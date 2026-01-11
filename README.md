<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Simple Free Photoreal AI Upscaler → 12K</title>
  <style>
    :root { color-scheme: dark; }
    body { margin:0; font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; background:#0b0f14; color:#e7eef7; }
    header { padding:18px 16px; border-bottom:1px solid rgba(255,255,255,.08); display:flex; justify-content:space-between; align-items:center; gap:12px; }
    header h1 { font-size:16px; margin:0; letter-spacing:.2px; }
    header .badge { font-size:12px; padding:6px 10px; border:1px solid rgba(255,255,255,.12); border-radius:999px; opacity:.9; }
    main { padding:16px; max-width:980px; margin:0 auto; display:grid; gap:14px; }
    .card { background:#0f1620; border:1px solid rgba(255,255,255,.08); border-radius:16px; padding:14px; }
    .drop {
      border:2px dashed rgba(255,255,255,.16);
      border-radius:16px; padding:18px; text-align:center; transition:.15s;
      background: linear-gradient(180deg, rgba(255,255,255,.02), rgba(255,255,255,0));
    }
    .drop.dragover { border-color: rgba(120,200,255,.7); box-shadow: 0 0 0 3px rgba(120,200,255,.15) inset; }
    .row { display:flex; flex-wrap:wrap; gap:10px; align-items:center; justify-content:center; }
    button, input[type="file"] {
      background:#121b26; color:#e7eef7; border:1px solid rgba(255,255,255,.14);
      padding:10px 12px; border-radius:12px; cursor:pointer;
    }
    button:hover { border-color: rgba(255,255,255,.26); }
    button:disabled { opacity:.5; cursor:not-allowed; }
    .muted { opacity:.8; font-size:13px; line-height:1.35; }
    .status { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", monospace; font-size:12px; white-space:pre-wrap; }
    canvas { width:100%; height:auto; background:#06090d; border-radius:14px; border:1px solid rgba(255,255,255,.08); }
    code { background: rgba(255,255,255,.06); padding:2px 6px; border-radius:8px; }
  </style>
</head>
<body>
  <header>
    <h1>Simple Free Photoreal AI Upscaler → 12K</h1>
    <div class="badge" id="badge">Checking…</div>
  </header>

  <main>
    <div class="card">
      <div class="drop" id="drop">
        <div style="font-size:14px; margin-bottom:8px;">Drop an image, or choose a file</div>
        <div class="row">
          <input id="file" type="file" accept="image/*" />
          <button id="btn" disabled>Upscale to 12K (Download PNG)</button>
        </div>
        <div class="muted" style="margin-top:10px;">
          Free, runs in your browser. This version auto-matches the model’s required input size (fixes the 128×128 error).
        </div>
      </div>
    </div>

    <div class="card">
      <div class="muted" style="margin-bottom:8px;">Preview</div>
      <canvas id="preview"></canvas>
      <div class="status" id="status" style="margin-top:10px;">Status: idle</div>
    </div>
  </main>

  <script src="https://cdn.jsdelivr.net/npm/onnxruntime-web@1.20.0/dist/ort.min.js"></script>

  <script>
    // Model URL (photoreal super-res). If you change models, this page adapts to fixed input sizes too.
    const MODEL_URL = "https://huggingface.co/qualcomm/Real-ESRGAN-x4plus/resolve/01179a4da7bf5ac91faca650e6afbf282ac93933/Real-ESRGAN-x4plus.onnx?download=true";

    const TARGET_EDGE = 12000;    // 12K max edge
    const HARD_MAX_EDGE = 16000;  // absolute safety to avoid memory blowups
    const OVERLAP = 16;           // seam reduction (auto-adjusted if needed)

    // IndexedDB cache
    const DB_NAME = "simple_ai_upscaler_db";
    const STORE = "models";
    const CACHE_KEY = "realesrgan_fixed_input_v3";

    const el = (id) => document.getElementById(id);
    const badge = el("badge");
    const drop = el("drop");
    const file = el("file");
    const btn = el("btn");
    const status = el("status");
    const preview = el("preview");
    const pctx = preview.getContext("2d");

    let imgBitmap = null;
    let session = null;

    let inputName = null;
    let outputName = null;

    // Model constraints discovered at runtime:
    let reqH = null, reqW = null;   // fixed input H/W if required
    let scale = null;              // output scale factor (computed)
    let overlap = OVERLAP;

    function log(s){ status.textContent = "Status: " + s; }
    function clamp(v, lo, hi){ return Math.max(lo, Math.min(hi, v)); }

    badge.textContent = (navigator.gpu ? "WebGPU ready" : "WASM mode");

    function setCanvasSize(c, w, h) { c.width = Math.max(1, Math.floor(w)); c.height = Math.max(1, Math.floor(h)); }

    function drawPreview(bitmap) {
      const cw = preview.clientWidth || 900;
      const ch = Math.max(360, Math.round(cw * 0.7));
      setCanvasSize(preview, cw * devicePixelRatio, ch * devicePixelRatio);
      pctx.clearRect(0,0,preview.width,preview.height);
      pctx.imageSmoothingEnabled = true;
      pctx.imageSmoothingQuality = "high";

      const ar = bitmap.width / bitmap.height;
      const car = preview.width / preview.height;
      let w,h;
      if (ar > car) { w = preview.width; h = w / ar; }
      else { h = preview.height; w = h * ar; }
      const x = (preview.width - w)/2;
      const y = (preview.height - h)/2;
      pctx.drawImage(bitmap, x, y, w, h);
    }

    function idbOpen() {
      return new Promise((resolve, reject) => {
        const req = indexedDB.open(DB_NAME, 1);
        req.onupgradeneeded = () => {
          const db = req.result;
          if (!db.objectStoreNames.contains(STORE)) db.createObjectStore(STORE);
        };
        req.onsuccess = () => resolve(req.result);
        req.onerror = () => reject(req.error);
      });
    }
    async function idbGet(key) {
      const db = await idbOpen();
      return new Promise((resolve, reject) => {
        const tx = db.transaction(STORE, "readonly");
        const st = tx.objectStore(STORE);
        const req = st.get(key);
        req.onsuccess = () => resolve(req.result || null);
        req.onerror = () => reject(req.error);
      });
    }
    async function idbSet(key, val) {
      const db = await idbOpen();
      return new Promise((resolve, reject) => {
        const tx = db.transaction(STORE, "readwrite");
        const st = tx.objectStore(STORE);
        const req = st.put(val, key);
        req.onsuccess = () => resolve(true);
        req.onerror = () => reject(req.error);
      });
    }

    async function loadModelIfNeeded() {
      if (session) return;

      log("loading AI model… (first time may take a while)");
      let bytes = await idbGet(CACHE_KEY);

      if (!bytes) {
        log("downloading AI model…");
        const res = await fetch(MODEL_URL);
        if (!res.ok) throw new Error("Model download failed: HTTP " + res.status);
        bytes = await res.arrayBuffer();
        log("caching model…");
        await idbSet(CACHE_KEY, bytes);
      } else {
        log("using cached model…");
      }

      const providers = navigator.gpu ? ["webgpu", "wasm"] : ["wasm"];
      ort.env.wasm.numThreads = Math.max(1, Math.min(4, navigator.hardwareConcurrency || 2));
      ort.env.wasm.simd = true;

      session = await ort.InferenceSession.create(bytes, {
        executionProviders: providers,
        graphOptimizationLevel: "all"
      });

      inputName = session.inputNames?.[0] || "input";
      outputName = session.outputNames?.[0] || "output";

      // Detect fixed input dims from metadata, if present.
      const meta = session.inputMetadata?.[inputName];
      const dims = meta?.dimensions || meta?.dims || null;
      // Expected shape often [1,3,H,W] with fixed H/W (e.g., 128x128)
      if (dims && dims.length === 4) {
        const H = dims[2], W = dims[3];
        if (Number.isInteger(H) && H > 0) reqH = H;
        if (Number.isInteger(W) && W > 0) reqW = W;
      }
      if (!reqH || !reqW) {
        // If dynamic, choose a safe tile size that most models accept.
        reqH = 256; reqW = 256;
      }

      // overlap must be smaller than half tile
      overlap = Math.min(OVERLAP, Math.floor(Math.min(reqH, reqW) / 4), 32);
      if (overlap < 8) overlap = 8;

      log(`model ready ✅ input="${inputName}" expects ${reqW}×${reqH}`);
    }

    async function loadImageFromFile(f) {
      const url = URL.createObjectURL(f);
      try {
        const bmp = await createImageBitmap(await (await fetch(url)).blob());
        return bmp;
      } finally { URL.revokeObjectURL(url); }
    }

    function makeCanvas(w,h) {
      const c = document.createElement("canvas");
      c.width = w; c.height = h;
      const x = c.getContext("2d", { willReadFrequently: true });
      x.imageSmoothingEnabled = true;
      x.imageSmoothingQuality = "high";
      return { c, x };
    }

    function imageDataToNCHWFloat32(imgData) {
      const { data, width, height } = imgData;
      const hw = width * height;
      const out = new Float32Array(3 * hw);
      const rOff = 0, gOff = hw, bOff = hw * 2;
      for (let i=0, p=0; i<hw; i++, p+=4) {
        out[rOff + i] = data[p] / 255;
        out[gOff + i] = data[p+1] / 255;
        out[bOff + i] = data[p+2] / 255;
      }
      return out;
    }

    function nchwFloat32ToImageData(tensorData, outW, outH) {
      const hw = outW * outH;
      const r = tensorData.subarray(0, hw);
      const g = tensorData.subarray(hw, hw*2);
      const b = tensorData.subarray(hw*2, hw*3);
      const imgData = new ImageData(outW, outH);
      const d = imgData.data;
      for (let i=0, p=0; i<hw; i++, p+=4) {
        d[p]   = clamp(Math.round(r[i]*255), 0, 255);
        d[p+1] = clamp(Math.round(g[i]*255), 0, 255);
        d[p+2] = clamp(Math.round(b[i]*255), 0, 255);
        d[p+3] = 255;
      }
      return imgData;
    }

    async function runModel(tileC, tileX) {
      const imgData = tileX.getImageData(0,0,tileC.width,tileC.height);
      const inputData = imageDataToNCHWFloat32(imgData);
      const inputTensor = new ort.Tensor("float32", inputData, [1,3,tileC.height,tileC.width]);

      const feeds = {};
      feeds[inputName] = inputTensor;

      const results = await session.run(feeds);
      const out = results[outputName];
      if (!out) {
        const keys = Object.keys(results);
        throw new Error(`Unexpected model outputs: ${keys.join(", ")}`);
      }
      return out;
    }

    function downloadPNG(canvas) {
      const stamp = new Date().toISOString().replace(/[:.]/g,"-");
      canvas.toBlob((blob) => {
        if (!blob) return;
        const a = document.createElement("a");
        a.href = URL.createObjectURL(blob);
        a.download = `upscaled-12k-${stamp}.png`;
        a.click();
        setTimeout(() => URL.revokeObjectURL(a.href), 1500);
      }, "image/png");
    }

    async function aiUpscaleTo12k() {
      if (!imgBitmap) throw new Error("No image loaded.");
      await loadModelIfNeeded();

      // We do ONE model pass over tiles of exactly reqW×reqH, then downscale to 12K if needed.
      // Pre-resize input to keep final output within 12K/hard limit based on scale inferred from model.
      // First, infer model scale by running ONE tiny tile once.
      if (!scale) {
        log("warming up / detecting scale…");
        const { c: tC, x: tX } = makeCanvas(reqW, reqH);
        tX.drawImage(imgBitmap, 0, 0, reqW, reqH);
        const outTensor = await runModel(tC, tX);
        const ow = outTensor.dims[3], oh = outTensor.dims[2];
        scale = Math.round(ow / reqW);
        if (!Number.isFinite(scale) || scale < 1) scale = 4; // fallback
        log(`scale detected: x${scale}`);
      }

      const inW0 = imgBitmap.width, inH0 = imgBitmap.height;
      const inMax0 = Math.max(inW0, inH0);
      const allowedOutMax = Math.min(TARGET_EDGE, HARD_MAX_EDGE);
      const allowedInMax = Math.floor(allowedOutMax / scale);

      let preScale = 1;
      if (inMax0 > allowedInMax) preScale = allowedInMax / inMax0;

      const preW = Math.max(1, Math.round(inW0 * preScale));
      const preH = Math.max(1, Math.round(inH0 * preScale));

      log(`preparing ${inW0}×${inH0} → ${preW}×${preH}…`);

      const { c: preC, x: preX } = makeCanvas(preW, preH);
      preX.drawImage(imgBitmap, 0, 0, preW, preH);

      const outW = preW * scale;
      const outH = preH * scale;

      log(`AI upscaling → ${outW}×${outH}…`);

      const { c: outC, x: outX } = makeCanvas(outW, outH);

      // Step size
      const tile = Math.min(reqW, reqH);
      const step = tile - overlap * 2;
      if (step <= 0) throw new Error("Tile/overlap config invalid for this model.");

      const tileC = document.createElement("canvas");
      const tileX = tileC.getContext("2d", { willReadFrequently: true });

      // To satisfy fixed-size models: we always feed exactly reqW×reqH tiles.
      // At edges, we clamp the source window so it stays in-bounds (repeat-edge padding).
      const maxSx = Math.max(0, preW - reqW);
      const maxSy = Math.max(0, preH - reqH);

      for (let y = 0; y < preH; y += step) {
        for (let x = 0; x < preW; x += step) {

          // Desired top-left (with overlap)
          let sx = x - overlap;
          let sy = y - overlap;

          // Clamp so we can always take a full reqW×reqH window
          sx = Math.max(0, Math.min(maxSx, sx));
          sy = Math.max(0, Math.min(maxSy, sy));

          tileC.width = reqW; tileC.height = reqH;
          tileX.clearRect(0,0,reqW,reqH);
          tileX.drawImage(preC, sx, sy, reqW, reqH, 0, 0, reqW, reqH);

          log(`AI tile ${sx},${sy}…`);
          const outTensor = await runModel(tileC, tileX);

          const ow = outTensor.dims[3], oh = outTensor.dims[2];
          const outImg = nchwFloat32ToImageData(outTensor.data, ow, oh);

          // Crop overlap seams on internal edges
          const cropL = (sx === 0) ? 0 : overlap * scale;
          const cropT = (sy === 0) ? 0 : overlap * scale;
          const cropR = (sx === maxSx) ? 0 : overlap * scale;
          const cropB = (sy === maxSy) ? 0 : overlap * scale;

          const drawW = ow - cropL - cropR;
          const drawH = oh - cropT - cropB;

          const tmp = document.createElement("canvas");
          tmp.width = ow; tmp.height = oh;
          const tx = tmp.getContext("2d");
          tx.putImageData(outImg, 0, 0);

          const dx = (sx * scale) + cropL;
          const dy = (sy * scale) + cropT;

          outX.drawImage(tmp, cropL, cropT, drawW, drawH, dx, dy, drawW, drawH);

          await new Promise(r => setTimeout(r, 0));
        }
      }

      // Downscale to exactly 12K max edge if needed
      const outMax = Math.max(outC.width, outC.height);
      if (outMax > TARGET_EDGE) {
        const ratio = TARGET_EDGE / outMax;
        const finalW = Math.round(outC.width * ratio);
        const finalH = Math.round(outC.height * ratio);
        log(`final downscale → ${finalW}×${finalH}…`);
        const { c: finalC, x: finalX } = makeCanvas(finalW, finalH);
        finalX.drawImage(outC, 0, 0, finalW, finalH);
        drawPreview(await createImageBitmap(finalC));
        log("done ✅ downloading…");
        downloadPNG(finalC);
        return;
      }

      drawPreview(await createImageBitmap(outC));
      log("done ✅ downloading…");
      downloadPNG(outC);
    }

    // UI events
    drop.addEventListener("dragover", (e) => { e.preventDefault(); drop.classList.add("dragover"); });
    drop.addEventListener("dragleave", () => drop.classList.remove("dragover"));
    drop.addEventListener("drop", async (e) => {
      e.preventDefault(); drop.classList.remove("dragover");
      const f = e.dataTransfer.files && e.dataTransfer.files[0];
      if (!f) return;
      try {
        imgBitmap = await loadImageFromFile(f);
        drawPreview(imgBitmap);
        btn.disabled = false;
        log(`image loaded: ${f.name} (${imgBitmap.width}×${imgBitmap.height})`);
      } catch (err) {
        console.error(err);
        log("image load failed: " + (err.message || err));
      }
    });

    file.addEventListener("change", async () => {
      const f = file.files && file.files[0];
      if (!f) return;
      try {
        imgBitmap = await loadImageFromFile(f);
        drawPreview(imgBitmap);
        btn.disabled = false;
        log(`image loaded: ${f.name} (${imgBitmap.width}×${imgBitmap.height})`);
      } catch (err) {
        console.error(err);
        log("image load failed: " + (err.message || err));
      }
    });

    btn.addEventListener("click", async () => {
      try {
        btn.disabled = true;
        await aiUpscaleTo12k();
      } catch (err) {
        console.error(err);
        log("Upscale failed: " + (err.message || err));
        alert("Upscale failed:\n" + (err.message || err) + "\n\nIf this still fails, the model may not be a general upscaler. In that case we swap the model URL.");
      } finally {
        btn.disabled = false;
      }
    });
  </script>
</body>
</html>

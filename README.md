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
          Runs in your browser for free using a photoreal <b>Real-ESRGAN x4</b> AI model. It auto-resizes safely if your input is too large.
        </div>
      </div>
    </div>

    <div class="card">
      <div class="muted" style="margin-bottom:8px;">Preview</div>
      <canvas id="preview"></canvas>
      <div class="status" id="status" style="margin-top:10px;">Status: idle</div>
    </div>
  </main>

  <!-- ONNX Runtime Web -->
  <script src="https://cdn.jsdelivr.net/npm/onnxruntime-web@1.20.0/dist/ort.min.js"></script>

  <script>
    // ====== Hard-coded photoreal model (Real-ESRGAN x4plus ONNX) ======
    // NOTE: If this URL ever changes upstream, replace it with another direct .onnx link.
    const MODEL_URL = "https://huggingface.co/qualcomm/Real-ESRGAN-x4plus/resolve/01179a4da7bf5ac91faca650e6afbf282ac93933/Real-ESRGAN-x4plus.onnx?download=true";

    // Common ONNX tensor names (may vary between exports)
    const INPUT_NAME  = "input";
    const OUTPUT_NAME = "output";

    // Target / limits
    const TARGET_EDGE = 12000;   // 12K
    const MODEL_SCALE = 4;       // x4 Real-ESRGAN
    const TILE = 256;            // safest for mobile
    const OVERLAP = 16;          // seam reduction
    const HARD_MAX_EDGE = 16000; // absolute safety (avoid exploding memory)

    // IndexedDB cache
    const DB_NAME = "simple_ai_upscaler_db";
    const STORE = "models";
    const CACHE_KEY = "realesrgan_x4plus_v1";

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

    function log(s){ status.textContent = "Status: " + s; }
    function clamp(v, lo, hi){ return Math.max(lo, Math.min(hi, v)); }

    function setBadge() {
      badge.textContent = (navigator.gpu ? "WebGPU ready" : "WASM mode");
    }
    setBadge();

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
      log("model ready ✅");
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

    async function runModelOnTile(tileC, tileX) {
      const imgData = tileX.getImageData(0,0,tileC.width,tileC.height);
      const inputData = imageDataToNCHWFloat32(imgData);
      const inputTensor = new ort.Tensor("float32", inputData, [1,3,tileC.height,tileC.width]);
      const feeds = {};
      feeds[INPUT_NAME] = inputTensor;

      const results = await session.run(feeds);
      const out = results[OUTPUT_NAME];
      if (!out) {
        const keys = Object.keys(results);
        throw new Error(`Model output "${OUTPUT_NAME}" not found. Available: ${keys.join(", ")}. Change OUTPUT_NAME/INPUT_NAME.`);
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

      // ---- SAFETY FIX: auto pre-resize so ONE x4 pass stays within target/safety ----
      const inW0 = imgBitmap.width, inH0 = imgBitmap.height;
      const inMax0 = Math.max(inW0, inH0);

      // We will do exactly one AI pass (x4), then optionally downscale to exactly 12K max edge.
      // Pre-resize input so that after x4, the max edge is <= min(TARGET_EDGE, HARD_MAX_EDGE).
      const allowedOutMax = Math.min(TARGET_EDGE, HARD_MAX_EDGE);
      const allowedInMax = Math.floor(allowedOutMax / MODEL_SCALE);

      let preScale = 1;
      if (inMax0 > allowedInMax) {
        preScale = allowedInMax / inMax0; // < 1
      }

      const preW = Math.max(1, Math.round(inW0 * preScale));
      const preH = Math.max(1, Math.round(inH0 * preScale));

      log(`preparing (${inW0}×${inH0}) → (${preW}×${preH}) for safe x4…`);

      const { c: preC, x: preX } = makeCanvas(preW, preH);
      preX.drawImage(imgBitmap, 0, 0, preW, preH);

      // Output canvas from AI pass
      const outW = preW * MODEL_SCALE;
      const outH = preH * MODEL_SCALE;

      log(`AI upscaling (one pass) → (${outW}×${outH})…`);

      const { c: outC, x: outX } = makeCanvas(outW, outH);

      const step = TILE - OVERLAP * 2;
      if (step <= 0) throw new Error("Internal tile config invalid.");

      const tileC = document.createElement("canvas");
      const tileX = tileC.getContext("2d", { willReadFrequently: true });

      for (let y = 0; y < preH; y += step) {
        for (let x = 0; x < preW; x += step) {
          const x0 = Math.max(0, x - OVERLAP);
          const y0 = Math.max(0, y - OVERLAP);
          const x1 = Math.min(preW, x + step + OVERLAP);
          const y1 = Math.min(preH, y + step + OVERLAP);
          const tw = x1 - x0, th = y1 - y0;

          tileC.width = tw; tileC.height = th;
          tileX.clearRect(0,0,tw,th);
          tileX.drawImage(preC, x0, y0, tw, th, 0, 0, tw, th);

          log(`AI tile ${x0},${y0} (${tw}×${th})…`);
          const outTensor = await runModelOnTile(tileC, tileX);

          const ow = outTensor.dims[3], oh = outTensor.dims[2];
          const outImg = nchwFloat32ToImageData(outTensor.data, ow, oh);

          // crop overlap seams
          const cropL = (x0 === 0) ? 0 : OVERLAP * MODEL_SCALE;
          const cropT = (y0 === 0) ? 0 : OVERLAP * MODEL_SCALE;
          const cropR = (x1 === preW) ? 0 : OVERLAP * MODEL_SCALE;
          const cropB = (y1 === preH) ? 0 : OVERLAP * MODEL_SCALE;
          const drawW = ow - cropL - cropR;
          const drawH = oh - cropT - cropB;

          const tmp = document.createElement("canvas");
          tmp.width = ow; tmp.height = oh;
          const tx = tmp.getContext("2d");
          tx.putImageData(outImg, 0, 0);

          const dx = (x0 * MODEL_SCALE) + cropL;
          const dy = (y0 * MODEL_SCALE) + cropT;
          outX.drawImage(tmp, cropL, cropT, drawW, drawH, dx, dy, drawW, drawH);

          await new Promise(r => setTimeout(r, 0));
        }
      }

      // If output max edge > 12K, downscale to exactly 12K
      const outMax = Math.max(outC.width, outC.height);
      if (outMax > TARGET_EDGE) {
        const ratio = TARGET_EDGE / outMax;
        const finalW = Math.round(outC.width * ratio);
        const finalH = Math.round(outC.height * ratio);
        log(`final downscale to 12K max edge → (${finalW}×${finalH})…`);
        const { c: finalC, x: finalX } = makeCanvas(finalW, finalH);
        finalX.drawImage(outC, 0, 0, finalW, finalH);
        log("done ✅ starting download…");
        drawPreview(await createImageBitmap(finalC));
        downloadPNG(finalC);
        return;
      }

      log("done ✅ starting download…");
      drawPreview(await createImageBitmap(outC));
      downloadPNG(outC);
    }

    // ---- UI events ----
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
        alert("Upscale failed:\n" + (err.message || err) + "\n\nIf this keeps happening on mobile, try a smaller input image.");
      } finally {
        btn.disabled = false;
      }
    });
  </script>
</body>
</html>

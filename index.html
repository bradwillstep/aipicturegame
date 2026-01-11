<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Photoreal AI Upscale (Local) + Face-Realistic Image→Video (Embedded)</title>
  <style>
    :root { color-scheme: dark; }
    body { margin:0; font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; background:#0b0f14; color:#e7eef7; }
    header { padding:18px 16px; border-bottom:1px solid rgba(255,255,255,.08); display:flex; gap:12px; align-items:center; justify-content:space-between; }
    header h1 { font-size:16px; margin:0; letter-spacing:.2px; }
    header .badge { font-size:12px; padding:6px 10px; border:1px solid rgba(255,255,255,.12); border-radius:999px; opacity:.9; }
    main { padding:16px; display:grid; gap:14px; max-width:1120px; margin:0 auto; }
    .card { background:#0f1620; border:1px solid rgba(255,255,255,.08); border-radius:16px; padding:14px; }
    .row { display:flex; flex-wrap:wrap; gap:10px; align-items:center; }
    button, select, input[type="number"], input[type="text"], textarea {
      background:#121b26; color:#e7eef7; border:1px solid rgba(255,255,255,.14);
      padding:10px 12px; border-radius:12px; cursor:pointer;
    }
    textarea { width: min(920px, 100%); min-height: 92px; resize: vertical; line-height: 1.35; }
    button:hover { border-color: rgba(255,255,255,.26); }
    button:disabled { opacity:.5; cursor:not-allowed; }
    .drop {
      border:2px dashed rgba(255,255,255,.16);
      border-radius:16px; padding:18px; text-align:center; transition:.15s;
      background: linear-gradient(180deg, rgba(255,255,255,.02), rgba(255,255,255,0));
    }
    .drop.dragover { border-color: rgba(120,200,255,.7); box-shadow: 0 0 0 3px rgba(120,200,255,.15) inset; }
    .muted { opacity:.82; font-size:13px; line-height:1.35; }
    .grid { display:grid; grid-template-columns: 1fr 1fr; gap:12px; }
    @media (max-width: 900px){ .grid { grid-template-columns: 1fr; } }
    canvas { width:100%; height:auto; background:#06090d; border-radius:14px; border:1px solid rgba(255,255,255,.08); }
    .status { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", monospace; font-size:12px; white-space:pre-wrap; }
    .kpi { display:flex; gap:10px; flex-wrap:wrap; }
    .pill { padding:6px 10px; border-radius:999px; border:1px solid rgba(255,255,255,.12); font-size:12px; opacity:.92; }
    .warn { border-color: rgba(255,160,90,.35); }
    .ok { border-color: rgba(120,255,180,.22); }
    .tabs { display:flex; gap:10px; flex-wrap:wrap; }
    .tabbtn[aria-selected="true"] { border-color: rgba(120,200,255,.7); box-shadow: 0 0 0 3px rgba(120,200,255,.12) inset; }
    .hidden { display:none !important; }
    iframe { width:100%; height:740px; border:1px solid rgba(255,255,255,.10); border-radius:16px; background:#06090d; }
    .note { border-left: 3px solid rgba(120,200,255,.55); padding-left:10px; }
    code { background: rgba(255,255,255,.06); padding:2px 6px; border-radius:8px; }
    .chips { display:flex; flex-wrap:wrap; gap:8px; margin-top:10px; }
    .chip { font-size:12px; padding:7px 10px; border-radius:999px; border:1px solid rgba(255,255,255,.14); background: rgba(255,255,255,.03); cursor:pointer; user-select:none; }
    .chip:hover { border-color: rgba(120,200,255,.55); }
    a { color:#8dd3ff; text-decoration:none; }
    a:hover { text-decoration:underline; }
  </style>
</head>
<body>
  <header>
    <h1>Photoreal AI Upscale (Local) + Realistic Human-Face Image→Video (Embedded)</h1>
    <div class="badge" id="runtimeBadge">Runtime: checking…</div>
  </header>

  <main>
    <div class="card">
      <div class="tabs">
        <button class="tabbtn" id="tabUpscale" aria-selected="true">🖼️ Upscale (Photoreal)</button>
        <button class="tabbtn" id="tabVideo" aria-selected="false">🎬 Image → Video (Faces)</button>
      </div>
      <div class="muted" style="margin-top:8px;">
        Upscale runs on-device in your browser (free). Image→Video is embedded from free public Spaces (runs on their servers).
      </div>
    </div>

    <!-- ===================== UPSCALE PANEL ===================== -->
    <section id="panelUpscale">
      <div class="card">
        <div class="drop" id="drop">
          <div style="font-size:14px; margin-bottom:8px;">Drop an image here, or choose a file</div>
          <div class="row" style="justify-content:center;">
            <input id="file" type="file" accept="image/*" />
            <button id="btnLoadModel">Load AI Upscale Model</button>
            <button id="btnClearCache">Clear Cached Model</button>
          </div>
          <div class="muted note" style="margin-top:10px;">
            Default: <b>Real-ESRGAN x4plus (ONNX)</b>. First run downloads once, then it caches in your browser (IndexedDB).
          </div>
        </div>
      </div>

      <div class="card">
        <div class="row">
          <label class="muted">ONNX model URL (direct .onnx download):</label>
          <input id="modelUrl" type="text" style="width:min(800px, 100%);"
            value="https://huggingface.co/qualcomm/Real-ESRGAN-x4plus/resolve/01179a4da7bf5ac91faca650e6afbf282ac93933/Real-ESRGAN-x4plus.onnx?download=true" />
          <button id="btnTestUrl">Test URL</button>
        </div>

        <div class="row" style="margin-top:10px;">
          <label class="muted">Backend:</label>
          <select id="backend">
            <option value="webgpu">WebGPU (fast, if supported)</option>
            <option value="wasm">WASM (compatible fallback)</option>
          </select>

          <label class="muted">Target max edge:</label>
          <select id="targetEdge">
            <option value="12000" selected>12,000 px (12K)</option>
            <option value="8000">8,000 px</option>
            <option value="6000">6,000 px</option>
            <option value="4096">4096 px</option>
          </select>

          <label class="muted">Tile size:</label>
          <select id="tileSize">
            <option value="256">256 (safest)</option>
            <option value="384">384</option>
            <option value="512" selected>512 (balanced)</option>
            <option value="768">768 (faster, strong devices)</option>
          </select>

          <label class="muted">Overlap:</label>
          <input id="overlap" type="number" min="8" max="64" step="4" value="16" style="width:90px;" />

          <button id="btnUpscale" disabled>AI Upscale → Download PNG</button>
        </div>

        <div class="kpi" style="margin-top:10px;">
          <div class="pill" id="kpiIn">Input: —</div>
          <div class="pill" id="kpiOut">Output: —</div>
          <div class="pill warn" id="kpiModel">Model: not loaded</div>
          <div class="pill" id="kpiCache">Cache: —</div>
        </div>

        <div class="status" id="status" style="margin-top:10px;">Status: idle</div>
      </div>

      <div class="grid">
        <div class="card">
          <div class="muted" style="margin-bottom:8px;">Preview (input)</div>
          <canvas id="cvIn"></canvas>
        </div>
        <div class="card">
          <div class="muted" style="margin-bottom:8px;">Preview (output)</div>
          <canvas id="cvOut"></canvas>
        </div>
      </div>

      <div class="card muted">
        <b>Face-quality tips (upscale):</b>
        <ul>
          <li>Use a sharp input image (avoid heavy blur).</li>
          <li>If the device struggles: set <code>Backend = WASM</code> and <code>Tile size = 256</code>.</li>
          <li>Going to 12K is heavy—start with a smaller input if you crash.</li>
        </ul>
      </div>
    </section>

    <!-- ===================== VIDEO PANEL ===================== -->
    <section id="panelVideo" class="hidden">
      <div class="card">
        <div class="row">
          <label class="muted">Provider (most face-realistic first):</label>
          <select id="videoProvider" style="min-width: 380px;">
            <option value="https://huggingface.co/spaces/multimodalart/stable-video-diffusion" selected>
              Stable Video Diffusion 1.1 (Image→Video) — best free default
            </option>
            <option value="https://huggingface.co/spaces/Lightricks/ltx-video-distilled">
              LTX Video Fast — good realism + motion
            </option>
            <option value="https://huggingface.co/spaces/KlingTeam/LivePortrait">
              LivePortrait (portrait motion transfer) — great face motion, not text-driven
            </option>
          </select>
          <button id="btnLoadSpace">Load Embed</button>
          <button id="btnOpenSpace">Open in New Tab</button>
        </div>

        <div class="muted note" style="margin-top:10px;">
          <b>Important:</b> Your site can’t auto-send your image into the embed (browser security). Upload inside the embedded UI or open it in a new tab.
        </div>

        <div class="row" style="margin-top:10px;">
          <label class="muted">Face-realistic prompt (copy/paste into the tool):</label>
        </div>

        <textarea id="videoPrompt" placeholder="Example: photoreal close-up portrait, natural skin texture, subtle breathing, micro-expressions, gentle head turn, soft daylight, 4 seconds, stabilized camera, no face morphing, identity consistent."></textarea>

        <div class="row" style="margin-top:10px;">
          <button id="btnCopyPrompt">Copy Prompt</button>
          <button id="btnClearPrompt">Clear</button>
        </div>

        <div class="muted" style="margin-top:8px;">
          Quick templates (tap to append):
          <div class="chips" id="chips"></div>
        </div>
      </div>

      <div class="card">
        <iframe id="videoFrame" title="Embedded Image to Video"></iframe>
        <div class="muted" style="margin-top:10px;">
          If the embed won’t load, that Space blocks iframes. Use “Open in New Tab”.
        </div>
      </div>

      <div class="card muted">
        <b>Best settings for realistic human faces:</b>
        <ul>
          <li><b>Keep motion subtle:</b> micro-expressions, blink, gentle head turn. Too much motion often breaks identity.</li>
          <li><b>Lighting:</b> “soft daylight”, “studio softbox”, “natural skin texture”.</li>
          <li><b>Identity lock language:</b> “no face morphing”, “identity consistent”, “no age change”.</li>
          <li><b>Camera:</b> “stabilized”, “slow push-in”, “50–85mm portrait lens”.</li>
        </ul>
      </div>
    </section>
  </main>

  <!-- ONNX Runtime Web (upscaling) -->
  <script src="https://cdn.jsdelivr.net/npm/onnxruntime-web@1.20.0/dist/ort.min.js"></script>

  <script>
    // ================== TAB UI ==================
    const tabUpscale = document.getElementById("tabUpscale");
    const tabVideo = document.getElementById("tabVideo");
    const panelUpscale = document.getElementById("panelUpscale");
    const panelVideo = document.getElementById("panelVideo");

    function setTab(which) {
      const isUpscale = which === "upscale";
      tabUpscale.setAttribute("aria-selected", String(isUpscale));
      tabVideo.setAttribute("aria-selected", String(!isUpscale));
      panelUpscale.classList.toggle("hidden", !isUpscale);
      panelVideo.classList.toggle("hidden", isUpscale);
    }
    tabUpscale.addEventListener("click", () => setTab("upscale"));
    tabVideo.addEventListener("click", () => setTab("video"));

    // ================== UPSCALE (LOCAL AI) ==================
    // Tensor names vary by ONNX export. If you get input/output errors, change these.
    const INPUT_NAME  = "input";
    const OUTPUT_NAME = "output";

    // IndexedDB cache
    const DB_NAME = "ai_upscaler_db";
    const STORE = "models";
    const CACHE_KEY = "realesrgan_x4plus_v1";

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
    async function idbDel(key) {
      const db = await idbOpen();
      return new Promise((resolve, reject) => {
        const tx = db.transaction(STORE, "readwrite");
        const st = tx.objectStore(STORE);
        const req = st.delete(key);
        req.onsuccess = () => resolve(true);
        req.onerror = () => reject(req.error);
      });
    }

    // UI refs (upscale)
    const el = (id) => document.getElementById(id);
    const drop = el("drop");
    const file = el("file");
    const btnLoadModel = el("btnLoadModel");
    const btnUpscale = el("btnUpscale");
    const btnClearCache = el("btnClearCache");
    const btnTestUrl = el("btnTestUrl");

    const backendSel = el("backend");
    const targetEdgeSel = el("targetEdge");
    const tileSel = el("tileSize");
    const overlapInp = el("overlap");
    const modelUrlInp = el("modelUrl");

    const cvIn = el("cvIn"), cxIn = cvIn.getContext("2d");
    const cvOut = el("cvOut"), cxOut = cvOut.getContext("2d");
    const status = el("status");
    const runtimeBadge = el("runtimeBadge");

    const kpiIn = el("kpiIn");
    const kpiOut = el("kpiOut");
    const kpiModel = el("kpiModel");
    const kpiCache = el("kpiCache");

    let imgBitmap = null;
    let session = null;

    function log(s) { status.textContent = `Status: ${s}`; }
    function clamp(v, lo, hi){ return Math.max(lo, Math.min(hi, v)); }
    function bytesToMB(n){ return (n/1024/1024).toFixed(1); }

    function setBadge() {
      const webgpu = !!navigator.gpu;
      runtimeBadge.textContent = `Runtime: ${webgpu ? "WebGPU available" : "WebGPU not found (WASM only)"}`;
    }
    setBadge();

    function setCanvasSize(canvas, w, h) {
      canvas.width = Math.max(1, Math.floor(w));
      canvas.height = Math.max(1, Math.floor(h));
    }

    function drawContain(bitmap, canvas, ctx) {
      const cw = canvas.clientWidth || 900;
      const ch = Math.max(320, Math.round(cw * 0.65));
      setCanvasSize(canvas, cw * devicePixelRatio, ch * devicePixelRatio);

      ctx.clearRect(0,0,canvas.width,canvas.height);
      ctx.imageSmoothingEnabled = true;
      ctx.imageSmoothingQuality = "high";

      const ar = bitmap.width / bitmap.height;
      const car = canvas.width / canvas.height;
      let w, h;
      if (ar > car) { w = canvas.width; h = w / ar; }
      else { h = canvas.height; w = h * ar; }
      const x = (canvas.width - w) / 2;
      const y = (canvas.height - h) / 2;
      ctx.drawImage(bitmap, x, y, w, h);
    }

    async function refreshCacheBadge() {
      const cached = await idbGet(CACHE_KEY);
      if (cached && cached.byteLength) kpiCache.textContent = `Cache: ${bytesToMB(cached.byteLength)} MB`;
      else kpiCache.textContent = `Cache: none`;
    }
    refreshCacheBadge().catch(()=>{});

    async function loadImageFromFile(f) {
      const url = URL.createObjectURL(f);
      try {
        const bmp = await createImageBitmap(await (await fetch(url)).blob());
        return bmp;
      } finally {
        URL.revokeObjectURL(url);
      }
    }

    async function testUrl() {
      const url = modelUrlInp.value.trim();
      if (!url) { alert("Paste a direct .onnx URL first."); return; }
      log("testing model URL…");
      const res = await fetch(url, { method: "GET" });
      if (!res.ok) throw new Error(`URL fetch failed: HTTP ${res.status}`);
      const ct = res.headers.get("content-type") || "";
      log(`URL OK ✅ content-type: ${ct || "unknown"}`);
      alert("URL reachable ✅");
    }

    async function fetchModelBytes(url) {
      const res = await fetch(url);
      if (!res.ok) throw new Error(`Model download failed: HTTP ${res.status}`);
      return await res.arrayBuffer();
    }

    async function loadModel() {
      const backend = backendSel.value;
      const url = modelUrlInp.value.trim();
      if (!url) throw new Error("Model URL is empty.");

      log("checking cache…");
      let modelBytes = await idbGet(CACHE_KEY);

      if (modelBytes) log(`using cached model (${bytesToMB(modelBytes.byteLength)} MB)…`);
      else {
        log("downloading model… (first run only)");
        modelBytes = await fetchModelBytes(url);
        log("caching model locally…");
        await idbSet(CACHE_KEY, modelBytes);
      }
      await refreshCacheBadge();

      log(`creating session (${backend})…`);
      ort.env.wasm.numThreads = Math.max(1, Math.min(4, navigator.hardwareConcurrency || 2));
      ort.env.wasm.simd = true;

      const providers = backend === "webgpu" ? ["webgpu", "wasm"] : ["wasm"];
      session = await ort.InferenceSession.create(modelBytes, {
        executionProviders: providers,
        graphOptimizationLevel: "all"
      });

      kpiModel.textContent = "Model: loaded";
      kpiModel.classList.remove("warn");
      kpiModel.classList.add("ok");
      btnUpscale.disabled = !imgBitmap;
      log("model loaded ✅");
    }

    function getTargetScaleToMaxEdge(inW, inH, maxEdge) {
      const inMax = Math.max(inW, inH);
      if (inMax >= maxEdge) return 1;
      return maxEdge / inMax;
    }
    function computePasses(scaleNeeded, perPass=4) {
      let passes = 0, s = 1;
      while (s * perPass < scaleNeeded && passes < 4) { s *= perPass; passes++; }
      if (s < scaleNeeded && passes < 5) passes++;
      return passes;
    }

    function makeWorkingCanvas(w, h) {
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
        out[rOff + i] = data[p]     / 255;
        out[gOff + i] = data[p + 1] / 255;
        out[bOff + i] = data[p + 2] / 255;
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

    async function runModelOnTile(tileCanvas, tileCtx) {
      const imgData = tileCtx.getImageData(0, 0, tileCanvas.width, tileCanvas.height);
      const inputData = imageDataToNCHWFloat32(imgData);
      const inputTensor = new ort.Tensor("float32", inputData, [1, 3, tileCanvas.height, tileCanvas.width]);

      const feeds = {};
      feeds[INPUT_NAME] = inputTensor;

      const results = await session.run(feeds);
      const out = results[OUTPUT_NAME];
      if (!out) {
        const keys = Object.keys(results);
        throw new Error(`Output "${OUTPUT_NAME}" not found. Available outputs: ${keys.join(", ")}. Update OUTPUT_NAME.`);
      }
      return out;
    }

    async function aiUpscaleOnceRGBA(srcCanvas, tileSize, overlap) {
      const inW = srcCanvas.width;
      const inH = srcCanvas.height;
      const scale = 4;

      const outW = inW * scale;
      const outH = inH * scale;

      const { c: outC, x: outX } = makeWorkingCanvas(outW, outH);

      const step = tileSize - overlap * 2;
      if (step <= 0) throw new Error("Tile size too small vs overlap.");

      const tileC = document.createElement("canvas");
      const tileX = tileC.getContext("2d", { willReadFrequently: true });

      for (let y = 0; y < inH; y += step) {
        for (let x = 0; x < inW; x += step) {
          const x0 = Math.max(0, x - overlap);
          const y0 = Math.max(0, y - overlap);
          const x1 = Math.min(inW, x + step + overlap);
          const y1 = Math.min(inH, y + step + overlap);
          const tw = x1 - x0;
          const th = y1 - y0;

          tileC.width = tw;
          tileC.height = th;
          tileX.clearRect(0,0,tw,th);
          tileX.drawImage(srcCanvas, x0, y0, tw, th, 0, 0, tw, th);

          log(`AI tile (${x0},${y0}) ${tw}×${th}…`);
          const outTensor = await runModelOnTile(tileC, tileX);

          const shape = outTensor.dims; // [1,3,th*4,tw*4]
          const oh = shape[2], ow = shape[3];

          const outImg = nchwFloat32ToImageData(outTensor.data, ow, oh);

          const cropL = (x0 === 0) ? 0 : overlap * scale;
          const cropT = (y0 === 0) ? 0 : overlap * scale;
          const cropR = (x1 === inW) ? 0 : overlap * scale;
          const cropB = (y1 === inH) ? 0 : overlap * scale;

          const drawW = ow - cropL - cropR;
          const drawH = oh - cropT - cropB;

          const tmp = document.createElement("canvas");
          tmp.width = ow; tmp.height = oh;
          const tx = tmp.getContext("2d");
          tx.putImageData(outImg, 0, 0);

          const dx = (x0 * scale) + cropL;
          const dy = (y0 * scale) + cropT;

          outX.drawImage(tmp, cropL, cropT, drawW, drawH, dx, dy, drawW, drawH);
          await new Promise(r => setTimeout(r, 0));
        }
      }
      return outC;
    }

    async function upscaleToTarget() {
      if (!imgBitmap) throw new Error("No image loaded.");
      if (!session) throw new Error("Model not loaded.");

      const maxEdge = parseInt(targetEdgeSel.value, 10);
      const tileSize = parseInt(tileSel.value, 10);
      const overlap = clamp(parseInt(overlapInp.value, 10) || 16, 8, 64);

      const { c: workC, x: workX } = makeWorkingCanvas(imgBitmap.width, imgBitmap.height);
      workX.drawImage(imgBitmap, 0, 0);

      const inW = workC.width, inH = workC.height;
      const scaleNeeded = getTargetScaleToMaxEdge(inW, inH, maxEdge);

      kpiIn.textContent = `Input: ${inW}×${inH}`;
      log(`target=${maxEdge}px edge; needed≈${scaleNeeded.toFixed(2)}x`);

      const passes = computePasses(scaleNeeded, 4);
      if (passes === 0) return workC;

      let curC = workC;
      for (let p=1; p<=passes; p++) {
        const curMax = Math.max(curC.width, curC.height);
        if (curMax >= maxEdge) break;

        const nextW = curC.width * 4, nextH = curC.height * 4;
        if (Math.max(nextW, nextH) > 16000) {
          throw new Error(`Safety stop: next pass exceeds ~16K edge (${nextW}×${nextH}). Start smaller or lower target.`);
        }

        log(`AI pass ${p}/${passes}…`);
        curC = await aiUpscaleOnceRGBA(curC, tileSize, overlap);
      }

      // Downscale to exact target edge if overshot
      const outMax = Math.max(curC.width, curC.height);
      let finalC = curC;

      if (outMax > maxEdge) {
        const ratio = maxEdge / outMax;
        const finalW = Math.round(curC.width * ratio);
        const finalH = Math.round(curC.height * ratio);
        log(`downscaling to exact target… (${finalW}×${finalH})`);
        const scaled = makeWorkingCanvas(finalW, finalH);
        scaled.x.imageSmoothingEnabled = true;
        scaled.x.imageSmoothingQuality = "high";
        scaled.x.drawImage(curC, 0, 0, finalW, finalH);
        finalC = scaled.c;
      }

      kpiOut.textContent = `Output: ${finalC.width}×${finalC.height}`;
      log("done ✅");
      return finalC;
    }

    function downloadCanvasPNG(canvas, filename="upscaled.png") {
      canvas.toBlob((blob) => {
        if (!blob) return;
        const a = document.createElement("a");
        a.href = URL.createObjectURL(blob);
        a.download = filename;
        a.click();
        setTimeout(() => URL.revokeObjectURL(a.href), 1500);
      }, "image/png");
    }

    // DnD + file pick
    drop.addEventListener("dragover", (e) => { e.preventDefault(); drop.classList.add("dragover"); });
    drop.addEventListener("dragleave", () => drop.classList.remove("dragover"));
    drop.addEventListener("drop", async (e) => {
      e.preventDefault();
      drop.classList.remove("dragover");
      const f = e.dataTransfer.files && e.dataTransfer.files[0];
      if (!f) return;
      try {
        imgBitmap = await loadImageFromFile(f);
        drawContain(imgBitmap, cvIn, cxIn);
        btnUpscale.disabled = !session;
        kpiIn.textContent = `Input: ${imgBitmap.width}×${imgBitmap.height}`;
        log(`loaded image: ${f.name}`);
      } catch (err) {
        console.error(err);
        log(`image load failed: ${err.message || err}`);
      }
    });

    file.addEventListener("change", async () => {
      const f = file.files && file.files[0];
      if (!f) return;
      try {
        imgBitmap = await loadImageFromFile(f);
        drawContain(imgBitmap, cvIn, cxIn);
        btnUpscale.disabled = !session;
        kpiIn.textContent = `Input: ${imgBitmap.width}×${imgBitmap.height}`;
        log(`loaded image: ${f.name}`);
      } catch (err) {
        console.error(err);
        log(`image load failed: ${err.message || err}`);
      }
    });

    btnTestUrl.addEventListener("click", async () => {
      try { await testUrl(); }
      catch (err) { log(`URL test failed: ${err.message || err}`); alert(`URL test failed:\n${err.message || err}`); }
    });

    btnClearCache.addEventListener("click", async () => {
      await idbDel(CACHE_KEY);
      await refreshCacheBadge();
      session = null;
      kpiModel.textContent = "Model: not loaded";
      kpiModel.classList.add("warn");
      kpiModel.classList.remove("ok");
      btnUpscale.disabled = true;
      log("cache cleared ✅");
    });

    btnLoadModel.addEventListener("click", async () => {
      try {
        btnLoadModel.disabled = true;
        await loadModel();
      } catch (err) {
        console.error(err);
        session = null;
        kpiModel.textContent = "Model: not loaded";
        kpiModel.classList.add("warn");
        kpiModel.classList.remove("ok");
        btnUpscale.disabled = true;
        log(`model load failed: ${err.message || err}`);
        alert(
          "Model load failed.\n\nMost common reasons:\n" +
          "1) ONNX uses different input/output names\n" +
          "2) Browser memory limits\n\n" +
          "Paste the error text into ChatGPT and I’ll fix the names."
        );
      } finally {
        btnLoadModel.disabled = false;
      }
    });

    btnUpscale.addEventListener("click", async () => {
      try {
        btnUpscale.disabled = true;
        const outCanvas = await upscaleToTarget();

        const bmp = await createImageBitmap(outCanvas);
        drawContain(bmp, cvOut, cxOut);

        const stamp = new Date().toISOString().replace(/[:.]/g,"-");
        downloadCanvasPNG(outCanvas, `ai-upscale-${stamp}.png`);
        log("download started ✅");
      } catch (err) {
        console.error(err);
        log(`upscale failed: ${err.message || err}`);
        alert(`Upscale failed:\n${err.message || err}\n\nTry:\n- Backend WASM\n- Tile size 256\n- Smaller target edge\n- Smaller input`);
      } finally {
        btnUpscale.disabled = false;
      }
    });

    // ================== VIDEO (EMBED) ==================
    const videoProvider = el("videoProvider");
    const videoFrame = el("videoFrame");
    const btnLoadSpace = el("btnLoadSpace");
    const btnOpenSpace = el("btnOpenSpace");
    const videoPrompt = el("videoPrompt");
    const btnCopyPrompt = el("btnCopyPrompt");
    const btnClearPrompt = el("btnClearPrompt");

    function loadEmbed() {
      const url = (videoProvider.value || "").trim();
      if (!url) { alert("Select a provider first."); return; }
      videoFrame.src = url;
    }
    btnLoadSpace.addEventListener("click", loadEmbed);
    btnOpenSpace.addEventListener("click", () => {
      const url = (videoProvider.value || "").trim();
      if (!url) return;
      window.open(url, "_blank", "noopener,noreferrer");
    });

    btnCopyPrompt.addEventListener("click", async () => {
      try {
        await navigator.clipboard.writeText(videoPrompt.value || "");
        alert("Copied ✅ Paste it into the Image→Video tool.");
      } catch {
        alert("Copy failed. Manually select + copy the text.");
      }
    });
    btnClearPrompt.addEventListener("click", () => { videoPrompt.value = ""; });

    // Prompt chips (realistic faces)
    const chipItems = [
      "photoreal close-up portrait, 85mm lens",
      "natural skin texture, pores, fine hair detail",
      "micro-expressions, subtle smile, gentle blink",
      "slow head turn 10–15 degrees, no face morphing",
      "identity consistent, no age change, no face swap",
      "soft daylight window light, realistic shadows",
      "cinematic shallow depth of field, bokeh background",
      "stabilized camera, slow push-in, minimal motion blur",
      "4 seconds, realistic motion, no warping artifacts"
    ];
    const chips = document.getElementById("chips");
    chipItems.forEach(text => {
      const b = document.createElement("div");
      b.className = "chip";
      b.textContent = text;
      b.addEventListener("click", () => {
        const cur = videoPrompt.value.trim();
        videoPrompt.value = cur ? (cur + (cur.endsWith(",") ? " " : ", ") + text) : text;
        videoPrompt.focus();
      });
      chips.appendChild(b);
    });

    // Auto-load the default embed (best-effort)
    loadEmbed();
  </script>
</body>
</html>

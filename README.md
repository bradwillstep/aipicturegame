<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Image Game - Control Characters & Break Objects</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/matter-js/0.19.0/matter.min.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700;900&family=Rajdhani:wght@400;600&display=swap" rel="stylesheet">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Rajdhani', sans-serif;
            background: #0a0e27;
            color: #e0e0e0;
            overflow-x: hidden;
            position: relative;
        }

        body::before {
            content: '';
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: 
                radial-gradient(circle at 20% 50%, rgba(0, 255, 157, 0.1) 0%, transparent 50%),
                radial-gradient(circle at 80% 80%, rgba(255, 0, 128, 0.1) 0%, transparent 50%);
            pointer-events: none;
            z-index: 0;
        }

        #app {
            position: relative;
            z-index: 1;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 40px 20px;
        }

        h1 {
            font-family: 'Orbitron', sans-serif;
            font-size: 3.5em;
            font-weight: 900;
            text-align: center;
            margin-bottom: 15px;
            background: linear-gradient(135deg, #00ff9d 0%, #00b8ff 50%, #ff0080 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            text-transform: uppercase;
            letter-spacing: 4px;
            text-shadow: 0 0 40px rgba(0, 255, 157, 0.3);
            animation: glow 2s ease-in-out infinite alternate;
        }

        @keyframes glow {
            from { filter: drop-shadow(0 0 20px rgba(0, 255, 157, 0.4)); }
            to { filter: drop-shadow(0 0 35px rgba(0, 255, 157, 0.8)); }
        }

        .subtitle {
            text-align: center;
            font-size: 1.2em;
            color: #00ff9d;
            margin-bottom: 40px;
            font-weight: 600;
            letter-spacing: 2px;
        }

        #uploadSection {
            background: rgba(255, 255, 255, 0.03);
            backdrop-filter: blur(20px);
            border: 2px solid rgba(0, 255, 157, 0.2);
            border-radius: 20px;
            padding: 40px;
            max-width: 700px;
            width: 100%;
            margin-bottom: 30px;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
        }

        .upload-zone {
            border: 3px dashed rgba(0, 255, 157, 0.3);
            border-radius: 15px;
            padding: 60px 30px;
            text-align: center;
            cursor: pointer;
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
        }

        .upload-zone::before {
            content: '';
            position: absolute;
            top: -50%;
            left: -50%;
            width: 200%;
            height: 200%;
            background: radial-gradient(circle, rgba(0, 255, 157, 0.1) 0%, transparent 70%);
            animation: rotate 10s linear infinite;
            opacity: 0;
            transition: opacity 0.3s;
        }

        @keyframes rotate {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }

        .upload-zone:hover::before {
            opacity: 1;
        }

        .upload-zone:hover {
            border-color: #00ff9d;
            background: rgba(0, 255, 157, 0.05);
            transform: scale(1.02);
        }

        .upload-icon {
            font-size: 4em;
            margin-bottom: 20px;
            display: block;
        }

        .upload-text {
            font-size: 1.3em;
            color: #00ff9d;
            font-weight: 600;
            margin-bottom: 10px;
        }

        .upload-hint {
            color: #888;
            font-size: 0.95em;
        }

        #imageUpload {
            display: none;
        }

        #processingSection {
            display: none;
            text-align: center;
            padding: 40px;
            background: rgba(255, 255, 255, 0.03);
            backdrop-filter: blur(20px);
            border: 2px solid rgba(0, 255, 157, 0.2);
            border-radius: 20px;
            max-width: 700px;
            width: 100%;
            margin-bottom: 30px;
        }

        .processing-spinner {
            width: 80px;
            height: 80px;
            border: 4px solid rgba(0, 255, 157, 0.1);
            border-top: 4px solid #00ff9d;
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin: 0 auto 20px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .processing-text {
            font-size: 1.3em;
            color: #00ff9d;
            font-weight: 600;
            margin-bottom: 10px;
        }

        .processing-detail {
            color: #888;
            font-size: 1em;
        }

        #gameSection {
            display: none;
            width: 100%;
            max-width: 1400px;
        }

        #gameCanvas {
            width: 100%;
            max-width: 100%;
            border: 3px solid #00ff9d;
            border-radius: 15px;
            box-shadow: 
                0 0 40px rgba(0, 255, 157, 0.3),
                inset 0 0 60px rgba(0, 0, 0, 0.4);
            display: block;
            margin: 0 auto 30px;
            background: #000;
        }

        #controls {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        /* Mobile Controls */
        #mobileControls {
            position: fixed;
            bottom: 20px;
            left: 0;
            right: 0;
            display: none;
            justify-content: space-between;
            align-items: flex-end;
            padding: 0 20px;
            pointer-events: none;
            z-index: 1000;
        }

        .mobile-control-group {
            pointer-events: all;
        }

        .mobile-dpad {
            display: grid;
            grid-template-columns: repeat(3, 70px);
            grid-template-rows: repeat(3, 70px);
            gap: 5px;
        }

        .mobile-btn {
            background: rgba(0, 255, 157, 0.3);
            backdrop-filter: blur(10px);
            border: 2px solid rgba(0, 255, 157, 0.6);
            border-radius: 15px;
            color: #00ff9d;
            font-size: 2em;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            user-select: none;
            transition: all 0.1s ease;
            box-shadow: 0 4px 15px rgba(0, 255, 157, 0.3);
            font-family: 'Orbitron', sans-serif;
        }

        .mobile-btn:active {
            background: rgba(0, 255, 157, 0.6);
            transform: scale(0.95);
            box-shadow: 0 2px 8px rgba(0, 255, 157, 0.5);
        }

        .mobile-btn.empty {
            background: transparent;
            border: none;
            box-shadow: none;
            pointer-events: none;
        }

        .mobile-action-btns {
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .mobile-btn-large {
            width: 80px;
            height: 80px;
            background: rgba(255, 0, 128, 0.3);
            backdrop-filter: blur(10px);
            border: 3px solid rgba(255, 0, 128, 0.6);
            border-radius: 50%;
            color: #ff0080;
            font-size: 2.5em;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            user-select: none;
            transition: all 0.1s ease;
            box-shadow: 0 4px 20px rgba(255, 0, 128, 0.4);
            font-family: 'Orbitron', sans-serif;
        }

        .mobile-btn-large:active {
            background: rgba(255, 0, 128, 0.6);
            transform: scale(0.95);
            box-shadow: 0 2px 10px rgba(255, 0, 128, 0.6);
        }

        .mobile-btn-shoot {
            width: 100px;
            height: 100px;
            background: rgba(255, 215, 0, 0.3);
            border-color: rgba(255, 215, 0, 0.6);
            color: #ffd700;
            font-size: 3em;
        }

        .mobile-btn-shoot:active {
            background: rgba(255, 215, 0, 0.6);
        }

        @media (max-width: 768px) {
            h1 {
                font-size: 2.2em;
            }

            .subtitle {
                font-size: 1em;
            }

            #mobileControls {
                display: flex;
            }

            .instructions {
                font-size: 0.9em;
            }

            .mobile-dpad {
                grid-template-columns: repeat(3, 60px);
                grid-template-rows: repeat(3, 60px);
            }

            .mobile-btn-large {
                width: 70px;
                height: 70px;
                font-size: 2em;
            }

            .mobile-btn-shoot {
                width: 85px;
                height: 85px;
                font-size: 2.5em;
            }
        }

        .control-card {
            background: rgba(255, 255, 255, 0.03);
            backdrop-filter: blur(20px);
            border: 2px solid rgba(0, 255, 157, 0.2);
            border-radius: 15px;
            padding: 25px;
            transition: all 0.3s ease;
        }

        .control-card:hover {
            border-color: #00ff9d;
            background: rgba(0, 255, 157, 0.05);
            transform: translateY(-5px);
            box-shadow: 0 10px 30px rgba(0, 255, 157, 0.2);
        }

        .control-title {
            font-family: 'Orbitron', sans-serif;
            font-size: 1.1em;
            color: #00ff9d;
            margin-bottom: 10px;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .control-desc {
            color: #ccc;
            font-size: 1.05em;
            line-height: 1.6;
        }

        #stats {
            background: linear-gradient(135deg, rgba(0, 255, 157, 0.1) 0%, rgba(0, 184, 255, 0.1) 100%);
            backdrop-filter: blur(20px);
            border: 2px solid rgba(0, 255, 157, 0.3);
            border-radius: 15px;
            padding: 30px;
            text-align: center;
            margin-bottom: 30px;
        }

        #stats h3 {
            font-family: 'Orbitron', sans-serif;
            color: #00ff9d;
            font-size: 1.4em;
            margin-bottom: 20px;
            text-transform: uppercase;
            letter-spacing: 2px;
        }

        .stat-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
        }

        .stat-item {
            background: rgba(0, 0, 0, 0.3);
            padding: 20px;
            border-radius: 10px;
            border: 1px solid rgba(0, 255, 157, 0.2);
        }

        .stat-value {
            font-family: 'Orbitron', sans-serif;
            font-size: 2.5em;
            color: #00ff9d;
            font-weight: 900;
            display: block;
            margin-bottom: 5px;
        }

        .stat-label {
            color: #888;
            font-size: 0.95em;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .btn {
            font-family: 'Orbitron', sans-serif;
            background: linear-gradient(135deg, #00ff9d 0%, #00b8ff 100%);
            color: #0a0e27;
            border: none;
            padding: 15px 40px;
            font-size: 1.1em;
            font-weight: 700;
            border-radius: 50px;
            cursor: pointer;
            text-transform: uppercase;
            letter-spacing: 2px;
            transition: all 0.3s ease;
            box-shadow: 0 5px 20px rgba(0, 255, 157, 0.4);
            margin: 10px;
        }

        .btn:hover {
            transform: translateY(-3px);
            box-shadow: 0 10px 35px rgba(0, 255, 157, 0.6);
        }

        .btn:active {
            transform: translateY(-1px);
        }

        .instructions {
            background: rgba(255, 0, 128, 0.1);
            border: 2px solid rgba(255, 0, 128, 0.3);
            border-left: 5px solid #ff0080;
            padding: 25px;
            border-radius: 10px;
            margin-bottom: 30px;
            backdrop-filter: blur(10px);
        }

        .instructions h3 {
            font-family: 'Orbitron', sans-serif;
            color: #ff0080;
            margin-bottom: 15px;
            font-size: 1.3em;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .instructions ol {
            margin-left: 25px;
            color: #ccc;
            line-height: 2;
        }

        .instructions li {
            margin-bottom: 8px;
        }

        #detectionResults {
            background: rgba(0, 184, 255, 0.1);
            border: 2px solid rgba(0, 184, 255, 0.3);
            border-radius: 10px;
            padding: 20px;
            margin-bottom: 20px;
            display: none;
        }

        #detectionResults h4 {
            font-family: 'Orbitron', sans-serif;
            color: #00b8ff;
            margin-bottom: 15px;
            font-size: 1.2em;
        }

        .detection-item {
            background: rgba(0, 0, 0, 0.3);
            padding: 10px 15px;
            margin: 8px 0;
            border-radius: 8px;
            border-left: 3px solid #00b8ff;
            color: #ccc;
        }
    </style>
</head>
<body>
    <div id="app">
        <h1>⚡ AI Image Game ⚡</h1>
        <p class="subtitle">Control Characters • Break Objects • Realistic Physics</p>

        <div class="instructions">
            <h3>🎮 How It Works</h3>
            <ol>
                <li><strong>Upload an image</strong> with people in it (the clearest person becomes playable)</li>
                <li><strong>AI analyzes</strong> and identifies the main person and breakable objects</li>
                <li><strong>Control the person</strong> using arrow keys, WASD, or touch controls on mobile</li>
                <li><strong>Break objects</strong> by running into them or shooting</li>
                <li><strong>Watch realistic physics</strong> as objects shatter and fly apart!</li>
            </ol>
        </div>

        <div id="uploadSection">
            <label for="imageUpload" class="upload-zone">
                <span class="upload-icon">👤</span>
                <div class="upload-text">Click to Upload Image</div>
                <div class="upload-hint">Best with clear photos of people and objects</div>
            </label>
            <input type="file" id="imageUpload" accept="image/*">
        </div>

        <div id="processingSection">
            <div class="processing-spinner"></div>
            <div class="processing-text">🤖 AI Analyzing Your Image...</div>
            <div class="processing-detail" id="processingDetail">Detecting characters and objects</div>
        </div>

        <div id="gameSection">
            <div id="detectionResults"></div>
            
            <canvas id="gameCanvas"></canvas>

            <!-- Mobile Touch Controls -->
            <div id="mobileControls">
                <div class="mobile-control-group">
                    <div class="mobile-dpad">
                        <div class="mobile-btn empty"></div>
                        <div class="mobile-btn" data-key="up">▲</div>
                        <div class="mobile-btn empty"></div>
                        <div class="mobile-btn" data-key="left">◀</div>
                        <div class="mobile-btn empty"></div>
                        <div class="mobile-btn" data-key="right">▶</div>
                        <div class="mobile-btn empty"></div>
                        <div class="mobile-btn" data-key="down">▼</div>
                        <div class="mobile-btn empty"></div>
                    </div>
                </div>
                <div class="mobile-control-group">
                    <div class="mobile-action-btns">
                        <div class="mobile-btn-large" id="jumpBtn">⬆</div>
                        <div class="mobile-btn-large mobile-btn-shoot" id="shootBtn">💥</div>
                    </div>
                </div>
            </div>

            <div id="stats">
                <h3>📊 Game Stats</h3>
                <div class="stat-grid">
                    <div class="stat-item">
                        <span class="stat-value" id="objectsBroken">0</span>
                        <span class="stat-label">Objects Destroyed</span>
                    </div>
                    <div class="stat-item">
                        <span class="stat-value" id="collisions">0</span>
                        <span class="stat-label">Collisions</span>
                    </div>
                    <div class="stat-item">
                        <span class="stat-value" id="shotsfired">0</span>
                        <span class="stat-label">Shots Fired</span>
                    </div>
                    <div class="stat-item">
                        <span class="stat-value" id="gameTime">0:00</span>
                        <span class="stat-label">Play Time</span>
                    </div>
                </div>
            </div>

            <div id="controls">
                <div class="control-card">
                    <div class="control-title">⌨️ Movement</div>
                    <div class="control-desc">Desktop: <strong>Arrow Keys</strong> or <strong>WASD</strong><br>Mobile: <strong>Touch D-Pad</strong> (bottom left)</div>
                </div>
                <div class="control-card">
                    <div class="control-title">🚀 Jump</div>
                    <div class="control-desc">Desktop: Press <strong>SPACE</strong><br>Mobile: Tap <strong>⬆ button</strong></div>
                </div>
                <div class="control-card">
                    <div class="control-title">💥 Shoot</div>
                    <div class="control-desc">Desktop: <strong>Click anywhere</strong><br>Mobile: Tap <strong>💥 button</strong> (bottom right)</div>
                </div>
                <div class="control-card">
                    <div class="control-title">🔄 Reset</div>
                    <div class="control-desc">Press <strong>R</strong> key or use the Reset button below</div>
                </div>
            </div>

            <div style="text-align: center;">
                <button class="btn" onclick="resetGame()">🔄 Reset Game</button>
                <button class="btn" onclick="location.reload()">📁 Upload New Image</button>
            </div>
        </div>
    </div>

    <script>
        const { Engine, Render, Runner, Bodies, World, Events, Body, Composite } = Matter;

        let engine, render, runner, player, ground;
        let uploadedImage = null;
        let imageCanvas = null;
        let keys = {};
        let canJump = false;
        let breakableObjects = [];
        let bullets = [];
        let gameStartTime;
        let gameTimeInterval;

        // Stats
        let stats = {
            objectsBroken: 0,
            collisions: 0,
            shotsFired: 0
        };

        const MOVE_FORCE = 0.015;
        const JUMP_FORCE = 0.18;
        const MAX_SPEED = 10;

        // Handle image upload
        document.getElementById('imageUpload').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(event) {
                    const img = new Image();
                    img.onload = function() {
                        uploadedImage = img;
                        processImageWithAI(img);
                    };
                    img.src = event.target.result;
                };
                reader.readAsDataURL(file);
            }
        });

        async function processImageWithAI(img) {
            // Show processing
            document.getElementById('uploadSection').style.display = 'none';
            document.getElementById('processingSection').style.display = 'block';

            // Simulate AI processing with detailed steps
            const steps = [
                "Analyzing image composition...",
                "Detecting characters and entities...",
                "Identifying breakable objects...",
                "Extracting character regions...",
                "Mapping physics boundaries...",
                "Initializing game engine..."
            ];

            for (let i = 0; i < steps.length; i++) {
                document.getElementById('processingDetail').textContent = steps[i];
                await new Promise(resolve => setTimeout(resolve, 800));
            }

            // Use Claude AI to analyze the image
            await analyzeImageWithClaude(img);
        }

        async function analyzeImageWithClaude(img) {
            // Convert image to base64
            const canvas = document.createElement('canvas');
            canvas.width = img.width;
            canvas.height = img.height;
            const ctx = canvas.getContext('2d');
            ctx.drawImage(img, 0, 0);
            const base64Data = canvas.toDataURL('image/jpeg').split(',')[1];

            try {
                const response = await fetch("https://api.anthropic.com/v1/messages", {
                    method: "POST",
                    headers: {
                        "Content-Type": "application/json",
                    },
                    body: JSON.stringify({
                        model: "claude-sonnet-4-20250514",
                        max_tokens: 1000,
                        messages: [
                            {
                                role: "user",
                                content: [
                                    {
                                        type: "image",
                                        source: {
                                            type: "base64",
                                            media_type: "image/jpeg",
                                            data: base64Data
                                        }
                                    },
                                    {
                                        type: "text",
                                        text: `Analyze this image for a physics-based game. Your task:

1. FIND THE CLEAREST PERSON: Identify the MOST CLEAR, PROMINENT, and IN-FOCUS person in the image. This will be the playable character. Choose the person who is:
   - Most clearly visible (not blurry or obscured)
   - Largest or most prominent in the frame
   - Most centered or well-positioned
   - Has the most distinct features
   Provide their position as percentage from left (x) and top (y), and approximate size (width and height) as percentage of image dimensions.

2. BREAKABLE OBJECTS: Identify 8-15 distinct physical objects in the scene that could be broken in a game (furniture, items, props, decorations, etc.). Avoid selecting other people. Provide position and size for each.

Respond in this EXACT JSON format with no preamble or markdown:
{
  "character": {
    "description": "brief description of the person (e.g., 'woman in red dress', 'man with glasses')",
    "clarity": "high/medium",
    "x": 0.0-1.0,
    "y": 0.0-1.0,
    "width": 0.0-1.0,
    "height": 0.0-1.0
  },
  "objects": [
    {
      "name": "object name",
      "type": "furniture/item/prop/decoration",
      "x": 0.0-1.0,
      "y": 0.0-1.0,
      "width": 0.0-1.0,
      "height": 0.0-1.0
    }
  ]
}`
                                    }
                                ]
                            }
                        ]
                    })
                });

                const data = await response.json();
                const aiResponse = data.content[0].text;
                
                // Parse AI response
                const jsonMatch = aiResponse.match(/\{[\s\S]*\}/);
                if (jsonMatch) {
                    const detectionData = JSON.parse(jsonMatch[0]);
                    initializeGame(img, detectionData);
                } else {
                    // Fallback to default positions
                    initializeGame(img, null);
                }
            } catch (error) {
                console.error("AI Analysis error:", error);
                // Fallback to default positions
                initializeGame(img, null);
            }
        }

        function initializeGame(img, detectionData) {
            // Hide processing, show game
            document.getElementById('processingSection').style.display = 'none';
            document.getElementById('gameSection').style.display = 'block';

            // Display detection results
            if (detectionData) {
                const resultsDiv = document.getElementById('detectionResults');
                resultsDiv.style.display = 'block';
                resultsDiv.innerHTML = `
                    <h4>🎯 AI Detection Results</h4>
                    <div class="detection-item">
                        <strong>Playable Person:</strong> ${detectionData.character.description}
                        ${detectionData.character.clarity ? ` (${detectionData.character.clarity} clarity)` : ''}
                    </div>
                    <div class="detection-item">
                        <strong>Breakable Objects:</strong> ${detectionData.objects.length} items detected in scene
                    </div>
                `;
            }

            const canvas = document.getElementById('gameCanvas');
            const containerWidth = Math.min(window.innerWidth - 40, 1400);
            const aspectRatio = img.height / img.width;
            const canvasWidth = containerWidth;
            const canvasHeight = Math.min(canvasWidth * aspectRatio, 800);

            canvas.width = canvasWidth;
            canvas.height = canvasHeight;

            // Create image canvas for background
            imageCanvas = document.createElement('canvas');
            imageCanvas.width = canvasWidth;
            imageCanvas.height = canvasHeight;
            const imgCtx = imageCanvas.getContext('2d');
            imgCtx.drawImage(img, 0, 0, canvasWidth, canvasHeight);

            // Create engine
            engine = Engine.create();
            engine.gravity.y = 1.2;

            // Create renderer
            render = Render.create({
                canvas: canvas,
                engine: engine,
                options: {
                    width: canvasWidth,
                    height: canvasHeight,
                    wireframes: false,
                    background: 'transparent'
                }
            });

            // Draw background image
            const ctx = canvas.getContext('2d');
            Events.on(render, 'afterRender', function() {
                ctx.save();
                ctx.globalAlpha = 0.8;
                ctx.drawImage(imageCanvas, 0, 0);
                ctx.restore();
            });

            // Create boundaries
            createBoundaries(canvasWidth, canvasHeight);

            // Create player based on AI detection or default
            if (detectionData && detectionData.character) {
                const char = detectionData.character;
                const playerX = canvasWidth * char.x;
                const playerY = canvasHeight * char.y;
                const playerSize = Math.max(30, canvasWidth * char.width * 0.8);
                
                player = Bodies.circle(playerX, playerY, playerSize / 2, {
                    restitution: 0.3,
                    friction: 0.1,
                    frictionAir: 0.02,
                    density: 0.05,
                    render: {
                        fillStyle: 'rgba(255, 64, 129, 0.7)',
                        strokeStyle: '#FF0080',
                        lineWidth: 4
                    },
                    label: 'player'
                });
            } else {
                // Default player position
                player = Bodies.circle(100, canvasHeight - 100, 35, {
                    restitution: 0.3,
                    friction: 0.1,
                    frictionAir: 0.02,
                    density: 0.05,
                    render: {
                        fillStyle: 'rgba(255, 64, 129, 0.7)',
                        strokeStyle: '#FF0080',
                        lineWidth: 4
                    },
                    label: 'player'
                });
            }

            // Create breakable objects based on AI detection
            if (detectionData && detectionData.objects) {
                detectionData.objects.forEach(obj => {
                    const objX = canvasWidth * obj.x;
                    const objY = canvasHeight * obj.y;
                    const objWidth = Math.max(25, canvasWidth * obj.width);
                    const objHeight = Math.max(25, canvasHeight * obj.height);
                    
                    createBreakableObject(objX, objY, objWidth, objHeight, obj.name);
                });
            } else {
                // Create default objects
                createDefaultObjects(canvasWidth, canvasHeight);
            }

            // Add all bodies to world
            World.add(engine.world, [player, ...breakableObjects]);

            // Collision detection
            Events.on(engine, 'collisionStart', function(event) {
                event.pairs.forEach(pair => {
                    if (pair.bodyA === player || pair.bodyB === player) {
                        canJump = true;
                        stats.collisions++;
                        updateStats();
                    }

                    // Check for high-speed collisions to break objects
                    const otherBody = pair.bodyA === player ? pair.bodyB : pair.bodyA;
                    if (otherBody.label === 'breakable') {
                        const velocity = Math.sqrt(
                            Math.pow(otherBody.velocity.x, 2) + 
                            Math.pow(otherBody.velocity.y, 2)
                        );
                        if (velocity > 12) {
                            shatterObject(otherBody);
                        }
                    }
                });
            });

            // Game loop
            Events.on(engine, 'beforeUpdate', function() {
                updatePlayer();
                updateBullets();
                cleanupObjects();
            });

            // Run engine
            runner = Runner.create();
            Runner.run(runner, engine);
            Render.run(render);

            setupControls(canvas);
            startGameTimer();
        }

        function createBoundaries(width, height) {
            ground = Bodies.rectangle(width / 2, height - 10, width, 20, {
                isStatic: true,
                render: {
                    fillStyle: 'rgba(0, 255, 157, 0.3)',
                    strokeStyle: '#00FF9D',
                    lineWidth: 2
                }
            });

            const leftWall = Bodies.rectangle(10, height / 2, 20, height, {
                isStatic: true,
                render: {
                    fillStyle: 'rgba(0, 255, 157, 0.2)',
                    strokeStyle: '#00FF9D',
                    lineWidth: 2
                }
            });

            const rightWall = Bodies.rectangle(width - 10, height / 2, 20, height, {
                isStatic: true,
                render: {
                    fillStyle: 'rgba(0, 255, 157, 0.2)',
                    strokeStyle: '#00FF9D',
                    lineWidth: 2
                }
            });

            World.add(engine.world, [ground, leftWall, rightWall]);
        }

        function createBreakableObject(x, y, width, height, name) {
            const colors = ['#00FF9D', '#00B8FF', '#FF0080', '#FFD700', '#FF6B6B'];
            const color = colors[Math.floor(Math.random() * colors.length)];
            
            const obj = Bodies.rectangle(x, y, width, height, {
                restitution: 0.7,
                friction: 0.5,
                density: 0.002,
                render: {
                    fillStyle: `${color}80`,
                    strokeStyle: color,
                    lineWidth: 3
                },
                label: 'breakable',
                customName: name
            });
            
            breakableObjects.push(obj);
        }

        function createDefaultObjects(width, height) {
            const objectTypes = [
                { w: 40, h: 40, name: 'Box' },
                { w: 60, h: 30, name: 'Crate' },
                { w: 35, h: 50, name: 'Vase' },
                { w: 50, h: 50, name: 'Block' }
            ];

            for (let i = 0; i < 12; i++) {
                const type = objectTypes[Math.floor(Math.random() * objectTypes.length)];
                const x = Math.random() * (width - 200) + 100;
                const y = Math.random() * (height - 400) + 50;
                createBreakableObject(x, y, type.w, type.h, type.name);
            }
        }

        function setupControls(canvas) {
            // Keyboard controls
            document.addEventListener('keydown', (e) => {
                keys[e.key.toLowerCase()] = true;
                
                if (e.key === ' ' || e.key === 'Spacebar') {
                    e.preventDefault();
                    if (canJump) {
                        Body.applyForce(player, player.position, { x: 0, y: -JUMP_FORCE });
                        canJump = false;
                    }
                }

                if (e.key.toLowerCase() === 'r') {
                    resetGame();
                }
            });

            document.addEventListener('keyup', (e) => {
                keys[e.key.toLowerCase()] = false;
            });

            // Mouse controls for shooting
            canvas.addEventListener('click', (e) => {
                const rect = canvas.getBoundingClientRect();
                const mouseX = (e.clientX - rect.left) * (canvas.width / rect.width);
                const mouseY = (e.clientY - rect.top) * (canvas.height / rect.height);
                shootBullet(mouseX, mouseY);
            });

            // Mobile touch controls
            setupMobileControls(canvas);
        }

        function setupMobileControls(canvas) {
            // D-Pad controls
            const dpadButtons = document.querySelectorAll('.mobile-btn[data-key]');
            dpadButtons.forEach(btn => {
                const key = btn.getAttribute('data-key');
                
                btn.addEventListener('touchstart', (e) => {
                    e.preventDefault();
                    if (key === 'up') keys['w'] = true;
                    else if (key === 'down') keys['s'] = true;
                    else if (key === 'left') keys['a'] = true;
                    else if (key === 'right') keys['d'] = true;
                });
                
                btn.addEventListener('touchend', (e) => {
                    e.preventDefault();
                    if (key === 'up') keys['w'] = false;
                    else if (key === 'down') keys['s'] = false;
                    else if (key === 'left') keys['a'] = false;
                    else if (key === 'right') keys['d'] = false;
                });
            });

            // Jump button
            const jumpBtn = document.getElementById('jumpBtn');
            jumpBtn.addEventListener('touchstart', (e) => {
                e.preventDefault();
                if (canJump) {
                    Body.applyForce(player, player.position, { x: 0, y: -JUMP_FORCE });
                    canJump = false;
                }
            });

            // Shoot button
            const shootBtn = document.getElementById('shootBtn');
            shootBtn.addEventListener('touchstart', (e) => {
                e.preventDefault();
                // Shoot in the direction the player is facing or forward
                const targetX = player.position.x + (player.velocity.x > 0 ? 200 : -200);
                const targetY = player.position.y - 100;
                shootBullet(targetX, targetY);
            });

            // Touch to shoot anywhere on canvas (mobile alternative)
            let lastTapTime = 0;
            canvas.addEventListener('touchstart', (e) => {
                const currentTime = Date.now();
                const timeSinceLastTap = currentTime - lastTapTime;
                
                // Double tap to shoot
                if (timeSinceLastTap < 300 && timeSinceLastTap > 0) {
                    e.preventDefault();
                    const rect = canvas.getBoundingClientRect();
                    const touch = e.touches[0];
                    const touchX = (touch.clientX - rect.left) * (canvas.width / rect.width);
                    const touchY = (touch.clientY - rect.top) * (canvas.height / rect.height);
                    shootBullet(touchX, touchY);
                }
                
                lastTapTime = currentTime;
            });
        }

        function updatePlayer() {
            if (!player) return;

            if (keys['arrowleft'] || keys['a']) {
                if (Math.abs(player.velocity.x) < MAX_SPEED) {
                    Body.applyForce(player, player.position, { x: -MOVE_FORCE, y: 0 });
                }
            }
            if (keys['arrowright'] || keys['d']) {
                if (Math.abs(player.velocity.x) < MAX_SPEED) {
                    Body.applyForce(player, player.position, { x: MOVE_FORCE, y: 0 });
                }
            }

            if (Math.abs(player.velocity.x) > MAX_SPEED) {
                Body.setVelocity(player, { 
                    x: Math.sign(player.velocity.x) * MAX_SPEED, 
                    y: player.velocity.y 
                });
            }
        }

        function shootBullet(targetX, targetY) {
            const angle = Math.atan2(targetY - player.position.y, targetX - player.position.x);
            const speed = 25;
            
            const bullet = Bodies.circle(player.position.x, player.position.y, 10, {
                restitution: 0.95,
                friction: 0,
                frictionAir: 0,
                density: 0.08,
                render: {
                    fillStyle: '#FFD700',
                    strokeStyle: '#FF8C00',
                    lineWidth: 3
                },
                label: 'bullet'
            });

            Body.setVelocity(bullet, {
                x: Math.cos(angle) * speed,
                y: Math.sin(angle) * speed
            });

            World.add(engine.world, bullet);
            bullets.push({ body: bullet, life: 120 });
            stats.shotsFired++;
            updateStats();
        }

        function updateBullets() {
            bullets = bullets.filter(bulletObj => {
                bulletObj.life--;
                
                if (bulletObj.life <= 0) {
                    World.remove(engine.world, bulletObj.body);
                    return false;
                }

                // Check collisions with breakable objects
                breakableObjects.forEach(obj => {
                    const distance = Math.hypot(
                        bulletObj.body.position.x - obj.position.x,
                        bulletObj.body.position.y - obj.position.y
                    );

                    if (distance < 60) {
                        const angle = Math.atan2(
                            obj.position.y - bulletObj.body.position.y,
                            obj.position.x - bulletObj.body.position.x
                        );
                        Body.applyForce(obj, obj.position, {
                            x: Math.cos(angle) * 0.15,
                            y: Math.sin(angle) * 0.15
                        });
                    }
                });

                return true;
            });
        }

        function shatterObject(obj) {
            const index = breakableObjects.indexOf(obj);
            if (index > -1) {
                breakableObjects.splice(index, 1);
                World.remove(engine.world, obj);
                stats.objectsBroken++;
                updateStats();

                // Create shatter particles
                for (let i = 0; i < 6; i++) {
                    const particle = Bodies.circle(
                        obj.position.x,
                        obj.position.y,
                        Math.random() * 8 + 4,
                        {
                            restitution: 0.8,
                            friction: 0.3,
                            density: 0.001,
                            render: {
                                fillStyle: obj.render.strokeStyle + '80'
                            },
                            label: 'particle'
                        }
                    );
                    
                    Body.setVelocity(particle, {
                        x: (Math.random() - 0.5) * 20,
                        y: (Math.random() - 0.5) * 20
                    });
                    
                    World.add(engine.world, particle);
                    
                    setTimeout(() => {
                        World.remove(engine.world, particle);
                    }, 2000);
                }
            }
        }

        function cleanupObjects() {
            // Remove objects that fell off screen
            breakableObjects = breakableObjects.filter(obj => {
                if (obj.position.y > render.canvas.height + 200) {
                    World.remove(engine.world, obj);
                    return false;
                }
                return true;
            });

            // Check for objects with high velocity
            breakableObjects.forEach(obj => {
                const velocity = Math.sqrt(
                    Math.pow(obj.velocity.x, 2) + 
                    Math.pow(obj.velocity.y, 2)
                );
                if (velocity > 20) {
                    shatterObject(obj);
                }
            });
        }

        function startGameTimer() {
            gameStartTime = Date.now();
            gameTimeInterval = setInterval(() => {
                const elapsed = Math.floor((Date.now() - gameStartTime) / 1000);
                const minutes = Math.floor(elapsed / 60);
                const seconds = elapsed % 60;
                document.getElementById('gameTime').textContent = 
                    `${minutes}:${seconds.toString().padStart(2, '0')}`;
            }, 1000);
        }

        function updateStats() {
            document.getElementById('objectsBroken').textContent = stats.objectsBroken;
            document.getElementById('collisions').textContent = stats.collisions;
            document.getElementById('shotsFired').textContent = stats.shotsFired;
        }

        function resetGame() {
            // Reset stats
            stats = {
                objectsBroken: 0,
                collisions: 0,
                shotsFired: 0
            };
            updateStats();

            // Clear bullets
            bullets.forEach(bulletObj => World.remove(engine.world, bulletObj.body));
            bullets = [];

            // Reset breakable objects
            breakableObjects.forEach(obj => World.remove(engine.world, obj));
            breakableObjects = [];

            // Create new objects
            if (imageCanvas) {
                createDefaultObjects(render.canvas.width, render.canvas.height);
                World.add(engine.world, breakableObjects);
            }

            // Reset player
            Body.setPosition(player, { x: 100, y: render.canvas.height - 100 });
            Body.setVelocity(player, { x: 0, y: 0 });
            Body.setAngularVelocity(player, 0);

            // Reset timer
            clearInterval(gameTimeInterval);
            startGameTimer();
        }
    </script>
</body>
</html>

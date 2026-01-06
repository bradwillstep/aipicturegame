<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Play As Yourself - AI Image Game</title>
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
            margin-bottom: 15px;
        }

        .extraction-preview {
            display: none;
            margin-top: 20px;
            padding: 20px;
            background: rgba(0, 0, 0, 0.3);
            border-radius: 10px;
            border: 2px solid rgba(0, 255, 157, 0.3);
        }

        .extraction-preview canvas {
            max-width: 200px;
            max-height: 200px;
            border: 2px solid #00ff9d;
            border-radius: 10px;
            margin: 10px auto;
            display: block;
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
        }

        .mobile-btn:active {
            background: rgba(0, 255, 157, 0.6);
            transform: scale(0.95);
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
        }

        .mobile-btn-large:active {
            background: rgba(255, 0, 128, 0.6);
            transform: scale(0.95);
        }

        @media (max-width: 768px) {
            h1 { font-size: 2.2em; }
            .subtitle { font-size: 1em; }
            #mobileControls { display: flex; }
            .mobile-dpad {
                grid-template-columns: repeat(3, 60px);
                grid-template-rows: repeat(3, 60px);
            }
            .mobile-btn-large {
                width: 70px;
                height: 70px;
                font-size: 2em;
            }
        }

        #controls {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
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
        <h1>🎮 Play As Yourself</h1>
        <p class="subtitle">AI-Powered Character Extraction • Realistic Movement • Physics Interactions</p>

        <div class="instructions">
            <h3>🚀 How It Works</h3>
            <ol>
                <li><strong>Upload a photo</strong> with a person in it (yourself, friend, anyone!)</li>
                <li><strong>AI extracts the person</strong> and creates a playable character sprite</li>
                <li><strong>The environment becomes your playground</strong> - move around the scene</li>
                <li><strong>Interact with detected objects</strong> - push, break, and destroy them!</li>
                <li><strong>Realistic walking/running</strong> animations and physics</li>
            </ol>
        </div>

        <div id="uploadSection">
            <label for="imageUpload" class="upload-zone">
                <span class="upload-icon">👤</span>
                <div class="upload-text">Upload Your Photo</div>
                <div class="upload-hint">Works best with clear photos of people standing</div>
            </label>
            <input type="file" id="imageUpload" accept="image/*">
        </div>

        <div id="processingSection">
            <div class="processing-spinner"></div>
            <div class="processing-text">🤖 AI Processing Your Image...</div>
            <div class="processing-detail" id="processingDetail">Analyzing image...</div>
            <div class="extraction-preview" id="extractionPreview">
                <div style="color: #00ff9d; margin-bottom: 10px; font-weight: 600;">Character Extracted:</div>
                <canvas id="characterPreview"></canvas>
            </div>
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
                        <div class="mobile-btn-large" id="kickBtn" style="background: rgba(255, 215, 0, 0.3); border-color: rgba(255, 215, 0, 0.6); color: #ffd700;">👊</div>
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
                        <span class="stat-value" id="interactions">0</span>
                        <span class="stat-label">Interactions</span>
                    </div>
                    <div class="stat-item">
                        <span class="stat-value" id="distance">0m</span>
                        <span class="stat-label">Distance Traveled</span>
                    </div>
                    <div class="stat-item">
                        <span class="stat-value" id="gameTime">0:00</span>
                        <span class="stat-label">Play Time</span>
                    </div>
                </div>
            </div>

            <div id="controls">
                <div class="control-card">
                    <div class="control-title">🏃 Move & Run</div>
                    <div class="control-desc">Desktop: <strong>A/D or Arrows</strong><br>Mobile: <strong>Left/Right buttons</strong><br>Character runs with realistic animation!</div>
                </div>
                <div class="control-card">
                    <div class="control-title">🚀 Jump</div>
                    <div class="control-desc">Desktop: <strong>SPACE or W</strong><br>Mobile: <strong>⬆ button</strong><br>Jump over obstacles and platforms</div>
                </div>
                <div class="control-card">
                    <div class="control-title">👊 Kick/Push</div>
                    <div class="control-desc">Desktop: <strong>E or Click</strong><br>Mobile: <strong>👊 button</strong><br>Push and break objects!</div>
                </div>
                <div class="control-card">
                    <div class="control-title">🔄 Reset</div>
                    <div class="control-desc">Press <strong>R</strong> or click Reset button below</div>
                </div>
            </div>

            <div style="text-align: center;">
                <button class="btn" onclick="resetGame()">🔄 Reset Game</button>
                <button class="btn" onclick="location.reload()">📁 Upload New Image</button>
            </div>
        </div>
    </div>

    <script>
        const { Engine, Render, Runner, Bodies, World, Events, Body } = Matter;

        let engine, render, runner, player;
        let uploadedImage = null;
        let playerSprite = null;
        let backgroundCanvas = null;
        let keys = {};
        let canJump = false;
        let interactiveObjects = [];
        let facingRight = true;
        let isWalking = false;
        let walkCycle = 0;
        let totalDistance = 0;
        let lastPlayerX = 0;

        // Game stats
        let stats = {
            objectsBroken: 0,
            interactions: 0,
            distance: 0
        };

        let gameStartTime;
        let gameTimeInterval;

        const MOVE_FORCE = 0.02;
        const JUMP_FORCE = 0.2;
        const MAX_SPEED = 8;
        const KICK_FORCE = 0.25;

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
            document.getElementById('uploadSection').style.display = 'none';
            document.getElementById('processingSection').style.display = 'block';

            const steps = [
                "Analyzing image composition...",
                "Detecting the main person...",
                "Extracting character boundaries...",
                "Creating character sprite...",
                "Identifying interactive objects...",
                "Building game environment...",
                "Setting up physics engine..."
            ];

            for (let i = 0; i < steps.length; i++) {
                document.getElementById('processingDetail').textContent = steps[i];
                await new Promise(resolve => setTimeout(resolve, 700));
            }

            await analyzeImageWithClaude(img);
        }

        async function analyzeImageWithClaude(img) {
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
                        max_tokens: 1500,
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
                                        text: `Analyze this image for an interactive physics game where the person becomes a playable character.

CRITICAL TASK 1 - FIND THE CLEAREST PERSON:
Identify the MOST CLEAR, PROMINENT, and WELL-DEFINED person who will become the playable character:
- Choose the person with the sharpest focus and clearest outline
- Prefer people who are standing/full-body visible
- Select the largest or most prominent person in frame
- Provide PRECISE boundary box (x, y, width, height as 0.0-1.0 percentages)

CRITICAL TASK 2 - IDENTIFY INTERACTIVE OBJECTS:
Find 10-20 distinct PHYSICAL OBJECTS in the scene that can be interacted with:
- Furniture (chairs, tables, desks, shelves, cabinets)
- Items (bottles, boxes, books, vases, plants, decorations)
- Props (signs, frames, equipment, tools)
- Structures (walls, doors, windows - mark as static)
- DO NOT include other people
- DO NOT include ground/floor/ceiling by themselves
- Provide position and size for each object
- Mark if object should be "static" (unmovable like walls) or "dynamic" (can be pushed/broken)

Respond in EXACT JSON format with no markdown:
{
  "character": {
    "description": "detailed description of person's appearance and pose",
    "clarity": "high/medium/low",
    "x": 0.0-1.0,
    "y": 0.0-1.0,
    "width": 0.0-1.0,
    "height": 0.0-1.0,
    "posture": "standing/sitting/action"
  },
  "objects": [
    {
      "name": "descriptive name",
      "type": "furniture/item/prop/structure",
      "static": false,
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
                
                const jsonMatch = aiResponse.match(/\{[\s\S]*\}/);
                if (jsonMatch) {
                    const detectionData = JSON.parse(jsonMatch[0]);
                    await extractCharacterSprite(img, detectionData.character);
                    initializeGame(img, detectionData);
                } else {
                    // Fallback to default positions if AI fails
                    console.log('AI detection failed, using fallback');
                    
                    // Create a simple sprite from the center of the image
                    const fallbackCanvas = document.createElement('canvas');
                    fallbackCanvas.width = img.width * 0.15;
                    fallbackCanvas.height = img.height * 0.25;
                    const fallbackCtx = fallbackCanvas.getContext('2d');
                    
                    fallbackCtx.drawImage(
                        img,
                        img.width * 0.4, img.height * 0.3,
                        img.width * 0.15, img.height * 0.25,
                        0, 0,
                        fallbackCanvas.width, fallbackCanvas.height
                    );
                    
                    playerSprite = fallbackCanvas;
                    
                    initializeGame(img, null);
                }
            } catch (error) {
                console.error("AI Analysis error:", error);
                initializeGame(img, null);
            }
        }

        async function extractCharacterSprite(img, characterData) {
            const tempCanvas = document.createElement('canvas');
            const tempCtx = tempCanvas.getContext('2d');
            
            // Calculate character bounds with padding
            const padding = 0.05; // 5% padding around character
            let charX = Math.max(0, img.width * (characterData.x - characterData.width * padding));
            let charY = Math.max(0, img.height * (characterData.y - characterData.height * padding));
            let charWidth = Math.min(img.width - charX, img.width * characterData.width * (1 + padding * 2));
            let charHeight = Math.min(img.height - charY, img.height * characterData.height * (1 + padding * 2));
            
            // Ensure minimum size
            charWidth = Math.max(50, charWidth);
            charHeight = Math.max(50, charHeight);
            
            tempCanvas.width = charWidth;
            tempCanvas.height = charHeight;
            
            // Extract character from image
            tempCtx.drawImage(
                img,
                charX, charY, charWidth, charHeight,
                0, 0, charWidth, charHeight
            );
            
            playerSprite = tempCanvas;
            
            console.log('Character sprite extracted:', {
                width: charWidth,
                height: charHeight,
                x: charX,
                y: charY
            });
            
            // Show preview
            const previewCanvas = document.getElementById('characterPreview');
            previewCanvas.width = Math.min(charWidth, 200);
            previewCanvas.height = Math.min(charHeight, 200);
            const previewCtx = previewCanvas.getContext('2d');
            previewCtx.drawImage(
                tempCanvas, 
                0, 0, charWidth, charHeight,
                0, 0, previewCanvas.width, previewCanvas.height
            );
            
            document.getElementById('extractionPreview').style.display = 'block';
            await new Promise(resolve => setTimeout(resolve, 1500));
        }

        function initializeGame(img, detectionData) {
            document.getElementById('processingSection').style.display = 'none';
            document.getElementById('gameSection').style.display = 'block';

            if (detectionData) {
                const resultsDiv = document.getElementById('detectionResults');
                resultsDiv.style.display = 'block';
                resultsDiv.innerHTML = `
                    <h4>🎯 AI Detection Results</h4>
                    <div class="detection-item">
                        <strong>Playable Character:</strong> ${detectionData.character.description}
                        (${detectionData.character.clarity} clarity, ${detectionData.character.posture})
                    </div>
                    <div class="detection-item">
                        <strong>Interactive Objects:</strong> ${detectionData.objects.length} objects detected
                        (${detectionData.objects.filter(o => !o.static).length} movable, ${detectionData.objects.filter(o => o.static).length} static)
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

            // Create background (environment) canvas
            backgroundCanvas = document.createElement('canvas');
            backgroundCanvas.width = canvasWidth;
            backgroundCanvas.height = canvasHeight;
            const bgCtx = backgroundCanvas.getContext('2d');
            bgCtx.drawImage(img, 0, 0, canvasWidth, canvasHeight);

            // Create physics engine
            engine = Engine.create();
            engine.gravity.y = 1.5;

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

            // Custom render with character sprite
            const ctx = canvas.getContext('2d');
            Events.on(render, 'afterRender', function() {
                // Draw background environment
                ctx.save();
                ctx.globalAlpha = 0.7;
                ctx.drawImage(backgroundCanvas, 0, 0);
                ctx.restore();

                // Draw character sprite on player body
                if (player && playerSprite) {
                    ctx.save();
                    ctx.translate(player.position.x, player.position.y);
                    
                    // Don't rotate the character sprite with physics body
                    // ctx.rotate(player.angle);
                    
                    // Flip sprite if facing left
                    if (!facingRight) {
                        ctx.scale(-1, 1);
                    }
                    
                    // Apply walking animation (subtle bob)
                    const bobOffset = isWalking ? Math.sin(walkCycle) * 3 : 0;
                    
                    // Calculate sprite size to match physics body
                    const spriteScale = 1.5; // Make sprite bigger than physics body
                    const spriteWidth = playerSprite.width * 0.15 * spriteScale;
                    const spriteHeight = playerSprite.height * 0.15 * spriteScale;
                    
                    // Draw sprite centered on physics body
                    ctx.drawImage(
                        playerSprite,
                        -spriteWidth / 2,
                        -spriteHeight / 2 + bobOffset,
                        spriteWidth,
                        spriteHeight
                    );
                    
                    // Debug: Draw outline around character
                    ctx.strokeStyle = '#00FF9D';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(
                        -spriteWidth / 2,
                        -spriteHeight / 2 + bobOffset,
                        spriteWidth,
                        spriteHeight
                    );
                    
                    ctx.restore();
                } else {
                    // Fallback: draw a visible circle if no sprite
                    if (player) {
                        ctx.save();
                        ctx.fillStyle = '#FF0080';
                        ctx.strokeStyle = '#00FF9D';
                        ctx.lineWidth = 3;
                        ctx.beginPath();
                        ctx.arc(player.position.x, player.position.y, 30, 0, Math.PI * 2);
                        ctx.fill();
                        ctx.stroke();
                        ctx.restore();
                    }
                }

                // Update walk cycle
                if (isWalking) {
                    walkCycle += 0.3;
                }
            });

            // Create boundaries
            const ground = Bodies.rectangle(canvasWidth / 2, canvasHeight - 15, canvasWidth, 30, {
                isStatic: true,
                render: {
                    fillStyle: 'rgba(50, 50, 50, 0.5)',
                    strokeStyle: '#00FF9D',
                    lineWidth: 2
                },
                friction: 0.8
            });

            const leftWall = Bodies.rectangle(15, canvasHeight / 2, 30, canvasHeight, {
                isStatic: true,
                render: { fillStyle: 'rgba(50, 50, 50, 0.3)' }
            });

            const rightWall = Bodies.rectangle(canvasWidth - 15, canvasHeight / 2, 30, canvasHeight, {
                isStatic: true,
                render: { fillStyle: 'rgba(50, 50, 50, 0.3)' }
            });

            World.add(engine.world, [ground, leftWall, rightWall]);

            // Create player physics body
            let playerStartX = canvasWidth * 0.15;
            let playerStartY = canvasHeight * 0.8;
            let playerWidth = 50;
            let playerHeight = 80;

            if (detectionData && detectionData.character) {
                // Use AI-detected position but ensure it's on screen
                playerStartX = Math.max(100, Math.min(canvasWidth - 100, canvasWidth * detectionData.character.x));
                playerStartY = Math.max(100, Math.min(canvasHeight - 100, canvasHeight * detectionData.character.y));
                
                // Size based on detected character
                const detectedWidth = Math.max(40, Math.min(100, canvasWidth * detectionData.character.width * 0.3));
                const detectedHeight = Math.max(60, Math.min(150, canvasHeight * detectionData.character.height * 0.3));
                playerWidth = detectedWidth;
                playerHeight = detectedHeight;
            }

            player = Bodies.rectangle(playerStartX, playerStartY, playerWidth, playerHeight, {
                restitution: 0.1,
                friction: 0.8,
                frictionAir: 0.02,
                density: 0.002,
                render: {
                    visible: false // We'll draw the sprite instead
                },
                label: 'player',
                collisionFilter: {
                    group: -1,
                    category: 2,
                    mask: 1
                }
            });

            World.add(engine.world, player);
            lastPlayerX = player.position.x;

            console.log('Player created at:', {
                x: playerStartX,
                y: playerStartY,
                width: playerWidth,
                height: playerHeight,
                hasSprite: !!playerSprite
            });

            // Create interactive objects
            if (detectionData && detectionData.objects) {
                detectionData.objects.forEach(obj => {
                    createInteractiveObject(obj, canvasWidth, canvasHeight);
                });
            } else {
                createDefaultObjects(canvasWidth, canvasHeight);
            }

            World.add(engine.world, interactiveObjects);

            // Collision detection
            Events.on(engine, 'collisionStart', function(event) {
                event.pairs.forEach(pair => {
                    if (pair.bodyA === player || pair.bodyB === player) {
                        canJump = true;
                        
                        const otherBody = pair.bodyA === player ? pair.bodyB : pair.bodyA;
                        if (otherBody.label === 'interactive') {
                            stats.interactions++;
                            updateStats();
                        }
                    }
                });
            });

            // Game loop
            Events.on(engine, 'beforeUpdate', function() {
                updatePlayer();
                updatePhysics();
                updateDistance();
            });

            runner = Runner.create();
            Runner.run(runner, engine);
            Render.run(render);

            setupControls(canvas);
            startGameTimer();
        }

        function createInteractiveObject(objData, canvasWidth, canvasHeight) {
            const objX = canvasWidth * objData.x;
            const objY = canvasHeight * objData.y;
            const objWidth = Math.max(20, canvasWidth * objData.width);
            const objHeight = Math.max(20, canvasHeight * objData.height);

            const colors = ['#00FF9D', '#00B8FF', '#FF0080', '#FFD700', '#FF6B6B'];
            const color = colors[Math.floor(Math.random() * colors.length)];

            const obj = Bodies.rectangle(objX, objY, objWidth, objHeight, {
                isStatic: objData.static || false,
                restitution: objData.static ? 0 : 0.4,
                friction: 0.6,
                density: objData.static ? 0 : 0.001,
                render: {
                    fillStyle: objData.static ? 'rgba(100, 100, 100, 0.3)' : `${color}40`,
                    strokeStyle: objData.static ? '#666' : color,
                    lineWidth: 2
                },
                label: 'interactive',
                customName: objData.name,
                breakable: !objData.static
            });

            interactiveObjects.push(obj);
        }

        function createDefaultObjects(width, height) {
            // Create some default objects if AI detection failed
            const objects = [
                { x: 0.3, y: 0.7, w: 0.08, h: 0.08, name: 'Box 1' },
                { x: 0.5, y: 0.6, w: 0.1, h: 0.15, name: 'Crate' },
                { x: 0.7, y: 0.75, w: 0.06, h: 0.12, name: 'Bottle' },
                { x: 0.4, y: 0.5, w: 0.12, h: 0.08, name: 'Platform', static: true },
                { x: 0.8, y: 0.8, w: 0.07, h: 0.07, name: 'Box 2' }
            ];

            objects.forEach(obj => {
                createInteractiveObject({
                    x: obj.x,
                    y: obj.y,
                    width: obj.w,
                    height: obj.h,
                    name: obj.name,
                    type: 'item',
                    static: obj.static || false
                }, width, height);
            });
        }

        function setupControls(canvas) {
            console.log('Setting up controls...');
            
            // Keyboard
            document.addEventListener('keydown', (e) => {
                const key = e.key.toLowerCase();
                keys[key] = true;
                
                console.log('Key pressed:', key);
                
                if ((e.key === ' ' || key === 'w') && canJump) {
                    e.preventDefault();
                    Body.applyForce(player, player.position, { x: 0, y: -JUMP_FORCE });
                    canJump = false;
                    console.log('Jump!');
                }

                if (key === 'e') {
                    kickNearbyObjects();
                }

                if (key === 'r') {
                    resetGame();
                }
            });

            document.addEventListener('keyup', (e) => {
                keys[e.key.toLowerCase()] = false;
            });

            // Mouse click to kick
            canvas.addEventListener('click', (e) => {
                console.log('Canvas clicked');
                kickNearbyObjects();
            });

            // Mobile controls
            setupMobileControls();
            
            console.log('Controls setup complete');
        }

        function setupMobileControls() {
            console.log('Setting up mobile controls...');
            
            const dpadButtons = document.querySelectorAll('.mobile-btn[data-key]');
            dpadButtons.forEach(btn => {
                const key = btn.getAttribute('data-key');
                
                btn.addEventListener('touchstart', (e) => {
                    e.preventDefault();
                    console.log('Mobile button pressed:', key);
                    if (key === 'left') keys['a'] = true;
                    else if (key === 'right') keys['d'] = true;
                });
                
                btn.addEventListener('touchend', (e) => {
                    e.preventDefault();
                    if (key === 'left') keys['a'] = false;
                    else if (key === 'right') keys['d'] = false;
                });
                
                // Also add mouse events for desktop testing
                btn.addEventListener('mousedown', (e) => {
                    e.preventDefault();
                    console.log('Mobile button clicked:', key);
                    if (key === 'left') keys['a'] = true;
                    else if (key === 'right') keys['d'] = true;
                });
                
                btn.addEventListener('mouseup', (e) => {
                    e.preventDefault();
                    if (key === 'left') keys['a'] = false;
                    else if (key === 'right') keys['d'] = false;
                });
            });

            const jumpBtn = document.getElementById('jumpBtn');
            if (jumpBtn) {
                jumpBtn.addEventListener('touchstart', (e) => {
                    e.preventDefault();
                    if (canJump) {
                        Body.applyForce(player, player.position, { x: 0, y: -JUMP_FORCE });
                        canJump = false;
                        console.log('Mobile jump!');
                    }
                });
                
                jumpBtn.addEventListener('click', (e) => {
                    e.preventDefault();
                    if (canJump) {
                        Body.applyForce(player, player.position, { x: 0, y: -JUMP_FORCE });
                        canJump = false;
                        console.log('Mobile jump (click)!');
                    }
                });
            }

            const kickBtn = document.getElementById('kickBtn');
            if (kickBtn) {
                kickBtn.addEventListener('touchstart', (e) => {
                    e.preventDefault();
                    kickNearbyObjects();
                    console.log('Mobile kick!');
                });
                
                kickBtn.addEventListener('click', (e) => {
                    e.preventDefault();
                    kickNearbyObjects();
                    console.log('Mobile kick (click)!');
                });
            }
            
            console.log('Mobile controls ready');
        }

        function updatePlayer() {
            if (!player) return;

            isWalking = false;
            let moveX = 0;

            // Horizontal movement
            if (keys['arrowleft'] || keys['a']) {
                moveX = -1;
                facingRight = false;
                isWalking = true;
            }
            if (keys['arrowright'] || keys['d']) {
                moveX = 1;
                facingRight = true;
                isWalking = true;
            }

            // Apply movement force
            if (moveX !== 0) {
                if (Math.abs(player.velocity.x) < MAX_SPEED) {
                    Body.applyForce(player, player.position, { 
                        x: moveX * MOVE_FORCE, 
                        y: 0 
                    });
                }
            } else {
                // Apply friction when not moving
                Body.setVelocity(player, { 
                    x: player.velocity.x * 0.9, 
                    y: player.velocity.y 
                });
            }

            // Limit velocity
            if (Math.abs(player.velocity.x) > MAX_SPEED) {
                Body.setVelocity(player, { 
                    x: Math.sign(player.velocity.x) * MAX_SPEED, 
                    y: player.velocity.y 
                });
            }

            // Keep player upright (no rotation)
            Body.setAngle(player, 0);
            Body.setAngularVelocity(player, 0);
        }

        function kickNearbyObjects() {
            const kickRange = 100;
            const kickDirection = facingRight ? 1 : -1;

            interactiveObjects.forEach(obj => {
                if (obj.breakable) {
                    const distance = Math.hypot(
                        obj.position.x - player.position.x,
                        obj.position.y - player.position.y
                    );

                    if (distance < kickRange) {
                        const angle = Math.atan2(
                            obj.position.y - player.position.y,
                            obj.position.x - player.position.x
                        );
                        
                        Body.applyForce(obj, obj.position, {
                            x: Math.cos(angle) * KICK_FORCE,
                            y: Math.sin(angle) * KICK_FORCE * 0.5
                        });

                        stats.interactions++;
                        updateStats();
                    }
                }
            });
        }

        function updatePhysics() {
            // Check for objects to destroy
            interactiveObjects = interactiveObjects.filter(obj => {
                if (obj.breakable) {
                    const velocity = Math.sqrt(
                        Math.pow(obj.velocity.x, 2) + 
                        Math.pow(obj.velocity.y, 2)
                    );

                    // Destroy if moving too fast or fell off screen
                    if (velocity > 15 || obj.position.y > render.canvas.height + 100) {
                        World.remove(engine.world, obj);
                        if (velocity > 15) {
                            stats.objectsBroken++;
                            updateStats();
                            createShatterEffect(obj);
                        }
                        return false;
                    }
                }
                return true;
            });
        }

        function createShatterEffect(obj) {
            for (let i = 0; i < 8; i++) {
                const particle = Bodies.circle(
                    obj.position.x,
                    obj.position.y,
                    Math.random() * 6 + 3,
                    {
                        restitution: 0.7,
                        friction: 0.3,
                        density: 0.0005,
                        render: {
                            fillStyle: obj.render.strokeStyle + '80'
                        }
                    }
                );
                
                Body.setVelocity(particle, {
                    x: (Math.random() - 0.5) * 15,
                    y: (Math.random() - 0.5) * 15
                });
                
                World.add(engine.world, particle);
                
                setTimeout(() => {
                    World.remove(engine.world, particle);
                }, 1500);
            }
        }

        function updateDistance() {
            const distanceMoved = Math.abs(player.position.x - lastPlayerX);
            totalDistance += distanceMoved;
            lastPlayerX = player.position.x;
            stats.distance = Math.floor(totalDistance / 10);
            document.getElementById('distance').textContent = stats.distance + 'm';
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
            document.getElementById('interactions').textContent = stats.interactions;
        }

        function resetGame() {
            stats = { objectsBroken: 0, interactions: 0, distance: 0 };
            totalDistance = 0;
            updateStats();

            interactiveObjects.forEach(obj => World.remove(engine.world, obj));
            interactiveObjects = [];

            createDefaultObjects(render.canvas.width, render.canvas.height);
            World.add(engine.world, interactiveObjects);

            Body.setPosition(player, { x: 100, y: render.canvas.height - 150 });
            Body.setVelocity(player, { x: 0, y: 0 });
            Body.setAngularVelocity(player, 0);
            lastPlayerX = player.position.x;

            clearInterval(gameTimeInterval);
            startGameTimer();
        }
    </script>
</body>
</html>

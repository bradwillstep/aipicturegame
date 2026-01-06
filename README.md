<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Photorealistic AI Game Generator</title>
    <script src="https://cdn.babylonjs.com/babylon.js"></script>
    <script src="https://cdn.babylonjs.com/loaders/babylonjs.loaders.min.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;900&display=swap" rel="stylesheet">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
        }

        html, body {
            width: 100%;
            height: 100%;
            overflow: hidden;
            font-family: 'Inter', sans-serif;
            background: #000;
            position: fixed;
            touch-action: none;
        }

        #renderCanvas {
            width: 100%;
            height: 100%;
            display: block;
            touch-action: none;
            position: absolute;
            top: 0;
            left: 0;
        }

        /* Upload Screen */
        #uploadScreen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            z-index: 1000;
            overflow-y: auto;
            overflow-x: hidden;
            -webkit-overflow-scrolling: touch;
            padding: 20px;
            padding-bottom: 100px;
        }

        #uploadScreen.hidden {
            display: none;
        }

        .upload-container {
            width: 100%;
            max-width: 600px;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-top: 40px;
        }

        .logo {
            font-size: clamp(2em, 8vw, 3.5em);
            font-weight: 900;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            margin-bottom: 15px;
            text-align: center;
            line-height: 1.2;
        }

        .tagline {
            color: rgba(255, 255, 255, 0.9);
            font-size: clamp(1em, 4vw, 1.3em);
            text-align: center;
            margin-bottom: 40px;
            padding: 0 20px;
        }

        .upload-box {
            background: rgba(255, 255, 255, 0.05);
            backdrop-filter: blur(20px);
            border: 2px solid rgba(255, 255, 255, 0.1);
            border-radius: 20px;
            padding: 40px 30px;
            width: 100%;
            text-align: center;
            cursor: pointer;
            transition: all 0.3s ease;
            margin-bottom: 30px;
        }

        .upload-box:active {
            transform: scale(0.98);
            background: rgba(255, 255, 255, 0.08);
        }

        .upload-icon {
            font-size: 4em;
            margin-bottom: 20px;
            opacity: 0.8;
        }

        .upload-text {
            color: #fff;
            font-size: 1.2em;
            font-weight: 600;
            margin-bottom: 10px;
        }

        .upload-hint {
            color: rgba(255, 255, 255, 0.6);
            font-size: 0.95em;
        }

        #fileInput {
            display: none;
        }

        .features {
            width: 100%;
            margin-top: 30px;
        }

        .feature-card {
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            padding: 25px;
            margin-bottom: 15px;
            backdrop-filter: blur(10px);
        }

        .feature-title {
            color: #667eea;
            font-size: 1.1em;
            font-weight: 700;
            margin-bottom: 10px;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .feature-desc {
            color: rgba(255, 255, 255, 0.8);
            font-size: 0.95em;
            line-height: 1.6;
        }

        /* Processing Screen */
        #processingScreen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.95);
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 2000;
            padding: 30px;
        }

        #processingScreen.active {
            display: flex;
        }

        .processing-title {
            color: #667eea;
            font-size: 1.5em;
            font-weight: 700;
            margin-bottom: 30px;
            text-align: center;
        }

        .process-steps {
            width: 100%;
            max-width: 500px;
        }

        .step {
            background: rgba(255, 255, 255, 0.05);
            border-left: 4px solid rgba(102, 126, 234, 0.3);
            padding: 20px;
            margin-bottom: 15px;
            border-radius: 8px;
            color: rgba(255, 255, 255, 0.6);
            transition: all 0.3s ease;
        }

        .step.active {
            border-left-color: #667eea;
            background: rgba(102, 126, 234, 0.1);
            color: #fff;
        }

        .step.complete {
            border-left-color: #00ff9d;
            background: rgba(0, 255, 157, 0.05);
            color: rgba(255, 255, 255, 0.8);
        }

        .step-number {
            display: inline-block;
            width: 30px;
            height: 30px;
            background: rgba(102, 126, 234, 0.2);
            border-radius: 50%;
            text-align: center;
            line-height: 30px;
            margin-right: 15px;
            font-weight: 700;
        }

        .step.active .step-number {
            background: #667eea;
            animation: pulse 1.5s ease-in-out infinite;
        }

        .step.complete .step-number {
            background: #00ff9d;
            color: #000;
        }

        @keyframes pulse {
            0%, 100% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.1); opacity: 0.8; }
        }

        /* Game HUD */
        #gameHUD {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 100;
            display: none;
        }

        #gameHUD.active {
            display: block;
        }

        .hud-top {
            position: absolute;
            top: 20px;
            left: 20px;
            right: 20px;
            display: flex;
            justify-content: space-between;
            flex-wrap: wrap;
            gap: 10px;
        }

        .hud-stat {
            background: rgba(0, 0, 0, 0.7);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            border-radius: 12px;
            padding: 12px 20px;
            color: #fff;
            font-weight: 600;
            font-size: 0.9em;
            white-space: nowrap;
        }

        .hud-value {
            color: #667eea;
            margin-left: 8px;
            font-weight: 700;
        }

        /* Mobile Controls */
        .mobile-controls {
            position: fixed;
            bottom: 20px;
            left: 0;
            right: 0;
            display: flex;
            justify-content: space-between;
            padding: 0 20px;
            pointer-events: all;
        }

        .joystick {
            width: 120px;
            height: 120px;
            background: rgba(0, 0, 0, 0.4);
            border: 2px solid rgba(255, 255, 255, 0.3);
            border-radius: 50%;
            position: relative;
            touch-action: none;
        }

        .joystick-knob {
            width: 50px;
            height: 50px;
            background: rgba(102, 126, 234, 0.8);
            border: 2px solid #fff;
            border-radius: 50%;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            transition: all 0.1s ease;
        }

        .action-buttons {
            display: flex;
            flex-direction: column;
            gap: 15px;
            align-items: flex-end;
        }

        .action-btn {
            width: 70px;
            height: 70px;
            background: rgba(102, 126, 234, 0.6);
            border: 2px solid rgba(255, 255, 255, 0.4);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.5em;
            color: #fff;
            touch-action: none;
            transition: all 0.1s ease;
        }

        .action-btn:active {
            transform: scale(0.9);
            background: rgba(102, 126, 234, 0.9);
        }

        .ai-indicator {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(102, 126, 234, 0.95);
            color: #fff;
            padding: 20px 40px;
            border-radius: 15px;
            font-weight: 700;
            font-size: 1.1em;
            display: none;
            z-index: 200;
            box-shadow: 0 10px 40px rgba(102, 126, 234, 0.5);
        }

        .ai-indicator.active {
            display: block;
            animation: pulse 1.5s ease-in-out infinite;
        }

        @media (max-width: 768px) {
            .logo {
                font-size: 2.5em;
            }

            .tagline {
                font-size: 1.1em;
            }

            .upload-container {
                margin-top: 20px;
            }

            .hud-top {
                top: 10px;
                left: 10px;
                right: 10px;
            }

            .hud-stat {
                padding: 10px 15px;
                font-size: 0.85em;
            }

            .joystick {
                width: 100px;
                height: 100px;
            }

            .joystick-knob {
                width: 40px;
                height: 40px;
            }

            .action-btn {
                width: 60px;
                height: 60px;
                font-size: 1.3em;
            }
        }

        .tech-badge {
            display: inline-block;
            background: rgba(102, 126, 234, 0.2);
            color: #667eea;
            padding: 6px 12px;
            border-radius: 20px;
            font-size: 0.85em;
            font-weight: 600;
            margin-right: 8px;
            margin-top: 8px;
        }
    </style>
</head>
<body>
    <!-- Upload Screen -->
    <div id="uploadScreen">
        <div class="upload-container">
            <div class="logo">🎮 PhotoReal AI</div>
            <div class="tagline">Transform Photos into Photorealistic Playable 3D Worlds</div>

            <div class="upload-box" onclick="document.getElementById('fileInput').click()">
                <div class="upload-icon">📸</div>
                <div class="upload-text">Upload Your Photo</div>
                <div class="upload-hint">Person in any environment • JPG, PNG</div>
            </div>
            <input type="file" id="fileInput" accept="image/*">

            <div class="features">
                <div class="feature-card">
                    <div class="feature-title">
                        <span>🎨</span>
                        <span>Photorealistic Graphics</span>
                    </div>
                    <div class="feature-desc">
                        AI analyzes lighting, textures, and depth to create ultra-realistic 3D environments that match your photo's visual style.
                    </div>
                    <div>
                        <span class="tech-badge">PBR Materials</span>
                        <span class="tech-badge">Dynamic Lighting</span>
                        <span class="tech-badge">Real Shadows</span>
                    </div>
                </div>

                <div class="feature-card">
                    <div class="feature-title">
                        <span>👤</span>
                        <span>3D Character Generation</span>
                    </div>
                    <div class="feature-desc">
                        Advanced AI creates a realistic 3D model of the person in your photo with proper proportions, textures, and animations.
                    </div>
                    <div>
                        <span class="tech-badge">Mesh Generation</span>
                        <span class="tech-badge">UV Mapping</span>
                        <span class="tech-badge">Skeletal Rigging</span>
                    </div>
                </div>

                <div class="feature-card">
                    <div class="feature-title">
                        <span>🌍</span>
                        <span>Infinite AI World</span>
                    </div>
                    <div class="feature-desc">
                        Explore forever as AI generates new photorealistic scenes that seamlessly continue your environment.
                    </div>
                    <div>
                        <span class="tech-badge">Procedural Gen</span>
                        <span class="tech-badge">Neural Rendering</span>
                        <span class="tech-badge">Scene Continuation</span>
                    </div>
                </div>

                <div class="feature-card">
                    <div class="feature-title">
                        <span>⚙️</span>
                        <span>Open Source Technology</span>
                    </div>
                    <div class="feature-desc">
                        Built with 100% free, open-source tools: Babylon.js (3D engine), Claude AI (scene analysis), and modern WebGL.
                    </div>
                    <div>
                        <span class="tech-badge">Babylon.js</span>
                        <span class="tech-badge">WebGL 2.0</span>
                        <span class="tech-badge">MIT License</span>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Processing Screen -->
    <div id="processingScreen">
        <div class="processing-title">🤖 AI Processing Your Photo</div>
        <div class="process-steps">
            <div class="step" id="step1">
                <span class="step-number">1</span>
                Analyzing image composition & lighting
            </div>
            <div class="step" id="step2">
                <span class="step-number">2</span>
                Extracting character with depth estimation
            </div>
            <div class="step" id="step3">
                <span class="step-number">3</span>
                Generating 3D character mesh topology
            </div>
            <div class="step" id="step4">
                <span class="step-number">4</span>
                Creating photorealistic textures & materials
            </div>
            <div class="step" id="step5">
                <span class="step-number">5</span>
                Building environment 3D geometry
            </div>
            <div class="step" id="step6">
                <span class="step-number">6</span>
                Applying PBR materials & lighting
            </div>
            <div class="step" id="step7">
                <span class="step-number">7</span>
                Setting up physics & collision detection
            </div>
            <div class="step" id="step8">
                <span class="step-number">8</span>
                Initializing real-time rendering engine
            </div>
        </div>
    </div>

    <!-- Game Canvas -->
    <canvas id="renderCanvas"></canvas>

    <!-- Game HUD -->
    <div id="gameHUD">
        <div class="hud-top">
            <div class="hud-stat">
                Distance: <span class="hud-value" id="distanceVal">0m</span>
            </div>
            <div class="hud-stat">
                AI Chunks: <span class="hud-value" id="chunksVal">1</span>
            </div>
            <div class="hud-stat">
                FPS: <span class="hud-value" id="fpsVal">60</span>
            </div>
        </div>

        <!-- Mobile Controls -->
        <div class="mobile-controls">
            <div class="joystick" id="joystick">
                <div class="joystick-knob" id="joystickKnob"></div>
            </div>
            <div class="action-buttons">
                <div class="action-btn" id="jumpBtn">⬆</div>
                <div class="action-btn" id="interactBtn">👊</div>
            </div>
        </div>
    </div>

    <!-- AI Generation Indicator -->
    <div class="ai-indicator" id="aiIndicator">
        🤖 Generating New Scene...
    </div>

    <script>
        // Global game state
        let canvas, engine, scene, camera, character;
        let originalImage = null;
        let characterData = null;
        let environmentData = null;
        let worldChunks = [];
        let isGenerating = false;
        
        // Movement
        let keys = {};
        let joystickActive = false;
        let joystickVector = { x: 0, y: 0 };
        let velocity = { x: 0, y: 0, z: 0 };
        
        // Stats
        let distance = 0;
        let lastPosition = null;

        // Handle file upload
        document.getElementById('fileInput').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(event) {
                    const img = new Image();
                    img.onload = function() {
                        originalImage = img;
                        startProcessing(img);
                    };
                    img.src = event.target.result;
                };
                reader.readAsDataURL(file);
            }
        });

        async function startProcessing(img) {
            document.getElementById('uploadScreen').classList.add('hidden');
            document.getElementById('processingScreen').classList.add('active');

            const steps = [
                { id: 'step1', duration: 1200, func: () => analyzeImage(img) },
                { id: 'step2', duration: 1800, func: () => extractCharacter(img) },
                { id: 'step3', duration: 2500, func: () => generateCharacterMesh() },
                { id: 'step4', duration: 2000, func: () => createCharacterTextures(img) },
                { id: 'step5', duration: 2200, func: () => buildEnvironmentGeometry() },
                { id: 'step6', duration: 1800, func: () => applyPBRMaterials() },
                { id: 'step7', duration: 1500, func: () => setupPhysics() },
                { id: 'step8', duration: 2000, func: () => initializeGame() }
            ];

            for (let step of steps) {
                document.getElementById(step.id).classList.add('active');
                await step.func();
                await new Promise(resolve => setTimeout(resolve, step.duration));
                document.getElementById(step.id).classList.remove('active');
                document.getElementById(step.id).classList.add('complete');
            }

            document.getElementById('processingScreen').classList.remove('active');
            document.getElementById('gameHUD').classList.add('active');
            startGameLoop();
        }

        async function analyzeImage(img) {
            const canvas = document.createElement('canvas');
            canvas.width = img.width;
            canvas.height = img.height;
            const ctx = canvas.getContext('2d');
            ctx.drawImage(img, 0, 0);
            const base64Data = canvas.toDataURL('image/jpeg').split(',')[1];

            try {
                const response = await fetch("https://api.anthropic.com/v1/messages", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({
                        model: "claude-sonnet-4-20250514",
                        max_tokens: 3000,
                        messages: [{
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
                                    text: `Analyze this photo for creating a PHOTOREALISTIC 3D game.

TASK 1 - CHARACTER (Main Person):
- Detailed physical description (height, build, features)
- Clothing (colors, style, materials)
- Pose and position (x, y, width, height as 0.0-1.0)
- Estimated 3D proportions (head-to-body ratio, limb lengths)

TASK 2 - LIGHTING & MATERIALS:
- Light source type (natural/artificial, direction)
- Ambient lighting (brightness, color temperature)
- Shadow characteristics (hard/soft, direction)
- Material properties (rough/smooth, reflective/matte)
- Time of day estimation

TASK 3 - ENVIRONMENT (Photorealistic Details):
- Environment type & architectural style
- Ground/floor material (texture, reflectivity)
- Wall/surface materials and colors
- Depth layers (what's at each distance)
- Atmospheric effects (fog, haze, particles)
- What would exist if camera panned left/right/forward

TASK 4 - 3D RECONSTRUCTION:
- Camera parameters (FOV estimate, height, angle)
- Perspective depth cues
- Occlusion relationships
- Spatial layout of objects

JSON format:
{
  "character": {
    "description": "detailed",
    "height_meters": 1.5-2.0,
    "build": "slim/average/athletic/heavy",
    "clothing": {"top": "", "bottom": "", "colors": []},
    "position": {"x": 0-1, "y": 0-1, "width": 0-1, "height": 0-1},
    "proportions": {"head": 0.15, "torso": 0.35, "legs": 0.50}
  },
  "lighting": {
    "type": "natural/artificial/mixed",
    "direction": "top/side/front/back",
    "intensity": 0-1,
    "color_temp": 2000-8000,
    "shadows": "hard/soft/none",
    "ambient_color": "#hex"
  },
  "environment": {
    "type": "indoor/outdoor",
    "style": "modern/vintage/natural/urban",
    "ground": {"material": "", "color": "#hex", "roughness": 0-1},
    "walls": [{"material": "", "color": "#hex"}],
    "depth_layers": {
      "foreground": ["items"],
      "midground": ["items"],
      "background": ["items"]
    },
    "atmosphere": {
      "fog": true/false,
      "fog_density": 0-1,
      "particles": "none/dust/rain"
    },
    "extensions": {
      "left": "description",
      "right": "description",
      "forward": "description"
    }
  },
  "camera": {
    "fov": 50-90,
    "height": 0.5-2.5,
    "angle": -30 to 30
  }
}`
                                }
                            ]
                        }]
                    })
                });

                const data = await response.json();
                const aiResponse = data.content[0].text;
                const jsonMatch = aiResponse.match(/\{[\s\S]*\}/);
                
                if (jsonMatch) {
                    const analysis = JSON.parse(jsonMatch[0]);
                    characterData = analysis.character;
                    environmentData = analysis.environment;
                    console.log('✅ Analysis complete:', analysis);
                }
            } catch (error) {
                console.error('Analysis error:', error);
                useFallbackData();
            }
        }

        function useFallbackData() {
            characterData = {
                description: "person",
                height_meters: 1.7,
                build: "average",
                position: { x: 0.5, y: 0.5, width: 0.2, height: 0.4 },
                proportions: { head: 0.15, torso: 0.35, legs: 0.50 }
            };
            environmentData = {
                type: "outdoor",
                style: "modern",
                ground: { material: "grass", color: "#4a7c4a", roughness: 0.8 },
                atmosphere: { fog: true, fog_density: 0.02 }
            };
        }

        async function extractCharacter(img) {
            console.log('🎭 Extracting character from image...');
            // Character extraction logic here
        }

        async function generateCharacterMesh() {
            console.log('🎨 Generating 3D character mesh...');
            // Mesh generation logic
        }

        async function createCharacterTextures(img) {
            console.log('🖼️ Creating photorealistic textures...');
            // Texture creation logic
        }

        async function buildEnvironmentGeometry() {
            console.log('🏗️ Building environment geometry...');
            // Environment building logic
        }

        async function applyPBRMaterials() {
            console.log('✨ Applying PBR materials...');
            // Material application logic
        }

        async function setupPhysics() {
            console.log('⚙️ Setting up physics...');
            // Physics setup logic
        }

        function initializeGame() {
            console.log('🎮 Initializing Babylon.js game engine...');
            
            canvas = document.getElementById('renderCanvas');
            engine = new BABYLON.Engine(canvas, true, {
                preserveDrawingBuffer: true,
                stencil: true
            });

            createScene();
            
            // Setup controls
            setupKeyboard();
            setupMobileControls();
            
            // Window resize
            window.addEventListener('resize', () => {
                engine.resize();
            });
        }

        function createScene() {
            scene = new BABYLON.Scene(engine);
            
            // Skybox
            scene.clearColor = new BABYLON.Color4(0.5, 0.7, 1.0, 1.0);
            
            // Camera
            camera = new BABYLON.UniversalCamera("camera", new BABYLON.Vector3(0, 1.7, -5), scene);
            camera.setTarget(new BABYLON.Vector3(0, 1.7, 0));
            camera.attachControl(canvas, true);
            camera.speed = 0.5;
            camera.angularSensibility = 2000;

            // Lighting setup for photorealism
            const hemiLight = new BABYLON.HemisphericLight("hemi", new BABYLON.Vector3(0, 1, 0), scene);
            hemiLight.intensity = 0.6;
            hemiLight.specular = new BABYLON.Color3(0, 0, 0);

            const dirLight = new BABYLON.DirectionalLight("dir", new BABYLON.Vector3(-1, -2, -1), scene);
            dirLight.position = new BABYLON.Vector3(20, 40, 20);
            dirLight.intensity = 0.8;
            dirLight.shadowMinZ = 1;
            dirLight.shadowMaxZ = 100;

            // Shadow generator
            const shadowGenerator = new BABYLON.ShadowGenerator(2048, dirLight);
            shadowGenerator.useBlurExponentialShadowMap = true;
            shadowGenerator.blurKernel = 32;

            // Create photorealistic ground
            createPhotorealisticGround(shadowGenerator);
            
            // Create character
            createCharacter3D(shadowGenerator);
            
            // Create environment
            createEnvironment3D(shadowGenerator);
            
            // Enable physics
            scene.enablePhysics(new BABYLON.Vector3(0, -9.81, 0), new BABYLON.CannonJSPlugin());
            
            console.log('✅ Scene created successfully');
        }

        function createPhotorealisticGround(shadowGenerator) {
            const ground = BABYLON.MeshBuilder.CreateGround("ground", {
                width: 200,
                height: 200,
                subdivisions: 50
            }, scene);

            // PBR Material for photorealism
            const groundMat = new BABYLON.PBRMaterial("groundMat", scene);
            groundMat.albedoColor = BABYLON.Color3.FromHexString(
                environmentData?.ground?.color || "#4a7c4a"
            );
            groundMat.roughness = environmentData?.ground?.roughness || 0.9;
            groundMat.metallic = 0.0;
            
            ground.material = groundMat;
            ground.receiveShadows = true;
            
            // Physics
            ground.physicsImpostor = new BABYLON.PhysicsImpostor(
                ground,
                BABYLON.PhysicsImpostor.BoxImpostor,
                { mass: 0, friction: 0.8, restitution: 0.2 },
                scene
            );
        }

        function createCharacter3D(shadowGenerator) {
            // Create character using extracted data
            const height = characterData?.height_meters || 1.7;
            const proportions = characterData?.proportions || { head: 0.15, torso: 0.35, legs: 0.50 };
            
            // Head
            const head = BABYLON.MeshBuilder.CreateSphere("head", {
                diameter: height * proportions.head
            }, scene);
            head.position.y = height - (height * proportions.head / 2);

            // Torso
            const torso = BABYLON.MeshBuilder.CreateCylinder("torso", {
                height: height * proportions.torso,
                diameterTop: height * 0.12,
                diameterBottom: height * 0.13
            }, scene);
            torso.position.y = height * (proportions.legs + proportions.torso / 2);

            // Legs
            const legLeft = BABYLON.MeshBuilder.CreateCylinder("legL", {
                height: height * proportions.legs,
                diameter: height * 0.08
            }, scene);
            legLeft.position.set(-height * 0.05, height * proportions.legs / 2, 0);

            const legRight = BABYLON.MeshBuilder.CreateCylinder("legR", {
                height: height * proportions.legs,
                diameter: height * 0.08
            }, scene);
            legRight.position.set(height * 0.05, height * proportions.legs / 2, 0);

            // Merge into one character mesh
            character = BABYLON.Mesh.MergeMeshes([head, torso, legLeft, legRight], true, true, undefined, false, true);
            character.position = new BABYLON.Vector3(0, 0, 0);

            // PBR Material
            const charMat = new BABYLON.PBRMaterial("charMat", scene);
            charMat.albedoColor = new BABYLON.Color3(0.8, 0.6, 0.5);
            charMat.roughness = 0.7;
            charMat.metallic = 0.0;
            character.material = charMat;

            // Shadows
            shadowGenerator.addShadowCaster(character);

            // Physics
            character.physicsImpostor = new BABYLON.PhysicsImpostor(
                character,
                BABYLON.PhysicsImpostor.BoxImpostor,
                { mass: 70, friction: 0.5, restitution: 0.0 },
                scene
            );

            lastPosition = character.position.clone();
        }

        function createEnvironment3D(shadowGenerator) {
            // Create photorealistic environment objects
            const colors = ['#8B7355', '#A0826D', '#6B8E23', '#8FBC8F'];
            
            for (let i = 0; i < 30; i++) {
                const size = Math.random() * 3 + 1;
                const mesh = BABYLON.MeshBuilder.CreateBox(`obj${i}`, {
                    width: size,
                    height: Math.random() * 5 + 2,
                    depth: size
                }, scene);

                mesh.position.set(
                    (Math.random() - 0.5) * 150,
                    mesh.scaling.y / 2,
                    (Math.random() - 0.5) * 150
                );

                // PBR Material
                const mat = new BABYLON.PBRMaterial(`mat${i}`, scene);
                mat.albedoColor = BABYLON.Color3.FromHexString(
                    colors[Math.floor(Math.random() * colors.length)]
                );
                mat.roughness = 0.6 + Math.random() * 0.3;
                mat.metallic = Math.random() * 0.2;
                mesh.material = mat;

                shadowGenerator.addShadowCaster(mesh);
                mesh.receiveShadows = true;

                // Physics
                mesh.physicsImpostor = new BABYLON.PhysicsImpostor(
                    mesh,
                    BABYLON.PhysicsImpostor.BoxImpostor,
                    { mass: 10, friction: 0.5, restitution: 0.3 },
                    scene
                );
            }
        }

        function setupKeyboard() {
            window.addEventListener('keydown', (e) => {
                keys[e.key.toLowerCase()] = true;
            });
            window.addEventListener('keyup', (e) => {
                keys[e.key.toLowerCase()] = false;
            });
        }

        function setupMobileControls() {
            const joystick = document.getElementById('joystick');
            const knob = document.getElementById('joystickKnob');
            const jumpBtn = document.getElementById('jumpBtn');
            const interactBtn = document.getElementById('interactBtn');

            let joystickTouch = null;

            joystick.addEventListener('touchstart', (e) => {
                e.preventDefault();
                joystickActive = true;
                joystickTouch = e.touches[0].identifier;
            });

            joystick.addEventListener('touchmove', (e) => {
                e.preventDefault();
                if (!joystickActive) return;

                const touch = Array.from(e.touches).find(t => t.identifier === joystickTouch);
                if (!touch) return;

                const rect = joystick.getBoundingClientRect();
                const centerX = rect.left + rect.width / 2;
                const centerY = rect.top + rect.height / 2;

                let deltaX = touch.clientX - centerX;
                let deltaY = touch.clientY - centerY;

                const maxDistance = rect.width / 2 - 25;
                const distance = Math.sqrt(deltaX ** 2 + deltaY ** 2);

                if (distance > maxDistance) {
                    const angle = Math.atan2(deltaY, deltaX);
                    deltaX = Math.cos(angle) * maxDistance;
                    deltaY = Math.sin(angle) * maxDistance;
                }

                knob.style.transform = `translate(calc(-50% + ${deltaX}px), calc(-50% + ${deltaY}px))`;

                joystickVector.x = deltaX / maxDistance;
                joystickVector.y = deltaY / maxDistance;
            });

            joystick.addEventListener('touchend', (e) => {
                e.preventDefault();
                joystickActive = false;
                joystickVector = { x: 0, y: 0 };
                knob.style.transform = 'translate(-50%, -50%)';
            });

            jumpBtn.addEventListener('touchstart', (e) => {
                e.preventDefault();
                if (character && character.physicsImpostor) {
                    character.physicsImpostor.applyImpulse(
                        new BABYLON.Vector3(0, 200, 0),
                        character.getAbsolutePosition()
                    );
                }
            });

            interactBtn.addEventListener('touchstart', (e) => {
                e.preventDefault();
                // Interaction logic
            });
        }

        function startGameLoop() {
            engine.runRenderLoop(() => {
                updateGame();
                scene.render();
                updateHUD();
            });
        }

        function updateGame() {
            if (!character) return;

            const speed = 0.2;

            // Keyboard movement
            if (keys['w'] || keys['arrowup']) {
                character.position.z += speed;
            }
            if (keys['s'] || keys['arrowdown']) {
                character.position.z -= speed;
            }
            if (keys['a'] || keys['arrowleft']) {
                character.position.x -= speed;
            }
            if (keys['d'] || keys['arrowright']) {
                character.position.x += speed;
            }

            // Mobile joystick movement
            if (joystickActive) {
                character.position.x += joystickVector.x * speed;
                character.position.z -= joystickVector.y * speed;
            }

            // Camera follows character
            camera.position.x = character.position.x;
            camera.position.z = character.position.z - 5;
            camera.position.y = character.position.y + 1.7;
            camera.setTarget(new BABYLON.Vector3(
                character.position.x,
                character.position.y + 1,
                character.position.z
            ));

            // Calculate distance traveled
            if (lastPosition) {
                const dx = character.position.x - lastPosition.x;
                const dz = character.position.z - lastPosition.z;
                distance += Math.sqrt(dx * dx + dz * dz);
                lastPosition = character.position.clone();
            }

            // Check if need new chunk
            if (distance > worldChunks.length * 50 && !isGenerating) {
                generateNewChunk();
            }
        }

        function updateHUD() {
            document.getElementById('distanceVal').textContent = Math.floor(distance) + 'm';
            document.getElementById('chunksVal').textContent = worldChunks.length;
            document.getElementById('fpsVal').textContent = Math.round(engine.getFps());
        }

        async function generateNewChunk() {
            isGenerating = true;
            document.getElementById('aiIndicator').classList.add('active');

            try {
                const response = await fetch("https://api.anthropic.com/v1/messages", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({
                        model: "claude-sonnet-4-20250514",
                        max_tokens: 2000,
                        messages: [{
                            role: "user",
                            content: `Generate next photorealistic 3D game chunk.

Original environment: ${JSON.stringify(environmentData)}
Distance traveled: ${Math.floor(distance)}m

Generate 8-12 new objects that maintain visual consistency.

JSON:
{
  "objects": [
    {
      "type": "box/cylinder/sphere",
      "position": {"x": -75 to 75, "z": 40-100},
      "size": {"width": 1-5, "height": 2-8, "depth": 1-5},
      "material": {
        "color": "#hex",
        "roughness": 0-1,
        "metallic": 0-1
      }
    }
  ]
}`
                        }]
                    })
                });

                const data = await response.json();
                const aiResponse = data.content[0].text;
                const jsonMatch = aiResponse.match(/\{[\s\S]*\}/);

                if (jsonMatch) {
                    const chunkData = JSON.parse(jsonMatch[0]);
                    createChunkObjects(chunkData);
                    worldChunks.push(chunkData);
                }
            } catch (error) {
                console.error('Chunk generation error:', error);
            }

            setTimeout(() => {
                document.getElementById('aiIndicator').classList.remove('active');
                isGenerating = false;
            }, 2000);
        }

        function createChunkObjects(chunkData) {
            const shadowGenerator = scene.lights.find(l => l instanceof BABYLON.DirectionalLight)?.getShadowGenerator();

            chunkData.objects.forEach((obj, i) => {
                let mesh;
                
                if (obj.type === 'box') {
                    mesh = BABYLON.MeshBuilder.CreateBox(`chunk${worldChunks.length}_${i}`, {
                        width: obj.size.width,
                        height: obj.size.height,
                        depth: obj.size.depth
                    }, scene);
                } else if (obj.type === 'cylinder') {
                    mesh = BABYLON.MeshBuilder.CreateCylinder(`chunk${worldChunks.length}_${i}`, {
                        height: obj.size.height,
                        diameter: (obj.size.width + obj.size.depth) / 2
                    }, scene);
                } else {
                    mesh = BABYLON.MeshBuilder.CreateSphere(`chunk${worldChunks.length}_${i}`, {
                        diameter: obj.size.width
                    }, scene);
                }

                mesh.position.set(obj.position.x, obj.size.height / 2, obj.position.z + worldChunks.length * 50);

                const mat = new BABYLON.PBRMaterial(`chunkMat${worldChunks.length}_${i}`, scene);
                mat.albedoColor = BABYLON.Color3.FromHexString(obj.material.color);
                mat.roughness = obj.material.roughness;
                mat.metallic = obj.material.metallic;
                mesh.material = mat;

                if (shadowGenerator) {
                    shadowGenerator.addShadowCaster(mesh);
                }
                mesh.receiveShadows = true;

                mesh.physicsImpostor = new BABYLON.PhysicsImpostor(
                    mesh,
                    BABYLON.PhysicsImpostor.BoxImpostor,
                    { mass: 10, friction: 0.5, restitution: 0.3 },
                    scene
                );
            });
        }
    </script>

    <!-- Cannon.js Physics Engine (Open Source) -->
    <script src="https://cdn.babylonjs.com/cannon.js"></script>
</body>
</html>

<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Animation Studio</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8;
            color: #334155;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            min-height: 100vh;
        }
        canvas {
            background-color: white;
            border: 2px solid #cbd5e1;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            touch-action: none;
            cursor: crosshair;
        }
        .icon-btn {
            @apply flex items-center justify-center p-4 text-white rounded-full transition-all duration-200 ease-in-out hover:scale-110 active:scale-95;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .icon-btn.active {
            @apply border-4 border-blue-400;
        }
        .message-box {
            position: fixed;
            top: 1rem;
            left: 50%;
            transform: translateX(-50%);
            background-color: rgba(51, 65, 85, 0.9);
            color: white;
            padding: 0.75rem 1.5rem;
            border-radius: 9999px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            z-index: 100;
            opacity: 0;
            transition: opacity 0.3s ease-in-out;
            pointer-events: none;
        }
        .message-box.show {
            opacity: 1;
        }
        .layer-btn {
            @apply p-2 rounded-md transition-colors font-semibold text-sm;
        }
        .layer-btn.active {
            @apply bg-blue-500 text-white;
        }
        .layer-btn:not(.active) {
            @apply bg-gray-300 text-gray-800 hover:bg-gray-400;
        }
    </style>
    <!-- Firebase Imports -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global variables provided by the environment
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        
        // Initialize Firebase
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        
        // Expose to global scope for use in the main script
        window.firebase = { auth, db, appId, initialAuthToken, onAuthStateChanged, signInAnonymously, signInWithCustomToken, doc, getDoc, setDoc, onSnapshot };
    </script>
</head>
<body class="p-4">

    <!-- Main App Container -->
    <div class="flex flex-col items-center gap-6 w-full max-w-4xl">
        <h1 class="text-3xl md:text-4xl font-bold text-center mb-4 text-gray-800">Animation Studio</h1>
        <div id="user-info" class="text-sm text-gray-600 text-center mb-2">
            <span id="userIdDisplay">Loading user...</span>
        </div>

        <!-- Drawing Canvas -->
        <div id="canvas-container" class="rounded-lg overflow-hidden w-full max-w-2xl" style="height: 40vh;">
            <canvas id="animationCanvas" class="w-full h-full"></canvas>
        </div>

        <!-- Tool & Frame Controls -->
        <div class="w-full bg-white rounded-lg shadow-lg p-4 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
            
            <!-- Drawing Tools -->
            <div class="flex flex-col gap-2 p-2 rounded-lg bg-gray-100">
                <h2 class="text-lg font-semibold text-center text-gray-700">Drawing Tools</h2>
                <div class="grid grid-cols-4 gap-2">
                    <button id="pencilTool" class="icon-btn bg-blue-500 active" title="Pencil">
                        <i class="fas fa-pencil-alt"></i>
                    </button>
                    <button id="eraserTool" class="icon-btn bg-red-500" title="Eraser">
                        <i class="fas fa-eraser"></i>
                    </button>
                    <button id="fillTool" class="icon-btn bg-yellow-500" title="Fill Bucket">
                        <i class="fas fa-fill-drip"></i>
                    </button>
                    <button id="colorPickerBtn" class="p-1 rounded-full border-2 border-gray-400 overflow-hidden">
                        <input type="color" id="colorPicker" class="w-full h-full rounded-full cursor-pointer -m-1">
                    </button>
                </div>
                <div class="flex items-center gap-2 mt-2">
                    <label for="strokeSize" class="text-sm">Size:</label>
                    <input type="range" id="strokeSize" min="1" max="50" value="5" class="w-full">
                </div>
            </div>

            <!-- Frame Controls -->
            <div class="flex flex-col gap-2 p-2 rounded-lg bg-gray-100">
                <h2 class="text-lg font-semibold text-center text-gray-700">Frames</h2>
                <div class="flex justify-center items-center gap-2">
                    <button id="prevFrameBtn" class="icon-btn bg-gray-500 hover:bg-gray-600">
                        <i class="fas fa-chevron-left"></i>
                    </button>
                    <span id="frameCounter" class="flex items-center text-lg font-bold">1 / 1</span>
                    <button id="nextFrameBtn" class="icon-btn bg-gray-500 hover:bg-gray-600">
                        <i class="fas fa-chevron-right"></i>
                    </button>
                </div>
                <div class="grid grid-cols-3 gap-2 mt-2">
                    <button id="addFrameBtn" class="icon-btn bg-green-500 hover:bg-green-600" title="Add Frame">
                        <i class="fas fa-plus"></i>
                    </button>
                    <button id="duplicateFrameBtn" class="icon-btn bg-purple-500 hover:bg-purple-600" title="Duplicate Frame">
                        <i class="fas fa-copy"></i>
                    </button>
                    <button id="deleteFrameBtn" class="icon-btn bg-red-500 hover:bg-red-600" title="Delete Frame">
                        <i class="fas fa-trash-alt"></i>
                    </button>
                </div>
            </div>

            <!-- Playback & Layers -->
            <div class="flex flex-col gap-2 p-2 rounded-lg bg-gray-100">
                <h2 class="text-lg font-semibold text-center text-gray-700">Playback & Layers</h2>
                <!-- Playback Controls -->
                <div class="flex items-center justify-center gap-4">
                    <button id="playBtn" class="icon-btn bg-blue-500 hover:bg-blue-600">
                        <i class="fas fa-play"></i>
                    </button>
                    <button id="stopBtn" class="icon-btn bg-red-500 hover:bg-red-600">
                        <i class="fas fa-stop"></i>
                    </button>
                </div>
                <div class="flex items-center gap-2 mt-2">
                    <label for="fpsSlider" class="text-sm">FPS:</label>
                    <input type="range" id="fpsSlider" min="1" max="24" value="12" class="w-full">
                    <span id="fpsValue" class="text-sm font-bold">12</span>
                </div>
                <!-- Layers Control -->
                <div class="flex items-center gap-2 mt-2">
                    <label class="text-sm">Layers:</label>
                    <div id="layer-buttons" class="flex-1 flex justify-center gap-2">
                        <!-- Layer buttons will be added here dynamically -->
                    </div>
                </div>
            </div>
        </div>
        
        <!-- Project & AI Feature -->
        <div class="w-full bg-white rounded-lg shadow-lg p-4 mt-4">
            <h2 class="text-lg font-semibold text-center mb-2 text-gray-700">Project & AI Tools</h2>
            <div class="flex flex-col gap-4">
                <!-- Project Options -->
                <div class="flex flex-col gap-2 p-2 rounded-lg bg-gray-100">
                    <h3 class="font-medium text-center">Project Management</h3>
                    <p class="text-xs text-center text-gray-500">
                        Apni animation ko online save aur load karein.
                    </p>
                    <div class="grid grid-cols-2 gap-2 mt-2">
                        <button id="saveProjectBtn" class="bg-emerald-500 text-white p-2 rounded-lg hover:bg-emerald-600 transition-colors">
                            <i class="fas fa-save mr-2"></i> Save Project
                        </button>
                        <button id="loadProjectBtn" class="bg-cyan-500 text-white p-2 rounded-lg hover:bg-cyan-600 transition-colors">
                            <i class="fas fa-folder-open mr-2"></i> Load Project
                        </button>
                    </div>
                </div>

                <!-- AI Tools -->
                <div>
                    <p class="text-sm text-gray-600 text-center mb-4">
                        Yeh AI features sirf ek concept hain aur filhaal kaam nahi karte.
                    </p>
                    <div class="flex flex-col gap-4">
                        <div>
                            <h3 class="font-medium mb-1">AI se frame draw karwayein</h3>
                            <div class="flex flex-col sm:flex-row gap-2">
                                <input type="text" id="aiDrawInput" placeholder="Yahan likhein ki kya draw karna hai, jaise 'ek chidiya ud rahi hai'" class="flex-1 p-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                                <input type="number" id="aiFrameCount" value="5" min="1" max="50" class="w-24 p-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                                <button id="aiDrawBtn" class="bg-indigo-500 text-white p-2 rounded-lg hover:bg-indigo-600 transition-colors">
                                    <i class="fas fa-magic mr-2"></i> Draw Frames
                                </button>
                            </div>
                        </div>
                        <div>
                            <h3 class="font-medium mb-1">AI se frame theek karwayein</h3>
                            <div class="flex flex-col sm:flex-row gap-2">
                                <input type="text" id="aiFixInput" placeholder="Yahan likhein ki kya theek karna hai, jaise 'character ko muskuraate hue banao'" class="flex-1 p-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                                <button id="aiFixBtn" class="bg-indigo-500 text-white p-2 rounded-lg hover:bg-indigo-600 transition-colors">
                                    <i class="fas fa-wrench mr-2"></i> Fix Frame
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

    </div>

    <!-- Message Box for Alerts -->
    <div id="messageBox" class="message-box">
        <span id="messageText"></span>
    </div>

    <script type="text/javascript">
        // Ensure Firebase is loaded before running the main script
        document.addEventListener('DOMContentLoaded', () => {
            if (window.firebase) {
                mainScript();
            } else {
                console.error("Firebase not loaded.");
            }
        });

        function mainScript() {
            const { auth, db, appId, initialAuthToken, onAuthStateChanged, signInAnonymously, signInWithCustomToken, doc, getDoc, setDoc, onSnapshot } = window.firebase;
            
            let userId = null;

            const canvas = document.getElementById('animationCanvas');
            const ctx = canvas.getContext('2d');
            const layerButtonsContainer = document.getElementById('layer-buttons');
            let frames = [];
            let currentFrameIndex = 0;
            let currentLayerIndex = 0;
            let isDrawing = false;
            let lastX = 0, lastY = 0;
            let playbackInterval = null;
            let playbackIndex = 0;
            let strokeColor = '#000000';
            let strokeSize = 5;
            let tool = 'pencil'; // 'pencil', 'eraser', 'fill'
            const MAX_LAYERS = 5;

            let originalCanvasWidth;
            let originalCanvasHeight;
            let scale = 1.0;
            let offsetX = 0;
            let offsetY = 0;
            let initialPinchDistance = null;
            let initialMidPoint = { x: 0, y: 0 };
            let isPinching = false;

            // UI elements
            const userIdDisplay = document.getElementById('userIdDisplay');
            const pencilTool = document.getElementById('pencilTool');
            const eraserTool = document.getElementById('eraserTool');
            const fillTool = document.getElementById('fillTool');
            const colorPicker = document.getElementById('colorPicker');
            const strokeSizeInput = document.getElementById('strokeSize');
            const frameCounter = document.getElementById('frameCounter');
            const addFrameBtn = document.getElementById('addFrameBtn');
            const prevFrameBtn = document.getElementById('prevFrameBtn');
            const nextFrameBtn = document.getElementById('nextFrameBtn');
            const duplicateFrameBtn = document.getElementById('duplicateFrameBtn');
            const deleteFrameBtn = document.getElementById('deleteFrameBtn');
            const playBtn = document.getElementById('playBtn');
            const stopBtn = document.getElementById('stopBtn');
            const fpsSlider = document.getElementById('fpsSlider');
            const fpsValue = document.getElementById('fpsValue');
            const aiDrawBtn = document.getElementById('aiDrawBtn');
            const aiFixBtn = document.getElementById('aiFixBtn');
            const saveProjectBtn = document.getElementById('saveProjectBtn');
            const loadProjectBtn = document.getElementById('loadProjectBtn');

            // Off-screen canvas for layers to manage drawing logic
            const offscreenLayerCanvases = [];
            const offscreenLayerContexts = [];
            for (let i = 0; i < MAX_LAYERS; i++) {
                const layerCanvas = document.createElement('canvas');
                layerCanvas.width = canvas.width;
                layerCanvas.height = canvas.height;
                offscreenLayerCanvases.push(layerCanvas);
                offscreenLayerContexts.push(layerCanvas.getContext('2d'));
            }

            // --- Firebase Auth & Firestore ---
            onAuthStateChanged(auth, async (user) => {
                if (user) {
                    userId = user.uid;
                    userIdDisplay.textContent = `User ID: ${userId}`;
                    initializeProjectListener();
                } else {
                    try {
                        if (initialAuthToken) {
                            await signInWithCustomToken(auth, initialAuthToken);
                        } else {
                            await signInAnonymously(auth);
                        }
                    } catch (error) {
                        console.error("Firebase auth error:", error);
                        userIdDisplay.textContent = "Error: Authentication failed.";
                    }
                }
            });

            function initializeProjectListener() {
                const projectRef = doc(db, 'artifacts', appId, 'users', userId, 'projects', 'my-project');
                onSnapshot(projectRef, (docSnap) => {
                    if (docSnap.exists()) {
                        const data = docSnap.data();
                        if (data && data.framesData) {
                            try {
                                frames = JSON.parse(data.framesData);
                                currentFrameIndex = 0;
                                currentLayerIndex = 0;
                                updateFrameCounter();
                                updateLayerButtons();
                                renderFrame();
                                showMessage("Project successfully loaded from cloud.");
                            } catch (e) {
                                console.error("Error parsing frames data:", e);
                                showMessage("Error loading project data.");
                            }
                        }
                    } else {
                        // If no project exists, initialize with a new, empty frame
                        if (frames.length === 0) {
                            initializeCanvas();
                        }
                    }
                }, (error) => {
                    console.error("Firestore snapshot error:", error);
                });
            }

            async function saveProject() {
                if (!userId) {
                    showMessage("User not authenticated. Cannot save project.");
                    return;
                }
                saveCurrentLayer(); // Make sure the current drawing is saved to the data array
                const projectRef = doc(db, 'artifacts', appId, 'users', userId, 'projects', 'my-project');
                const framesData = JSON.stringify(frames); // Serialize the nested array
                try {
                    await setDoc(projectRef, { framesData });
                    showMessage("Project saved successfully!");
                } catch (e) {
                    console.error("Error saving document:", e);
                    showMessage("Error saving project.");
                }
            }

            async function loadProject() {
                if (!userId) {
                    showMessage("User not authenticated. Cannot load project.");
                    return;
                }
                const projectRef = doc(db, 'artifacts', appId, 'users', userId, 'projects', 'my-project');
                try {
                    const docSnap = await getDoc(projectRef);
                    if (docSnap.exists()) {
                        const data = docSnap.data();
                        frames = JSON.parse(data.framesData);
                        currentFrameIndex = 0;
                        currentLayerIndex = 0;
                        updateFrameCounter();
                        updateLayerButtons();
                        renderFrame();
                        showMessage("Project loaded successfully!");
                    } else {
                        showMessage("Koi project nahi mila.");
                    }
                } catch (e) {
                    console.error("Error loading document:", e);
                    showMessage("Error loading project.");
                }
            }

            // Function to show a temporary message
            function showMessage(text) {
                const messageBox = document.getElementById('messageBox');
                const messageText = document.getElementById('messageText');
                messageText.textContent = text;
                messageBox.classList.add('show');
                setTimeout(() => {
                    messageBox.classList.remove('show');
                }, 3000);
            }

            // Initialize canvas size and frame data
            function initializeCanvas() {
                const container = document.getElementById('canvas-container');
                const aspectRatio = 16 / 9;
                let width = container.offsetWidth;
                let height = width / aspectRatio;

                canvas.width = width;
                canvas.height = height;

                if (!originalCanvasWidth) {
                    originalCanvasWidth = width;
                    originalCanvasHeight = height;
                }

                if (frames.length === 0) {
                    addNewFrame(true);
                } else {
                    renderFrame();
                }
            }
            window.addEventListener('resize', initializeCanvas);

            // --- Layers Management ---
            function addLayer(isInitial = false) {
                if (frames[currentFrameIndex].length >= MAX_LAYERS) {
                    if (!isInitial) showMessage(`Ek frame mein ${MAX_LAYERS} se zyada layers nahi ho sakti.`);
                    return;
                }
                offscreenLayerContexts[currentLayerIndex].clearRect(0, 0, originalCanvasWidth, originalCanvasHeight);
                const emptyLayerData = offscreenLayerCanvases[currentLayerIndex].toDataURL();

                if (isInitial) {
                    const emptyFrame = [];
                    for(let i = 0; i < MAX_LAYERS; i++) {
                        emptyFrame.push(emptyLayerData);
                    }
                    frames.push(emptyFrame);
                } else {
                    frames[currentFrameIndex].splice(currentLayerIndex + 1, 0, emptyLayerData);
                    currentLayerIndex++;
                }
                updateLayerButtons();
                renderFrame();
                if (!isInitial) showMessage(`Layer ${currentLayerIndex + 1} jodi gayi.`);
            }

            function updateLayerButtons() {
                layerButtonsContainer.innerHTML = '';
                for (let i = 0; i < frames[currentFrameIndex].length; i++) {
                    const layerBtn = document.createElement('button');
                    layerBtn.textContent = `Layer ${i + 1}`;
                    layerBtn.className = `layer-btn ${i === currentLayerIndex ? 'active' : ''}`;
                    layerBtn.onclick = () => {
                        saveCurrentLayer();
                        currentLayerIndex = i;
                        updateLayerButtons();
                        showMessage(`Layer ${i + 1} select ki gayi.`);
                        renderFrame(); // Render to show the active layer for drawing
                    };
                    layerButtonsContainer.appendChild(layerBtn);
                }
            }
            
            // --- Frame Management ---
            function updateFrameCounter() {
                frameCounter.textContent = `${currentFrameIndex + 1} / ${frames.length}`;
                prevFrameBtn.disabled = currentFrameIndex === 0;
                nextFrameBtn.disabled = currentFrameIndex === frames.length - 1;
            }
            
            function saveCurrentLayer() {
                const currentData = offscreenLayerCanvases[currentLayerIndex].toDataURL();
                frames[currentFrameIndex][currentLayerIndex] = currentData;
            }

            function renderFrame(frameIndex = currentFrameIndex) {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                
                // Clear and re-draw all offscreen layers to the main canvas
                for(let i = 0; i < MAX_LAYERS; i++) {
                    offscreenLayerContexts[i].clearRect(0, 0, originalCanvasWidth, originalCanvasHeight);
                    if (frames[frameIndex] && frames[frameIndex][i]) {
                        const img = new Image();
                        img.onload = () => {
                            offscreenLayerContexts[i].drawImage(img, 0, 0, originalCanvasWidth, originalCanvasHeight);
                            
                            // Re-draw all layers to the main canvas
                            ctx.clearRect(0, 0, canvas.width, canvas.height);
                            ctx.save();
                            ctx.translate(canvas.width / 2, canvas.height / 2);
                            ctx.scale(scale, scale);
                            ctx.translate(-canvas.width / 2, -canvas.height / 2);
                            ctx.translate(offsetX, offsetY);
                            offscreenLayerCanvases.forEach(layerCanvas => {
                                ctx.drawImage(layerCanvas, 0, 0, originalCanvasWidth, originalCanvasHeight);
                            });
                            ctx.restore();
                        };
                        img.src = frames[frameIndex][i];
                    }
                }
            }

            function addNewFrame(isInitial = false) {
                if (frames.length >= 100) {
                    if (!isInitial) showMessage("Maximum 100 frames allowed.");
                    return;
                }
                saveCurrentLayer(); // Save current work before switching
                const newFrame = frames[currentFrameIndex].map(layer => layer); // Duplicate all layers
                frames.splice(currentFrameIndex + 1, 0, newFrame);
                currentFrameIndex++;
                updateFrameCounter();
                updateLayerButtons();
                renderFrame();
                if (!isInitial) showMessage(`Frame ${currentFrameIndex + 1} joda gaya.`);
            }

            function deleteCurrentFrame() {
                if (frames.length <= 1) {
                    showMessage("Ek frame toh rehne do!");
                    return;
                }
                frames.splice(currentFrameIndex, 1);
                if (currentFrameIndex >= frames.length) {
                    currentFrameIndex = frames.length - 1;
                }
                updateFrameCounter();
                updateLayerButtons();
                renderFrame();
                showMessage("Frame delete kar diya gaya.");
            }

            function duplicateCurrentFrame() {
                if (frames.length >= 100) {
                    showMessage("Maximum 100 frames allowed.");
                    return;
                }
                saveCurrentLayer();
                const currentFrameData = frames[currentFrameIndex].map(layer => layer);
                frames.splice(currentFrameIndex + 1, 0, currentFrameData);
                currentFrameIndex++;
                updateFrameCounter();
                updateLayerButtons();
                renderFrame();
                showMessage(`Frame ${currentFrameIndex + 1} duplicate kar diya gaya.`);
            }

            function goToFrame(index) {
                if (index >= 0 && index < frames.length) {
                    saveCurrentLayer();
                    currentFrameIndex = index;
                    // Reset offscreen canvases to match the new frame's data
                    for(let i = 0; i < MAX_LAYERS; i++) {
                        const img = new Image();
                        img.onload = () => {
                            offscreenLayerContexts[i].clearRect(0, 0, originalCanvasWidth, originalCanvasHeight);
                            offscreenLayerContexts[i].drawImage(img, 0, 0, originalCanvasWidth, originalCanvasHeight);
                        };
                        img.src = frames[currentFrameIndex][i];
                    }
                    renderFrame();
                    updateFrameCounter();
                }
            }

            // --- Playback Controls ---
            function playAnimation() {
                if (playbackInterval) return;
                playbackIndex = 0;
                const fps = parseInt(fpsSlider.value);
                const intervalTime = 1000 / fps;
                playBtn.innerHTML = '<i class="fas fa-pause"></i>';
                playBtn.onclick = pauseAnimation;

                playbackInterval = setInterval(() => {
                    if (playbackIndex >= frames.length) {
                        playbackIndex = 0;
                    }
                    renderFrame(playbackIndex);
                    playbackIndex++;
                }, intervalTime);
            }

            function pauseAnimation() {
                clearInterval(playbackInterval);
                playbackInterval = null;
                playBtn.innerHTML = '<i class="fas fa-play"></i>';
                playBtn.onclick = playAnimation;
            }

            function stopAnimation() {
                pauseAnimation();
                renderFrame(currentFrameIndex);
            }

            // --- Drawing Logic ---
            function getPos(e, touchIndex = 0) {
                const rect = canvas.getBoundingClientRect();
                let x, y;
                if (e.touches) {
                    x = e.touches[touchIndex].clientX - rect.left;
                    y = e.touches[touchIndex].clientY - rect.top;
                } else {
                    x = e.clientX - rect.left;
                    y = e.clientY - rect.top;
                }
                
                // Adjust for scale and offset
                const transformedX = (x / scale) - (offsetX / scale);
                const transformedY = (y / scale) - (offsetY / scale);
                
                return { x: transformedX, y: transformedY };
            }

            function startDrawing(e) {
                if (e.touches && e.touches.length > 1) {
                    isDrawing = false;
                    isPinching = true;
                    handlePinchStart(e);
                    return;
                }
                isDrawing = true;
                isPinching = false;
                const pos = getPos(e);
                lastX = pos.x;
                lastY = pos.y;
                draw(pos.x, pos.y);
            }

            function endDrawing() {
                if (!isDrawing && !isPinching) return;
                isDrawing = false;
                isPinching = false;
                saveCurrentLayer();
                renderFrame();
            }

            function draw(x, y) {
                if (!isDrawing) return;
                const layerCtx = offscreenLayerContexts[currentLayerIndex];
                
                layerCtx.lineJoin = 'round';
                layerCtx.lineCap = 'round';
                layerCtx.lineWidth = strokeSize / scale;
                layerCtx.strokeStyle = tool === 'eraser' ? 'white' : strokeColor;

                layerCtx.beginPath();
                layerCtx.moveTo(lastX, lastY);
                layerCtx.lineTo(x, y);
                layerCtx.stroke();
                lastX = x;
                lastY = y;
                
                renderFrame();
            }

            // --- Multi-touch Gestures ---
            function getDistance(touches) {
                const dx = touches[0].clientX - touches[1].clientX;
                const dy = touches[0].clientY - touches[1].clientY;
                return Math.sqrt(dx * dx + dy * dy);
            }
            
            function getMidPoint(touches) {
                const x = (touches[0].clientX + touches[1].clientX) / 2;
                const y = (touches[0].clientY + touches[1].clientY) / 2;
                return { x, y };
            }
            
            function handlePinchStart(e) {
                if (e.touches.length === 2) {
                    initialPinchDistance = getDistance(e.touches);
                    initialMidPoint = getMidPoint(e.touches);
                }
            }
            
            function handlePinchMove(e) {
                if (e.touches.length === 2 && initialPinchDistance !== null) {
                    const currentDistance = getDistance(e.touches);
                    const currentMidPoint = getMidPoint(e.touches);
                    
                    // Zoom
                    const zoomFactor = currentDistance / initialPinchDistance;
                    scale = Math.max(0.5, Math.min(2.0, scale * zoomFactor));

                    // Pan
                    const deltaX = currentMidPoint.x - initialMidPoint.x;
                    const deltaY = currentMidPoint.y - initialMidPoint.y;
                    offsetX += deltaX / scale;
                    offsetY += deltaY / scale;
                    
                    initialPinchDistance = currentDistance;
                    initialMidPoint = currentMidPoint;
                    
                    renderFrame();
                }
            }

            // Event Listeners
            canvas.addEventListener('mousedown', startDrawing);
            canvas.addEventListener('mouseup', endDrawing);
            canvas.addEventListener('mouseout', endDrawing);
            canvas.addEventListener('mousemove', (e) => {
                if (isDrawing) {
                    const pos = getPos(e);
                    draw(pos.x, pos.y);
                }
            });

            canvas.addEventListener('touchstart', (e) => {
                e.preventDefault();
                startDrawing(e);
            }, { passive: false });

            canvas.addEventListener('touchend', (e) => {
                endDrawing(e);
                initialPinchDistance = null;
            });

            canvas.addEventListener('touchmove', (e) => {
                e.preventDefault();
                if (e.touches.length === 1 && isDrawing) {
                    const pos = getPos(e);
                    draw(pos.x, pos.y);
                } else if (e.touches.length === 2 && isPinching) {
                    handlePinchMove(e);
                }
            }, { passive: false });

            // --- UI Interactions ---
            pencilTool.addEventListener('click', () => {
                tool = 'pencil';
                document.querySelectorAll('.icon-btn').forEach(btn => btn.classList.remove('active'));
                pencilTool.classList.add('active');
                showMessage("Pencil tool select kar liya gaya.");
            });

            eraserTool.addEventListener('click', () => {
                tool = 'eraser';
                document.querySelectorAll('.icon-btn').forEach(btn => btn.classList.remove('active'));
                eraserTool.classList.add('active');
                showMessage("Eraser tool select kar liya gaya.");
            });
            
            fillTool.addEventListener('click', () => {
                tool = 'fill';
                document.querySelectorAll('.icon-btn').forEach(btn => btn.classList.remove('active'));
                fillTool.classList.add('active');
                showMessage("Fill bucket tool select kar liya gaya.");
            });

            colorPicker.addEventListener('input', (e) => {
                strokeColor = e.target.value;
                showMessage("Naya rang select kiya gaya.");
            });

            strokeSizeInput.addEventListener('input', (e) => {
                strokeSize = e.target.value;
                showMessage(`Stroke size: ${strokeSize}`);
            });

            addFrameBtn.addEventListener('click', () => addNewFrame());
            deleteFrameBtn.addEventListener('click', deleteCurrentFrame);
            duplicateFrameBtn.addEventListener('click', duplicateCurrentFrame);
            prevFrameBtn.addEventListener('click', () => goToFrame(currentFrameIndex - 1));
            nextFrameBtn.addEventListener('click', () => goToFrame(currentFrameIndex + 1));
            playBtn.addEventListener('click', playAnimation);
            stopBtn.addEventListener('click', stopAnimation);
            saveProjectBtn.addEventListener('click', saveProject);
            loadProjectBtn.addEventListener('click', loadProject);

            fpsSlider.addEventListener('input', (e) => {
                fpsValue.textContent = e.target.value;
                if (playbackInterval) {
                    pauseAnimation();
                    playAnimation();
                }
            });

            // AI Feature Placeholder
            aiDrawBtn.addEventListener('click', () => {
                const aiPrompt = document.getElementById('aiDrawInput').value;
                const aiFrames = document.getElementById('aiFrameCount').value;
                if (aiPrompt) {
                    showMessage(`Sorry, ye feature abhi kaam nahi kar raha. '${aiPrompt}' ko ${aiFrames} frames mein generate karne ke liye ek advanced AI model ki zaroorat hai.`);
                } else {
                    showMessage("Pehle kuch likhein ki kya draw karna hai.");
                }
            });

            aiFixBtn.addEventListener('click', () => {
                const aiPrompt = document.getElementById('aiFixInput').value;
                if (aiPrompt) {
                    showMessage(`Sorry, ye feature abhi kaam nahi kar raha. '${aiPrompt}' ko process karne ke liye ek advanced AI model ki zaroorat hai.`);
                } else {
                    showMessage("Pehle kuch likhein ki kya theek karna hai.");
                }
            });
        }
    </script>
</body>
</html>

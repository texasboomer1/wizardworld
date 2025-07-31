<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Magical Adventure</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #1a1a2e; font-family: sans-serif; }
        canvas { display: block; }
        #loading-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 2em;
            z-index: 1000;
        }
        #interactButton {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            padding: 10px 20px;
            background-color: #8800ff; /* Magical purple */
            color: white;
            border: none;
            border-radius: 8px; /* Rounded corners */
            cursor: pointer;
            display: none; /* Hidden by default */
            font-size: 1.2em;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
            transition: background-color 0.3s ease, transform 0.1s ease;
            z-index: 500; /* Above canvas */
        }
        #interactButton:hover {
            background-color: #9933ff;
            transform: translateX(-50%) scale(1.05);
        }
        #interactButton:active {
            transform: translateX(-50%) scale(0.98);
        }

        /* Dialogue Box Styles */
        #dialogue-container {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 300px; /* Height of the dialogue box */
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            display: none; /* Hidden by default */
            flex-direction: column;
            padding: 15px;
            box-sizing: border-box;
            z-index: 600; /* Above interact button */
            border-top: 2px solid #8800ff;
        }
        #dialogue-history {
            flex-grow: 1;
            overflow-y: auto;
            margin-bottom: 10px;
            padding-right: 10px;
            font-size: 0.9em;
            line-height: 1.4;
        }
        #dialogue-input-area {
            display: flex;
            gap: 10px;
        }
        #dialogue-input {
            flex-grow: 1;
            padding: 8px;
            border: none;
            border-radius: 5px;
            background-color: #333;
            color: white;
        }
        #dialogue-send-button, #dialogue-close-button {
            padding: 8px 15px;
            background-color: #8800ff;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }
        #dialogue-send-button:hover, #dialogue-close-button:hover {
            background-color: #9933ff;
        }
        #dialogue-loading {
            text-align: center;
            padding: 5px;
            font-style: italic;
            color: #aaa;
            display: none;
        }
        .message-user {
            color: #add8e6; /* Light blue for user */
        }
        .message-sage {
            color: #90ee90; /* Light green for sage */
        }
    </style>
</head>
<body>
    <div id="loading-overlay">Loading Magical World...</div>
    <button id="interactButton">Talk to Sage ✨</button>

    <div id="dialogue-container">
        <div id="dialogue-history"></div>
        <div id="dialogue-loading">Sage is thinking...</div>
        <div id="dialogue-input-area">
            <input type="text" id="dialogue-input" placeholder="Ask the Sage...">
            <button id="dialogue-send-button">Send</button>
            <button id="dialogue-close-button">Close</button>
        </div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <script>
        // --- Global Variables ---
        let scene, camera, renderer, wizard, controls, homeGroup, oldSage;
        let moveForward = false;
        let moveBackward = false;
        let moveLeft = false;
        let moveRight = false;
        const playerSpeed = 0.15;
        let isInsideHouse = false;
        const interactionDistance = 8;
        const sageInteractionDistance = 10; // Distance to talk to sage
        let isDialogueOpen = false; // New state for dialogue system

        // --- House Coordinates ---
        const homePosition = new THREE.Vector3(10, 0, -10);
        const wizardInsidePosition = new THREE.Vector3(10, 1, -8);
        const wizardOutsidePosition = new THREE.Vector3(10, 1, -5);

        // --- Sage Coordinates ---
        const sagePosition = new THREE.Vector3(-10, 0, 0); // Position for the Old Sage

        // --- DOM Elements ---
        const interactButton = document.getElementById('interactButton');
        const dialogueContainer = document.getElementById('dialogue-container');
        const dialogueHistory = document.getElementById('dialogue-history');
        const dialogueInput = document.getElementById('dialogue-input');
        const dialogueSendButton = document.getElementById('dialogue-send-button');
        const dialogueCloseButton = document.getElementById('dialogue-close-button');
        const dialogueLoading = document.getElementById('dialogue-loading');

        // --- Chat History for LLM ---
        let chatHistory = [];

        // --- Initialization Function ---
        function init() {
            // 1. Scene Setup
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x0a0a1a);

            // 2. Camera Setup
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 5, 10);

            // 3. Renderer Setup
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            document.body.appendChild(renderer.domElement);

            // 4. Lighting
            const ambientLight = new THREE.AmbientLight(0x404040, 2);
            scene.add(ambientLight);

            const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight.position.set(5, 10, 5).normalize();
            scene.add(directionalLight);

            const pointLight = new THREE.PointLight(0x8800ff, 5, 50);
            pointLight.position.set(0, 5, 0);
            scene.add(pointLight);

            // 5. Ground Plane (grass)
            const groundGeometry = new THREE.PlaneGeometry(50, 50);
            const groundMaterial = new THREE.MeshLambertMaterial({ color: 0x228B22, side: THREE.DoubleSide });
            const ground = new THREE.Mesh(groundGeometry, groundMaterial);
            ground.rotation.x = Math.PI / 2;
            scene.add(ground);

            // 6. Wizard Character
            wizard = createWizard();
            wizard.position.y = 1;
            scene.add(wizard);

            // 7. Add a 2-story Home with a door
            homeGroup = new THREE.Group();

            const houseBodyGeometry = new THREE.BoxGeometry(5, 8, 5);
            const houseBodyMaterial = new THREE.MeshStandardMaterial({ color: 0x5c3a2e });
            const houseBody = new THREE.Mesh(houseBodyGeometry, houseBodyMaterial);
            houseBody.position.y = 4;
            homeGroup.add(houseBody);

            const roofGeometry = new THREE.ConeGeometry(4, 3, 4);
            const roofMaterial = new THREE.MeshStandardMaterial({ color: 0x7a2e2e });
            const roof = new THREE.Mesh(roofGeometry, roofMaterial);
            roof.rotation.y = Math.PI / 4;
            roof.position.y = 8 + 1.5;
            homeGroup.add(roof);

            const doorGeometry = new THREE.BoxGeometry(1.5, 3, 0.1);
            const doorMaterial = new THREE.MeshStandardMaterial({ color: 0x3d2a21 });
            const door = new THREE.Mesh(doorGeometry, doorMaterial);
            door.position.set(0, -2.5, 2.5);
            homeGroup.add(door);

            homeGroup.position.copy(homePosition);
            scene.add(homeGroup);

            // 8. Add Trees
            function createTree(x, z) {
                const treeGroup = new THREE.Group();

                const trunkGeometry = new THREE.CylinderGeometry(0.5, 0.7, 5, 8);
                const trunkMaterial = new THREE.MeshStandardMaterial({ color: 0x4a2c20 });
                const trunk = new THREE.Mesh(trunkGeometry, trunkMaterial);
                trunk.position.y = 2.5;
                treeGroup.add(trunk);

                const leavesGeometry = new THREE.SphereGeometry(2.5, 16, 16);
                const leavesMaterial = new THREE.MeshStandardMaterial({ color: 0x228b22, transparent: true, opacity: 0.8 });
                const leaves = new THREE.Mesh(leavesGeometry, leavesMaterial);
                leaves.position.y = 6;
                treeGroup.add(leaves);

                treeGroup.position.set(x, 0, z);
                scene.add(treeGroup);
            }

            createTree(-15, 5);
            createTree(5, 15);
            createTree(-10, -15);
            createTree(20, 0);
            createTree(-5, -20);
            createTree(-20, 10);
            createTree(15, -5);

            // 9. Add Unicorn and Sheep
            const unicorn = createUnicorn();
            unicorn.position.set(-8, 0, -5);
            scene.add(unicorn);

            const sheep = createSheep();
            sheep.position.set(-12, 0, -8);
            scene.add(sheep);

            // 10. Add a Dirt Path to the Home
            createDirtPath(homeGroup.position.x, homeGroup.position.z);

            // 11. Add an Apple Tree with Apples
            createAppleTree(-18, -10);

            // 12. Add the Old Sage NPC
            oldSage = createOldSage();
            oldSage.position.copy(sagePosition);
            scene.add(oldSage);

            // 13. Orbit Controls (for mouse view control)
            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.dampingFactor = 0.25;
            controls.screenSpacePanning = false;
            controls.maxPolarAngle = Math.PI / 2;
            controls.target.copy(wizard.position);

            // 14. Event Listeners
            window.addEventListener('resize', onWindowResize, false);
            document.addEventListener('keydown', onKeyDown, false);
            document.addEventListener('keyup', onKeyUp, false);
            interactButton.addEventListener('click', onInteractButtonClick, false);
            dialogueSendButton.addEventListener('click', sendDialogue, false);
            dialogueInput.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') {
                    sendDialogue();
                }
            }, false);
            dialogueCloseButton.addEventListener('click', closeDialogue, false);

            // Hide loading overlay
            document.getElementById('loading-overlay').style.display = 'none';
        }

        // --- Function to create the wizard character ---
        function createWizard() {
            const wizardGroup = new THREE.Group();

            const bodyGeometry = new THREE.CylinderGeometry(0.8, 1, 1.5, 8);
            const bodyMaterial = new THREE.MeshStandardMaterial({ color: 0x4b0082 });
            const body = new THREE.Mesh(bodyGeometry, bodyMaterial);
            body.position.y = 0.75;
            wizardGroup.add(body);

            const headGeometry = new THREE.SphereGeometry(0.5, 16, 16);
            const headMaterial = new THREE.MeshStandardMaterial({ color: 0xdeb887 });
            const head = new THREE.Mesh(headGeometry, headMaterial);
            head.position.y = 1.8;
            wizardGroup.add(head);

            const hatGeometry = new THREE.ConeGeometry(0.7, 1.5, 32);
            const hatMaterial = new THREE.MeshStandardMaterial({ color: 0x3a0066 });
            const hat = new THREE.Mesh(hatGeometry, hatMaterial);
            hat.position.y = 2.8;
            wizardGroup.add(hat);

            const sleeveGeometry = new THREE.CylinderGeometry(0.3, 0.3, 0.8, 8);
            const sleeveMaterial = new THREE.MeshStandardMaterial({ color: 0x4b0082 });

            const leftSleeve = new THREE.Mesh(sleeveGeometry, sleeveMaterial);
            leftSleeve.position.set(-0.9, 1.2, 0);
            leftSleeve.rotation.z = Math.PI / 2;
            wizardGroup.add(leftSleeve);

            const rightSleeve = new THREE.Mesh(sleeveGeometry, sleeveMaterial);
            rightSleeve.position.set(0.9, 1.2, 0);
            rightSleeve.rotation.z = -Math.PI / 2;
            wizardGroup.add(rightSleeve);

            const handGeometry = new THREE.SphereGeometry(0.2, 12, 12);
            const handMaterial = new THREE.MeshStandardMaterial({ color: 0xdeb887 });
            const leftHand = new THREE.Mesh(handGeometry, handMaterial);
            leftHand.position.set(-1.3, 1.2, 0);
            wizardGroup.add(leftHand);

            const rightHand = new THREE.Mesh(handGeometry, handMaterial);
            rightHand.position.set(1.3, 1.2, 0);
            wizardGroup.add(rightHand);

            const wandGroup = new THREE.Group();
            const wandGeometry = new THREE.CylinderGeometry(0.05, 0.05, 1, 8);
            const wandMaterial = new THREE.MeshStandardMaterial({ color: 0x8b4513 });
            const wand = new THREE.Mesh(wandGeometry, wandMaterial);
            wand.position.y = 0.5;
            wandGroup.add(wand);

            const wandTipGeometry = new THREE.SphereGeometry(0.1, 8, 8);
            const wandTipMaterial = new THREE.MeshStandardMaterial({ color: 0x40e0d0 });
            const wandTip = new THREE.Mesh(wandTipGeometry, wandTipMaterial);
            wandTip.position.y = 1;
            wandGroup.add(wandTip);
            
            wandGroup.rotation.z = Math.PI / 4;
            wandGroup.position.set(1.5, 1.2, 0.5);
            wizardGroup.add(wandGroup);

            return wizardGroup;
        }

        // --- Function to create a unicorn character ---
        function createUnicorn() {
            const unicornGroup = new THREE.Group();

            const bodyGeometry = new THREE.CylinderGeometry(0.6, 0.8, 2, 8);
            const bodyMaterial = new THREE.MeshStandardMaterial({ color: 0xffffff });
            const body = new THREE.Mesh(bodyGeometry, bodyMaterial);
            body.rotation.x = Math.PI / 2;
            unicornGroup.add(body);

            const headGeometry = new THREE.SphereGeometry(0.4, 16, 16);
            const headMaterial = new THREE.MeshStandardMaterial({ color: 0xffffff });
            const head = new THREE.Mesh(headGeometry, headMaterial);
            head.position.set(1.2, 0, 0);
            unicornGroup.add(head);

            const hornGeometry = new THREE.ConeGeometry(0.15, 0.8, 8);
            const hornMaterial = new THREE.MeshStandardMaterial({ color: 0xffd700 });
            const horn = new THREE.Mesh(hornGeometry, hornMaterial);
            horn.position.set(1.5, 0.5, 0);
            horn.rotation.x = Math.PI / 2;
            unicornGroup.add(horn);

            const legGeometry = new THREE.CylinderGeometry(0.15, 0.15, 1, 8);
            const legMaterial = new THREE.MeshStandardMaterial({ color: 0xcccccc });
            const legPositions = [
                { x: 0.5, y: -0.5, z: 0.5 },
                { x: 0.5, y: -0.5, z: -0.5 },
                { x: -0.5, y: -0.5, z: 0.5 },
                { x: -0.5, y: -0.5, z: -0.5 }
            ];
            legPositions.forEach(pos => {
                const leg = new THREE.Mesh(legGeometry, legMaterial);
                leg.position.set(pos.x, pos.y - 0.5, pos.z);
                unicornGroup.add(leg);
            });

            unicornGroup.position.y = 1;
            return unicornGroup;
        }

        // --- Function to create a sheep character ---
        function createSheep() {
            const sheepGroup = new THREE.Group();

            const bodyMaterial = new THREE.MeshStandardMaterial({ color: 0xf0f0f0 });
            const bodySphere1 = new THREE.Mesh(new THREE.SphereGeometry(0.8, 16, 16), bodyMaterial);
            bodySphere1.position.x = 0.5;
            sheepGroup.add(bodySphere1);

            const bodySphere2 = new THREE.Mesh(new THREE.SphereGeometry(0.7, 16, 16), bodyMaterial);
            bodySphere2.position.x = -0.3;
            sheepGroup.add(bodySphere2);

            const headGeometry = new THREE.SphereGeometry(0.3, 12, 12);
            const headMaterial = new THREE.MeshStandardMaterial({ color: 0x8b4513 });
            const head = new THREE.Mesh(headGeometry, headMaterial);
            head.position.set(1.2, 0.2, 0);
            sheepGroup.add(head);

            const legGeometry = new THREE.CylinderGeometry(0.1, 0.1, 0.5, 8);
            const legMaterial = new THREE.MeshStandardMaterial({ color: 0x8b4513 });
            const legPositions = [
                { x: 0.4, z: 0.4 },
                { x: 0.4, z: -0.4 },
                { x: -0.4, z: 0.4 },
                { x: -0.4, z: -0.4 }
            ];
            legPositions.forEach(pos => {
                const leg = new THREE.Mesh(legGeometry, legMaterial);
                leg.position.set(pos.x, -0.75, pos.z);
                sheepGroup.add(leg);
            });

            sheepGroup.position.y = 0.75;
            return sheepGroup;
        }

        // --- Function to create a dirt path ---
        function createDirtPath(targetX, targetZ) {
            const pathMaterial = new THREE.MeshStandardMaterial({ color: 0x8B4513 });
            const pathWidth = 2;
            const segmentLength = 5;
            const numSegments = 5;

            const startX = 0;
            const startZ = 0;

            for (let i = 0; i < numSegments; i++) {
                const pathGeometry = new THREE.PlaneGeometry(pathWidth, segmentLength);
                const pathSegment = new THREE.Mesh(pathGeometry, pathMaterial);
                pathSegment.rotation.x = -Math.PI / 2;

                pathSegment.position.x = startX + (targetX - startX) * (i / (numSegments - 1));
                pathSegment.position.z = startZ + (targetZ - startZ) * (i / (numSegments - 1));
                pathSegment.position.y = 0.01;

                scene.add(pathSegment);
            }
        }

        // --- Function to create an apple tree ---
        function createAppleTree(x, z) {
            const treeGroup = new THREE.Group();

            const trunkGeometry = new THREE.CylinderGeometry(0.4, 0.6, 4, 8);
            const trunkMaterial = new THREE.MeshStandardMaterial({ color: 0x654321 });
            const trunk = new THREE.Mesh(trunkGeometry, trunkMaterial);
            trunk.position.y = 2;
            treeGroup.add(trunk);

            const branchGeometry = new THREE.CylinderGeometry(0.2, 0.3, 2, 8);
            const branchMaterial = new THREE.MeshStandardMaterial({ color: 0x654321 });

            const branch1 = new THREE.Mesh(branchGeometry, branchMaterial);
            branch1.rotation.z = Math.PI / 2;
            branch1.rotation.y = Math.PI / 4;
            branch1.position.set(1, 3.5, 0);
            treeGroup.add(branch1);

            const branch2 = new THREE.Mesh(branchGeometry, branchMaterial);
            branch2.rotation.z = Math.PI / 2;
            branch2.rotation.y = -Math.PI / 4;
            branch2.position.set(-1, 4, 0);
            treeGroup.add(branch2);
            
            const leavesGeometry = new THREE.SphereGeometry(2.2, 16, 16);
            const leavesMaterial = new THREE.MeshStandardMaterial({ color: 0x32cd32 });
            const leaves = new THREE.Mesh(leavesGeometry, leavesMaterial);
            leaves.position.y = 4.5;
            treeGroup.add(leaves);

            const appleGeometry = new THREE.SphereGeometry(0.2, 8, 8);
            const appleMaterial = new THREE.MeshStandardMaterial({ color: 0xff0000 });

            for (let i = 0; i < 10; i++) {
                const apple = new THREE.Mesh(appleGeometry, appleMaterial);
                const radius = Math.random() * 1.8;
                const theta = Math.random() * Math.PI * 2;
                const phi = Math.acos(Math.random() * 2 - 1);
                apple.position.set(
                    radius * Math.sin(phi) * Math.cos(theta),
                    radius * Math.cos(phi),
                    radius * Math.sin(phi) * Math.sin(theta)
                );
                apple.position.y += leaves.position.y;
                treeGroup.add(apple);
            }

            treeGroup.position.set(x, 0, z);
            scene.add(treeGroup);
        }

        // --- Function to create the Old Sage NPC ---
        function createOldSage() {
            const sageGroup = new THREE.Group();

            // Body (robe-like cylinder)
            const bodyGeometry = new THREE.CylinderGeometry(0.7, 0.9, 1.8, 8);
            const bodyMaterial = new THREE.MeshStandardMaterial({ color: 0x708090 }); // Slate gray robe
            const body = new THREE.Mesh(bodyGeometry, bodyMaterial);
            body.position.y = 0.9;
            sageGroup.add(body);

            // Head (sphere)
            const headGeometry = new THREE.SphereGeometry(0.4, 16, 16);
            const headMaterial = new THREE.MeshStandardMaterial({ color: 0xdeb887 }); // Pale skin tone
            const head = new THREE.Mesh(headGeometry, headMaterial);
            head.position.y = 2.1;
            sageGroup.add(head);

            // Beard (simple cone or box)
            const beardGeometry = new THREE.ConeGeometry(0.3, 0.8, 8);
            const beardMaterial = new THREE.MeshStandardMaterial({ color: 0xc0c0c0 }); // Silver
            const beard = new THREE.Mesh(beardGeometry, beardMaterial);
            beard.position.set(0, 1.6, 0.3);
            beard.rotation.x = Math.PI / 6;
            sageGroup.add(beard);

            // Hat (wider, shorter cone)
            const hatGeometry = new THREE.ConeGeometry(0.8, 1.2, 32);
            const hatMaterial = new THREE.MeshStandardMaterial({ color: 0x5a2d72 }); // Darker purple
            const hat = new THREE.Mesh(hatGeometry, hatMaterial);
            hat.position.y = 2.8;
            sageGroup.add(hat);

            sageGroup.position.y = 1; // Sit on the ground
            return sageGroup;
        }

        // --- Event Handlers ---
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        function onKeyDown(event) {
            if (isDialogueOpen) return; // Prevent movement when dialogue is open
            switch (event.code) {
                case 'KeyW':
                    moveForward = true;
                    break;
                case 'KeyS':
                    moveBackward = true;
                    break;
                case 'KeyA':
                    moveLeft = true;
                    break;
                case 'KeyD':
                    moveRight = true;
                    break;
            }
        }

        function onKeyUp(event) {
            if (isDialogueOpen) return; // Prevent movement when dialogue is open
            switch (event.code) {
                case 'KeyW':
                    moveForward = false;
                    break;
                case 'KeyS':
                    moveBackward = false;
                    break;
                case 'KeyA':
                    moveLeft = false;
                    break;
                case 'KeyD':
                    moveRight = false;
                    break;
            }
        }

        function onInteractButtonClick() {
            // This button handles both house and sage interaction based on context
            const distanceToHome = wizard.position.distanceTo(homeGroup.position);
            const distanceToSage = wizard.position.distanceTo(oldSage.position);

            if (distanceToHome < interactionDistance && !isDialogueOpen) {
                if (!isInsideHouse) {
                    wizard.position.copy(wizardInsidePosition);
                    interactButton.textContent = "Exit Home";
                    isInsideHouse = true;
                } else {
                    wizard.position.copy(wizardOutsidePosition);
                    interactButton.textContent = "Enter Home";
                    isInsideHouse = false;
                }
            } else if (distanceToSage < sageInteractionDistance && !isInsideHouse) {
                openDialogue();
            }
            controls.target.copy(wizard.position);
        }

        // --- Dialogue Functions ---
        function openDialogue() {
            isDialogueOpen = true;
            dialogueContainer.style.display = 'flex';
            interactButton.style.display = 'none'; // Hide general interact button
            dialogueInput.focus(); // Focus on input field
            // Initial greeting from the Sage
            displayMessage("Sage", "Greetings, traveler! What wisdom do you seek?", 'message-sage');
        }

        function closeDialogue() {
            isDialogueOpen = false;
            dialogueContainer.style.display = 'none';
            dialogueHistory.innerHTML = ''; // Clear chat history
            chatHistory = []; // Clear LLM chat history
            interactButton.textContent = "Talk to Sage ✨"; // Reset button text
        }

        async function sendDialogue() {
            const userMessage = dialogueInput.value.trim();
            if (userMessage === "") return;

            displayMessage("You", userMessage, 'message-user');
            dialogueInput.value = "";
            dialogueLoading.style.display = 'block'; // Show loading indicator

            chatHistory.push({ role: "user", parts: [{ text: userMessage }] });

            const payload = { contents: chatHistory };
            const apiKey = ""; // If you want to use models other than gemini-2.5-flash-preview-05-20 or imagen-3.0-generate-002, provide an API key here. Otherwise, leave this as-is.
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

            try {
                let response;
                let result;
                let retries = 0;
                const maxRetries = 5;
                const baseDelay = 1000; // 1 second

                while (retries < maxRetries) {
                    response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (response.ok) {
                        result = await response.json();
                        break; // Success, exit loop
                    } else if (response.status === 429) { // Too Many Requests
                        const delay = baseDelay * Math.pow(2, retries) + Math.random() * 1000; // Exponential backoff with jitter
                        console.warn(`Rate limit hit. Retrying in ${delay / 1000}s...`);
                        await new Promise(resolve => setTimeout(resolve, delay));
                        retries++;
                    } else {
                        throw new Error(`API error: ${response.status} ${response.statusText}`);
                    }
                }

                if (!result) {
                    throw new Error("Failed to get a response from the API after multiple retries.");
                }

                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    const sageResponseText = result.candidates[0].content.parts[0].text;
                    displayMessage("Sage", sageResponseText, 'message-sage');
                    chatHistory.push({ role: "model", parts: [{ text: sageResponseText }] });
                } else {
                    displayMessage("Sage", "I'm sorry, I didn't quite understand that. Can you rephrase?", 'message-sage');
                }
            } catch (error) {
                console.error("Error calling Gemini API:", error);
                displayMessage("Sage", "My apologies, a magical disturbance prevents me from responding. Please try again later.", 'message-sage');
            } finally {
                dialogueLoading.style.display = 'none'; // Hide loading indicator
                dialogueHistory.scrollTop = dialogueHistory.scrollHeight; // Scroll to bottom
            }
        }

        function displayMessage(sender, message, className) {
            const messageElement = document.createElement('p');
            messageElement.classList.add(className);
            messageElement.innerHTML = `<strong>${sender}:</strong> ${message}`;
            dialogueHistory.appendChild(messageElement);
            dialogueHistory.scrollTop = dialogueHistory.scrollHeight; // Auto-scroll to latest message
        }

        // --- Event Handlers ---
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        function onInteractButtonClick() {
            const distanceToHome = wizard.position.distanceTo(homeGroup.position);
            const distanceToSage = wizard.position.distanceTo(oldSage.position);

            if (distanceToHome < interactionDistance && !isDialogueOpen) {
                if (!isInsideHouse) {
                    wizard.position.copy(wizardInsidePosition);
                    interactButton.textContent = "Exit Home";
                    isInsideHouse = true;
                } else {
                    wizard.position.copy(wizardOutsidePosition);
                    interactButton.textContent = "Enter Home";
                    isInsideHouse = false;
                }
            } else if (distanceToSage < sageInteractionDistance && !isInsideHouse) {
                openDialogue();
            }
            controls.target.copy(wizard.position);
        }

        // --- Animation Loop ---
        function animate() {
            requestAnimationFrame(animate);

            if (wizard) {
                // Movement logic always active
                const moveDirection = new THREE.Vector3();
                const cameraDirection = new THREE.Vector3();
                camera.getWorldDirection(cameraDirection);
                cameraDirection.y = 0;
                cameraDirection.normalize();

                const cameraRight = new THREE.Vector3();
                cameraRight.crossVectors(new THREE.Vector3(0, 1, 0), cameraDirection);

                let isMoving = false;

                if (moveForward) {
                    moveDirection.add(cameraDirection);
                    isMoving = true;
                }
                if (moveBackward) {
                    moveDirection.addScaledVector(cameraDirection, -1);
                    isMoving = true;
                }
                if (moveLeft) {
                    moveDirection.addScaledVector(cameraRight, -1);
                    isMoving = true;
                }
                if (moveRight) {
                    moveDirection.addScaledVector(cameraRight, 1);
                    isMoving = true;
                }

                if (isMoving) {
                    moveDirection.normalize();
                    wizard.position.addScaledVector(moveDirection, playerSpeed);

                    const targetAngle = Math.atan2(moveDirection.x, moveDirection.z);
                    wizard.rotation.y = THREE.MathUtils.lerp(wizard.rotation.y, targetAngle, 0.1);
                }

                // Update button visibility based on proximity and dialogue state
                const distanceToHome = wizard.position.distanceTo(homeGroup.position);
                const distanceToSage = wizard.position.distanceTo(oldSage.position);

                if (!isDialogueOpen) {
                    if (distanceToHome < interactionDistance) {
                        interactButton.style.display = 'block';
                        interactButton.textContent = isInsideHouse ? "Exit Home" : "Enter Home";
                    } else if (distanceToSage < sageInteractionDistance) {
                        interactButton.style.display = 'block';
                        interactButton.textContent = "Talk to Sage ✨";
                    } else {
                        interactButton.style.display = 'none';
                    }
                } else {
                    interactButton.style.display = 'none'; // Hide general interact button when dialogue is open
                }

                controls.target.copy(wizard.position);
            }

            controls.update();
            renderer.render(scene, camera);
        }

        // --- Start the game when the window loads ---
        window.onload = function() {
            init();
            animate();
        };
    </script>
</body>
</html>

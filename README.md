# merpppppp
cool
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Game with Save Feature</title>
    <style>
        body { margin: 0; overflow: hidden; }
        #controls { position: fixed; top: 10px; left: 10px; z-index: 10; }
        button { margin: 5px; padding: 10px; font-size: 16px; }
    </style>
</head>
<body>
    <div id="controls">
        <button onclick="saveGame()">Save Game</button>
        <button onclick="loadGame()">Load Game</button>
    </div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        // Set up Three.js scene, camera, and renderer
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // Create player cube
        const cubeGeometry = new THREE.BoxGeometry(1, 1, 1);
        const cubeMaterial = new THREE.MeshStandardMaterial({ color: 0xff0000 });
        const playerCube = new THREE.Mesh(cubeGeometry, cubeMaterial);
        playerCube.position.y = 0.5; // Set above ground level
        scene.add(playerCube);

        // Ground
        const planeGeometry = new THREE.PlaneGeometry(50, 50);
        const planeMaterial = new THREE.MeshStandardMaterial({ color: 0x00ff00 });
        const ground = new THREE.Mesh(planeGeometry, planeMaterial);
        ground.rotation.x = -Math.PI / 2; // Lay the plane flat
        scene.add(ground);

        // Lighting
        const light = new THREE.DirectionalLight(0xffffff, 1);
        light.position.set(5, 10, 5);
        scene.add(light);

        // Position camera
        camera.position.set(0, 5, 10);
        camera.lookAt(0, 0, 0);

        // Player controls
        const keys = {};
        document.addEventListener("keydown", (event) => { keys[event.key] = true; });
        document.addEventListener("keyup", (event) => { keys[event.key] = false; });

        // Move player cube based on key input
        function movePlayer() {
            const speed = 0.1;
            if (keys["ArrowUp"] || keys["w"]) playerCube.position.z -= speed;
            if (keys["ArrowDown"] || keys["s"]) playerCube.position.z += speed;
            if (keys["ArrowLeft"] || keys["a"]) playerCube.position.x -= speed;
            if (keys["ArrowRight"] || keys["d"]) playerCube.position.x += speed;
        }

        // Game loop
        function animate() {
            requestAnimationFrame(animate);
            movePlayer();
            renderer.render(scene, camera);
        }
        animate();

        // Saving game state
        async function saveGame() {
            const gameState = {
                position: {
                    x: playerCube.position.x,
                    y: playerCube.position.y,
                    z: playerCube.position.z
                }
            };
            const gameStateBlob = new Blob([JSON.stringify(gameState, null, 2)], { type: 'application/json' });

            const handle = await window.showSaveFilePicker({
                suggestedName: "game-save.json",
                types: [{ description: "JSON File", accept: { "application/json": [".json"] } }]
            });
            const writable = await handle.createWritable();
            await writable.write(gameStateBlob);
            await writable.close();
            alert("Game Saved!");
        }

        // Loading game state
        async function loadGame() {
            const [fileHandle] = await window.showOpenFilePicker({
                types: [{ description: "JSON Files", accept: { "application/json": [".json"] } }]
            });
            const file = await fileHandle.getFile();
            const contents = await file.text();
            const gameState = JSON.parse(contents);

            // Update player position
            playerCube.position.set(gameState.position.x, gameState.position.y, gameState.position.z);
            alert("Game Loaded!");
        }

        // Handle window resize
        window.addEventListener("resize", () => {
            renderer.setSize(window.innerWidth, window.innerHeight);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
        });
    </script>
</body>
</html>

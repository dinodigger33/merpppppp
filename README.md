<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Basic Minecraft-like Game</title>
    <style>
        body { margin: 0; overflow: hidden; }
        canvas { display: block; }
    </style>
</head>
<body>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        // Basic setup
        let scene, camera, renderer;
        const blockSize = 1;
        const worldSize = 10;
        let blocks = {};

        function init() {
            // Create scene
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x87ceeb);

            // Set up camera
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(worldSize / 2, worldSize / 2, worldSize * 1.5);
            camera.lookAt(worldSize / 2, 0, worldSize / 2);

            // Renderer
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            document.body.appendChild(renderer.domElement);

            // Add lights
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
            scene.add(ambientLight);
            const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
            directionalLight.position.set(5, 10, 5).normalize();
            scene.add(directionalLight);

            // Controls for adding and removing blocks
            window.addEventListener("click", onBlockClick);

            // Create the ground
            createGround();
            animate();
        }

        // Create ground layer of blocks
        function createGround() {
            for (let x = 0; x < worldSize; x++) {
                for (let z = 0; z < worldSize; z++) {
                    addBlock(x, 0, z, 0x8B4513); // Brown ground blocks
                }
            }
        }

        // Add a block to the scene and store its position in `blocks`
        function addBlock(x, y, z, color = 0x00ff00) {
            const geometry = new THREE.BoxGeometry(blockSize, blockSize, blockSize);
            const material = new THREE.MeshStandardMaterial({ color: color });
            const block = new THREE.Mesh(geometry, material);
            block.position.set(x, y, z);
            scene.add(block);

            blocks[`${x},${y},${z}`] = block; // Store block by position
        }

        // Remove a block at a specific position
        function removeBlock(x, y, z) {
            const key = `${x},${y},${z}`;
            if (blocks[key]) {
                scene.remove(blocks[key]);
                delete blocks[key];
            }
        }

        // Handles adding/removing blocks on click
        function onBlockClick(event) {
            event.preventDefault();

            // Raycaster for detecting where we click in the 3D world
            const raycaster = new THREE.Raycaster();
            const mouse = new THREE.Vector2(
                (event.clientX / window.innerWidth) * 2 - 1,
                -(event.clientY / window.innerHeight) * 2 + 1
            );
            raycaster.setFromCamera(mouse, camera);

            const intersects = raycaster.intersectObjects(Object.values(blocks));
            if (intersects.length > 0) {
                // Get the position of the intersected block
                const intersectedBlock = intersects[0].object;
                const pos = intersectedBlock.position;

                // Remove block if shift key is held, otherwise add one
                if (event.shiftKey) {
                    removeBlock(pos.x, pos.y, pos.z);
                } else {
                    // Place a new block on top
                    addBlock(pos.x, pos.y + blockSize, pos.z);
                }
            }
        }

        // Render loop
        function animate() {
            requestAnimationFrame(animate);
            renderer.render(scene, camera);
        }

        // Start the game
        init();
    </script>
</body>
</html>


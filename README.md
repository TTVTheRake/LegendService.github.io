<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Car Game</title>
    <style>
        body { margin: 0; overflow: hidden; }
        canvas { display: block; }
    </style>
</head>
<body>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        let scene, camera, renderer, car, carSpeed = 0.1, carDirection = 0, carRotationSpeed = 0.03;
        let carControls = { left: false, right: false, up: false, down: false };

        intt();

        function init() {
            // Scene
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x87ceeb);

            // Camera
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.z = 10;

            // Renderer
            renderer = new THREE.WebGLRenderer();
            renderer.setSize(window.innerWidth, window.innerHeight);
            document.body.appendChild(renderer.domElement);

            // Car (Cube for simplicity)
            let carGeometry = new THREE.BoxGeometry(2, 1, 4);
            let carMaterial = new THREE.MeshBasicMaterial({ color: 0x0000ff });
            car = new THREE.Mesh(carGeometry, carMaterial);
            scene.add(car);

            // Basic ground (Plane)
            let groundGeometry = new THREE.PlaneGeometry(2000, 2000);
            let groundMaterial = new THREE.MeshBasicMaterial({ color: 0x228B22 });
            let ground = new THREE.Mesh(groundGeometry, groundMaterial);
            ground.rotation.x = -Math.PI / 2;
            scene.add(ground);

            // Light source
            let light = new THREE.PointLight(0xffffff, 1, 100);
            light.position.set(0, 10, 10);
            scene.add(light);

            // Event listeners for controls
            document.addEventListener('keydown', onKeyDown);
            document.addEventListener('keyup', onKeyUp);

            // Start animation loop
            animate();
        }

        function onKeyDown(event) {
            if (event.key === "ArrowUp") carControls.up = true;
            if (event.key === "ArrowDown") carControls.down = true;
            if (event.key === "ArrowLeft") carControls.left = true;
            if (event.key === "ArrowRight") carControls.right = true;
        }

        function onKeyUp(event) {
            if (event.key === "ArrowUp") carControls.up = false;
            if (event.key === "ArrowDown") carControls.down = false;
            if (event.key === "ArrowLeft") carControls.left = false;
            if (event.key === "ArrowRight") carControls.right = false;
        }

        function animate() {
            requestAnimationFrame(animate);
            
            if (carControls.up) car.position.z -= carSpeed;
            if (carControls.down) car.position.z += carSpeed;
            if (carControls.left) car.rotation.y += carRotationSpeed;
            if (carControls.right) car.rotation.y -= carRotationSpeed;

            // Update camera position
            camera.position.x = car.position.x;
            camera.position.y = car.position.y + 5;
            camera.position.z = car.position.z + 10;
            camera.lookAt(car.position);

            renderer.render(scene, camera);
        }

        // Handle window resizing
        window.addEventListener('resize', () => {
            renderer.setSize(window.innerWidth, window.innerHeight);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
        });
    </script>
</body>
</html>

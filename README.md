<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fibonacci Coin Clicker</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: white;
            font-family: Arial, sans-serif;
        }
        #game-container {
            width: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        #sphere-container {
            width: 400px;
            height: 400px;
            margin: 20px 0;
        }
        #stats {
            margin: 20px 0;
            font-size: 18px;
        }
        #upgrade {
            margin: 20px 0;
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
        #upgrade:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
        #chart-container {
            width: 100%;
            height: 300px;
            margin-top: 20px;
        }
        #save-load {
            margin-top: 20px;
        }
        #save-load button {
            margin: 0 10px;
            padding: 10px 20px;
            background-color: #008CBA;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <h1>Fibonacci Coin Clicker</h1>
        <div id="sphere-container"></div>
        <div id="stats">
            Coins: <span id="coin-count">0</span>
            <br>
            Click Value: <span id="click-value">1</span>
        </div>
        <button id="upgrade">Upgrade (Cost: 1)</button>
    </div>
    <div id="chart-container">
        <canvas id="coinChart"></canvas>
    </div>
    <div id="save-load">
        <button id="save-button">Save Game</button>
        <button id="load-button">Load Game</button>
    </div>

    <script>
        let coins = 0;
        let clickValue = 1;
        let upgradeCost = 1;
        let upgradeIndex = 0;
        let coinHistory = [];
        let timeLabels = [];

        const sphereContainer = document.getElementById('sphere-container');
        const coinCountElement = document.getElementById('coin-count');
        const clickValueElement = document.getElementById('click-value');
        const upgradeButton = document.getElementById('upgrade');
        const chartCtx = document.getElementById('coinChart').getContext('2d');
        const saveButton = document.getElementById('save-button');
        const loadButton = document.getElementById('load-button');

        // Three.js setup
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, sphereContainer.clientWidth / sphereContainer.clientHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
        renderer.setClearColor(0xffffff, 0);
        renderer.setSize(sphereContainer.clientWidth, sphereContainer.clientHeight);
        sphereContainer.appendChild(renderer.domElement);

        camera.position.z = 2;

        // Create a semi-transparent sphere to represent the clickable area
        const globeGeometry = new THREE.SphereGeometry(1, 32, 32);
        const globeMaterial = new THREE.MeshBasicMaterial({
            color: 0x000000,
            transparent: true,
            opacity: 0.1,
            wireframe: true
        });
        const globe = new THREE.Mesh(globeGeometry, globeMaterial);
        scene.add(globe);

        // Line material
        const lineMaterial = new THREE.LineBasicMaterial({ color: 0x000000 });

        // Chart.js setup
        const coinChart = new Chart(chartCtx, {
            type: 'line',
            data: {
                labels: timeLabels,
                datasets: [{
                    label: 'Coin Balance',
                    data: coinHistory,
                    borderColor: 'black',
                    tension: 0.1
                }]
            },
            options: {
                responsive: true,
                scales: {
                    x: { title: { display: true, text: 'Time' } },
                    y: { title: { display: true, text: 'Coins' } }
                }
            }
        });

        function clickSphere() {
            coins += clickValue;
            createFloatingText(clickValue);
            updateStats();
        }

        function updateStats() {
            coinCountElement.textContent = Math.floor(coins);
            clickValueElement.textContent = clickValue;
            upgradeButton.textContent = `Upgrade (Cost: ${upgradeCost})`;
            upgradeButton.disabled = coins < upgradeCost;
            updateChart();
        }

        function updateChart() {
            const now = new Date();
            const timeLabel = `${now.getHours()}:${now.getMinutes()}:${now.getSeconds()}`;
            if (timeLabels.length > 50) {
                timeLabels.shift();
                coinHistory.shift();
            }
            timeLabels.push(timeLabel);
            coinHistory.push(coins);
            coinChart.update();
        }

        function fibonacci(n) {
            if (n <= 1) return n;
            return fibonacci(n - 1) + fibonacci(n - 2);
        }

        function upgrade() {
            if (coins >= upgradeCost) {
                coins -= upgradeCost;
                upgradeIndex++;
                clickValue = fibonacci(upgradeIndex + 1);
                upgradeCost = fibonacci(upgradeIndex + 3) * 10;
                updateStats();
                addUpgradeToSphere(upgradeIndex);
            }
        }

        function addUpgradeToSphere(index) {
            const phi = Math.acos(-1 + (2 * index) / 100);
            const theta = Math.sqrt(100 * Math.PI) * phi;
            const x = Math.cos(theta) * Math.sin(phi);
            const y = Math.sin(theta) * Math.sin(phi);
            const z = Math.cos(phi);

            const upgradeDot = new THREE.Mesh(
                new THREE.SphereGeometry(0.02, 8, 8),
                new THREE.MeshBasicMaterial({ color: 0x000000 })
            );
            upgradeDot.position.set(x, y, z);
            scene.add(upgradeDot);

            if (index > 0) {
                const prevPhi = Math.acos(-1 + (2 * (index - 1)) / 100);
                const prevTheta = Math.sqrt(100 * Math.PI) * prevPhi;
                const prevX = Math.cos(prevTheta) * Math.sin(prevPhi);
                const prevY = Math.sin(prevTheta) * Math.sin(prevPhi);
                const prevZ = Math.cos(prevPhi);

                const lineGeometry = new THREE.BufferGeometry().setFromPoints([
                    new THREE.Vector3(prevX, prevY, prevZ),
                    new THREE.Vector3(x, y, z)
                ]);
                const line = new THREE.Line(lineGeometry, lineMaterial);
                scene.add(line);
            }
        }

        function rotateCameraAroundSphere() {
            const radius = 2;
            const speed = 0.0005;
            camera.position.x = radius * Math.cos(Date.now() * speed);
            camera.position.z = radius * Math.sin(Date.now() * speed);
            camera.lookAt(scene.position);
        }

        function animate() {
            requestAnimationFrame(animate);
            rotateCameraAroundSphere();
            globe.rotation.y += 0.005;
            renderer.render(scene, camera);
        }

        function saveGame() {
            const gameState = {
                coins,
                clickValue,
                upgradeCost,
                upgradeIndex
            };
            localStorage.setItem('fibonacciClickerSave', JSON.stringify(gameState));
            alert('Game saved!');
        }

        function loadGame() {
            const savedState = localStorage.getItem('fibonacciClickerSave');
            if (savedState) {
                const gameState = JSON.parse(savedState);
                coins = gameState.coins;
                clickValue = gameState.clickValue;
                upgradeCost = gameState.upgradeCost;
                upgradeIndex = gameState.upgradeIndex;

                // Recreate the upgrade dots and lines
                scene.clear();
                scene.add(globe);
                for (let i = 0; i <= upgradeIndex; i++) {
                    addUpgradeToSphere(i);
                }

                updateStats();
                alert('Game loaded!');
            } else {
                alert('No saved game found.');
            }
        }

        function createFloatingText(value) {
            const text = document.createElement('div');
            text.textContent = `+${value}`;
            text.style.position = 'absolute';
            text.style.left = `${sphereContainer.offsetLeft + sphereContainer.offsetWidth / 2}px`;
            text.style.top = `${sphereContainer.offsetTop + sphereContainer.offsetHeight / 2}px`;
            text.style.color = 'black';
            text.style.fontSize = '20px';
            text.style.pointerEvents = 'none';
            document.body.appendChild(text);

            let opacity = 1;
            const animateText = () => {
                opacity -= 0.02;
                text.style.opacity = opacity;
                text.style.transform = `translateY(${(1 - opacity) * -50}px)`;
                if (opacity > 0) {
                    requestAnimationFrame(animateText);
                } else {
                    document.body.removeChild(text);
                }
            };
            animateText();
        }

        function onSphereClick(event) {
            const rect = sphereContainer.getBoundingClientRect();
            const x = ((event.clientX - rect.left) / sphereContainer.clientWidth) * 2 - 1;
            const y = -((event.clientY - rect.top) / sphereContainer.clientHeight) * 2 + 1;

            const raycaster = new THREE.Raycaster();
            raycaster.setFromCamera(new THREE.Vector2(x, y), camera);

            const intersects = raycaster.intersectObject(globe);

            if (intersects.length > 0) {
                clickSphere();
            }
        }

        // Event listeners
        sphereContainer.addEventListener('click', onSphereClick);
        upgradeButton.addEventListener('click', upgrade);
        saveButton.addEventListener('click', saveGame);
        loadButton.addEventListener('click', loadGame);
        window.addEventListener('resize', () => {
            camera.aspect = sphereContainer.clientWidth / sphereContainer.clientHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(sphereContainer.clientWidth, sphereContainer.clientHeight);
        });

        // Start animation
        animate();

        // Update stats periodically
        setInterval(updateStats, 1000);

        // Auto-save every 5 minutes
        setInterval(saveGame, 5 * 60 * 1000);

        // Initial update
        updateStats();
    </script>
</body>
</html>

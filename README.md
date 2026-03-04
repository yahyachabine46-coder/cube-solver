<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Rubik's PRO Fixed</title>
    <style>
        body { margin: 0; background: #111; color: white; font-family: sans-serif; text-align: center; }
        #canvas-container { width: 100%; height: 400px; background: #000; }
        .controls { padding: 20px; }
        button { padding: 10px 20px; font-size: 16px; cursor: pointer; background: #4caf50; border: none; color: white; border-radius: 5px; }
        #timer { font-size: 40px; margin: 10px; font-family: monospace; }
    </style>
</head>
<body>

    <h1>🧩 Rubik's Cube PRO</h1>
    <div id="canvas-container"></div>
    
    <div class="controls">
        <div id="scramble">Generating...</div>
        <div id="timer">0.000</div>
        <button id="scrambleBtn">Scramble & Play</button>
        <p>Press <b>Space</b> to Time | Use <b>L, R, U, D, F, B</b> keys to turn</p>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    
    <script>
        // Check if Three.js loaded
        if (typeof THREE === 'undefined') {
            document.body.innerHTML = "<h1>Error: Could not load Three.js. Check internet connection.</h1>";
        }

        let scene, camera, renderer, cubeGroup;
        let isAnimating = false;

        // INITIALIZE SCENE
        function init() {
            scene = new THREE.Scene();
            const container = document.getElementById('canvas-container');
            camera = new THREE.PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 1000);
            camera.position.set(3, 3, 5);
            camera.lookAt(0, 0, 0);

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(container.clientWidth, container.clientHeight);
            container.appendChild(renderer.domElement);

            const light = new THREE.DirectionalLight(0xffffff, 1);
            light.position.set(5, 5, 5);
            scene.add(light);
            scene.add(new THREE.AmbientLight(0x404040));

            createCube();
            animate();
        }

        function createCube() {
            if (cubeGroup) scene.remove(cubeGroup);
            cubeGroup = new THREE.Group();
            
            const pieceGeom = new THREE.BoxGeometry(0.95, 0.95, 0.95);
            const colors = [0xff0000, 0xff8800, 0xffffff, 0xffff00, 0x00ff00, 0x0000ff]; // R, L, U, D, F, B
            
            for (let x = -1; x <= 1; x++) {
                for (let y = -1; y <= 1; y++) {
                    for (let z = -1; z <= 1; z++) {
                        const mats = colors.map(c => new THREE.MeshLambertMaterial({ color: c }));
                        const mesh = new THREE.Mesh(pieceGeom, mats);
                        mesh.position.set(x, y, z);
                        cubeGroup.add(mesh);
                    }
                }
            }
            scene.add(cubeGroup);
        }

        async function rotateFace(face) {
            if (isAnimating) return;
            isAnimating = true;

            const pivot = new THREE.Group();
            scene.add(pivot);

            // Select pieces
            const eps = 0.1;
            const pieces = cubeGroup.children.filter(p => {
                if (face === 'R') return p.position.x > eps;
                if (face === 'L') return p.position.x < -eps;
                if (face === 'U') return p.position.y > eps;
                if (face === 'D') return p.position.y < -eps;
                if (face === 'F') return p.position.z > eps;
                if (face === 'B') return p.position.z < -eps;
            });

            pieces.forEach(p => pivot.attach(p));

            const axis = (face==='R'||face==='L') ? 'x' : (face==='U'||face==='D' ? 'y' : 'z');
            const targetAngle = (face==='L'||face==='D'||face==='B') ? Math.PI/2 : -Math.PI/2;

            const duration = 300;
            const start = performance.now();

            await new Promise(resolve => {
                function step() {
                    const now = performance.now();
                    const t = Math.min((now - start) / duration, 1);
                    pivot.rotation[axis] = targetAngle * t;
                    if (t < 1) requestAnimationFrame(step);
                    else resolve();
                }
                requestAnimationFrame(step);
            });

            pivot.updateMatrixWorld();
            [...pivot.children].forEach(p => {
                cubeGroup.attach(p);
                p.position.set(Math.round(p.position.x), Math.round(p.position.y), Math.round(p.position.z));
            });
            scene.remove(pivot);
            isAnimating = false;
        }

        function animate() {
            requestAnimationFrame(animate);
            if (!isAnimating) {
                cubeGroup.rotation.y += 0.005; // Slow idle spin
            }
            renderer.render(scene, camera);
        }

        // INPUTS
        window.addEventListener('keydown', e => {
            const key = e.key.toUpperCase();
            if ("RLUDFB".includes(key)) rotateFace(key);
        });

        document.getElementById('scrambleBtn').onclick = async () => {
            const moves = ["R", "L", "U", "D", "F", "B"];
            for(let i=0; i<10; i++) {
                await rotateFace(moves[Math.floor(Math.random()*6)]);
            }
        };

        init();
    </script>
</body>
</html>

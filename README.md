<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Neon Orbit Merge</title>
    <!-- Using a reliable CDN for Matter.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/matter-js/0.19.0/matter.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap');

        * {
            box-sizing: border-box;
            touch-action: none; /* Critical for mobile games */
        }

        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: #050505;
            color: #fff;
            font-family: 'Orbitron', sans-serif;
            width: 100vw;
            height: 100vh;
        }

        /* --- Layers --- */
        #game-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
            background: radial-gradient(circle at center, #1a1a2e 0%, #000000 100%);
        }

        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 10;
            pointer-events: none; /* Let clicks pass through to game */
            display: flex;
            flex-direction: column;
            padding: 20px;
        }

        #overlay-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 20;
            pointer-events: auto; /* Catch clicks for menus */
            display: flex;
            justify-content: center;
            align-items: center;
            background: rgba(0, 0, 0, 0.85);
            backdrop-filter: blur(5px);
            transition: opacity 0.3s;
        }

        /* --- Canvas --- */
        canvas {
            display: block;
            margin: 0 auto;
            max-width: 100%;
            max-height: 100%;
            box-shadow: 0 0 50px rgba(0, 255, 255, 0.1);
        }

        /* --- HUD --- */
        .hud-top {
            display: flex;
            justify-content: space-between;
            width: 100%;
            max-width: 500px;
            margin: 0 auto;
            margin-top: 20px; /* Space for timer */
        }

        .score-box { text-align: left; }
        .score-label {
            font-size: 12px; color: #00d2ff;
            text-transform: uppercase; letter-spacing: 2px;
            text-shadow: 0 0 10px #00d2ff;
        }
        #score {
            font-size: 32px; font-weight: 700; color: #fff;
            text-shadow: 0 0 15px rgba(255, 255, 255, 0.5);
        }

        .next-box { text-align: right; display: flex; flex-direction: column; align-items: flex-end; }
        .next-label {
            font-size: 12px; color: #ff00de;
            margin-bottom: 5px; text-transform: uppercase; letter-spacing: 2px;
            text-shadow: 0 0 10px #ff00de;
        }
        #next-ball-display {
            width: 50px; height: 50px;
            display: flex; justify-content: center; align-items: center;
            border-radius: 8px;
        }
        
        /* --- Timer Bar --- */
        #timer-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 8px;
            background: rgba(255,255,255,0.1);
            z-index: 15;
        }
        #timer-bar {
            width: 100%;
            height: 100%;
            background: #00FF9D;
            box-shadow: 0 0 10px #00FF9D;
            transition: width 0.1s linear, background-color 0.3s;
        }
        .timer-danger {
            background-color: #FF0055 !important;
            box-shadow: 0 0 15px #FF0055 !important;
        }

        /* --- Menus --- */
        .menu-content {
            text-align: center;
            padding: 20px;
            max-width: 90%;
        }

        h1 {
            font-size: 40px; margin: 0 0 10px 0;
            background: linear-gradient(90deg, #00d2ff, #ff00de);
            -webkit-background-clip: text; -webkit-text-fill-color: transparent;
            text-transform: uppercase; letter-spacing: 5px;
        }

        p { color: #aaa; margin-bottom: 30px; line-height: 1.5; }

        .btn {
            background: transparent; color: #fff;
            border: 2px solid #00d2ff; padding: 15px 40px;
            font-size: 18px; font-family: 'Orbitron', sans-serif;
            text-transform: uppercase; letter-spacing: 2px;
            cursor: pointer; transition: 0.2s;
            box-shadow: 0 0 20px rgba(0, 210, 255, 0.2);
        }
        .btn:active { background: #00d2ff; color: #000; transform: scale(0.95); }

        .hidden { display: none !important; }

        /* Danger Line */
        #danger-line {
            position: absolute;
            top: 20%;
            left: 0;
            width: 100%;
            border-top: 2px dashed rgba(255, 0, 80, 0.5);
            pointer-events: none;
            display: none; /* Show only when running */
        }
    </style>
</head>
<body>

    <!-- Game Layer -->
    <div id="game-layer">
        <canvas id="world"></canvas>
        <div id="danger-line"></div>
    </div>

    <!-- UI Layer -->
    <div id="ui-layer">
        <div id="timer-container">
            <div id="timer-bar"></div>
        </div>
        <div class="hud-top">
            <div class="score-box">
                <div class="score-label">Score</div>
                <div id="score">0</div>
            </div>
            <div class="next-box">
                <div class="next-label">Next</div>
                <div id="next-ball-display"></div>
            </div>
        </div>
    </div>

    <!-- Overlay Layer (Start / Game Over) -->
    <div id="overlay-layer">
        <div id="start-screen" class="menu-content">
            <h1>Neon Orbit</h1>
            <p>Drag to Aim. Release to Drop.<br><strong>New Rule:</strong> 5 Seconds per drop!</p>
            <button class="btn" id="start-btn">Loading...</button>
        </div>

        <div id="game-over-screen" class="menu-content hidden">
            <h1>Collapsed</h1>
            <p>Final Score</p>
            <div id="final-score" style="font-size: 40px; color:white; margin-bottom:20px;">0</div>
            <button class="btn" id="restart-btn">Retry</button>
        </div>
    </div>

    <script>
        // Wait for Matter.js to load
        window.addEventListener('load', () => {
            if (typeof Matter === 'undefined') {
                alert('Error: Physics engine failed to load. Please check your internet connection.');
                return;
            }
            const btn = document.getElementById('start-btn');
            btn.innerText = "START GAME";
            btn.addEventListener('click', startGame);
            
            document.getElementById('restart-btn').addEventListener('click', resetGame);
        });

        // --- Configuration ---
        const BALL_TYPES = [
            { radius: 15, color: '#FF00DE', glow: '#FF00DE', score: 10,  pitch: 800 },
            { radius: 25, color: '#00D2FF', glow: '#00D2FF', score: 20,  pitch: 700 },
            { radius: 35, color: '#00FF9D', glow: '#00FF9D', score: 40,  pitch: 600 },
            { radius: 45, color: '#FFE600', glow: '#FFE600', score: 80,  pitch: 500 },
            { radius: 60, color: '#FF5722', glow: '#FF5722', score: 160, pitch: 400 },
            { radius: 75, color: '#9D00FF', glow: '#9D00FF', score: 320, pitch: 300 },
            { radius: 90, color: '#FF0055', glow: '#FF0055', score: 640, pitch: 200 },
            { radius: 110, color: '#FFFFFF', glow: '#FFFFFF', score: 1280, pitch: 100 }
        ];

        // --- Variables ---
        let engine, world, runner;
        let canvas, ctx, width, height;
        let isGameActive = false;
        let canDrop = true;
        let currentScore = 0;
        
        let currentBallIndex = 0;
        let upcomingBallIndex = 0;
        
        // Timer Variables
        const DROP_TIME_LIMIT = 5000; // 5 seconds
        let timeRemaining = DROP_TIME_LIMIT;
        let lastTime = 0;

        let mouseX = 0;
        let particles = [];
        let audioCtx;

        // --- Audio ---
        function initAudio() {
            if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            if (audioCtx.state === 'suspended') audioCtx.resume();
        }

        function playSound(type, pitch = 600) {
            if (!audioCtx) return;
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);

            if (type === 'pop') {
                osc.type = 'sine';
                osc.frequency.setValueAtTime(pitch, audioCtx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(pitch * 0.1, audioCtx.currentTime + 0.15);
                gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.1);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.15);
            } else if (type === 'drop') {
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(600, audioCtx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(100, audioCtx.currentTime + 0.2);
                gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
                gain.gain.linearRampToValueAtTime(0, audioCtx.currentTime + 0.2);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.2);
            }
        }

        // --- Initialization ---
        function startGame() {
            initAudio();
            document.getElementById('start-screen').classList.add('hidden');
            document.getElementById('overlay-layer').classList.add('hidden');
            document.getElementById('danger-line').style.display = 'block';
            
            initPhysics();
            isGameActive = true;
            
            // Initial State
            currentBallIndex = Math.floor(Math.random() * 3);
            upcomingBallIndex = Math.floor(Math.random() * 3);
            
            // Reset Timer
            timeRemaining = DROP_TIME_LIMIT;
            lastTime = performance.now();
            
            updateUI();
            
            // Attach inputs
            const layer = document.getElementById('game-layer');
            layer.addEventListener('mousedown', handleInput);
            layer.addEventListener('mousemove', handleInput);
            window.addEventListener('mouseup', handleInput);
            
            layer.addEventListener('touchstart', handleInput, {passive: false});
            layer.addEventListener('touchmove', handleInput, {passive: false});
            layer.addEventListener('touchend', handleInput, {passive: false});

            requestAnimationFrame(gameLoop);
        }

        function initPhysics() {
            const Engine = Matter.Engine,
                  Runner = Matter.Runner,
                  Bodies = Matter.Bodies,
                  Composite = Matter.Composite,
                  Events = Matter.Events;

            engine = Engine.create();
            world = engine.world;
            
            canvas = document.getElementById('world');
            ctx = canvas.getContext('2d');
            
            width = window.innerWidth > 600 ? 500 : window.innerWidth;
            height = window.innerHeight;
            canvas.width = width;
            canvas.height = height;
            mouseX = width / 2;

            const wallThick = 100;
            const ground = Bodies.rectangle(width/2, height + wallThick/2 - 20, width, wallThick, { isStatic: true });
            const left = Bodies.rectangle(0 - wallThick/2, height/2, wallThick, height*2, { isStatic: true });
            const right = Bodies.rectangle(width + wallThick/2, height/2, wallThick, height*2, { isStatic: true });

            Composite.add(world, [ground, left, right]);

            Events.on(engine, 'collisionStart', (e) => {
                e.pairs.forEach(pair => {
                    const A = pair.bodyA;
                    const B = pair.bodyB;
                    if (Number.isInteger(A.label) && Number.isInteger(B.label)) {
                        if (A.label === B.label && A.id !== B.id) {
                            if (A.toRemove || B.toRemove) return;
                            A.toRemove = true;
                            B.toRemove = true;
                            mergeBalls(A, B);
                        }
                    }
                });
            });

            runner = Runner.create();
            Runner.run(runner, engine);
        }

        // --- Core Logic ---
        function mergeBalls(A, B) {
            const idx = A.label;
            const midX = (A.position.x + B.position.x) / 2;
            const midY = (A.position.y + B.position.y) / 2;
            
            Matter.Composite.remove(world, [A, B]);
            
            currentScore += BALL_TYPES[idx].score;
            createExplosion(midX, midY, BALL_TYPES[idx].color);

            if (idx < BALL_TYPES.length - 1) {
                const nextIdx = idx + 1;
                playSound('pop', BALL_TYPES[nextIdx].pitch);
                const newBall = Matter.Bodies.circle(midX, midY, BALL_TYPES[nextIdx].radius, {
                    restitution: 0.2,
                    label: nextIdx
                });
                Matter.Composite.add(world, newBall);
            } else {
                currentScore += 5000;
                playSound('pop', 100);
            }
            updateUI();
        }

        function dropBall() {
            if (!canDrop || !isGameActive) return;
            
            const r = BALL_TYPES[currentBallIndex].radius;
            let dropX = mouseX;
            // Clamp so ball doesn't spawn in wall
            if (dropX < r + 5) dropX = r + 5;
            if (dropX > width - r - 5) dropX = width - r - 5;

            canDrop = false;
            playSound('drop');

            // Drop current ball
            const ball = Matter.Bodies.circle(dropX, 50, r, {
                restitution: 0.2,
                label: currentBallIndex
            });
            Matter.Composite.add(world, ball);

            // Shift
            currentBallIndex = upcomingBallIndex; 
            upcomingBallIndex = Math.floor(Math.random() * 4); // New random
            
            // UI Update
            updateUI(); 

            // Cooldown & Timer Reset
            setTimeout(() => {
                canDrop = true;
                timeRemaining = DROP_TIME_LIMIT; // Reset Timer
                checkGameOver();
            }, 600);
        }

        // --- Input ---
        function handleInput(e) {
            if (!isGameActive) return;

            let cx;
            if (e.touches) {
                cx = e.touches[0].clientX;
            } else {
                cx = e.clientX;
            }

            const rect = canvas.getBoundingClientRect();
            mouseX = cx - rect.left;

            if (e.type === 'touchend' || e.type === 'mouseup') {
                dropBall();
            }
        }

        // --- Game Loop (Rendering & Time) ---
        function gameLoop(timestamp) {
            if (!isGameActive) return;

            // Timer Logic
            const dt = timestamp - lastTime;
            lastTime = timestamp;

            if (canDrop) {
                timeRemaining -= dt;
                
                // Update Timer Bar
                const percent = Math.max(0, (timeRemaining / DROP_TIME_LIMIT) * 100);
                const bar = document.getElementById('timer-bar');
                bar.style.width = percent + '%';
                
                // Urgency Color
                if (percent < 25) {
                    bar.classList.add('timer-danger');
                } else {
                    bar.classList.remove('timer-danger');
                }

                // Time's Up!
                if (timeRemaining <= 0) {
                    dropBall();
                }
            }

            // Draw Everything
            ctx.clearRect(0, 0, width, height);

            // 1. Draw Physics Bodies
            const bodies = Matter.Composite.allBodies(world);
            bodies.forEach(body => {
                if (body.isStatic) return;
                const type = BALL_TYPES[body.label];
                if (!type) return;

                ctx.translate(body.position.x, body.position.y);
                ctx.rotate(body.angle);
                
                ctx.shadowBlur = 15;
                ctx.shadowColor = type.glow;
                
                ctx.beginPath();
                ctx.arc(0, 0, type.radius, 0, Math.PI*2);
                ctx.fillStyle = type.color;
                ctx.fill();

                ctx.shadowBlur = 0;
                ctx.fillStyle = 'rgba(255,255,255,0.3)';
                ctx.beginPath();
                ctx.arc(-type.radius*0.3, -type.radius*0.3, type.radius*0.2, 0, Math.PI*2);
                ctx.fill();

                ctx.rotate(-body.angle);
                ctx.translate(-body.position.x, -body.position.y);
            });

            // 2. Draw Aim Line & Ghost
            if (canDrop) {
                const r = BALL_TYPES[currentBallIndex].radius;
                let aimX = mouseX;
                if (aimX < r + 5) aimX = r + 5;
                if (aimX > width - r - 5) aimX = width - r - 5;

                ctx.beginPath();
                ctx.strokeStyle = 'rgba(255,255,255,0.2)';
                ctx.moveTo(aimX, 50);
                ctx.lineTo(aimX, height);
                ctx.stroke();

                ctx.beginPath();
                ctx.arc(aimX, 50, r, 0, Math.PI*2);
                ctx.fillStyle = BALL_TYPES[currentBallIndex].color; 
                ctx.globalAlpha = 0.5;
                ctx.fill();
                ctx.globalAlpha = 1.0;
            }

            updateParticles();
            requestAnimationFrame(gameLoop);
        }

        // --- Particles ---
        function createExplosion(x, y, color) {
            for(let i=0; i<10; i++) {
                particles.push({
                    x: x, y: y,
                    vx: (Math.random()-0.5)*10,
                    vy: (Math.random()-0.5)*10,
                    life: 1.0, color: color
                });
            }
        }

        function updateParticles() {
            for(let i=particles.length-1; i>=0; i--) {
                let p = particles[i];
                p.x += p.vx; p.y += p.vy;
                p.life -= 0.05;
                if(p.life <= 0) {
                    particles.splice(i, 1);
                    continue;
                }
                ctx.globalAlpha = p.life;
                ctx.fillStyle = p.color;
                ctx.beginPath();
                ctx.arc(p.x, p.y, 3, 0, Math.PI*2);
                ctx.fill();
                ctx.globalAlpha = 1.0;
            }
        }

        // --- UI & Game Over ---
        function updateUI() {
            document.getElementById('score').innerText = currentScore;
            
            // Draw Next Ball
            const container = document.getElementById('next-ball-display');
            container.innerHTML = '';
            
            const type = BALL_TYPES[upcomingBallIndex]; 
            const div = document.createElement('div');
            
            let displaySize = type.radius * 2;
            if (displaySize > 40) displaySize = 40;
            
            div.style.width = displaySize + 'px';
            div.style.height = displaySize + 'px';
            div.style.backgroundColor = type.color;
            div.style.borderRadius = '50%';
            div.style.boxShadow = `0 0 10px ${type.glow}`;
            container.appendChild(div);
        }

        function checkGameOver() {
            const bodies = Matter.Composite.allBodies(world);
            const dangerLimit = height * 0.2; 
            
            for (let b of bodies) {
                if (b.isStatic) continue;
                if (b.position.y < dangerLimit && Math.abs(b.velocity.y) < 0.2) {
                    endGame();
                    break;
                }
            }
        }

        function endGame() {
            isGameActive = false;
            document.getElementById('final-score').innerText = currentScore;
            document.getElementById('overlay-layer').classList.remove('hidden');
            document.getElementById('game-over-screen').classList.remove('hidden');
            document.getElementById('start-screen').classList.add('hidden');
            Matter.Runner.stop(runner);
        }

        function resetGame() {
            Matter.World.clear(world);
            Matter.Engine.clear(engine);
            
            const wallThick = 100;
            const ground = Matter.Bodies.rectangle(width/2, height + wallThick/2 - 20, width, wallThick, { isStatic: true });
            const left = Matter.Bodies.rectangle(0 - wallThick/2, height/2, wallThick, height*2, { isStatic: true });
            const right = Matter.Bodies.rectangle(width + wallThick/2, height/2, wallThick, height*2, { isStatic: true });
            Matter.Composite.add(world, [ground, left, right]);

            currentScore = 0;
            isGameActive = true;
            canDrop = true;
            currentBallIndex = Math.floor(Math.random() * 3);
            upcomingBallIndex = Math.floor(Math.random() * 3);
            
            // Reset Timer logic
            timeRemaining = DROP_TIME_LIMIT;
            lastTime = performance.now();
            
            document.getElementById('overlay-layer').classList.add('hidden');
            document.getElementById('game-over-screen').classList.add('hidden');
            
            Matter.Runner.run(runner, engine);
            requestAnimationFrame(gameLoop);
            updateUI();
        }
    </script>
</body>
</html>

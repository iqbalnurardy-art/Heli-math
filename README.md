```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Game Helikopter Penjumlahan</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Fredoka+One&family=Nunito:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: #87CEEB; /* Sky blue */
            font-family: 'Nunito', sans-serif;
            touch-action: none; /* Prevent scrolling on touch */
        }
        
        #game-container {
            position: relative;
            width: 100vw;
            height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }

        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 10;
            pointer-events: none; /* Let clicks pass through to canvas */
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 20px;
            box-sizing: border-box;
        }

        .hud-text {
            font-family: 'Fredoka One', cursive;
            color: white;
            text-shadow: 2px 2px 0 #000, -1px -1px 0 #000, 1px -1px 0 #000, -1px 1px 0 #000, 1px 1px 0 #000;
            font-size: 1.5rem;
            z-index: 20;
        }

        #question-box {
            background: linear-gradient(to bottom, #ffffff, #e0f2fe);
            border: 4px solid #3B82F6; /* Biru */
            border-radius: 50px; /* Bentuk pil */
            padding: 10px 40px;
            font-size: 2.5rem;
            color: #1E40AF;
            text-shadow: none;
            box-shadow: 0 8px 0 #1D4ED8, 0 15px 15px rgba(0,0,0,0.3);
            text-align: center;
            pointer-events: auto;
            white-space: nowrap; /* Mencegah teks turun ke bawah / tumpang tindih */
        }

        /* Penyesuaian ukuran font untuk layar HP agar tidak saling bertabrakan */
        @media (max-width: 768px) {
            .hud-text { font-size: 1.1rem; }
            #question-box { 
                font-size: 1.8rem; 
                padding: 8px 25px; 
                border-width: 3px; 
                box-shadow: 0 5px 0 #1D4ED8, 0 10px 10px rgba(0,0,0,0.3); 
            }
        }

        #start-screen, #game-over-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            z-index: 20;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            pointer-events: auto;
        }

        .btn {
            background: linear-gradient(to bottom, #4ade80, #22c55e);
            border: 3px solid #166534;
            border-radius: 15px;
            padding: 15px 40px;
            font-family: 'Fredoka One', cursive;
            font-size: 1.8rem;
            color: white;
            cursor: pointer;
            box-shadow: 0 6px 0 #14532d, 0 10px 10px rgba(0,0,0,0.5);
            transition: transform 0.1s, box-shadow 0.1s;
            text-shadow: 1px 1px 0 #000;
        }

        .btn:active {
            transform: translateY(6px);
            box-shadow: 0 0px 0 #14532d, 0 4px 4px rgba(0,0,0,0.5);
        }

        .hidden {
            display: none !important;
        }
    </style>
</head>
<body>

<div id="game-container">
    <canvas id="gameCanvas"></canvas>
    
    <div id="ui-layer">
        <!-- Layout atas menggunakan absolute positioning agar tidak tumpang tindih -->
        <div class="relative w-full h-32">
            <div class="absolute top-0 left-0 hud-text">Skor: <span id="scoreDisplay">0</span></div>
            <div class="absolute top-0 right-0 hud-text text-red-500">Nyawa: <span id="livesDisplay">❤️❤️❤️</span></div>
            
            <div class="absolute top-2 left-1/2 transform -translate-x-1/2">
                <div id="question-box" class="hud-text">? + ? =</div>
            </div>
        </div>

        <!-- Instruksi di bagian bawah -->
        <div class="absolute bottom-16 left-0 w-full text-center text-white/80 text-sm md:text-base font-bold drop-shadow-md">
            Ketuk & Tahan untuk terbang naik. Lepas untuk turun.
        </div>

        <!-- Watermark Iqbal -->
        <div class="absolute bottom-4 left-0 w-full text-center text-white/60 text-sm md:text-base font-bold tracking-widest uppercase drop-shadow-sm" style="font-family: 'Nunito', sans-serif;">
            Created by Iqbal
        </div>
    </div>

    <div id="start-screen">
        <h1 class="text-5xl md:text-7xl font-bold mb-4 text-center" style="font-family: 'Fredoka One', cursive; color: #FCD34D; text-shadow: 4px 4px 0 #B45309;">Helikopter<br>Penjumlahan</h1>
        <p class="text-xl mb-8 text-center max-w-md px-4">Terbangkan helikopter dan tabrak awan yang berisi jawaban yang benar!</p>
        <button class="btn" id="startBtn">MAIN SEKARANG</button>
    </div>

    <div id="game-over-screen" class="hidden">
        <h1 class="text-6xl font-bold mb-4 text-red-500" style="font-family: 'Fredoka One', cursive; text-shadow: 3px 3px 0 #000;">YAH, JATUH!</h1>
        <p class="text-2xl mb-2">Skor Akhirmu:</p>
        <p class="text-6xl text-yellow-400 font-bold mb-8" id="finalScoreDisplay">0</p>
        <button class="btn" id="restartBtn">MAIN LAGI</button>
    </div>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    
    // UI Elements
    const startScreen = document.getElementById('start-screen');
    const gameOverScreen = document.getElementById('game-over-screen');
    const startBtn = document.getElementById('startBtn');
    const restartBtn = document.getElementById('restartBtn');
    const scoreDisplay = document.getElementById('scoreDisplay');
    const livesDisplay = document.getElementById('livesDisplay');
    const questionBox = document.getElementById('question-box');
    const finalScoreDisplay = document.getElementById('finalScoreDisplay');

    // Game State Variables
    let gameState = 'start'; // start, playing, gameover
    let score = 0;
    let lives = 3;
    let animationId;
    let currentProblem = { num1: 0, num2: 0, answer: 0 };
    let frameCount = 0;
    let difficultyMultiplier = 1;
    let particles = [];

    // Screen setup
    function resizeCanvas() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // Helicopter Object
    const heli = {
        x: 100,
        y: canvas.height / 2,
        width: 80,
        height: 50,
        velocity: 0,
        gravity: 0.25,
        lift: -6,
        isFlying: false,
        angle: 0,
        rotorAngle: 0,
        
        draw: function() {
            ctx.save();
            ctx.translate(this.x + this.width/2, this.y + this.height/2);
            // Tilt based on velocity
            this.angle = Math.min(Math.max(this.velocity * 0.05, -0.2), 0.2);
            ctx.rotate(this.angle);

            // Helicopter Body
            ctx.fillStyle = '#EF4444'; // Red
            ctx.beginPath();
            ctx.ellipse(0, 0, this.width/2, this.height/2, 0, 0, Math.PI * 2);
            ctx.fill();
            
            // Cockpit window
            ctx.fillStyle = '#60A5FA'; // Light blue
            ctx.beginPath();
            ctx.ellipse(15, -5, 15, 15, 0, 0, Math.PI * 2);
            ctx.fill();
            
            // Tail
            ctx.fillStyle = '#DC2626';
            ctx.fillRect(-this.width/2 - 30, -5, 40, 10);
            
            // Tail Rotor
            this.rotorAngle += 0.5;
            ctx.save();
            ctx.translate(-this.width/2 - 25, 0);
            ctx.rotate(this.rotorAngle);
            ctx.fillStyle = '#9CA3AF';
            ctx.fillRect(-2, -15, 4, 30);
            ctx.restore();

            // Main Rotor
            ctx.fillStyle = '#4B5563';
            ctx.fillRect(-5, -this.height/2 - 10, 10, 10); // Mast
            
            ctx.save();
            ctx.translate(0, -this.height/2 - 10);
            // Animate main rotor to look like a disc
            ctx.scale(Math.cos(this.rotorAngle * 2), 1);
            ctx.fillStyle = '#D1D5DB';
            ctx.beginPath();
            ctx.ellipse(0, 0, 45, 2, 0, 0, Math.PI * 2);
            ctx.fill();
            ctx.restore();

            // Skids
            ctx.strokeStyle = '#4B5563';
            ctx.lineWidth = 3;
            ctx.beginPath();
            ctx.moveTo(-15, this.height/2);
            ctx.lineTo(-15, this.height/2 + 10);
            ctx.moveTo(15, this.height/2);
            ctx.lineTo(15, this.height/2 + 10);
            ctx.moveTo(-30, this.height/2 + 10);
            ctx.lineTo(30, this.height/2 + 10);
            ctx.stroke();

            ctx.restore();
        },
        
        update: function() {
            if (this.isFlying) {
                this.velocity += this.lift * 0.1; // Smooth lift
                // Create smoke particles
                if (Math.random() < 0.3) {
                    createParticle(this.x - 20, this.y + this.height/2, 'smoke');
                }
            } else {
                this.velocity += this.gravity;
            }
            
            // Terminal velocity
            this.velocity = Math.max(Math.min(this.velocity, 10), -8);
            this.y += this.velocity;
            
            // Ceiling and floor collision
            if (this.y < 0) {
                this.y = 0;
                this.velocity = 0;
            }
            if (this.y + this.height > canvas.height) {
                this.y = canvas.height - this.height;
                this.velocity = 0;
                loseLife("Jatuh ke tanah!");
            }
        },
        
        reset: function() {
            this.y = canvas.height / 2;
            this.velocity = 0;
        }
    };

    let clouds = [];
    class Cloud {
        constructor(x, y, number, isCorrect) {
            this.x = x;
            this.y = y;
            this.width = 90;
            this.height = 60;
            this.speed = (3 + Math.random() * 2) * difficultyMultiplier;
            this.number = number;
            this.isCorrect = isCorrect;
            this.markedForDeletion = false;
        }

        draw() {
            ctx.save();
            ctx.translate(this.x, this.y);
            
            // Draw Cloud Shape
            ctx.fillStyle = 'rgba(255, 255, 255, 0.9)';
            ctx.beginPath();
            ctx.arc(this.width/2, this.height/2, 25, 0, Math.PI * 2); // Center
            ctx.arc(this.width/2 - 20, this.height/2 + 5, 20, 0, Math.PI * 2); // Left
            ctx.arc(this.width/2 + 20, this.height/2 + 5, 20, 0, Math.PI * 2); // Right
            ctx.fill();
            
            // Draw Number
            ctx.fillStyle = '#1F2937';
            ctx.font = 'bold 28px "Nunito"';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(this.number, this.width/2, this.height/2);
            
            ctx.restore();
        }

        update() {
            this.x -= this.speed;
            if (this.x + this.width < 0) {
                this.markedForDeletion = true;
            }
        }
    }

    // Particle System for visual effects
    function createParticle(x, y, type) {
        particles.push({
            x: x,
            y: y,
            vx: (Math.random() - 0.5) * 5,
            vy: (Math.random() - 0.5) * 5,
            life: 1,
            type: type
        });
    }

    function updateParticles() {
        for (let i = particles.length - 1; i >= 0; i--) {
            let p = particles[i];
            p.x += p.vx;
            p.y += p.vy;
            p.life -= 0.05;
            
            if (p.life <= 0) {
                particles.splice(i, 1);
                continue;
            }

            ctx.save();
            ctx.globalAlpha = p.life;
            if (p.type === 'smoke') {
                ctx.fillStyle = 'white';
                ctx.beginPath();
                ctx.arc(p.x, p.y, 5 + (1-p.life)*10, 0, Math.PI*2);
                ctx.fill();
            } else if (p.type === 'star') {
                ctx.fillStyle = '#FBBF24'; // Yellow
                ctx.translate(p.x, p.y);
                ctx.rotate(Math.random() * Math.PI);
                ctx.fillRect(-5, -5, 10, 10);
            }
            ctx.restore();
        }
    }

    // Simple Sound Generator
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    
    function playSound(type) {
        if (audioCtx.state === 'suspended') audioCtx.resume();
        const osc = audioCtx.createOscillator();
        const gainNode = audioCtx.createGain();
        
        osc.connect(gainNode);
        gainNode.connect(audioCtx.destination);
        
        if (type === 'correct') {
            osc.type = 'sine';
            osc.frequency.setValueAtTime(400, audioCtx.currentTime);
            osc.frequency.exponentialRampToValueAtTime(800, audioCtx.currentTime + 0.1);
            gainNode.gain.setValueAtTime(0.5, audioCtx.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.3);
            osc.start();
            osc.stop(audioCtx.currentTime + 0.3);
        } else if (type === 'wrong') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(150, audioCtx.currentTime);
            osc.frequency.exponentialRampToValueAtTime(50, audioCtx.currentTime + 0.3);
            gainNode.gain.setValueAtTime(0.5, audioCtx.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.3);
            osc.start();
            osc.stop(audioCtx.currentTime + 0.3);
        } else if (type === 'flap') {
             // Engine hum simulation
             osc.type = 'square';
             osc.frequency.setValueAtTime(50, audioCtx.currentTime);
             gainNode.gain.setValueAtTime(0.1, audioCtx.currentTime);
             gainNode.gain.linearRampToValueAtTime(0, audioCtx.currentTime + 0.1);
             osc.start();
             osc.stop(audioCtx.currentTime + 0.1);
        }
    }

    function generateProblem() {
        // Increase range based on score
        let maxNum = 5 + Math.floor(score / 5) * 2;
        if (maxNum > 20) maxNum = 20;

        currentProblem.num1 = Math.floor(Math.random() * maxNum) + 1;
        currentProblem.num2 = Math.floor(Math.random() * maxNum) + 1;
        currentProblem.answer = currentProblem.num1 + currentProblem.num2;
        
        questionBox.textContent = `${currentProblem.num1} + ${currentProblem.num2} = ?`;
    }

    function spawnClouds() {
        if (frameCount % Math.max(60, 120 - score) === 0) { // Spawn rate increases slightly
            const y = Math.random() * (canvas.height - 150) + 50;
            const x = canvas.width;
            
            // Determine if this cloud should be the correct answer
            // Try to keep at least one correct answer on screen
            let hasCorrectOnScreen = clouds.some(c => c.isCorrect);
            let isCorrect = !hasCorrectOnScreen || Math.random() < 0.4;
            
            let num;
            if (isCorrect) {
                num = currentProblem.answer;
            } else {
                // Generate a believable wrong answer
                let offset = Math.floor(Math.random() * 5) + 1;
                num = Math.random() < 0.5 ? currentProblem.answer + offset : currentProblem.answer - offset;
                if (num < 0) num = 0;
                
                // Prevent duplicate correct answers from wrong generation
                if (num === currentProblem.answer) num++; 
            }
            
            clouds.push(new Cloud(x, y, num, isCorrect));
        }
    }

    function checkCollisions() {
        // Helicopter hitbox (slightly smaller than visual)
        const hBox = {
            x: heli.x - heli.width/2 + 10,
            y: heli.y - heli.height/2 + 10,
            w: heli.width - 20,
            h: heli.height - 20
        };

        for (let i = clouds.length - 1; i >= 0; i--) {
            let c = clouds[i];
            const cBox = {
                x: c.x,
                y: c.y,
                w: c.width,
                h: c.height
            };

            // Basic AABB Collision
            if (hBox.x < cBox.x + cBox.w &&
                hBox.x + hBox.w > cBox.x &&
                hBox.y < cBox.y + cBox.h &&
                hBox.y + hBox.h > cBox.y) {
                
                handleCollision(c);
                clouds.splice(i, 1); // Remove cloud
            }
        }
    }

    function handleCollision(cloud) {
        if (cloud.isCorrect) {
            score++;
            scoreDisplay.textContent = score;
            difficultyMultiplier += 0.02; // Slowly increase game speed
            playSound('correct');
            generateProblem(); // New question
            
            // Visual flair
            for(let j=0; j<10; j++) createParticle(heli.x, heli.y, 'star');
            
            // Clear existing clouds so screen isn't cluttered with old wrong answers
            clouds = [];
        } else {
            loseLife("Jawaban salah!");
        }
    }

    function loseLife(reason) {
        lives--;
        playSound('wrong');
        updateLivesDisplay();
        
        // Visual indication
        ctx.fillStyle = 'rgba(255, 0, 0, 0.3)';
        ctx.fillRect(0,0, canvas.width, canvas.height);

        if (lives <= 0) {
            endGame();
        } else {
            // Temporary invulnerability or reset position could go here
            // For now, clear nearby clouds to prevent instant double-hits
            clouds = clouds.filter(c => c.x > heli.x + 200);
        }
    }

    function updateLivesDisplay() {
        let hearts = '';
        for (let i = 0; i < lives; i++) hearts += '❤️';
        livesDisplay.textContent = hearts;
    }

    // Background Scenery Setup
    let bgMountains = [];
    function initScenery() {
        bgMountains = [
            { x: 0, w: 400, h: 200, color: '#3B82F6' },
            { x: 300, w: 500, h: 300, color: '#2563EB' },
            { x: 700, w: 350, h: 150, color: '#60A5FA' },
            { x: 1000, w: 600, h: 250, color: '#3B82F6' }
        ];
    }
    
    function drawBackground() {
        // Simple parallax mountains
        bgMountains.forEach(m => {
            m.x -= 0.5 * difficultyMultiplier;
            if (m.x + m.w < 0) {
                m.x = canvas.width; // Recycle
            }
            ctx.fillStyle = m.color;
            ctx.beginPath();
            ctx.moveTo(m.x, canvas.height);
            ctx.lineTo(m.x + m.w/2, canvas.height - m.h);
            ctx.lineTo(m.x + m.w, canvas.height);
            ctx.fill();
        });
    }

    function gameLoop() {
        if (gameState !== 'playing') return;
        
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        drawBackground();
        
        // Update & Draw Helicopter
        heli.update();
        heli.draw();
        
        // Update & Draw Clouds
        spawnClouds();
        for (let i = clouds.length - 1; i >= 0; i--) {
            clouds[i].update();
            clouds[i].draw();
            if (clouds[i].markedForDeletion) {
                clouds.splice(i, 1);
            }
        }
        
        checkCollisions();
        updateParticles();
        
        frameCount++;
        animationId = requestAnimationFrame(gameLoop);
    }

    function startGame() {
        gameState = 'playing';
        score = 0;
        lives = 3;
        difficultyMultiplier = 1;
        clouds = [];
        particles = [];
        frameCount = 0;
        
        heli.reset();
        initScenery();
        generateProblem();
        
        scoreDisplay.textContent = score;
        updateLivesDisplay();
        
        startScreen.classList.add('hidden');
        gameOverScreen.classList.add('hidden');
        
        // Start Audio Context if needed
        if (audioCtx.state === 'suspended') audioCtx.resume();
        
        gameLoop();
    }

    function endGame() {
        gameState = 'gameover';
        cancelAnimationFrame(animationId);
        finalScoreDisplay.textContent = score;
        gameOverScreen.classList.remove('hidden');
    }

    // Input Handling
    function flyUp() {
        if (gameState !== 'playing') return;
        heli.isFlying = true;
        if(frameCount % 5 === 0) playSound('flap');
    }
    
    function flyDown() {
        heli.isFlying = false;
    }

    // Mouse
    canvas.addEventListener('mousedown', flyUp);
    canvas.addEventListener('mouseup', flyDown);
    // Touch
    canvas.addEventListener('touchstart', (e) => { e.preventDefault(); flyUp(); }, {passive: false});
    canvas.addEventListener('touchend', (e) => { e.preventDefault(); flyDown(); }, {passive: false});
    // Keyboard support for desktop
    window.addEventListener('keydown', (e) => {
        if (e.code === 'Space' || e.code === 'ArrowUp') flyUp();
    });
    window.addEventListener('keyup', (e) => {
        if (e.code === 'Space' || e.code === 'ArrowUp') flyDown();
    });

    // UI Buttons
    startBtn.addEventListener('click', startGame);
    restartBtn.addEventListener('click', startGame);

    // Initial render
    initScenery();
    ctx.clearRect(0,0, canvas.width, canvas.height);
    drawBackground();
    heli.draw();

</script>
</body>
</html>

```

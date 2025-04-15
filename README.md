# aaravs-escape-
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Aarav's Escape - Full Game</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: #000;
      font-family: Arial, sans-serif;
    }
    #menu, #level1, #level2, #intro, #outro {
      display: none;
      width: 100vw;
      height: 100vh;
    }
    #menu {
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      background: linear-gradient(to bottom, #000, #222);
      color: white;
    }
    button {
      padding: 15px 30px;
      margin: 10px;
      font-size: 18px;
      border: none;
      background: #ff8800;
      color: white;
      border-radius: 10px;
      cursor: pointer;
    }
    canvas {
      display: block;
      margin: 0 auto;
    }
    #timer {
      position: absolute;
      top: 10px;
      left: 10px;
      color: white;
      font-size: 24px;
    }
    #intro {
      display: flex;
      justify-content: center;
      align-items: center;
      background: black;
      color: white;
      font-size: 24px;
      text-align: center;
      padding: 40px;
      flex-direction: column;
    }
    #introText {
      max-width: 700px;
      margin-bottom: 20px;
    }
    #outro {
      position: absolute;
      top: 0; left: 0;
      background: black;
      color: white;
      display: none;
      justify-content: center;
      align-items: center;
      font-size: 48px;
      font-weight: bold;
      letter-spacing: 2px;
      opacity: 0;
      transition: opacity 2s ease-in;
      z-index: 999;
    }
    #outroText {
      text-shadow: 0 0 10px #fff;
    }
  </style>
</head>
<body>

  <div id="intro">
    <div id="introText"></div>
    <button onclick="skipIntro()">Skip</button>
  </div>

  <div id="menu">
    <h1>Aarav's Escape</h1>
    <p>An MBBS student takes on a child trafficking mafia</p>
    <button onclick="startLevel1()">Start Game</button>
  </div>

  <div id="timer">Time Left: 45</div>
  <canvas id="level1" width="800" height="400"></canvas>
  <canvas id="level2" width="800" height="400"></canvas>

  <div id="outro">
    <div id="outroText">To Be Continued…</div>
  </div>

  <!-- Audio -->
  <audio id="introMusic" src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_a0f2f83245.mp3?filename=dark-trailer-18444.mp3"></audio>
  <audio id="chaseMusic" src="https://cdn.pixabay.com/download/audio/2023/05/11/audio_c89dcf8dc7.mp3?filename=intense-chase-146695.mp3"></audio>
  <audio id="windSound" src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_8b1c6dc5a3.mp3?filename=wind-loop-18448.mp3"></audio>
  <audio id="jumpSound" src="https://cdn.pixabay.com/download/audio/2023/05/28/audio_e17c875dd8.mp3?filename=jump-149751.mp3"></audio>
  <audio id="landingSound" src="https://cdn.pixabay.com/download/audio/2022/03/28/audio_03e4b51cf5.mp3?filename=impact-soft-19415.mp3"></audio>

  <script>
    const intro = document.getElementById('intro');
    const introTextDiv = document.getElementById('introText');
    const menu = document.getElementById('menu');
    const level1Canvas = document.getElementById('level1');
    const level2Canvas = document.getElementById('level2');
    const timerDisplay = document.getElementById('timer');
    const outro = document.getElementById('outro');

    const introMusic = document.getElementById('introMusic');
    const chaseMusic = document.getElementById('chaseMusic');
    const windSound = document.getElementById('windSound');
    const jumpSound = document.getElementById('jumpSound');
    const landingSound = document.getElementById('landingSound');

    let timeLeft = 45;
    let gameOver = false;
    let bridgeReached = false;

    const introLines = [
      "In the dark corners of the city, a trafficking mafia grows bolder.",
      "Aarav, a final-year MBBS student, uncovers the truth.",
      "They kidnapped someone he cares about.",
      "He must outrun, outfight, and outsmart them.",
      "Tonight, Aarav escapes... or dies trying."
    ];
    let currentLine = 0;

    window.onload = () => {
      intro.style.display = 'flex';
      introMusic.volume = 1;
      introMusic.play();
      showNextLine();
    };

    function showNextLine() {
      if (currentLine < introLines.length) {
        introTextDiv.innerText = introLines[currentLine];
        currentLine++;
        setTimeout(showNextLine, 3000);
      } else {
        setTimeout(skipIntro, 3000);
      }
    }

    function skipIntro() {
      intro.style.display = 'none';
      fadeOutMusic(introMusic);
      menu.style.display = 'flex';
    }

    function fadeOutMusic(audio, duration = 2000) {
      const stepTime = 50;
      const steps = duration / stepTime;
      const volumeStep = audio.volume / steps;
      const fade = setInterval(() => {
        if (audio.volume > volumeStep) {
          audio.volume -= volumeStep;
        } else {
          audio.volume = 0;
          audio.pause();
          audio.currentTime = 0;
          clearInterval(fade);
        }
      }, stepTime);
    }

    function fadeInMusic(audio, duration = 2000) {
      const stepTime = 50;
      const steps = duration / stepTime;
      const volumeStep = 1 / steps;
      audio.volume = 0;
      audio.play();
      const fade = setInterval(() => {
        if (audio.volume < 1 - volumeStep) {
          audio.volume += volumeStep;
        } else {
          audio.volume = 1;
          clearInterval(fade);
        }
      }, stepTime);
    }

    function startLevel1() {
      menu.style.display = 'none';
      level1Canvas.style.display = 'block';
      timerDisplay.style.display = 'block';
      fadeInMusic(chaseMusic, 2500);
      runLevel1();
    }

    function runLevel1() {
      const ctx = level1Canvas.getContext('2d');
      const bike = { x: 100, y: 280, width: 60, height: 40, speed: 6, color: 'lime' };
      const cheetahs = [
        { x: 800, y: 280, width: 60, height: 30, speed: 3, color: 'orange' },
        { x: 1000, y: 310, width: 60, height: 30, speed: 3.5, color: 'orange' }
      ];
      let keys = {}, bridgeX = 1400;
      document.addEventListener('keydown', e => keys[e.key] = true);
      document.addEventListener('keyup', e => keys[e.key] = false);

      const countdown = setInterval(() => {
        if (gameOver || bridgeReached) return;
        timeLeft--;
        timerDisplay.textContent = `Time Left: ${timeLeft}`;
        if (timeLeft <= 0) {
          gameOver = true;
          alert("Too late! Aarav was caught.");
        }
      }, 1000);

      function update() {
        if (keys['ArrowUp'] && bike.y > 200) bike.y -= bike.speed;
        if (keys['ArrowDown'] && bike.y < 320) bike.y += bike.speed;

        cheetahs.forEach(c => {
          c.x -= c.speed;
          if (c.x < -60) c.x = 800 + Math.random() * 400;
          if (
            bike.x < c.x + c.width &&
            bike.x + bike.width > c.x &&
            bike.y < c.y + c.height &&
            bike.y + bike.height > c.y
          ) {
            gameOver = true;
            alert("Caught by cheetahs!");
          }
        });

        bridgeX -= 4;
        if (bike.x + bike.width >= bridgeX && bridgeX < 800) {
          bridgeReached = true;
          jumpSound.play();
          setTimeout(startLevel2, 1000);
        }
      }

      function draw() {
        ctx.clearRect(0, 0, level1Canvas.width, level1Canvas.height);
        ctx.fillStyle = bike.color;
        ctx.fillRect(bike.x, bike.y, bike.width, bike.height);
        cheetahs.forEach(c => {
          ctx.fillStyle = c.color;
          ctx.fillRect(c.x, c.y, c.width, c.height);
        });
        ctx.fillStyle = 'brown';
        ctx.fillRect(bridgeX, 260, 200, 60);
      }

      function loop() {
        if (!gameOver && !bridgeReached) {
          update();
          draw();
          requestAnimationFrame(loop);
        }
      }

      loop();
    }

    function startLevel2() {
      level1Canvas.style.display = 'none';
      level2Canvas.style.display = 'block';
      timerDisplay.style.display = 'none';
      fadeInMusic(windSound, 2000);
      runLevel2();
    }

    function runLevel2() {
      const ctx = level2Canvas.getContext('2d');
      const aarav = { x: 400, y: 50, width: 30, height: 50, color: '#00ff99', dy: 2 };
      const birds = [];
      for (let i = 0; i < 5; i++) {
        birds.push({
          x: Math.random() * 800,
          y: Math.random() * 300,
          width: 40,
          height: 30,
          dx: (Math.random() * 2 + 1),
          image: new Image()
        });
        birds[i].image.src = 'https://i.imgur.com/p0mA9cg.png';
      }

      function update() {
        aarav.y += aarav.dy;
        if (aarav.y > 300) {
          aarav.dy = 0;
          landingSound.play();
          setTimeout(() => {
            alert("Aarav lands safely with parachute near the cave!");
            fadeOutMusic(windSound);
            showOutro();
          }, 500);
        }
        birds.forEach(bird => {
          bird.x -= bird.dx;
          if (bird.x < -50) {
            bird.x = 800 + Math.random() * 100;
            bird.y = Math.random() * 300;
          }
        });
      }

      function draw() {
        ctx.clearRect(0, 0, level2Canvas.width, level2Canvas.height);
        ctx.beginPath();
        ctx.arc(700, 80, 50, 0, Math.PI * 2);
        ctx.fillStyle = '#ffb347';
        ctx.fill();
        ctx.fillStyle = aarav.color;
        ctx.fillRect(aarav.x, aarav.y, aarav.width, aarav.height);
        birds.forEach(bird => {
          ctx.drawImage(bird.image, bird.x, bird.y, bird.width, bird.height);
        });
      }

      function loop() {
        update();
        draw();
        requestAnimationFrame(loop);
      }

      loop();
    }

    function showOutro() {
      outro.style.display = 'flex';
      setTimeout(() => {
        outro.style.opacity = 1;
      }, 100);
    }
  </script>
</body>
</html>

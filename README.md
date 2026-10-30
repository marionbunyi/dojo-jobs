```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Job Bingo Wheel</title>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  height: 100vh;
  overflow: hidden;
  font-family: "Trebuchet MS", Arial, sans-serif;
  background: linear-gradient(135deg, #e8f7ff, #fff6df);
  display: flex;
  justify-content: center;
  align-items: center;
}

.game {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 8px;
}

h1 {
  margin: 0 0 5px;
  font-size: clamp(24px, 4vh, 42px);
  color: #1558b0;
  text-shadow: 2px 2px white;
}

.wheel-area {
  position: relative;
  width: min(76vh, 76vw);
  height: min(76vh, 76vw);
  max-width: 740px;
  max-height: 740px;
  flex-shrink: 1;
}

#wheel {
  display: block;
  width: 100%;
  height: 100%;
}

.pointer {
  position: absolute;
  z-index: 2;
  left: 50%;
  top: -5px;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 22px solid transparent;
  border-right: 22px solid transparent;
  border-top: 42px solid #ff4545;
  filter: drop-shadow(0 3px 2px #777);
}

.controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  flex-wrap: wrap;
  margin-top: 8px;
}

button {
  border: 0;
  border-radius: 16px;
  padding: 12px 23px;
  font-size: clamp(16px, 2.4vh, 23px);
  font-weight: bold;
  cursor: pointer;
  box-shadow: 0 4px 0 #0002;
}

button:disabled {
  opacity: 0.6;
  cursor: wait;
}

#drawButton {
  background: #1688f8;
  color: white;
}

#restartButton {
  background: #ff9e28;
  color: white;
}

.volume-box {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 14px;
  background: white;
  color: #245078;
  border-radius: 14px;
  font-weight: bold;
}

#volume {
  width: 100px;
}

#status {
  margin-top: 7px;
  color: #245078;
  font-size: clamp(14px, 2vh, 19px);
  font-weight: bold;
  text-align: center;
}

#confetti {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100%;
  z-index: 20;
  pointer-events: none;
}

.overlay {
  display: none;
  position: fixed;
  inset: 0;
  z-index: 10;
  background: #10264dcc;
  justify-content: center;
  align-items: center;
  padding: 16px;
}

.overlay.show {
  display: flex;
}

.popup {
  width: min(520px, 90vw);
  max-height: 92vh;
  overflow: auto;
  background: white;
  border: 7px solid #ffd43b;
  border-radius: 30px;
  padding: 18px;
  text-align: center;
  box-shadow: 0 12px 40px #0005;
  animation: pop 0.3s ease-out;
}

@keyframes pop {
  from {
    transform: scale(0.65);
    opacity: 0;
  }
  to {
    transform: scale(1);
    opacity: 1;
  }
}

.popup h2 {
  margin: 0 0 10px;
  font-size: clamp(27px, 5vh, 42px);
  color: #1764c0;
}

#winnerImage {
  width: min(420px, 76vw);
  max-height: 68vh;
  object-fit: contain;
  display: block;
  margin: 0 auto;
}

#continueButton {
  margin-top: 12px;
  color: white;
  background: #1688f8;
}

@media (max-height: 650px) {
  h1 {
    font-size: 25px;
  }

  .wheel-area {
    width: min(65vh, 65vw);
    height: min(65vh, 65vw);
  }

  button {
    padding: 8px 16px;
  }

  .controls {
    margin-top: 4px;
  }
}
</style>
</head>

<body>

<div class="game">
  <h1>🎯 JOB BINGO WHEEL 🎯</h1>

  <div class="wheel-area">
    <div class="pointer"></div>
    <canvas id="wheel" width="1000" height="1000"></canvas>
  </div>

  <div class="controls">
    <button id="drawButton">🎡 DRAW AGAIN!</button>
    <button id="restartButton">🔄 RESTART</button>

    <div class="volume-box">
      🔊 Volume
      <input id="volume" type="range" min="0" max="100" value="45">
    </div>
  </div>

  <div id="status">Ready? Spin the wheel!</div>
</div>

<!-- Winner popup: picture only, no extra job name -->
<div class="overlay" id="overlay">
  <div class="popup">
    <h2>🎉 YOU GOT! 🎉</h2>
    <img id="winnerImage" alt="Selected job">
    <button id="continueButton">CONTINUE</button>
  </div>
</div>

<canvas id="confetti"></canvas>

<script>
/* =========================================
   INDIVIDUAL IMAGES FROM GITHUB
========================================= */

const BASE =
  "https://raw.githubusercontent.com/marionbunyi/dojo-jobs/main/";

const jobs = [
  { name: "driver",      file: "driver.png",      eligible: true },
  { name: "pilot",       file: "pilot.png",       eligible: true },
  { name: "dentist",     file: "dentist.png",     eligible: true },
  { name: "teacher",     file: "teacher.png",     eligible: true },
  { name: "doctor",      file: "doctor.png",      eligible: true },
  { name: "engineer",    file: "engineer.png",    eligible: true },
  { name: "farmer",      file: "farmer.png",      eligible: true },
  { name: "policeman",   file: "policeman.png",   eligible: true },
  { name: "firefighter", file: "firefighter.png", eligible: true },
  { name: "chef",        file: "chef.PNG",        eligible: true },

  { name: "nurse",       file: "nurse.png",       eligible: false },
  { name: "artist",      file: "artist.png",      eligible: false },
  { name: "scientist",   file: "scientist.png",   eligible: false },
  { name: "musician",    file: "musician.png",    eligible: false },
  { name: "baker",       file: "baker.png",       eligible: false }
];

const colors = [
  "#45B8FF", "#FFCE36", "#5BDFB9", "#B995FF", "#50B9FF",
  "#FF9B40", "#8BD34F", "#FF6666", "#F18EC9", "#43C9C4",
  "#FFD166", "#BA9AF5", "#64C9FF", "#F58E65", "#77D6B0"
];

const canvas = document.getElementById("wheel");
const ctx = canvas.getContext("2d");

const drawButton = document.getElementById("drawButton");
const restartButton = document.getElementById("restartButton");
const continueButton = document.getElementById("continueButton");
const statusText = document.getElementById("status");
const overlay = document.getElementById("overlay");
const winnerImage = document.getElementById("winnerImage");
const volumeSlider = document.getElementById("volume");

const confettiCanvas = document.getElementById("confetti");
const confettiCtx = confettiCanvas.getContext("2d");

const slice = Math.PI * 2 / jobs.length;

let rotation = 0;
let spinning = false;
let drawPool = [];
let audioContext = null;
let musicStarted = false;
let musicNote = 0;
let confettiPieces = [];
let confettiFrame = null;

/* =========================================
   LOAD ALL 15 INDIVIDUAL IMAGES
========================================= */

jobs.forEach(job => {
  job.image = new Image();

  job.image.onload = drawWheel;

  job.image.onerror = () => {
    console.error("Image failed to load:", BASE + job.file);
    statusText.textContent =
      "Could not load " + job.file + ". Check its GitHub filename.";
  };

  job.image.src = BASE + encodeURIComponent(job.file);
});

/* =========================================
   DRAW WHEEL — PICTURES ONLY
========================================= */

function drawWheel() {
  const W = canvas.width;
  const cx = W / 2;
  const cy = W / 2;
  const radius = W * 0.455;

  ctx.clearRect(0, 0, W, W);

  ctx.beginPath();
  ctx.arc(cx, cy, radius + 12, 0, Math.PI * 2);
  ctx.fillStyle = "#FFFFFF";
  ctx.fill();
  ctx.lineWidth = 12;
  ctx.strokeStyle = "#1558B0";
  ctx.stroke();

  jobs.forEach((job, i) => {
    const start = rotation + i * slice;
    const end = start + slice;
    const middle = start + slice / 2;

    // Colored wheel segment
    ctx.beginPath();
    ctx.moveTo(cx, cy);
    ctx.arc(cx, cy, radius, start, end);
    ctx.closePath();
    ctx.fillStyle = colors[i % colors.length];
    ctx.fill();

    ctx.lineWidth = 5;
    ctx.strokeStyle = "#FFFFFF";
    ctx.stroke();

    // Clip image to its wheel segment
    ctx.save();
    ctx.beginPath();
    ctx.moveTo(cx, cy);
    ctx.arc(cx, cy, radius, start, end);
    ctx.closePath();
    ctx.clip();

    const imageRadius = radius * 0.63;
    const px = cx + Math.cos(middle) * imageRadius;
    const py = cy + Math.sin(middle) * imageRadius;
    const imageSize = 140;

    if (job.image.complete && job.image.naturalWidth > 0) {
      ctx.save();
      ctx.translate(px, py);
      ctx.rotate(middle + Math.PI / 2);

      ctx.fillStyle = "#FFFFFF";
      roundedRect(
        ctx,
        -imageSize / 2 - 4,
        -imageSize / 2 - 4,
        imageSize + 8,
        imageSize + 8,
        12
      );
      ctx.fill();

      // The image contains its own job name.
      drawContainedImage(
        ctx,
        job.image,
        -imageSize / 2,
        -imageSize / 2,
        imageSize,
        imageSize
      );

      ctx.restore();
    }

    // No additional job-name labels.
    ctx.restore();
  });

  // Center circle
  ctx.beginPath();
  ctx.arc(cx, cy, 58, 0, Math.PI * 2);
  ctx.fillStyle = "#FFFFFF";
  ctx.fill();
  ctx.lineWidth = 8;
  ctx.strokeStyle = "#1558B0";
  ctx.stroke();

  ctx.fillStyle = "#1558B0";
  ctx.font = "bold 25px Trebuchet MS, Arial";
  ctx.textAlign = "center";
  ctx.textBaseline = "middle";
  ctx.fillText("GO!", cx, cy);
}

function roundedRect(ctx, x, y, w, h, r) {
  ctx.beginPath();
  ctx.moveTo(x + r, y);
  ctx.lineTo(x + w - r, y);
  ctx.quadraticCurveTo(x + w, y, x + w, y + r);
  ctx.lineTo(x + w, y + h - r);
  ctx.quadraticCurveTo(
    x + w, y + h, x + w - r, y + h
  );
  ctx.lineTo(x + r, y + h);
  ctx.quadraticCurveTo(x, y + h, x, y + h - r);
  ctx.lineTo(x, y + r);
  ctx.quadraticCurveTo(x, y, x + r, y);
  ctx.closePath();
}

function drawContainedImage(ctx, img, x, y, w, h) {
  const scale = Math.min(
    w / img.naturalWidth,
    h / img.naturalHeight
  );

  const dw = img.naturalWidth * scale;
  const dh = img.naturalHeight * scale;

  ctx.drawImage(
    img,
    x + (w - dw) / 2,
    y + (h - dh) / 2,
    dw,
    dh
  );
}

/* =========================================
   RANDOM DRAW POOL — ONLY 10 JOBS
========================================= */

function shuffle(array) {
  const result = [...array];

  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [result[i], result[j]] = [result[j], result[i]];
  }

  return result;
}

function resetDrawPool() {
  drawPool = shuffle(jobs.filter(job => job.eligible));
}

/* =========================================
   SOUND EFFECTS
========================================= */

function getAudio() {
  if (!audioContext) {
    const AudioClass =
      window.AudioContext || window.webkitAudioContext;

    if (AudioClass) audioContext = new AudioClass();
  }

  if (audioContext && audioContext.state === "suspended") {
    audioContext.resume();
  }

  return audioContext;
}

function playTone(freq, duration, type = "sine", loudness = 0.12) {
  const ac = getAudio();
  if (!ac) return;

  const osc = ac.createOscillator();
  const gain = ac.createGain();
  const volume = Number(volumeSlider.value) / 100;

  osc.type = type;
  osc.frequency.setValueAtTime(freq, ac.currentTime);

  gain.gain.setValueAtTime(
    Math.max(0.0001, loudness * volume),
    ac.currentTime
  );

  gain.gain.exponentialRampToValueAtTime(
    0.0001,
    ac.currentTime + duration
  );

  osc.connect(gain);
  gain.connect(ac.destination);

  osc.start();
  osc.stop(ac.currentTime + duration);
}

function spinTick() {
  playTone(260, 0.07, "square", 0.08);
}

function victorySound() {
  [523, 659, 784, 1047].forEach((freq, i) => {
    setTimeout(() => {
      playTone(freq, 0.28, "sine", 0.18);
    }, i * 130);
  });
}

/* =========================================
   BACKGROUND MUSIC
========================================= */

function startMusic() {
  if (musicStarted) return;

  getAudio();
  musicStarted = true;

  const melody = [
    523, 659, 784, 659,
    587, 698, 880, 698,
    523, 659, 784, 1047,
    880, 784, 659, 523
  ];

  setInterval(() => {
    if (Number(volumeSlider.value) === 0) return;

    playTone(
      melody[musicNote % melody.length],
      0.24,
      "sine",
      0.035
    );

    musicNote++;
  }, 300);
}

/* =========================================
   CONFETTI
========================================= */

function resizeConfetti() {
  confettiCanvas.width =
    window.innerWidth * devicePixelRatio;
  confettiCanvas.height =
    window.innerHeight * devicePixelRatio;

  confettiCtx.setTransform(
    devicePixelRatio, 0, 0, devicePixelRatio, 0, 0
  );
}

window.addEventListener("resize", resizeConfetti);
resizeConfetti();

function launchConfetti() {
  confettiPieces = [];

  for (let i = 0; i < 180; i++) {
    confettiPieces.push({
      x: Math.random() * window.innerWidth,
      y: -Math.random() * window.innerHeight,
      size: 5 + Math.random() * 9,
      speedY: 3 + Math.random() * 5,
      speedX: -2 + Math.random() * 4,
      rotation: Math.random() * 6,
      spin: -0.15 + Math.random() * 0.3,
      color: colors[Math.floor(Math.random() * colors.length)]
    });
  }

  if (confettiFrame) {
    cancelAnimationFrame(confettiFrame);
  }

  animateConfetti();
}

function animateConfetti() {
  confettiCtx.clearRect(
    0, 0, window.innerWidth, window.innerHeight
  );

  confettiPieces.forEach(p => {
    p.x += p.speedX;
    p.y += p.speedY;
    p.rotation += p.spin;

    confettiCtx.save();
    confettiCtx.translate(p.x, p.y);
    confettiCtx.rotate(p.rotation);
    confettiCtx.fillStyle = p.color;
    confettiCtx.fillRect(
      -p.size / 2,
      -p.size / 2,
      p.size,
      p.size * 0.65
    );
    confettiCtx.restore();
  });

  confettiPieces = confettiPieces.filter(
    p => p.y < window.innerHeight + 30
  );

  if (confettiPieces.length) {
    confettiFrame = requestAnimationFrame(animateConfetti);
  } else {
    confettiCtx.clearRect(
      0, 0, window.innerWidth, window.innerHeight
    );
    confettiFrame = null;
  }
}

/* =========================================
   WINNER POPUP — PICTURE ONLY
========================================= */

function showWinner(job) {
  // Show the complete job card, including its printed name.
  // No separate job-name text is added.
  winnerImage.src = job.image.src;

  overlay.classList.add("show");
  launchConfetti();
  victorySound();
}

/* =========================================
   SPIN THE WHEEL
========================================= */

function spinWheel() {
  if (spinning) return;

  startMusic();

  if (drawPool.length === 0) {
    resetDrawPool();
    statusText.textContent =
      "New round! All 10 target jobs are back.";
  }

  // Select only from the 10 eligible jobs.
  const winner = drawPool.pop();

  const winnerIndex = jobs.findIndex(
    job => job.name === winner.name
  );

  spinning = true;
  drawButton.disabled = true;
  restartButton.disabled = true;
  statusText.textContent = "Spinning...";

  const pointerAngle = -Math.PI / 2;
  const winnerCenter = (winnerIndex + 0.5) * slice;

  let desiredRotation = pointerAngle - winnerCenter;

  desiredRotation =
    ((desiredRotation % (Math.PI * 2)) + Math.PI * 2) %
    (Math.PI * 2);

  const currentNormalized =
    ((rotation % (Math.PI * 2)) + Math.PI * 2) %
    (Math.PI * 2);

  let delta = desiredRotation - currentNormalized;

  while (delta < 0) {
    delta += Math.PI * 2;
  }

  const extraTurns =
    (5 + Math.floor(Math.random() * 4)) * Math.PI * 2;

  const startRotation = rotation;
  const finalRotation = rotation + extraTurns + delta;
  const duration = 5000;
  const startTime = performance.now();

  let lastTick = 0;

  function animate(now) {
    const elapsed = now - startTime;
    const progress = Math.min(elapsed / duration, 1);
    const eased = 1 - Math.pow(1 - progress, 4);

    rotation =
      startRotation +
      (finalRotation - startRotation) * eased;

    drawWheel();

    if (now - lastTick > 115 && progress < 0.95) {
      spinTick();
      lastTick = now;
    }

    if (progress < 1) {
      requestAnimationFrame(animate);
    } else {
      rotation =
        ((finalRotation % (Math.PI * 2)) +
         Math.PI * 2) % (Math.PI * 2);

      drawWheel();

      spinning = false;
      drawButton.disabled = false;
      restartButton.disabled = false;

      statusText.textContent =
        drawPool.length + " target job(s) left to draw.";

      showWinner(winner);
    }
  }

  requestAnimationFrame(animate);
}

/* =========================================
   BUTTONS
========================================= */

drawButton.addEventListener("click", spinWheel);

continueButton.addEventListener("click", () => {
  overlay.classList.remove("show");

  if (drawPool.length === 0) {
    statusText.textContent =
      "All 10 jobs have been drawn! Draw again for a new round.";
  } else {
    statusText.textContent =
      drawPool.length + " target job(s) left to draw.";
  }
});

restartButton.addEventListener("click", () => {
  if (spinning) return;

  overlay.classList.remove("show");
  rotation = 0;

  resetDrawPool();
  drawWheel();

  statusText.textContent = "Restarted! Ready to spin again.";
});

/* =========================================
   START GAME
========================================= */

resetDrawPool();
drawWheel();
</script>

</body>
</html>
```

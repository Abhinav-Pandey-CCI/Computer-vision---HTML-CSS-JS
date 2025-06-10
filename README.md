# Object-detection-using-CV

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Iron Man Vision with Voice</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: black;
      font-family: 'Orbitron', sans-serif;
      color: #00fff7;
    }

    #videoElement {
      position: absolute;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      object-fit: cover;
      z-index: 1;
      filter: brightness(0.8) contrast(1.2);
    }

    canvas {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 2;
    }

    .hud {
      position: absolute;
      top: 20px;
      left: 20px;
      z-index: 3;
      font-size: 18px;
      color: #00fff7;
      text-shadow: 0 0 5px #00fff7;
      animation: blink 1s infinite alternate;
    }

    @keyframes blink {
      from { opacity: 0.8; }
      to { opacity: 1; }
    }
  </style>

  <!-- Google Fonts for Orbitron -->
  <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700&display=swap" rel="stylesheet">
</head>
<body>

<video id="videoElement" autoplay muted></video>
<canvas id="overlay"></canvas>
<div class="hud">🧠Voice Detection Enabled...</div>

<!-- TensorFlow + COCO-SSD -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script>

<script>
  const video = document.getElementById('videoElement');
  const canvas = document.getElementById('overlay');
  const ctx = canvas.getContext('2d');

  let lastSpoken = {};
  const speakCooldown = 3000; // 3 seconds

  async function setupCamera() {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: { width: window.innerWidth, height: window.innerHeight },
      audio: false
    });
    video.srcObject = stream;

    return new Promise(resolve => {
      video.onloadedmetadata = () => resolve(video);
    });
  }

  function speak(text) {
    if ('speechSynthesis' in window) {
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.lang = 'en-US';
      window.speechSynthesis.cancel(); // Prevent overlap
      window.speechSynthesis.speak(utterance);
    }
  }

  async function main() {
    await setupCamera();
    video.play();

    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    const model = await cocoSsd.load();
    detectFrame(video, model);
  }

  async function detectFrame(video, model) {
    const predictions = await model.detect(video);
    renderPredictions(predictions);
    requestAnimationFrame(() => detectFrame(video, model));
  }

  function renderPredictions(predictions) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    const now = Date.now();

    predictions.forEach(pred => {
      const [x, y, width, height] = pred.bbox;

      // Draw bounding box
      ctx.beginPath();
      ctx.rect(x, y, width, height);
      ctx.lineWidth = 2;
      ctx.strokeStyle = '#00fff7';
      ctx.stroke();

      // Set bold and bigger font for label
      ctx.font = 'bold 20px Orbitron, sans-serif';
      ctx.fillStyle = '#00fff7';
      ctx.fillText(`${pred.class} - ${Math.round(pred.score * 100)}%`, x + 5, y - 10);

      // Voice feedback with cooldown
      if (!lastSpoken[pred.class] || now - lastSpoken[pred.class] > speakCooldown) {
        speak(pred.class);
        lastSpoken[pred.class] = now;
      }
    });
  }

  main();
</script>

</body>
</html>
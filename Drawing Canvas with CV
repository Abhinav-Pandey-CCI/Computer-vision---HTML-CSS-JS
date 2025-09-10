# Drawing-Canvas-with-CV

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>CV Canvas with Smart Speech</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      display: flex;
      height: 100vh;
      background: #111;
      color: white;
      font-family: sans-serif;
    }
    #canvas-container, #video-container {
      width: 50%;
      height: 100%;
      position: relative;
    }
    #videoElement {
      width: 100%;
      height: 100%;
      object-fit: cover;
      transform: scaleX(-1);
    }
    #drawingCanvas, #handCanvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
    }
    #drawingCanvas {
      background: white;
      cursor: crosshair;
    }
    .mode-label {
      position: absolute;
      top: 10px;
      left: 10px;
      background: rgba(0,0,0,0.5);
      padding: 8px 12px;
      border-radius: 8px;
      font-size: 18px;
      z-index: 10;
    }
  </style>
</head>
<body>
  <!-- Left: Drawing Area -->
  <div id="canvas-container">
    <div class="mode-label" id="modeLabel">Mode: Draw</div>
    <canvas id="drawingCanvas"></canvas>
  </div>

  <!-- Right: Camera Area -->
  <div id="video-container">
    <video id="videoElement" autoplay muted playsinline></video>
    <canvas id="handCanvas"></canvas>
  </div>

  <!-- MediaPipe -->
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.min.js"></script>

  <script>
    const videoElement = document.getElementById('videoElement');
    const drawCanvas = document.getElementById('drawingCanvas');
    const handCanvas = document.getElementById('handCanvas');
    const drawCtx = drawCanvas.getContext('2d');
    const handCtx = handCanvas.getContext('2d');
    const modeLabel = document.getElementById('modeLabel');

    drawCanvas.width = drawCanvas.offsetWidth;
    drawCanvas.height = drawCanvas.offsetHeight;
    handCanvas.width = handCanvas.offsetWidth;
    handCanvas.height = handCanvas.offsetHeight;

    let drawing = false;
    let eraseMode = false;
    let lastX = 0;
    let lastY = 0;
    let lastSpokenMode = "";

    // Speak function
    function speak(text) {
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.rate = 1;
      utterance.pitch = 1;
      speechSynthesis.speak(utterance);
    }

    // On load speak once
    window.onload = () => {
      speak("Computer vision based canvas started");
    };

    function draw(x, y) {
      if (!drawing) return;
      drawCtx.strokeStyle = eraseMode ? "white" : "black";
      drawCtx.lineWidth = eraseMode ? 20 : 3;
      drawCtx.lineCap = "round";
      drawCtx.beginPath();
      drawCtx.moveTo(lastX, lastY);
      drawCtx.lineTo(x, y);
      drawCtx.stroke();
      [lastX, lastY] = [x, y];
    }

    function countExtendedFingers(landmarks) {
      const tips = [8, 12, 16, 20];
      let count = 0;
      tips.forEach((tipIdx) => {
        if (landmarks[tipIdx].y < landmarks[tipIdx - 2].y) count++;
      });
      if (landmarks[4].x > landmarks[3].x) count++;
      return count;
    }

    function detectEraseMode(landmarks) {
      const index = landmarks[8];
      const middle = landmarks[12];
      const ring = landmarks[16];
      const distIM = Math.hypot(index.x - middle.x, index.y - middle.y);
      const distIR = Math.hypot(index.x - ring.x, index.y - ring.y);
      return distIM < 0.1 && distIR > 0.1;
    }

    function clearCanvas() {
      drawCtx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
    }

    function drawBoundingBox(landmarks, mode = "Draw") {
      let minX = 1, minY = 1, maxX = 0, maxY = 0;
      for (const point of landmarks) {
        if (point.x < minX) minX = point.x;
        if (point.y < minY) minY = point.y;
        if (point.x > maxX) maxX = point.x;
        if (point.y > maxY) maxY = point.y;
      }

      const width = handCanvas.width;
      const height = handCanvas.height;
      const x = (1 - maxX) * width;
      const y = minY * height;
      const w = (maxX - minX) * width;
      const h = (maxY - minY) * height;

      handCtx.clearRect(0, 0, width, height);

      handCtx.strokeStyle = '#00f0ff';
      handCtx.lineWidth = 3;
      handCtx.shadowColor = '#00f0ff';
      handCtx.shadowBlur = 20;
      handCtx.strokeRect(x, y, w, h);
      handCtx.shadowBlur = 0;

      handCtx.font = '18px Orbitron, sans-serif';
      handCtx.fillStyle = '#00f0ff';
      handCtx.shadowColor = '#00f0ff';
      handCtx.shadowBlur = 10;
      handCtx.textBaseline = "bottom";
      handCtx.fillText(mode, x + 8, y - 8);
      handCtx.shadowBlur = 0;
    }

    const hands = new Hands({
      locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`,
    });

    hands.setOptions({
      maxNumHands: 1,
      modelComplexity: 1,
      minDetectionConfidence: 0.7,
      minTrackingConfidence: 0.7,
    });

    hands.onResults((results) => {
      handCtx.clearRect(0, 0, handCanvas.width, handCanvas.height);

      if (results.multiHandLandmarks.length > 0) {
        const landmarks = results.multiHandLandmarks[0];
        const fingerCount = countExtendedFingers(landmarks);

        // Hand fully open = clear
        if (fingerCount === 5) {
          clearCanvas();
          drawBoundingBox(landmarks, "Cleared");

          if (lastSpokenMode !== "Cleared") {
            speak("Canvas cleared");
            lastSpokenMode = "Cleared";
          }

          drawing = false;
          return;
        }

        // Detect draw vs erase
        eraseMode = detectEraseMode(landmarks);
        const modeText = eraseMode ? "Eraser" : "Drawing";

        if (lastSpokenMode !== modeText) {
          speak(modeText);
          lastSpokenMode = modeText;
        }

        modeLabel.textContent = `Mode: ${modeText}`;
        drawBoundingBox(landmarks, modeText);

        const indexTip = landmarks[8];
        const x = (1 - indexTip.x) * drawCanvas.width;
        const y = indexTip.y * drawCanvas.height;

        draw(x, y);

        if (!drawing) {
          drawing = true;
          [lastX, lastY] = [x, y];
        }
      } else {
        drawing = false;
      }
    });

    const camera = new Camera(videoElement, {
      onFrame: async () => {
        await hands.send({ image: videoElement });
      },
      width: 640,
      height: 480,
    });

    camera.start();
  </script>
</body>
</html>

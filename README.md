html<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>AR Drawing</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { background: #000; overflow: hidden; font-family: sans-serif; }

  #container {
    position: relative;
    width: 100vw;
    height: 100vh;
  }

  video {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
  }

  #overlay {
    position: absolute;
    top: 0; left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
  }

  #panel {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    background: rgba(0,0,0,0.75);
    padding: 14px 16px 40px;
    display: flex;
    flex-direction: column;
    gap: 10px;
    backdrop-filter: blur(6px);
    z-index: 9999;
}

  .row {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  label {
    color: #fff;
    font-size: 13px;
    min-width: 90px;
  }

  input[type=range] {
    flex: 1;
    accent-color: #fff;
  }

  #uploadBtn {
    background: #fff;
    color: #000;
    border: none;
    border-radius: 10px;
    padding: 10px;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    width: 100%;
  }

  #hint {
    color: rgba(255,255,255,0.5);
    font-size: 12px;
    text-align: center;
  }

  #fileInput { display: none; }
</style>
</head>
<body>
<div id="container">
  <video id="video" autoplay playsinline muted></video>
  <canvas id="overlay"></canvas>

  <div id="panel">
    <button id="uploadBtn">📁 Загрузить изображение</button>
    <input type="file" id="fileInput" accept="image/*">

    <div class="row">
      <label>Прозрачность</label>
      <input type="range" id="opacity" min="0" max="100" value="50">
    </div>
    <div class="row">
      <label>Размер</label>
      <input type="range" id="scale" min="10" max="300" value="80">
    </div>
    <div id="hint">Перемещай изображение пальцем</div>
  </div>
</div>

<script>
  const video = document.getElementById('video');
  const canvas = document.getElementById('overlay');
  const ctx = canvas.getContext('2d');
  const opacitySlider = document.getElementById('opacity');
  const scaleSlider = document.getElementById('scale');
  const uploadBtn = document.getElementById('uploadBtn');
  const fileInput = document.getElementById('fileInput');

  let img = null;
  let imgX = 0, imgY = 0;
  let dragging = false;
  let dragStartX, dragStartY, imgStartX, imgStartY;

  // Камера
  async function startCamera() {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: { ideal: 'environment' }, width: { ideal: 1280 }, height: { ideal: 720 } }
      });
      video.srcObject = stream;
    } catch (e) {
      alert('Нет доступа к камере: ' + e.message);
    }
  }

  // Размер canvas
  function resize() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    if (!img) { imgX = canvas.width / 2; imgY = canvas.height / 2; }
  }

  window.addEventListener('resize', resize);
  resize();
  startCamera();

  // Загрузка изображения
  uploadBtn.addEventListener('click', () => fileInput.click());
  fileInput.addEventListener('change', e => {
    const file = e.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = ev => {
      const image = new Image();
      image.onload = () => {
        img = image;
        imgX = canvas.width / 2;
        imgY = (canvas.height - 200) / 2; // чуть выше панели
      };
      image.src = ev.target.result;
    };
    reader.readAsDataURL(file);
  });

// Drag (touch)
canvas.addEventListener('touchstart', e => {
    if (!img) return;
    e.preventDefault();
    dragging = true;
    dragStartX = e.touches[0].clientX;
    dragStartY = e.touches[0].clientY;
    imgStartX = imgX;
    imgStartY = imgY;
}, { passive: false });

canvas.addEventListener('touchmove', e => {
    if (!dragging) return;
    e.preventDefault();
    imgX = imgStartX + (e.touches[0].clientX - dragStartX);
    imgY = imgStartY + (e.touches[0].clientY - dragStartY);
}, { passive: false });

canvas.addEventListener('touchend', e => {
    e.preventDefault();
    dragging = false;
}, { passive: false });

// Drag (mouse для теста на ПК)
canvas.addEventListener('mousedown', e => {
    if (!img) return;
    dragging = true;
    dragStartX = e.clientX;
    dragStartY = e.clientY;
    imgStartX = imgX;
    imgStartY = imgY;
});
canvas.addEventListener('mousemove', e => {
    if (!dragging) return;
    imgX = imgStartX + (e.clientX - dragStartX);
    imgY = imgStartY + (e.clientY - dragStartY);
});
canvas.addEventListener('mouseup', () => dragging = false);

  // Рендер
  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (img) {
      const scale = scaleSlider.value / 100;
      const opacity = opacitySlider.value / 100;
      const w = img.naturalWidth * scale;
      const h = img.naturalHeight * scale;

      ctx.globalAlpha = opacity;
      ctx.drawImage(img, imgX - w / 2, imgY - h / 2, w, h);
      ctx.globalAlpha = 1;
    }

    requestAnimationFrame(draw);
  }

  draw();
</script>
</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Universal Mask Creator</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    #canvas { border: 1px solid #ccc; cursor: crosshair; }
    .controls { margin-bottom: 10px; }
    .class-item { margin: 8px 0; }
    label { display: inline-block; width: 100px; }
    input[type="range"] { width: 180px; }
    .rgb-group { display: flex; gap: 10px; align-items: center; }
  </style>
</head>
<body>

  <h2>🎨 Universal Mask Creator</h2>
  <p>Upload an image, define your classes with RGB values, and paint the mask.</p>

  <div class="controls">
    <input type="file" id="imageUpload" accept="image/*"><br><br>

    <label>Brush Size:</label>
    <input type="range" id="brushSize" min="1" max="50" value="10"><br><br>

    <div id="classControls">
      <h3>Define Classes</h3>
      <div id="classList"></div>
      <button onclick="addClass()">+ Add Class</button>
    </div>
  </div>

  <canvas id="canvas" width="512" height="512"></canvas><br>
  <button onclick="downloadMask()">💾 Download Mask</button>

  <script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const img = new Image();
    let currentColor = 'rgb(255,0,0)';
    let isDrawing = false;
    let originalFileName = '';

    // Hidden mask canvas
    const maskCanvas = document.createElement('canvas');
    const maskCtx = maskCanvas.getContext('2d');

    // Load image
    document.getElementById('imageUpload').addEventListener('change', function (e) {
      const file = e.target.files[0];
      if (!file) return;
      originalFileName = file.name.split('.')[0];
      const reader = new FileReader();
      reader.onload = function (event) {
        img.onload = function () {
          canvas.width = img.width;
          canvas.height = img.height;
          maskCanvas.width = img.width;
          maskCanvas.height = img.height;

          // Clear both
          ctx.clearRect(0, 0, canvas.width, canvas.height);
          maskCtx.fillStyle = 'black';
          maskCtx.fillRect(0, 0, maskCanvas.width, maskCanvas.height);

          // Draw image on visible canvas only
          ctx.drawImage(img, 0, 0);
          ctx.globalCompositeOperation = 'source-over';
        };
        img.src = event.target.result;
      };
      reader.readAsDataURL(file);
    });

    let classes = [];

    function rgbToString(r, g, b) {
      return `rgb(${r},${g},${b})`;
    }

    function addClass(name = '', r = 255, g = 0, b = 0) {
      const id = classes.length;
      const div = document.createElement('div');
      div.className = 'class-item';
      div.innerHTML = `
        <label>Class ${id}:</label>
        <input type="text" placeholder="Name" id="name${id}" value="${name}" style="width:120px;">
        <div class="rgb-group">
          R: <input type="range" min="0" max="250" id="r${id}" value="${r}" oninput="updateClass(${id})">
          G: <input type="range" min="0" max="250" id="g${id}" value="${g}" oninput="updateClass(${id})">
          B: <input type="range" min="0" max="250" id="b${id}" value="${b}" oninput="updateClass(${id})">
        </div>
        <button onclick="selectClass(${id})">Use</button>
        <span id="preview${id}" style="margin-left:10px;padding:4px 12px;border:1px solid #ccc;">Color</span>
      `;
      document.getElementById('classList').appendChild(div);
      classes.push({ name, r, g, b });
      updateClass(id);
    }

    function updateClass(id) {
      const r = document.getElementById(`r${id}`).value;
      const g = document.getElementById(`g${id}`).value;
      const b = document.getElementById(`b${id}`).value;
      const color = rgbToString(r, g, b);
      document.getElementById(`preview${id}`).style.backgroundColor = color;
    }

    function selectClass(id) {
      const name = document.getElementById(`name${id}`).value || `class_${id}`;
      const r = document.getElementById(`r${id}`).value;
      const g = document.getElementById(`g${id}`).value;
      const b = document.getElementById(`b${id}`).value;
      currentColor = rgbToString(r, g, b);
      alert(`Selected: ${name} (${currentColor})`);
    }

    // Add default classes
    addClass('background', 0, 0, 0);
    addClass('class_1', 255, 0, 0);
    addClass('class_2', 0, 255, 0);
    addClass('class_3', 0, 0, 255);

    // Drawing logic: draw on both canvases
    canvas.addEventListener('mousedown', () => isDrawing = true);
    canvas.addEventListener('mouseup', () => isDrawing = false);
    canvas.addEventListener('mousemove', draw);

    function draw(e) {
      if (!isDrawing) return;
      const rect = canvas.getBoundingClientRect();
      const x = e.clientX - rect.left;
      const y = e.clientY - rect.top;
      const size = document.getElementById('brushSize').value;

      // Draw on visible canvas (image + color)
      ctx.globalCompositeOperation = 'source-over';
      ctx.fillStyle = currentColor;
      ctx.beginPath();
      ctx.arc(x, y, size, 0, Math.PI * 2);
      ctx.fill();

      // Draw on hidden mask canvas (color only)
      maskCtx.fillStyle = currentColor;
      maskCtx.beginPath();
      maskCtx.arc(x, y, size, 0, Math.PI * 2);
      maskCtx.fill();
    }

    // Download only the mask canvas (no image)
    function downloadMask() {
      const link = document.createElement('a');
      link.download = `mask_${originalFileName || 'untitled'}.png`;
      link.href = maskCanvas.toDataURL();
      link.click();
    }
  </script>

</body>
</html>

<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <title>Шахед-симулятор</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      background-color: #222;
    }
    #battlefield {
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      background: linear-gradient(to right, red 50%, blue 50%);
      cursor: crosshair;
    }
    .city {
      position: absolute;
      width: 30px;
      height: 30px;
      background-color: yellow;
      border: 2px solid #333;
      border-radius: 4px;
    }
    .drone {
      position: absolute;
      width: 60px;
      height: 60px;
      background: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADgAAAA4CAMAAAAo0r9XAAAAllBMVEUAAAD///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////9zqELgAAAAMHRSTlMAAQIDBAUGBwgJCgsMDQ4PEBESExQVFhcYGRobHB0eHyAhIiMkJSYnKC0uLzFJY20AAABoSURBVEjH7dSxDoAgDIXhIh4s+f//Z6OQCg6SKaHVXtT3WxrS3rAfw2IRAhISiW0BQzMJjVm0jQXnyl68INaK1e0aJ9+9ivXuJ6HrfhtV6HU+7zFbX8gXxva5L5yl36u5R+W6PCFyaPNdfCmAAAAAElFTkSuQmCC") no-repeat center/contain;
      pointer-events: none;
    }
    .explosion {
      position: absolute;
      width: 60px;
      height: 60px;
      background: orange;
      border-radius: 50%;
      opacity: 0.8;
      animation: explode 0.5s ease-out;
      pointer-events: none;
    }
    @keyframes explode {
      0% { transform: scale(0.5); opacity: 1; }
      100% { transform: scale(2); opacity: 0; }
    }
    #startBtn {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      padding: 20px 40px;
      font-size: 24px;
      background-color: #fff;
      border: none;
      cursor: pointer;
      z-index: 10;
    }
  </style>
</head>
<body>
  <button id="startBtn">Начать бой</button>
  <div id="battlefield" style="display:none;"></div>

  <script>
    const battlefield = document.getElementById('battlefield');
    const startBtn = document.getElementById('startBtn');

    startBtn.onclick = () => {
      startBtn.style.display = 'none';
      battlefield.style.display = 'block';
      spawnCities();
    };

    function spawnCities() {
      for (let i = 0; i < 10; i++) {
        const city = document.createElement('div');
        city.className = 'city';
        const x = Math.random() * (window.innerWidth - 30);
        const y = Math.random() * (window.innerHeight - 30);
        city.style.left = x + 'px';
        city.style.top = y + 'px';
        battlefield.appendChild(city);
      }
    }

    battlefield.onclick = (e) => {
      const drone = document.createElement('div');
      drone.className = 'drone';
      drone.style.left = '50%';
      drone.style.top = '100%';
      battlefield.appendChild(drone);

      const targetX = e.clientX;
      const targetY = e.clientY;

      const duration = 1000;
      const startX = window.innerWidth / 2;
      const startY = window.innerHeight;
      const deltaX = targetX - startX;
      const deltaY = targetY - startY;
      const startTime = performance.now();

      function animate(time) {
        const elapsed = time - startTime;
        const progress = Math.min(elapsed / duration, 1);
        const currentX = startX + deltaX * progress;
        const currentY = startY + deltaY * progress;
        drone.style.left = currentX + 'px';
        drone.style.top = currentY + 'px';

        if (progress < 1) {
          requestAnimationFrame(animate);
        } else {
          battlefield.removeChild(drone);
          const explosion = document.createElement('div');
          explosion.className = 'explosion';
          explosion.style.left = (targetX - 30) + 'px';
          explosion.style.top = (targetY - 30) + 'px';
          battlefield.appendChild(explosion);
          setTimeout(() => battlefield.removeChild(explosion), 500);
        }
      }

      requestAnimationFrame(animate);
    };
  </script>
</body>
</html>

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
      height: 100%;
      font-family: Arial, sans-serif;
      color: #fff;
    }
    #battlefield {
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      background: linear-gradient(to right, red 50%, blue 50%);
      cursor: default;
      padding-bottom: 100px; /* отступ для меню */
      box-sizing: border-box;
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
      background-size: contain;
      background-repeat: no-repeat;
      background-position: center;
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
      color: #000;
      border-radius: 8px;
    }

    /* Центрированное меню оружия снизу */
    #weaponPanel {
      position: fixed;
      bottom: 20px; /* немного отступ сверху от нижнего края */
      left: 50%;
      transform: translateX(-50%);
      background-color: #111;
      border: 2px solid #444;
      border-radius: 12px;
      padding: 10px 20px;
      display: flex;
      align-items: center;
      gap: 20px;
      box-sizing: border-box;
      z-index: 20;
      user-select: none;
      width: 160px; /* узкая по ширине */
      justify-content: center;
    }
    .weapon-btn {
      background-color: #333;
      border: 2px solid #555;
      padding: 6px;
      border-radius: 8px;
      cursor: pointer;
      transition: background-color 0.3s, border-color 0.3s;
      width: 60px;
      height: 60px;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .weapon-btn.selected {
      background-color: orange;
      border-color: yellow;
    }
    .weapon-btn img {
      max-width: 50px;
      max-height: 50px;
      pointer-events: none;
      user-select: none;
    }
  </style>
</head>
<body>
  <button id="startBtn">Начать бой</button>
  <div id="battlefield" style="display:none;"></div>

  <!-- Узкое меню оружия по центру снизу -->
  <div id="weaponPanel" style="display:none;">
    <div class="weapon-btn selected" data-weapon="shahed" title="Шахед">
      <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/f7/Drone_icon.svg/1024px-Drone_icon.svg.png" alt="Шахед" />
    </div>
    <!-- Можно добавить другие оружия здесь -->
  </div>

  <script>
    const battlefield = document.getElementById('battlefield');
    const startBtn = document.getElementById('startBtn');
    const weaponPanel = document.getElementById('weaponPanel');
    const weaponButtons = document.querySelectorAll('.weapon-btn');

    let selectedWeapon = 'shahed';

    weaponButtons.forEach(btn => {
      btn.addEventListener('click', () => {
        weaponButtons.forEach(b => b.classList.remove('selected'));
        btn.classList.add('selected');
        selectedWeapon = btn.dataset.weapon;
      });
    });

    startBtn.onclick = () => {
      startBtn.style.display = 'none';
      battlefield.style.display = 'block';
      weaponPanel.style.display = 'flex';
      spawnCities();
    };

    function spawnCities() {
      for (let i = 0; i < 10; i++) {
        const city = document.createElement('div');
        city.className = 'city';
        const x = Math.random() * (window.innerWidth - 30);
        const y = Math.random() * (window.innerHeight - 30 - 100); // не заходить под меню
        city.style.left = x + 'px';
        city.style.top = y + 'px';
        battlefield.appendChild(city);
      }
    }

    battlefield.onclick = (e) => {
      // Игнорируем клики по меню
      if (e.clientY > window.innerHeight - 100) return;

      if (selectedWeapon === 'shahed') {
        const drone = document.createElement('div');
        drone.className = 'drone';
        drone.style.backgroundImage = "url('https://upload.wikimedia.org/wikipedia/commons/thumb/f/f7/Drone_icon.svg/1024px-Drone_icon.svg.png')";

        const startX = window.innerWidth / 2;
        const startY = window.innerHeight - 100;

        drone.style.left = (startX - 30) + 'px';
        drone.style.top = startY + 'px';
        battlefield.appendChild(drone);

        const targetX = e.clientX;
        const targetY = e.clientY;

        const duration = 1000;
        const deltaX = targetX - startX;
        const deltaY = targetY - startY;
        const startTime = performance.now();

        function animate(time) {
          const elapsed = time - startTime;
          const progress = Math.min(elapsed / duration, 1);
          const currentX = startX + deltaX * progress;
          const currentY = startY + deltaY * progress;
          drone.style.left = (currentX - 30) + 'px';
          drone.style.top = (currentY - 30) + 'px';

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
      }
    };
  </script>
</body>
</html>

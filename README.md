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
      cursor: default; /* обычный курсор */
      overflow: hidden;
      padding-bottom: 80px; /* место для панели оружия */
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
      /* Фотка шахеда */
      background-image: url('https://upload.wikimedia.org/wikipedia/commons/thumb/f/f7/Drone_icon.svg/1024px-Drone_icon.svg.png');
      /* можно заменить на любую другую ссылку с прозрачным фоном */
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

    /* Панель выбора оружия снизу */
    #weaponPanel {
      position: fixed;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 80px;
      background-color: #111;
      border-top: 2px solid #444;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 20px;
      box-sizing: border-box;
      z-index: 20;
      color: white;
      user-select: none;
    }
    .weapon-btn {
      background-color: #333;
      border: 2px solid #555;
      padding: 10px 20px;
      border-radius: 6px;
      cursor: pointer;
      font-weight: bold;
      transition: background-color 0.3s, border-color 0.3s;
    }
    .weapon-btn.selected {
      background-color: orange;
      border-color: yellow;
      color: black;
    }
  </style>
</head>
<body>
  <button id="startBtn">Начать бой</button>
  <div id="battlefield" style="display:none;"></div>

  <!-- Панель оружия -->
  <div id="weaponPanel" style="display:none;">
    <div class="weapon-btn selected" data-weapon="shahed">Шахед</div>
    <!-- Можно добавить другие оружия -->
  </div>

  <script>
    const battlefield = document.getElementById('battlefield');
    const startBtn = document.getElementById('startBtn');
    const weaponPanel = document.getElementById('weaponPanel');
    const weaponButtons = document.querySelectorAll('.weapon-btn');

    let selectedWeapon = 'shahed'; // текущий выбранный "шахед"

    // Обработка выбора оружия (для расширения)
    weaponButtons.forEach(btn => {
      btn.addEventListener('click', () => {
        weaponButtons.forEach(b => b.classList.remove('selected'));
        btn.classList.add('selected');
        selectedWeapon = btn.dataset.weapon;
        // Здесь можно добавить логику для смены оружия
        // Пока у нас только шахед, так что это просто визуально
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
        const y = Math.random() * (window.innerHeight - 30 - 80); // не заезжать на панель оружия
        city.style.left = x + 'px';
        city.style.top = y + 'px';
        battlefield.appendChild(city);
      }
    }

    battlefield.onclick = (e) => {
      // Проверяем, что клик внутри battlefield, но не на панели оружия
      if (e.clientY > window.innerHeight - 80) return;

      if (selectedWeapon === 'shahed') {
        const drone = document.createElement('div');
        drone.className = 'drone';

        // старт снизу по центру
        const startX = window.innerWidth / 2;
        const startY = window.innerHeight - 80; // чуть выше панели оружия

        drone.style.left = (startX - 30) + 'px'; // -30 чтобы центрировать по ширине (60/2)
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
      // Можно добавить else if для других оружий
    };
  </script>
</body>
</html>

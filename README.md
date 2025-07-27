<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <title>Шахед-симулятор</title>
  <style>
    html, body {
      margin: 0; padding: 0; overflow: hidden;
      background-color: #222;
      height: 100%;
      font-family: Arial, sans-serif;
      color: #fff;
    }
    #battlefield {
      position: fixed;
      top: 0; left: 0;
      width: 100vw; height: 100vh;
      background: linear-gradient(to right, blue 50%, red 50%);
      cursor: default;
      padding-bottom: 100px; /* место для меню */
      box-sizing: border-box;
    }
    .city {
      position: absolute;
      width: 30px; height: 30px;
      background-color: yellow;
      border: 2px solid #333;
      border-radius: 4px;
    }
    .drone {
      position: absolute;
      width: 60px; height: 60px;
      background-size: contain;
      background-repeat: no-repeat;
      background-position: center;
      pointer-events: none;
    }
    .explosion {
      position: absolute;
      width: 60px; height: 60px;
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
      top: 50%; left: 50%;
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
    #weaponPanel {
      position: fixed;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      background-color: #111;
      border: 2px solid #444;
      border-radius: 12px;
      padding: 10px 20px;
      display: flex;
      align-items: center;
      gap: 40px;
      box-sizing: border-box;
      z-index: 20;
      user-select: none;
      width: auto;
      justify-content: center;
    }
    .weapon-btn {
      background-color: #333;
      border: 2px solid #555;
      padding: 6px 10px 30px 10px;
      border-radius: 8px;
      cursor: pointer;
      transition: background-color 0.3s, border-color 0.3s;
      width: 80px;
      height: 80px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      position: relative;
      color: white;
      font-weight: bold;
      font-size: 14px;
      text-align: center;
      user-select: none;
    }
    .weapon-btn.selected {
      background-color: orange;
      border-color: yellow;
      color: black;
    }
    .weapon-btn img {
      max-width: 60px;
      max-height: 60px;
      pointer-events: none;
      user-select: none;
      margin-bottom: 4px;
      flex-shrink: 0;
      object-fit: contain;
    }
    .weapon-btn .cooldown {
      position: absolute;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.6);
      border-radius: 8px;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 20px;
      font-weight: bold;
      color: yellow;
      user-select: none;
    }
  </style>
</head>
<body>
  <button id="startBtn">Начать бой</button>
  <div id="battlefield" style="display:none;"></div>

  <div id="weaponPanel" style="display:none;">
    <div class="weapon-btn selected" data-weapon="shahed" title="Шахед">
      <img src="https://encrypted-tbn3.gstatic.com/images?q=tbn:ANd9GcTR1ApeuEnozjK1xuvfmWoX4eY6gkg_5JuPvG02npFa_1Hs9SDl" alt="Шахед" />
      Шахед
    </div>
    <div class="weapon-btn" data-weapon="rocket" title="Ракета">
      <img src="https://cdn-icons-png.flaticon.com/512/622/622669.png" alt="Ракета" />
      Ракета
    </div>
  </div>

  <script>
    const battlefield = document.getElementById('battlefield');
    const startBtn = document.getElementById('startBtn');
    const weaponPanel = document.getElementById('weaponPanel');
    const weaponButtons = document.querySelectorAll('.weapon-btn');

    let selectedWeapon = 'shahed';
    let canShoot = true;

    weaponButtons.forEach(btn => {
      btn.addEventListener('click', () => {
        if (!canShoot) return;
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
        const y = Math.random() * (window.innerHeight - 30 - 100);
        city.style.left = x + 'px';
        city.style.top = y + 'px';
        battlefield.appendChild(city);
      }
    }

    function setCooldown(btn, seconds) {
      canShoot = false;
      const cdElem = document.createElement('div');
      cdElem.className = 'cooldown';
      cdElem.textContent = seconds;
      btn.appendChild(cdElem);

      let timeLeft = seconds;
      const interval = setInterval(() => {
        timeLeft--;
        if (timeLeft > 0) {
          cdElem.textContent = timeLeft;
        } else {
          clearInterval(interval);
          btn.removeChild(cdElem);
          canShoot = true;
        }
      }, 1000);
    }

    battlefield.onclick = (e) => {
      if (e.clientY > window.innerHeight - 100) return; // не запускать с меню
      if (!canShoot) return;

      const btn = document.querySelector('.weapon-btn.selected');

      // Старт с центра синей части (1/4 ширины, по центру высоты)
      const startX = window.innerWidth / 4;
      const startY = window.innerHeight / 2;

      const targetX = e.clientX;
      const targetY = e.clientY;

      const duration = 2000;
      const deltaX = targetX - startX;
      const deltaY = targetY - startY;

      const drone = document.createElement('div');
      drone.className = 'drone';

      if (selectedWeapon === 'shahed') {
        drone.style.backgroundImage = `url("https://encrypted-tbn3.gstatic.com/images?q=tbn:ANd9GcTR1ApeuEnozjK1xuvfmWoX4eY6gkg_5JuPvG02npFa_1Hs9SDl")`;
      } else if (selectedWeapon === 'rocket') {
        drone.style.backgroundImage = `url("https://cdn-icons-png.flaticon.com/512/622/622669.png")`;
      }

      drone.style.left = (startX - 30) + 'px';
      drone.style.top = (startY - 30) + 'px';

      battlefield.appendChild(drone);

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

      setCooldown(btn, 3);
    };
  </script>
</body>
</html>

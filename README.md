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
      background: linear-gradient(to right, red 50%, blue 50%);
      cursor: default;
      padding-bottom: 140px; /* место для меню и поля ввода */
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
      bottom: 60px; /* под меню ввода */
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
      width: 160px;
      justify-content: center;
    }
    .weapon-btn {
      background-color: #333;
      border: 2px solid #555;
      padding: 6px;
      border-radius: 8px;
      cursor: pointer;
      transition: background-color 0.3s, border-color 0.3s;
      width: 60px; height: 60px;
      display: flex;
      align-items: center;
      justify-content: center;
      position: relative;
      color: white;
      font-weight: bold;
      font-size: 12px;
      text-align: center;
    }
    .weapon-btn.selected {
      background-color: orange;
      border-color: yellow;
      color: black;
    }
    .weapon-btn img {
      max-width: 50px;
      max-height: 50px;
      pointer-events: none;
      user-select: none;
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
    #customUrlPanel {
      position: fixed;
      bottom: 10px;
      left: 50%;
      transform: translateX(-50%);
      background: #111;
      border: 2px solid #444;
      border-radius: 12px;
      padding: 10px 20px;
      width: 320px;
      display: flex;
      gap: 10px;
      box-sizing: border-box;
      z-index: 25;
    }
    #customUrlPanel input {
      flex-grow: 1;
      padding: 6px 8px;
      border-radius: 6px;
      border: 1px solid #555;
      font-size: 14px;
      background: #222;
      color: #fff;
      outline: none;
    }
    #customUrlPanel button {
      padding: 6px 12px;
      border-radius: 6px;
      border: none;
      background: orange;
      color: black;
      font-weight: bold;
      cursor: pointer;
      transition: background-color 0.3s;
    }
    #customUrlPanel button:hover {
      background: darkorange;
    }
  </style>
</head>
<body>
  <button id="startBtn">Начать бой</button>
  <div id="battlefield" style="display:none;"></div>

  <div id="weaponPanel" style="display:none;">
    <div class="weapon-btn selected" data-weapon="shahed" title="Шахед">
      <img id="weaponImg" src="https://i.postimg.cc/Z5tWNQ9Y/shahed.png" alt="Шахед" />
    </div>
  </div>

  <!-- Поле для вставки URL картинки шахеда -->
  <div id="customUrlPanel" style="display:none;">
    <input type="text" id="urlInput" placeholder="Вставьте URL картинки шахеда" />
    <button id="urlBtn">Обновить</button>
  </div>

  <script>
    const battlefield = document.getElementById('battlefield');
    const startBtn = document.getElementById('startBtn');
    const weaponPanel = document.getElementById('weaponPanel');
    const weaponButtons = document.querySelectorAll('.weapon-btn');
    const customUrlPanel = document.getElementById('customUrlPanel');
    const urlInput = document.getElementById('urlInput');
    const urlBtn = document.getElementById('urlBtn');
    const weaponImg = document.getElementById('weaponImg');

    let selectedWeapon = 'shahed';
    let canShoot = true;
    let currentDroneImg = weaponImg.src; // текущая картинка шахеда

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
      customUrlPanel.style.display = 'flex';
      spawnCities();
    };

    function spawnCities() {
      for (let i = 0; i < 10; i++) {
        const city = document.createElement('div');
        city.className = 'city';
        const x = Math.random() * (window.innerWidth - 30);
        const y = Math.random() * (window.innerHeight - 30 - 140);
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
      if (e.clientY > window.innerHeight - 140) return;
      if (!canShoot) return;

      if (selectedWeapon === 'shahed') {
        const drone = document.createElement('div');
        drone.className = 'drone';
        drone.style.backgroundImage = `url('${currentDroneImg}')`;

        const startX = window.innerWidth / 2;
        const startY = window.innerHeight - 140;

        // Центрируем картинку — сдвигаем на половину ширины и высоты
        drone.style.left = (startX - 30) + 'px';
        drone.style.top = (startY - 30) + 'px';
        battlefield.appendChild(drone);

        const targetX = e.clientX;
        const targetY = e.clientY;

        const duration = 2000; // 2 секунды!
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

        const btn = document.querySelector('.weapon-btn.selected');
        setCooldown(btn, 3);
      }
    };

    // Обновляем картинку шахеда при вводе URL
    urlBtn.onclick = () => {
      const url = urlInput.value.trim();
      if (!url) {
        alert('Введите URL картинки!');
        return;
      }
      // Проверим что url валидный простой (без сложных проверок)
      if (!/^https?:\/\/.+\.(png|jpg|jpeg|svg|gif)$/i.test(url)) {
        alert('Пожалуйста, введите URL, оканчивающийся на png, jpg, jpeg, svg или gif');
        return;
      }

      currentDroneImg = url;
      weaponImg.src = url;
      urlInput.value = '';
    };
  </script>
</body>
</html>

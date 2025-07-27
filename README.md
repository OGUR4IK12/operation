<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Операция: Контрнаступ — с дронами</title>
<style>
  body {
    font-family: Arial, sans-serif;
    background: linear-gradient(135deg, #223366 0%, #551111 100%);
    color: #eee;
    text-align: center;
    user-select: none;
  }
  #game {
    margin: 20px auto;
    width: 800px;
    height: 400px;
    position: relative;
    border: 3px solid #ccc;
    display: flex;
    box-shadow: 0 0 20px #444;
  }
  .zone {
    flex: 1;
    position: relative;
    border: 2px solid #333;
  }
  .zone.red {
    background: linear-gradient(45deg, #ff4444, #aa0000);
    filter: drop-shadow(0 0 10px #ff2222);
  }
  .zone.blue {
    background: linear-gradient(45deg, #4499ff, #0033aa);
    filter: drop-shadow(0 0 10px #3399ff);
  }
  .city {
    width: 32px;
    height: 32px;
    border-radius: 50%;
    background: yellow;
    border: 2px solid #444;
    position: absolute;
    cursor: default;
    transition: background-color 0.3s;
    box-shadow: 0 0 8px 2px #ffff55;
  }
  .city.captured-blue {
    background: #66f;
    border-color: #224;
    box-shadow: 0 0 15px 5px #66aaff;
  }
  .city.captured-red {
    background: #f66;
    border-color: #422;
    box-shadow: 0 0 15px 5px #ff6666;
  }
  #controls {
    margin-top: 15px;
  }
  button {
    background: #222;
    border: none;
    color: #eee;
    padding: 12px 30px;
    margin: 0 10px;
    font-size: 18px;
    cursor: pointer;
    border-radius: 8px;
    box-shadow: 0 0 10px #555 inset;
    transition: background 0.3s;
  }
  button:hover {
    background: #444;
  }
  #log {
    margin-top: 15px;
    min-height: 50px;
    font-size: 16px;
    color: #eee;
    font-weight: bold;
    min-height: 50px;
  }
  #capturedCount {
    font-weight: bold;
    font-size: 20px;
    margin-top: 10px;
  }
  /* Дрон */
  #drone {
    position: absolute;
    width: 40px;
    height: 40px;
    background: url('https://i.imgur.com/o9PvD5Y.png') no-repeat center center;
    background-size: contain;
    pointer-events: none;
    display: none;
    z-index: 100;
    filter: drop-shadow(0 0 5px #00ffff);
  }
  /* Взрыв */
  #explosion {
    position: absolute;
    width: 60px;
    height: 60px;
    background: url('https://i.imgur.com/BlB1yHo.png') no-repeat center center;
    background-size: contain;
    pointer-events: none;
    display: none;
    z-index: 101;
    animation: explodeAnim 0.5s forwards;
  }
  @keyframes explodeAnim {
    0% { transform: scale(0.5); opacity: 1; }
    100% { transform: scale(2); opacity: 0; }
  }
</style>
</head>
<body>

<h1>Операция: Контрнаступ</h1>

<div id="game">
  <div id="redZone" class="zone red"></div>
  <div id="blueZone" class="zone blue"></div>
  <div id="drone"></div>
  <div id="explosion"></div>
</div>

<div id="controls">
  <button id="startBattleBtn">Начать бой</button>
  <span style="font-size:18px; margin-left: 20px;">
    Захвачено городов: <span id="capturedCount">0</span>
  </span>
</div>

<div id="log"></div>

<script>
  const redZone = document.getElementById('redZone');
  const blueZone = document.getElementById('blueZone');
  const startBattleBtn = document.getElementById('startBattleBtn');
  const capturedCountSpan = document.getElementById('capturedCount');
  const log = document.getElementById('log');
  const drone = document.getElementById('drone');
  const explosion = document.getElementById('explosion');

  // Города (координаты в зонах)
  const redCities = [
    {id: 1, x: 60, y: 70, captured: false},
    {id: 2, x: 200, y: 130, captured: false},
    {id: 3, x: 130, y: 250, captured: false},
    {id: 7, x: 250, y: 320, captured: false},
    {id: 8, x: 50, y: 350, captured: false},
  ];
  const blueCities = [
    {id: 4, x: 70, y: 60, captured: true},
    {id: 5, x: 180, y: 150, captured: true},
    {id: 6, x: 90, y: 280, captured: true},
    {id: 9, x: 210, y: 300, captured: true},
    {id: 10, x: 30, y: 320, captured: true},
  ];

  let battleStarted = false;

  // Рисуем города
  function drawCities() {
    redZone.innerHTML = '';
    blueZone.innerHTML = '';
    redCities.forEach(city => {
      const div = document.createElement('div');
      div.classList.add('city');
      if(city.captured) div.classList.add('captured-blue');
      else div.classList.add('captured-red');
      div.style.left = city.x + 'px';
      div.style.top = city.y + 'px';
      div.title = `Красная зона — Город #${city.id} (${city.captured ? 'Захвачен' : 'Свободен'})`;
      redZone.appendChild(div);
    });
    blueCities.forEach(city => {
      const div = document.createElement('div');
      div.classList.add('city');
      if(city.captured) div.classList.add('captured-blue');
      else div.classList.add('captured-red');
      div.style.left = city.x + 'px';
      div.style.top = city.y + 'px';
      div.title = `Синяя зона — Город #${city.id} (${city.captured ? 'Захвачен' : 'Свободен'})`;
      blueZone.appendChild(div);
    });
  }

  function countCaptured() {
    return redCities.filter(c => c.captured).length;
  }

  // Анимация дрона — летит от центра синей зоны к клику в красной зоне
  function flyDroneTo(x, y) {
    const gameRect = document.getElementById('game').getBoundingClientRect();
    const blueRect = blueZone.getBoundingClientRect();

    // Начальная позиция — центр синей зоны относительно страницы
    let startX = blueRect.left + blueRect.width / 2 - gameRect.left - drone.offsetWidth/2;
    let startY = blueRect.top + blueRect.height / 2 - gameRect.top - drone.offsetHeight/2;

    drone.style.left = startX + 'px';
    drone.style.top = startY + 'px';
    drone.style.display = 'block';

    // Целевая позиция — там, где кликнули в красной зоне (относительно game)
    let targetX = x - drone.offsetWidth/2;
    let targetY = y - drone.offsetHeight/2;

    const duration = 1500; // время полета 1.5 секунды
    const startTime = performance.now();

    function animate(time) {
      let elapsed = time - startTime;
      let progress = Math.min(elapsed / duration, 1);

      let currentX = startX + (targetX - startX) * progress;
      let currentY = startY + (targetY - startY) * progress;

      drone.style.left = currentX + 'px';
      drone.style.top = currentY + 'px';

      if (progress < 1) {
        requestAnimationFrame(animate);
      } else {
        drone.style.display = 'none';
        showExplosion(targetX + drone.offsetWidth/2, targetY + drone.offsetHeight/2);
        processAttack(targetX + drone.offsetWidth/2, targetY + drone.offsetHeight/2);
      }
    }
    requestAnimationFrame(animate);
  }

  // Показываем анимацию взрыва в координатах
  function showExplosion(x, y) {
    explosion.style.left = (x - explosion.offsetWidth / 2) + 'px';
    explosion.style.top = (y - explosion.offsetHeight / 2) + 'px';
    explosion.style.display = 'block';
    explosion.style.animation = 'explodeAnim 0.5s forwards';

    setTimeout(() => {
      explosion.style.display = 'none';
    }, 500);
  }

  // Обработка атаки: если клик близко к городу, он захватывается
  function processAttack(x, y) {
    // Координаты клика относительно зоны redZone
    const rect = redZone.getBoundingClientRect();
    let localX = x - rect.left;
    let localY = y - rect.top;

    // Ищем ближайший город в радиусе 40px
    let attackedCity = null;
    for(let city of redCities) {
      if (!city.captured) {
        let dx = city.x + 16 - localX;
        let dy = city.y + 16 - localY;
        let dist = Math.sqrt(dx*dx + dy*dy);
        if (dist < 40) {
          attackedCity = city;
          break;
        }
      }
    }
    if(attackedCity) {
      attackedCity.captured = true;
      log.textContent = `Город #${attackedCity.id} захвачен!`;
    } else {
      log.textContent = 'Мимо! Город не поврежден.';
    }
    updateCapturedCount();
    drawCities();
  }

  // Обновление счетчика захваченных городов
  function updateCapturedCount() {
    let count = countCaptured();
    capturedCountSpan.textContent = count;
  }

  // Кнопка "Начать бой"
  startBattleBtn.onclick = () => {
    if(battleStarted) {
      alert('Бой уже идет!');
      return;
    }
    battleStarted = true;
    log.textContent = 'Бой начался! Кликайте по красной зоне, чтобы запустить шахед.';
    startBattleBtn.disabled = true;
  };

  // Обработка клика в красной зоне — запуск дрона
  redZone.addEventListener('click', (e) => {
    if(!battleStarted) {
      alert('Нажмите "Начать бой" для запуска атаки.');
      return;
    }
    const rect = redZone.getBoundingClientRect();
    const clickX = e.clientX - rect.left;
    const clickY = e.clientY - rect.top;

    flyDroneTo(e.clientX - document.getElementById('game').getBoundingClientRect().left, 
               e.clientY - document.getElementById('game').getBoundingClientRect().top);
  });

  // Начальная отрисовка
  drawCities();
  updateCapturedCount();
</script>

</body>
</html>

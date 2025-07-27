<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Операция: Контрнаступ</title>
<style>
  body { font-family: Arial, sans-serif; background: #222; color: #eee; text-align: center; }
  #game {
    margin: 20px auto; 
    width: 600px; 
    height: 300px; 
    position: relative; 
    border: 2px solid #555;
    display: flex;
  }
  .zone {
    flex: 1;
    position: relative;
  }
  .zone.red { background: #a33; }
  .zone.blue { background: #3366cc; }
  .city {
    width: 24px; height: 24px;
    border-radius: 50%;
    background: yellow;
    position: absolute;
    cursor: pointer;
    border: 2px solid #444;
    transition: background-color 0.3s;
  }
  .city.captured-blue { background: #66f; border-color: #224; }
  .city.captured-red { background: #f66; border-color: #422; }
  #controls { margin-top: 15px; }
  button {
    background: #444; 
    border: none; 
    color: #eee; 
    padding: 10px 20px; 
    margin: 0 10px; 
    font-size: 16px; 
    cursor: pointer;
    border-radius: 6px;
  }
  button:hover { background: #666; }
  #log {
    margin-top: 15px;
    min-height: 40px;
    font-size: 14px;
    color: #ddd;
  }
</style>
</head>
<body>

<h1>Операция: Контрнаступ</h1>

<div id="game">
  <div id="redZone" class="zone red"></div>
  <div id="blueZone" class="zone blue"></div>
</div>

<div id="controls">
  <button id="launchRocketBtn">Запустить ракету по красной зоне</button>
  <button id="nextTurnBtn" disabled>Ход врага</button>
</div>

<div id="log"></div>
<div>Захвачено городов: <span id="capturedCount">0</span></div>

<script>
  const redZone = document.getElementById('redZone');
  const blueZone = document.getElementById('blueZone');
  const launchBtn = document.getElementById('launchRocketBtn');
  const nextTurnBtn = document.getElementById('nextTurnBtn');
  const log = document.getElementById('log');
  const capturedCountSpan = document.getElementById('capturedCount');

  // Города — координаты внутри зон
  const redCities = [
    {id: 1, x: 50, y: 50, captured: false},
    {id: 2, x: 150, y: 100, captured: false},
    {id: 3, x: 80, y: 180, captured: false}
  ];
  const blueCities = [
    {id: 4, x: 50, y: 70, captured: true},
    {id: 5, x: 130, y: 140, captured: true},
    {id: 6, x: 90, y: 220, captured: true}
  ];

  // Функция для отрисовки городов
  function drawCities() {
    redZone.innerHTML = '';
    blueZone.innerHTML = '';
    redCities.forEach(city => {
      const div = document.createElement('div');
      div.classList.add('city');
      if(city.captured) div.classList.add('captured-blue');
      div.style.left = city.x + 'px';
      div.style.top = city.y + 'px';
      div.title = 'Красная страна - Город #' + city.id;
      redZone.appendChild(div);
    });
    blueCities.forEach(city => {
      const div = document.createElement('div');
      div.classList.add('city');
      if(city.captured) div.classList.add('captured-blue');
      else div.classList.add('captured-red');
      div.style.left = city.x + 'px';
      div.style.top = city.y + 'px';
      div.title = 'Синяя страна - Город #' + city.id;
      blueZone.appendChild(div);
    });
  }

  // Подсчет захваченных городов (синих)
  function countCaptured() {
    return redCities.filter(c => c.captured).length + blueCities.filter(c => c.captured).length;
  }

  // Логика атаки: выбираем случайный город в красной зоне, который не захвачен
  function attackRedCity() {
    const targets = redCities.filter(c => !c.captured);
    if(targets.length === 0) {
      log.textContent = 'Все города красной зоны захвачены!';
      launchBtn.disabled = true;
      nextTurnBtn.disabled = true;
      return;
    }
    const target = targets[Math.floor(Math.random() * targets.length)];
    // Атака успешна с вероятностью 70%
    if(Math.random() < 0.7) {
      target.captured = true;
      log.textContent = `Вы захватили город #${target.id} в красной зоне!`;
    } else {
      log.textContent = `Атака на город #${target.id} не удалась.`;
    }
    drawCities();
    capturedCountSpan.textContent = redCities.filter(c => c.captured).length;
    launchBtn.disabled = true;
    nextTurnBtn.disabled = false;
  }

  // Ход врага: атакует случайный город из синей зоны
  function enemyTurn() {
    const targets = blueCities.filter(c => c.captured);
    if(targets.length === 0) {
      log.textContent = 'Вы проиграли: все ваши города захвачены!';
      launchBtn.disabled = true;
      nextTurnBtn.disabled = true;
      return;
    }
    const target = targets[Math.floor(Math.random() * targets.length)];
    // Враг атакует с вероятностью 50%
    if(Math.random() < 0.5) {
      target.captured = false;
      log.textContent = `Враг захватил город #${target.id} на вашей территории!`;
    } else {
      log.textContent = `Враг попытался атаковать город #${target.id}, но безуспешно.`;
    }
    drawCities();
    nextTurnBtn.disabled = true;
    launchBtn.disabled = false;
  }

  launchBtn.onclick = attackRedCity;
  nextTurnBtn.onclick = enemyTurn;

  drawCities();
  capturedCountSpan.textContent = countCaptured();

</script>

</body>
</html>

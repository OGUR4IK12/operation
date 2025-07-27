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
    f

<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Monitoreo Presa</title>
  <style>
    body { font-family: Arial; background:#f5f5f5; padding:20px; }
    #connect { padding:10px 20px; font-size:16px; }
    .data { margin:10px 0; }
    .label { font-weight:bold; }
  </style>
</head>
<body>
  <button id="connect">Conectar Presa</button>
  <div id="status" class="data">Estado: Desconectado</div>
  <div class="data"><span class="label">Distancia:</span> <span id="dist">--</span> cm</div>
  <div class="data"><span class="label">Temp:</span> <span id="temp">--</span> °C</div>
  <div class="data"><span class="label">Humedad:</span> <span id="hum">--</span> %</div>
  <div class="data"><span class="label">Lluvia:</span> <span id="rain">--</span></div>
  <div class="data"><span class="label">RainAO:</span> <span id="rainAO">--</span></div>

  <script>
    const SERVICE = '4fafc201-1fb5-459e-8fcc-c5c9c331914b';
    const CHAR = 'beb5483e-36e1-4688-b7f5-e9a123456789';
    let characteristic;

    document.getElementById('connect').addEventListener('click', async() => {
      try {
        const dev = await navigator.bluetooth.requestDevice({
          filters: [{ name: 'Presa_HIDRO' }],
          optionalServices: [SERVICE]
        });
        document.getElementById('status').textContent = 'Estado: Conectando…';
        const srv = await dev.gatt.connect();
        const svc = await srv.getPrimaryService(SERVICE);
        characteristic = await svc.getCharacteristic(CHAR);
        await characteristic.startNotifications();
        characteristic.addEventListener('characteristicvaluechanged', onData);
        document.getElementById('status').textContent = 'Estado: Conectado';
      } catch (e) {
        console.error(e);
        document.getElementById('status').textContent = 'Error al conectar';
      }
    });

    function onData(e) {
      const txt = new TextDecoder().decode(e.target.value);
      const [d,t,h,lr,ao] = txt.split(',');
      document.getElementById('dist').textContent = d;
      document.getElementById('temp').textContent = t;
      document.getElementById('hum').textContent = h;
      document.getElementById('rain').textContent = lr=='1'?'SI':'NO';
      document.getElementById('rainAO').textContent = ao;
    }
  </script>
</body>
</html>
![image](https://github.com/user-attachments/assets/4d27db10-fee8-4682-a4bd-3ada9f6f514a)

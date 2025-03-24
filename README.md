<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Ruleta para decidir quién se queda fuera de convocatoria</title>
  <style>
    body {
      text-align: center;
      font-family: Arial, sans-serif;
      background: linear-gradient(135deg, #ff9a9e, #fad0c4);
    }
    .ruleta-container {
      position: relative;
      width: 300px;
      height: 300px;
      margin: 20px auto;
    }
    canvas {
      width: 100%;
      height: 100%;
      filter: drop-shadow(0px 0px 10px rgba(0, 0, 0, 0.5));
    }
    .flecha {
      position: absolute;
      top: -10px;
      left: 50%;
      transform: translateX(-50%);
      font-size: 30px;
      color: black;
    }
    #resultado {
      font-size: 20px;
      margin-top: 20px;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <h1>Ruleta para decidir quién se queda fuera de convocatoria</h1>
  <div class="ruleta-container">
    <canvas id="ruleta"></canvas>
    <div class="flecha">▼</div>
  </div>
  <button onclick="iniciarGiro()">Girar</button>
  <button onclick="reiniciarRuleta()">Reiniciar</button>
  <p id="resultado"></p>

  <audio id="sonidoPerdedor" src="https://www.fesliyanstudios.com/play-mp3/387"></audio>

  <script>
    const canvas = document.getElementById("ruleta");
    const ctx = canvas.getContext("2d");
    const nombresOriginales = ["Belén", "Teresa", "Virginia", "Nazaret", "Berta", "Irene"];
    let nombres = [...nombresOriginales];
    let currentAngle = 0;
    let spinning = false;
    let candidataEliminada = null;
    let esTrucada = true;

    canvas.width = 300;
    canvas.height = 300;
    const centroX = canvas.width / 2;
    const centroY = canvas.height / 2;
    const radio = centroX;

    function dibujarRuleta() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      const total = nombres.length;
      const sliceAngle = 360 / total;
      for (let i = 0; i < total; i++) {
        const inicio = (currentAngle + i * sliceAngle) * Math.PI / 180;
        const fin = (currentAngle + (i + 1) * sliceAngle) * Math.PI / 180;
        ctx.beginPath();
        ctx.moveTo(centroX, centroY);
        ctx.arc(centroX, centroY, radio, inicio, fin);
        ctx.closePath();
        ctx.fillStyle = ["red", "orange", "blue", "green", "yellow", "purple"][i % 6];
        ctx.fill();
        ctx.stroke();

        const centroSector = currentAngle + i * sliceAngle + (sliceAngle / 2);
        const radian = centroSector * Math.PI / 180;
        const xTexto = centroX + (radio / 1.5) * Math.cos(radian);
        const yTexto = centroY + (radio / 1.5) * Math.sin(radian);
        ctx.save();
        ctx.translate(xTexto, yTexto);
        ctx.rotate(radian + Math.PI / 2);
        ctx.fillStyle = "white";
        ctx.font = "16px Arial";
        ctx.textAlign = "center";
        ctx.fillText(nombres[i], 0, 0);
        ctx.restore();
      }
    }

    function iniciarGiro() {
      if (spinning) return;
      spinning = true;

      let opciones;
      if (esTrucada) {
        opciones = nombres.filter(n => n === "Belén" || n === "Teresa" || n === "Irene");
      } else {
        opciones = [...nombres];
      }

      if (candidataEliminada) {
        opciones = opciones.filter(n => n !== candidataEliminada);
      }
      if (opciones.length === 0) {
        spinning = false;
        return;
      }

      const candidata = opciones[Math.floor(Math.random() * opciones.length)];
      const indexObjetivo = nombres.indexOf(candidata);
      const total = nombres.length;
      const sliceAngle = 360 / total;
      const centroSector = currentAngle + indexObjetivo * sliceAngle + (sliceAngle / 2);
      let diff = (270 - centroSector + 360) % 360;
      if (diff < 10) diff += 360;

      const vueltasExtra = (Math.floor(Math.random() * 3) + 6) * 360;
      const targetAngle = currentAngle + vueltasExtra + diff;

      function animar() {
        let delta = targetAngle - currentAngle;
        if (delta < 0.5) {
          currentAngle = targetAngle;
          dibujarRuleta();
          setTimeout(() => {
            document.getElementById("resultado").innerText = `Ohhh, ${candidata} no jugará este fin de semana 😢, pero seguro que nos está apoyando desde fuera 💪`;
            document.getElementById("sonidoPerdedor").play();
            setTimeout(() => {
              nombres.splice(indexObjetivo, 1);
              candidataEliminada = candidata;
              spinning = false;
              dibujarRuleta();
            }, 2000);
          }, 500);
        } else {
          currentAngle += delta * 0.02;
          dibujarRuleta();
          requestAnimationFrame(animar);
        }
      }

      requestAnimationFrame(animar);
    }

    function reiniciarRuleta() {
      nombres = [...nombresOriginales];
      currentAngle = 0;
      candidataEliminada = null;
      spinning = false;
      esTrucada = !esTrucada; // Alterna entre ruleta trucada y aleatoria sin mostrar texto
      dibujarRuleta();
    }

    dibujarRuleta();
  </script>
</body>
</html>

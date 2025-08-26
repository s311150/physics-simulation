# physics-simulation
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <title>重力助推模擬</title>
  <style>
    body { font-family: sans-serif; display: flex; background: #111; color: #eee; margin: 0; }
    #controls { width: 250px; padding: 1em; background: #222; }
    #controls label { display: block; margin-top: 10px; }
    input { width: 100%; }
    #canvas { flex: 1; background: black; }
    #info { margin-top: 20px; font-size: 14px; }
  </style>
</head>
<body>
  <div id="controls">
    <h2>重力助推參數</h2>
    <label>行星質量
      <input id="planetMass" type="range" min="100" max="2000" step="50" value="800">
    </label>
    <label>行星速度 (像素/秒)
      <input id="planetV" type="range" min="0" max="2" step="0.1" value="1">
    </label>
    <label>飛船初速
      <input id="shipV" type="range" min="1" max="5" step="0.1" value="2.5">
    </label>
    <label>入射角度 (度)
      <input id="shipAngle" type="range" min="-60" max="60" step="1" value="0">
    </label>
    <button onclick="resetSim()">重新開始</button>
    <div id="info"></div>
  </div>

  <canvas id="canvas" width="800" height="600"></canvas>

  <script>
    const canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");

    let planet = {x: 400, y: 300, vx: 0, vy: 0, mass: 800};
    let ship = {x: 100, y: 300, vx: 2.5, vy: 0, trail: []};
    let t = 0, running = true;

    function resetSim() {
      planet.mass = parseFloat(document.getElementById("planetMass").value);
      let pv = parseFloat(document.getElementById("planetV").value);
      planet.x = 400; planet.y = 300; planet.vx = 0; planet.vy = -pv;

      let sv = parseFloat(document.getElementById("shipV").value);
      let angle = parseFloat(document.getElementById("shipAngle").value) * Math.PI/180;
      ship.x = 100; ship.y = 500; 
      ship.vx = sv * Math.cos(angle);
      ship.vy = -sv * Math.sin(angle);
      ship.trail = [];

      t = 0; running = true;
    }

    function step() {
      if (!running) return;

      // 行星移動（簡化為直線）
      planet.x += planet.vx;
      planet.y += planet.vy;

      // 飛船受行星重力影響
      let dx = planet.x - ship.x;
      let dy = planet.y - ship.y;
      let r2 = dx*dx + dy*dy;
      let r = Math.sqrt(r2);
      let G = 5000; // 簡化常數
      let F = G * planet.mass / r2;
      let ax = F * dx/r;
      let ay = F * dy/r;

      ship.vx += ax;
      ship.vy += ay;
      ship.x += ship.vx;
      ship.y += ship.vy;

      ship.trail.push([ship.x, ship.y]);
      if (ship.trail.length > 300) ship.trail.shift();

      // 繪圖
      ctx.clearRect(0,0,canvas.width,canvas.height);

      // 行星
      ctx.beginPath();
      ctx.arc(planet.x, planet.y, 15, 0, 2*Math.PI);
      ctx.fillStyle = "blue"; ctx.fill();

      // 飛船軌跡
      ctx.beginPath();
      ctx.strokeStyle = "lime";
      ctx.moveTo(ship.trail[0][0], ship.trail[0][1]);
      for (let p of ship.trail) ctx.lineTo(p[0], p[1]);
      ctx.stroke();

      // 飛船
      ctx.beginPath();
      ctx.arc(ship.x, ship.y, 5, 0, 2*Math.PI);
      ctx.fillStyle = "red"; ctx.fill();

      // 顯示速度
      let v = Math.sqrt(ship.vx**2 + ship.vy**2).toFixed(2);
      document.getElementById("info").innerText = `模擬時間 ${t}，飛船速度 = ${v}`;

      t++;
      requestAnimationFrame(step);
    }

    resetSim();
    step();
  </script>
</body>
</html>

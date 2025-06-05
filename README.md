<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>RoboRumble</title>
  <style>
    body { margin: 0; background: #111; font-family: sans-serif; }
    canvas { display: block; margin: 20px auto; background: #222; }
    #status { color: white; text-align: center; }
  </style>
</head>
<body>
  <canvas id="gameCanvas" width="800" height="600"></canvas>
  <div id="status">Connecting to server...</div>

  <!-- ✅ CDN Socket.IO -->
  <script src="https://cdn.socket.io/4.7.2/socket.io.min.js"></script>
  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const status = document.getElementById('status');
    const socket = io();

    let players = {}; // id: {x, y, hp}
    let myId = null;
    let keys = {};

    // Input
    document.addEventListener('keydown', e => keys[e.key] = true);
    document.addEventListener('keyup', e => keys[e.key] = false);

    setInterval(() => {
      socket.emit('move', keys);
    }, 1000 / 30);

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      for (let id in players) {
        const p = players[id];
        ctx.fillStyle = id === myId ? 'cyan' : 'red';
        ctx.fillRect(p.x, p.y, 50, 50);

        ctx.fillStyle = 'white';
        ctx.fillRect(p.x, p.y - 10, 50 * (p.hp / 100), 5);
      }
    }

    function loop() {
      draw();
      requestAnimationFrame(loop);
    }

    loop();

    socket.on('init', data => {
      myId = data.id;
      players = data.players;
      status.innerText = 'Connected! Invite a friend by sharing this room.';
    });

    socket.on('update', data => {
      players = data.players;
    });

    socket.on('gameOver', winner => {
      status.innerText = winner === myId ? 'You Win!' : 'You Lose!';
    });
  </script>
</body>
</html>


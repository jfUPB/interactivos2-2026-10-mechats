# Unidad 4

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
Lo primero que hice fue modificar el codigo de las visuales para que se modificaran cuando el publico enviara los datos y cambiara el fondo y posicion de las visuales

```javascript
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Set Audiovisual Interactivo</title>
  <script src="https://cdn.jsdelivr.net/npm/p5@1.11.1/lib/p5.min.js"></script>
  <script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
  <style> body { margin: 0; overflow: hidden; background: black; } </style>
</head>
<body>
<script>
const MAX_ANIMATIONS = 25;
let eventQueue = [];
let activeAnimations = [];
let globalScale = 1.0; 

// Variables para el público (Mobile)
let mobileData = { x: 0.5, y: 0.5 }; 
let targetBgHue = 0; 
let currentBgHue = 0; 

function setup() {
  createCanvas(windowWidth, windowHeight);
  colorMode(HSB, 360, 100, 100, 1); 
  rectMode(CENTER);
  noStroke();

  // --- 1. CONEXIÓN AL BRIDGE PRINCIPAL (Música y OSC) ---
  const bridgeP5 = new WebSocket("ws://localhost:8081");

  bridgeP5.onmessage = (event) => {
    const msg = JSON.parse(event.data);

    // Manejo de Datos de Open Stage Control (desde el relay 8082 -> bridge 8081)
    if (msg.type === "OSC") {
      // Ajustado para escuchar faderSize o fader1 según tu relay
      if (msg.address === "/faderSize" || msg.address === "/fader1") { 
        globalScale = map(msg.args[0], 0, 1, 0.1, 4.0); 
      }
      return;
    }

    // Manejo de Datos de Strudel
    let params = {};
    if (msg.args) {
      for (let i = 0; i < msg.args.length; i += 2) {
        params[msg.args[i]] = msg.args[i + 1];
      }
      eventQueue.push({
        timestamp: msg.timestamp || Date.now(),
        sound: params.s || "",
        delta: params.delta || 0.25,
        params: params
      });
      eventQueue.sort((a, b) => a.timestamp - b.timestamp);
    }
  };

  // --- 2. CONEXIÓN AL SERVIDOR MÓVIL (Socket.io) ---
  const socketMobile = io("http://localhost:3000");

  socketMobile.on('message', (data) => {
    if (data.type === 'mobile-touch') {
      mobileData.x = data.x;
      mobileData.y = data.y;
      targetBgHue = data.x * 360; // El público cambia el color del fondo

      // Efecto visual instantáneo por toque
      activeAnimations.push({
        startTime: Date.now(),
        duration: 400,
        type: "touch-feedback",
        x: data.x * width,
        y: data.y * height,
        color: [0, 0, 100]
      });
    }
  });
}

function draw() {
  // Color de fondo controlado por el público
  currentBgHue = lerp(currentBgHue, targetBgHue, 0.05);
  background(currentBgHue, 80, 10, 0.15); 

  let now = Date.now();

  // Procesar sonidos de Strudel en la ubicación del público
  while (eventQueue.length > 0 && now >= eventQueue[0].timestamp) {
    let ev = eventQueue.shift();
    if (activeAnimations.length >= MAX_ANIMATIONS) activeAnimations.shift();

    activeAnimations.push({
      startTime: ev.timestamp,
      duration: ev.delta * 1000,
      type: ev.sound,
      x: mobileData.x * width, // Sigue al público
      y: mobileData.y * height,
      color: getColorForSound(ev.sound)
    });
  }

  for (let i = activeAnimations.length - 1; i >= 0; i--) {
    let anim = activeAnimations[i];
    let elapsed = now - anim.startTime;
    let progress = elapsed / anim.duration;

    if (progress <= 1) {
      dibujarElemento(anim, progress);
    } else {
      activeAnimations.splice(i, 1);
    }
  }
}

function dibujarElemento(anim, p) {
  push();
  translate(anim.x, anim.y);
  scale(globalScale); // Controlado por el fader de Open Stage Control
  
  const s = anim.type;
  const c = anim.color;

  if (s === "touch-feedback") {
    noFill();
    stroke(0, 0, 100, 1 - p);
    circle(0, 0, p * 200);
  } else if (s.includes("bd")) {
    fill(c[0], c[1], c[2], 1 - p);
    circle(0, 0, lerp(100, 400, p));
  } else if (s.includes("hh")) {
    fill(c[0], c[1], c[2], 1 - p);
    rect(0, 0, lerp(30, 0, p), lerp(30, 0, p));
  } else {
    noFill();
    stroke(c[0], c[1], c[2], 1 - p);
    strokeWeight(2);
    rotate(p * PI);
    rect(0, 0, lerp(100, 20, p), lerp(100, 20, p));
  }
  pop();
}

function getColorForSound(s) {
  if (s.includes("bd")) return [200, 100, 100];
  if (s.includes("hh")) return [60, 100, 100];
  return [targetBgHue, 50, 100]; // Los sonidos armonizan con el color del fondo
}

function windowResized() { resizeCanvas(windowWidth, windowHeight); }
</script>
</body>
</html>
```

Despues de esto, use el proyecto que usamos en la clase 1 para conectar al publico y lo modifique de esta forma

server:

```javascript
const express = require('express');
const http = require('http');
const socketIO = require('socket.io');

const app = express();
const server = http.createServer(app); 

// ← SOLO CAMBIA ESTA LÍNEA
const io = socketIO(server, {
    cors: {
        origin: "*",  // permite cualquier origen
        methods: ["GET", "POST"]
    }
});

const port = 3000;

app.use(express.static('public'));

io.on('connection', (socket) => {
    console.log('New client connected');
    socket.on('message', (message) => {
        console.log('Received message =>', message);
        socket.broadcast.emit('message', message);
    });
    socket.on('disconnect', () => {
        console.log('Client disconnected');
    });
});

server.listen(port, () => {
    console.log(`Server is listening on http://localhost:${port}`);
});
```
index:

```javascript
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>Control XY</title>
  <script src="/socket.io/socket.io.js"></script>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Space+Mono&display=swap');

    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      background: #000;
      width: 100vw;
      height: 100vh;
      overflow: hidden;
      touch-action: none;
      font-family: 'Space Mono', monospace;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      user-select: none;
    }

    #pad {
      width: 80vw;
      height: 80vw;
      max-width: 360px;
      max-height: 360px;
      border: 1px solid rgba(255,255,255,0.15);
      border-radius: 12px;
      position: relative;
      background: radial-gradient(circle at 50% 50%, rgba(255,255,255,0.03), transparent);
      cursor: crosshair;
    }

    #pad::before {
      content: '';
      position: absolute;
      inset: 0;
      border-radius: 12px;
      background: 
        linear-gradient(rgba(255,255,255,0.04) 1px, transparent 1px),
        linear-gradient(90deg, rgba(255,255,255,0.04) 1px, transparent 1px);
      background-size: 40px 40px;
    }

    #cursor {
      position: absolute;
      width: 16px;
      height: 16px;
      border-radius: 50%;
      background: white;
      transform: translate(-50%, -50%);
      pointer-events: none;
      top: 50%;
      left: 50%;
      box-shadow: 0 0 12px white;
    }

    #crossh, #crossv {
      position: absolute;
      background: rgba(255,255,255,0.1);
      pointer-events: none;
    }
    #crossh { width: 100%; height: 1px; top: 50%; left: 0; }
    #crossv { height: 100%; width: 1px; left: 50%; top: 0; }

    #status {
      margin-top: 24px;
      font-size: 10px;
      letter-spacing: 0.3em;
      text-transform: uppercase;
      color: rgba(255,255,255,0.3);
    }

    #coords {
      margin-top: 10px;
      font-size: 9px;
      color: rgba(255,255,255,0.2);
      letter-spacing: 0.15em;
    }

    #hint {
      margin-top: 32px;
      font-size: 9px;
      letter-spacing: 0.3em;
      text-transform: uppercase;
      color: rgba(255,255,255,0.1);
      animation: blink 2.5s ease-in-out infinite;
    }

    @keyframes blink { 0%,100%{opacity:.3} 50%{opacity:1} }
  </style>
</head>
<body>

  <div id="pad">
    <div id="crossh"></div>
    <div id="crossv"></div>
    <div id="cursor"></div>
  </div>

  <div id="status">● conectando...</div>
  <div id="coords">x: — &nbsp; y: —</div>
  <div id="hint">arrastra para controlar</div>

  <script src="sketch.js"></script>
</body>
</html>
```
sketch:
```javascript
const socket = io();

socket.on('connect', () => console.log('✅ Conectado'));
socket.on('disconnect', () => console.log('❌ Desconectado'));

const pad = document.getElementById('pad');
const cursor = document.getElementById('cursor');
const crossh = document.getElementById('crossh');
const crossv = document.getElementById('crossv');
const coordsEl = document.getElementById('coords');

function getPos(e) {
    const rect = pad.getBoundingClientRect();
    const clientX = e.touches ? e.touches[0].clientX : e.clientX;
    const clientY = e.touches ? e.touches[0].clientY : e.clientY;
    const x = Math.max(0, Math.min(1, (clientX - rect.left) / rect.width));
    const y = Math.max(0, Math.min(1, (clientY - rect.top) / rect.height));
    return { x, y };
}

function onMove(e) {
    e.preventDefault();
    const { x, y } = getPos(e);

    cursor.style.left = (x * 100) + '%';
    cursor.style.top = (y * 100) + '%';
    crossh.style.top = (y * 100) + '%';
    crossv.style.left = (x * 100) + '%';

    const hue = Math.round(x * 360);
    cursor.style.background = `hsl(${hue}, 80%, 70%)`;
    cursor.style.boxShadow = `0 0 16px hsl(${hue}, 80%, 70%)`;

    coordsEl.textContent = `x: ${x.toFixed(3)}   y: ${y.toFixed(3)}`;

    socket.emit('message', {
        type: 'mobile-touch',
        x,
        y
    });
}

pad.addEventListener('touchstart', onMove, { passive: false });
pad.addEventListener('touchmove', onMove, { passive: false });
pad.addEventListener('mousedown', onMove);
pad.addEventListener('mousemove', (e) => { if (e.buttons > 0) onMove(e); });

```

no logre plantear la arquitectura que elegimos en clase, el publico y las visuales tienen una relación unidireccional, este es el diagrama de MI arquitectura:


![WhatsApp Image 2026-03-11 at 9 20 26 PM](https://github.com/user-attachments/assets/8516e0df-7e7a-4ac9-b0cd-ea5d78871e9b)


## Bitácora de reflexión



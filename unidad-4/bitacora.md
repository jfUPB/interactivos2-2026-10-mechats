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
index:

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/p5@1.11.0/lib/p5.min.js"></script>
    <script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
    <script src="sketch.js"></script>
    <title>Desktop p5.js Application</title>
</head>
<body></body>
</html>
```
sketch:
```javascript
let socket;
let circleX = 200;
let circleY = 200;
const port = 3000;

function setup() {
    createCanvas(300, 400);
    background(220);
    socket = io(); 

    socket.on('connect', () => {
        console.log('Connected to server');
    });

    socket.on('message', (data) => {
        console.log(`Received message:`, data);
        if (data && data.type === 'touch') {
            circleX = data.x;
            circleY = data.y;
        }
    });    

    socket.on('disconnect', () => {
        console.log('Disconnected from server');
    });

    socket.on('connect_error', (error) => {
        console.error('Socket.IO error:', error);
    });
}

function draw() {
    background(220);
    fill(255, 0, 0);
    ellipse(circleX, circleY, 50, 50);
}

```

no logre plantear la arquitectura que elegimos en clase, el publico y las visuales tienen una relación unidireccional, este es el diagrama de MI arquitectura:




## Bitácora de reflexión

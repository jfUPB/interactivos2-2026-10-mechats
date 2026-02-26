# Unidad 3

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 


Lo primero que tenía que hacer para usar el Open Stage era lograr que este se conectara con las visuales, y a la vez el puente de las visuales transmitiera ambas cosas a  el server del strudel. Para esto tocaba crear otro puente. El problema principal era que el poenstage estaba configurado para lanzar datos desde el puerto 8082. Pero no se puede comunicar por un mismo "canal" desde dos puentes diferentes, por eso tocaba enviar los datos desde otro, en este caso el 9000. Ademas el open stage necesitaba un lugar donde alojarse para enbiarle sus datos al bridge. En este caso el 8083, todo eso lo configure desde el open stage

<img width="779" height="573" alt="image" src="https://github.com/user-attachments/assets/2367ff1d-a9e0-4e5c-a99d-499c150d0990" />

Para verifiar que si se estuviesen recibiendo los datos use la misma linea de codigo que usamos en el puente de strudel, pero esta vez en open stage

<img width="1090" height="491" alt="image" src="https://github.com/user-attachments/assets/57e4b2dd-931e-43b7-8d5c-9925effdf817" />

## El descenso a la locura

No se por que empezo a fallarme el fader y por mas que lo movia no funcionaba, le miraba si los puertos estaban buenos y seguia sin funcionar, se supone que la idea era que cambiara de tamaño pero no funcionnaba. Me quede 4 horas tratanod de arreglar el problema, chat gpt incluso me llego a unir los dos bridges en uno solo, pero yo sabia que eso no tenia ningun sentido entonces no lo hice, pero de tanto cambio que ya le habia hecho habia arruinado completamente los dos bridges, entonces me toco volver al principio. Honestamente no se que fue lo que funciono exactamente, pero modificando algo de las visuales y las maneras en la que se comportaba con el fader logre que reaccionaran

```javascript
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdn.jsdelivr.net/npm/p5@1.11.11/lib/p5.min.js"></script>
  <style> body { margin: 0; overflow: hidden; background: black; } </style>
</head>
<body>
<script>
const MAX_ANIMATIONS = 18;
let eventQueue = [];
let activeAnimations = [];
let globalScale = 1.0; // <--- VARIABLE CONTROLADA POR EL FADER

function setup() {
  createCanvas(windowWidth, windowHeight);
  rectMode(CENTER);
  noStroke();

  const socket = new WebSocket("ws://localhost:8081");

  socket.onmessage = (event) => {
    const msg = JSON.parse(event.data);

    // SI EL MENSAJE ES DE OPEN STAGE CONTROL
    if (msg.type === "OSC") {
      if (msg.address === "/fader1") { // Ajusta esto al nombre de tu fader
        globalScale = map(msg.args[0], 0, 1, 0.1, 3.0); 
      }
      return; // No lo agregamos a la cola de sonidos
    }

    // SI EL MENSAJE ES DE STRUDEL
    let params = {};
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
  };
}

function draw() {
  background(0, 40);
  let now = Date.now();

  while (eventQueue.length > 0 && now >= eventQueue[0].timestamp) {
    let ev = eventQueue.shift();
    if (activeAnimations.length >= MAX_ANIMATIONS) activeAnimations.shift();

    activeAnimations.push({
      startTime: ev.timestamp,
      duration: ev.delta * 1000,
      type: ev.sound,
      x: random(width * 0.2, width * 0.8),
      y: random(height * 0.2, height * 0.8),
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
  // Aplicamos el escalado global aquí antes de dibujar
  translate(anim.x, anim.y);
  scale(globalScale); 
  
  const s = anim.type;
  const c = anim.color;

  if (s.includes("bd")) {
    dibujarBombo(p, c);
  } else if (s.includes("hh") || s.includes("oh")) {
    dibujarHat(p, c);
  } else if (s.includes("piano") || s.includes("square")) {
    dibujarPiano(p, c);
  } else {
    dibujarAmbient(p, c);
  }
  pop();
}

function dibujarBombo(p, c) {
  let d = lerp(120, 500, p);
  fill(c[0], c[1], c[2], lerp(200, 0, p));
  circle(0, 0, d); // Usamos 0,0 porque ya hicimos translate
}

function dibujarHat(p, c) {
  let sz = lerp(40, 0, p);
  fill(c[0], c[1], c[2], 150);
  rect(0, 0, sz, sz);
}

function dibujarPiano(p, c) {
  let size = lerp(140, 20, p);
  rotate(p * TWO_PI);
  stroke(c[0], c[1], c[2], 180);
  strokeWeight(2);
  noFill();
  rect(0, 0, size, size);
}

function dibujarAmbient(p, c) {
  let sz = lerp(100, 0, p);
  stroke(c[0], c[1], c[2], 100);
  noFill();
  ellipse(0, 0, sz);
}

function getColorForSound(s) {
  const colors = { "square": [255, 60, 80], "piano": [138, 20, 31] };
  if (colors[s]) return colors[s];
  return [200, 200, 200];
}

function windowResized() { resizeCanvas(windowWidth, windowHeight); }
</script>
</body>
</html>
```








## Bitácora de reflexión

<img width="1445" height="708" alt="image" src="https://github.com/user-attachments/assets/51c20d1c-c5e9-4e42-bfa3-006854206661" />




# Unidad 2

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

Muy bien, para emppezar a hacer las visuales de mi temita me di cuenta que estaba usando mal la funcion arrange, porque lo que estaba haciendo era crear un stack lleno de funciones arrange, y hacia que sonara uno mientras que los otros estaban en silencio, sabiendo que. Teniendo en cuenta que la forma en la que hice mi tema estaba pensada para tener la misma melodia pero con cosas nuevas, lo mas facil era crear solo una fioncion arrange, haciendo que primero sonase una cosa, y despues otra. Sin necesidad de hacer que cosas estuvieran en silencio por no se cuantos cilos, asi que modifique el codigo a este

```javascript

setcps(0.5)



const inicio =
  note("<[74 86]*4 [72 84]*4 [70 82]*4 [68 80]*4>")
    .s("piano")
    .room(0.3)

const principal =
  stack(
    note("<[50 62]*4 [48 60]*4 [46 58]*4 [44 56]*4>")
      .sound("gm_acoustic_bass"),
    inicio
  )

const preboom =
  stack(
    s("bd:5*4").gain(1.2),
    s("hh*8").gain(0.7),
    s("~ cp")
      .gain(0.8)
      .delay(0.2)
      .delayfb(0.3),
    s("~ [hh:4*2]").gain(0.9),
    principal.fast("1 1 2")
  )

const puente =
  stack(
    s("cp:2")
      .gain(1.3)
      .room(1)
      .sz(0.9)
      .sustain(2),
    s("noise").room(0.5)
  ).slow(5)

const final =
  stack(
    preboom.room(0.7),
    note("<[74 86]*4 [72 84]*4 [70 82]*4 [68 80]*4>")
      .s("square")
      .gain(0.4),
    note("<[50 62]*4 [48 60]*4 [46 58]*4 [44 56]*4>")
      .s("triangle")
      .gain(0.4)
  )



const tema = arrange(
  [4,  inicio],
  [4,  principal],
  [4,  preboom],
  [1,  puente],
  [8,  final]
)



$: stack(
  tema,
  tema
    .gain(0.05)
    .osc()
)

```


Despues, una vez hecho esto, busque sincronizar el audio con las visuales de la forma en que el profe nos mostro en el codigo, basicamente  agarraba el timestamp que me mandaba la pagina y con eso se podia saber en que momento exacto tenia que sonar, entonces con el eventquen se organizaban todos los sonidos que llegaban en orden cronologico y asi se evitaban los desfaces y hacia que cada sonido se activase en el momento correcto



<img width="966" height="427" alt="image" src="https://github.com/user-attachments/assets/e730f9fe-6ff1-4ea8-8310-1d7ef5c0e7c3" />

Esta parte del codigo es para aclarar por que aparece una linea que dice  const MAX_ANIMATIONS = 40; Y es que cuando heche a correr mi animacion me di cuenta que estaba demasiado saturada, y aunque en un principio esa era la idea genuinamente las visaules no se alcanzaban a entender, entonces esta linea hace que haya un maximo de 40 animaciones activas, y si aun nos parece mucho pues el valor se puede modificar para que sean cada vez menos

<img width="607" height="234" alt="image" src="https://github.com/user-attachments/assets/44e9147e-7a1d-4eeb-a14e-83ac8270e4c3" />


Esta parte tambien es importante porque comprendi como funcionaba el cambiar de color en las figuras geometricas que aparecian, y es que se usa un sistema RGB, entonces me toco buscar una herramienta en internet que me permitiera cambiar de sistema RGB a sistema HEX para poder poner los rombos que salen cuando sale el piano de color rojo



Codigo de las visuales:

```javascript
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdn.jsdelivr.net/npm/p5@1.11.11/lib/p5.min.js"></script>
  <style>
    body { margin: 0; overflow: hidden; background: black; }
  </style>
</head>
<body>
<script>

const MAX_ANIMATIONS = 18;

let eventQueue = [];
let activeAnimations = [];
const LATENCY_CORRECTION = 0;

function setup() {
  createCanvas(windowWidth, windowHeight);
  rectMode(CENTER);
  noStroke();

  const socket = new WebSocket("ws://localhost:8081");

  socket.onmessage = (event) => {
    const msg = JSON.parse(event.data);

    let params = {};
    for (let i = 0; i < msg.args.length; i += 2) {
      params[msg.args[i]] = msg.args[i + 1];
    }

    eventQueue.push({
      timestamp: msg.timestamp,
      sound: params.s || "",
      delta: params.delta || 0.25,
      params: params
    });

    eventQueue.sort((a, b) => a.timestamp - b.timestamp);
  };
}

function draw() {
  background(0, 40);

  let now = Date.now() + LATENCY_CORRECTION;

  while (eventQueue.length > 0 && now >= eventQueue[0].timestamp) {
    let ev = eventQueue.shift();

    if (ev.delta < 0.12) continue;

    if (activeAnimations.length >= MAX_ANIMATIONS) {
      activeAnimations.shift();
    }

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

  const s = anim.type;
  const c = anim.color;

  if (s.includes("bd")) {
    dibujarBombo(p, c);
  }
  else if (s.includes("hh") || s.includes("oh")) {
    dibujarHat(anim, p, c);
  }
 
  else if (s.includes("piano") || s.includes("square") || s.includes("triangle")) {
    dibujarPiano(anim, p, c);
  }
  else {
    dibujarAmbient(anim, p, c);
  }

  pop();
}

function dibujarBombo(p, c) {
  let d = lerp(120, 500, p);
  let alpha = lerp(200, 0, p);
  fill(c[0], c[1], c[2], alpha);
  circle(width / 2, height / 2, d);
}

function dibujarHat(anim, p, c) {
  let sz = lerp(20, 0, p);
  fill(c[0], c[1], c[2], 150);
  rect(anim.x, anim.y, sz, sz);
}



function dibujarPiano(anim, p, c) {
  let size = lerp(140, 20, p);
  let angle = p * TWO_PI;

  translate(anim.x, anim.y);
  rotate(angle);

  stroke(c[0], c[1], c[2], 180);
  strokeWeight(2);
  noFill();

  rect(0, 0, size, size);
}

function dibujarAmbient(anim, p, c) {
  let sz = lerp(100, 0, p);
  stroke(c[0], c[1], c[2], 100);
  noFill();
  ellipse(anim.x, anim.y, sz);
}

function getColorForSound(s) {
  const colors = {
    "square": [255, 60, 80],
    "triangle": [255, 255, 120],
    "tr909oh": [255, 180, 80],
    "piano": [138, 20, 31]
  };

  if (colors[s]) return colors[s];

  let charCode = s.charCodeAt(0) || 0;
  return [
    (charCode * 123) % 255,
    (charCode * 321) % 255,
    (charCode * 213) % 255
  ];
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}

</script>
</body>
</html>
```




## Bitácora de reflexión

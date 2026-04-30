# Unidad 6

## Bitácora de proceso de aprendizaje

 el objetivo de mis visuales es que se puedan usar en un lugar oscuro. Me las imagino en un rave proyectadas en paredes y techos, la idea principal que tengo es una especie de mosaicos por los cuales el publico navega a lo largo de la cancion, tienen formas geometricas y de mosaicos con una paleta de colores frios y poco saturados para dar la vibra soft que busca mi tema, todas estas fromas estaran generandose en un fondo negro no estatico que hace creer a la persona que lo ve que esta  en un espacio tridimensional
## Bitácora de aplicación 

A continuación expongo el codigo de mis visuales

```javascript
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdn.jsdelivr.net/npm/p5@1.11.11/lib/p5.min.js"></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { background: #000; overflow: hidden; font-family: 'Courier New', monospace; }
    #hud {
      position: fixed; bottom: 18px; left: 50%;
      transform: translateX(-50%);
      font-size: 10px; color: rgba(180,100,255,0.45);
      letter-spacing: 4px; pointer-events: none;
    }
    #section-hud {
      position: fixed; top: 14px; left: 18px;
      font-size: 9px; color: rgba(0,255,200,0.4);
      letter-spacing: 5px; pointer-events: none;
    }
    #conn-hud {
      position: fixed; bottom: 38px; left: 50%;
      transform: translateX(-50%);
      font-size: 8px; letter-spacing: 2px;
      pointer-events: none; transition: color 0.4s;
    }
    #conn-hud.connecting { color: rgba(255,192,64,0.8); }
    #conn-hud.connected  { color: rgba(0,255,140,0.7); }
    #conn-hud.error      { color: rgba(255,48,100,0.8); }
    #last-sound {
      position: fixed; bottom: 54px; left: 50%;
      transform: translateX(-50%);
      font-size: 8px; color: rgba(0,255,200,0.35);
      letter-spacing: 3px; pointer-events: none;
    }
    #debug-panel {
      position: fixed; top: 36px; left: 18px;
      font-size: 8px; color: rgba(255,192,64,0.6);
      letter-spacing: 1px; line-height: 1.7;
      max-width: 380px; pointer-events: none;
    }
    /* ── NUEVO: globos HUD ── */
    #globos-hud {
      position: fixed; top: 14px; right: 18px;
      font-size: 9px; color: rgba(0,255,220,0.5);
      letter-spacing: 3px; pointer-events: none; text-align: right;
    }
    #globos-barra-bg {
      position: fixed; top: 44px; right: 18px;
      width: 100px; height: 2px;
      background: rgba(255,255,255,0.06); pointer-events: none;
    }
    #globos-barra-fill {
      height: 100%; width: 0%;
      background: rgba(180,100,255,0.8);
      transition: width 0.4s ease, background 0.5s;
    }
    #nivel-flash {
      position: fixed; inset: 0;
      display: flex; align-items: center; justify-content: center;
      flex-direction: column; gap: 14px;
      pointer-events: none; opacity: 0;
      transition: opacity 0.2s; z-index: 200;
    }
    #nivel-flash.show { opacity: 1; }
    #nivel-flash-txt {
      font-size: clamp(20px, 5vw, 52px);
      letter-spacing: 8px; color: #00ffdc; text-align: center;
    }
    #nivel-flash-sub {
      font-size: 9px; letter-spacing: 4px; color: rgba(255,255,255,0.3);
    }
  </style>
</head>
<body>
<div id="hud">▲ MYSTIC TECHNO ▲</div>
<div id="section-hud">──</div>
<div id="conn-hud" class="connecting">◌ INICIANDO...</div>
<div id="last-sound"></div>
<div id="debug-panel"></div>

<!-- NUEVO -->
<div id="globos-hud">
  <div id="g-num">🎈 0</div>
  <div id="g-nivel" style="font-size:7px;color:rgba(180,100,255,0.35);margin-top:2px;">en calma</div>
</div>
<div id="globos-barra-bg"><div id="globos-barra-fill"></div></div>
<div id="nivel-flash">
  <div id="nivel-flash-txt"></div>
  <div id="nivel-flash-sub"></div>
</div>

<script>
// ─── CONFIG ───────────────────────────────────────────────────────
const WS_PORTS = [8081, 8080, 8082, 3000];
let currentPortIndex = 0;

const C = {
  purple: [180, 100, 255],
  cyan:   [0,   255, 220],
  pink:   [255,  48, 136],
  gold:   [255, 200,  64],
  violet: [120,  60, 220],
  lime:   [160, 255,  60],
  ice:    [100, 200, 255],
};

// ─── ESTADO ───────────────────────────────────────────────────────
let eventQueue  = [];
let activeAnims = [];
let particles   = [];
let pulseRings  = [];
let frame       = 0;
let inDrop      = false;
let dropIntensity = 0;
let keyFlash    = 0;
let wsConnected = false;
let totalMessages = 0;
let debugLines  = [];
let socket      = null;

const CAM = {
  x: 0, y: 0,
  tx: 0, ty: 0,
  vx: 0.18, vy: 0.10,
  zoom: 1.0, tzoom: 1.0,
  shake: 0,
  angle: 0, targetAngle: 0,
};

const TILES = [];
const TILE_GRID = 9;
const TILE_SIZE = 320;

// ─── NUEVO: estado de globos ──────────────────────────────────────
let globosCount    = 0;
let nivelActual    = 0;
let globoAnims     = [];   // partículas de explosión de globo
let nivelFlashTimer = 0;
const UMBRAL_1 = 10, UMBRAL_2 = 25, UMBRAL_3 = 50;

const NIVEL_LABELS = ['en calma', '▲ umbral 1', '▲▲ umbral 2', '▲▲▲ CAOS'];
const NIVEL_MSGS = {
  1: { txt: 'PRIMER UMBRAL',  sub: 'el sistema despierta' },
  2: { txt: 'SEGUNDO UMBRAL', sub: 'la energía se libera'  },
  3: { txt: 'MODO CAOS',      sub: '████████████████████'  },
};
// colores de barra por nivel
const BARRA_COLORES = [
  'rgba(180,100,255,0.8)',
  'rgba(180,100,255,1)',
  'rgba(255,48,136,0.9)',
  'rgba(255,200,64,1)',
];

// ─── UI ───────────────────────────────────────────────────────────
const connHud    = document.getElementById('conn-hud');
const lastSound  = document.getElementById('last-sound');
const debugPanel = document.getElementById('debug-panel');
const gNum       = document.getElementById('g-num');
const gNivel     = document.getElementById('g-nivel');
const barFill    = document.getElementById('globos-barra-fill');
const flashEl    = document.getElementById('nivel-flash');
const flashTxt   = document.getElementById('nivel-flash-txt');
const flashSub   = document.getElementById('nivel-flash-sub');

function addDebug(msg) {
  const t = new Date().toLocaleTimeString('es', {hour12:false});
  debugLines.unshift('['+t+'] '+msg);
  if (debugLines.length > 7) debugLines.pop();
  debugPanel.innerHTML = debugLines.join('<br>');
}

// ─── NUEVO: actualizar HUD de globos ─────────────────────────────
function actualizarHUDGlobos() {
  gNum.textContent = '🎈 ' + globosCount;
  gNivel.textContent = NIVEL_LABELS[nivelActual] || '';
  const pct = Math.min(100, (globosCount / UMBRAL_3) * 100);
  barFill.style.width = pct + '%';
  barFill.style.background = BARRA_COLORES[nivelActual];
}

// ─── NUEVO: cambio de nivel ───────────────────────────────────────
function cambiarNivel(nuevo) {
  nivelActual = nuevo;
  actualizarHUDGlobos();
  const info = NIVEL_MSGS[nuevo];
  if (info) {
    flashTxt.textContent = info.txt;
    flashSub.textContent = info.sub;
    flashEl.classList.add('show');
    nivelFlashTimer = 180;
  }
  // Nivel 3: sacudir cámara fuerte y acelerar tiles
  if (nuevo === 3) {
    CAM.shake = 25;
    CAM.tzoom = 0.75;
    for (const t of TILES) { t.active = 1; t.flash = 1; }
  }
  // Nivel 2: zoom out suave
  if (nuevo === 2) {
    CAM.tzoom = 0.88;
  }
}

// ─── NUEVO: explosión de globo en world space ─────────────────────
function spawnGloboExplosion(wx, wy) {
  const colors = [C.purple, C.cyan, C.pink, C.gold];
  const n = 20;
  for (let i = 0; i < n; i++) {
    const ang   = (i / n) * Math.PI * 2 + Math.random() * 0.3;
    const speed = 2 + Math.random() * 7;
    const col   = colors[Math.floor(Math.random() * colors.length)];
    globoAnims.push({
      wx, wy,
      vx: Math.cos(ang) * speed,
      vy: Math.sin(ang) * speed,
      r:  3 + Math.random() * 10,
      color: col,
      alpha: 1.0,
      born: Date.now(),
      lifetime: 600 + Math.random() * 600,
      shape: Math.random() < 0.5 ? 'circle' : 'diamond',
    });
  }
  // Pulso de onda de choque en world space
  pulseRings.push({ wx, wy, r: 15, color: C.cyan,   alpha: 0.9, big: true,  speed: 14 });
  pulseRings.push({ wx, wy, r: 5,  color: C.purple, alpha: 0.7, big: false, speed: 8  });
  // Activar tile más cercano
  let closest = null, closestD = Infinity;
  for (const t of TILES) {
    const d = Math.hypot(wx - t.wx, wy - t.wy);
    if (d < closestD) { closestD = d; closest = t; }
  }
  if (closest) { closest.active = 1; closest.flash = 1; }
  // Sacudida de cámara
  CAM.shake = Math.max(CAM.shake, 10);
}

// ─── NUEVO: dibujar partículas de globo (en world space) ──────────
function drawGloboAnims() {
  const now = Date.now();
  for (let i = globoAnims.length - 1; i >= 0; i--) {
    const g = globoAnims[i];
    const p = (now - g.born) / g.lifetime;
    if (p >= 1) { globoAnims.splice(i, 1); continue; }
    g.wx += g.vx;
    g.wy += g.vy;
    g.vy += 0.08;  // gravedad suave
    g.vx *= 0.97;
    const c = g.color;
    const a = (1 - p) * 200;
    noStroke();
    fill(c[0], c[1], c[2], a);
    push();
    translate(g.wx, g.wy);
    if (g.shape === 'circle') {
      circle(0, 0, g.r * 2 * (1 - p * 0.4));
    } else {
      rotate(p * Math.PI * 3);
      const sz = g.r * (1 - p * 0.4);
      beginShape();
      vertex(0, -sz); vertex(sz * 0.6, 0);
      vertex(0, sz);  vertex(-sz * 0.6, 0);
      endShape(CLOSE);
    }
    pop();
  }
}

// ─── PARSER ───────────────────────────────────────────────────────
function parseStrudelMessage(raw) {
  let msg;
  try { msg = JSON.parse(raw); }
  catch(e) { addDebug('⚠ no-JSON: '+raw.substring(0,38)); return null; }

  // ── NUEVO: interceptar mensajes de globos ──
  if (msg.type === 'globo') {
    globosCount = msg.globos;
    actualizarHUDGlobos();
    const wx = (msg.x || 0.5) * 2000 - 1000 + CAM.x;
    const wy = (msg.y || 0.5) * 2000 - 1000 + CAM.y;
    spawnGloboExplosion(wx, wy);
    if (msg.nivel !== nivelActual) cambiarNivel(msg.nivel);
    return null; // no es un evento de sonido, no lo encolar
  }
  if (msg.type === 'estado') {
    globosCount = msg.globos;
    actualizarHUDGlobos();
    return null;
  }

  let p = {};
  if (Array.isArray(msg.args) && msg.args.length > 0) {
    if (typeof msg.args[0] === 'string') {
      for (let i = 0; i < msg.args.length-1; i+=2)
        if (typeof msg.args[i] === 'string') p[msg.args[i]] = msg.args[i+1];
    } else if (typeof msg.args[0] === 'object' && msg.args[0] !== null) {
      for (const item of msg.args) {
        const k = item.key||item.name||item.type;
        const v = item.value||item.val;
        if (k !== undefined && v !== undefined) p[k] = v;
      }
    }
  }
  if (!Object.keys(p).length && (msg.s||msg.sound||msg.note)) p = msg;
  if (!Object.keys(p).length && msg.value) p = msg.value;
  if (!Object.keys(p).length && msg.address && Array.isArray(msg.args))
    for (let i=0; i<msg.args.length-1; i+=2) p[msg.args[i]] = msg.args[i+1];

  const sound = String(p.s||p.sound||p.n||'unknown');
  let ts = parseFloat(msg.timestamp||msg.time||0);
  if (!ts) ts = Date.now();
  else if (ts < 1e10) ts *= 1000;
  if (ts < Date.now()-2000) ts = Date.now();

  return {
    timestamp: ts,
    sound,
    delta: parseFloat(p.delta)||0.25,
    gain:  parseFloat(p.gain)||0.5,
    note:  p.note||p.midinote||null,
    params: p,
  };
}

// ─── WEBSOCKET ────────────────────────────────────────────────────
function connectWebSocket(port) {
  if (socket) try { socket.close(); } catch(e) {}
  const url = 'ws://localhost:'+port;
  connHud.className = 'connecting';
  connHud.style.opacity = '1';
  connHud.textContent = '◌ CONECTANDO '+url+'...';
  addDebug('Intentando '+url);
  socket = new WebSocket(url);
  socket.binaryType = 'arraybuffer';
  socket.onopen = () => {
    wsConnected = true;
    connHud.className = 'connected';
    connHud.textContent = '● CONECTADO '+url;
    addDebug('✓ CONECTADO puerto '+port);
    setTimeout(() => { connHud.style.opacity='0'; }, 5000);
  };
  socket.onmessage = (event) => {
    totalMessages++;
    let raw = event.data instanceof ArrayBuffer
      ? new TextDecoder().decode(event.data)
      : event.data;
    if (totalMessages <= 3) addDebug('MSG #'+totalMessages+': '+raw.substring(0,50));
    const ev = parseStrudelMessage(raw);
    if (!ev) return;
    lastSound.textContent = '▸ '+ev.sound+(ev.note?' '+ev.note:'')+' ['+totalMessages+']';
    setTimeout(() => { lastSound.textContent = ''; }, 350);
    eventQueue.push(ev);
    eventQueue.sort((a,b) => a.timestamp-b.timestamp);
  };
  socket.onerror = () => { addDebug('✕ Error puerto '+port); connHud.className='error'; connHud.style.opacity='1'; };
  socket.onclose = () => {
    wsConnected = false;
    connHud.className = 'error'; connHud.style.opacity = '1';
    currentPortIndex = (currentPortIndex+1) % WS_PORTS.length;
    const np = WS_PORTS[currentPortIndex];
    connHud.textContent = '◌ Probando '+np+'...';
    addDebug('Cerrado → puerto '+np+' en 2s...');
    setTimeout(() => connectWebSocket(np), 2000);
  };
}

// ─── TILES ────────────────────────────────────────────────────────
function initTiles() {
  TILES.length = 0;
  const half = Math.floor(TILE_GRID/2);
  for (let gx = -half; gx <= half; gx++) {
    for (let gy = -half; gy <= half; gy++) {
      const seed = (gx * 7919 + gy * 104729) & 0xFFFFFF;
      const r = (n) => { let s=seed^n*374761393; s=((s>>>17)|s<<15); s=(s^(s>>>13))^(s<<15); return (s>>>0)/4294967296; };
      TILES.push({
        gx, gy,
        wx: gx * TILE_SIZE,
        wy: gy * TILE_SIZE,
        type: Math.floor(r(1)*8),
        colorIdx: Math.floor(r(2)*Object.keys(C).length),
        rot: r(3) * Math.PI * 2,
        scale: 0.5 + r(4) * 0.8,
        phase: r(5) * Math.PI * 2,
        active: 0,
        flash: 0,
      });
    }
  }
}

// ─── SETUP ────────────────────────────────────────────────────────
function setup() {
  createCanvas(windowWidth, windowHeight);
  colorMode(RGB);
  rectMode(CENTER);
  textAlign(CENTER, CENTER);
  noStroke();
  initTiles();
  initParticles(80);
  connectWebSocket(WS_PORTS[0]);
  document.addEventListener('keydown', onKeyPress);
  addDebug('Visual lista. Esperando bridge...');
}

// ─── CÁMARA ────────────────────────────────────────────────────────
function updateCamera() {
  CAM.tx += CAM.vx;
  CAM.ty += CAM.vy;

  if (frame % 360 === 0) {
    const a = random(TWO_PI);
    const spd = 0.12 + dropIntensity * 0.3;
    CAM.vx = cos(a) * spd;
    CAM.vy = sin(a) * spd;
  }

  if (inDrop) {
    CAM.vx *= 1.004;
    CAM.vy *= 1.004;
    CAM.tzoom = 0.82 + sin(frame * 0.05) * 0.08;
  } else {
    CAM.vx = lerp(CAM.vx, CAM.vx * 0.99, 0.05);
    CAM.vy = lerp(CAM.vy, CAM.vy * 0.99, 0.05);
    CAM.tzoom = 1.0;
  }
  CAM.vx = constrain(CAM.vx, -0.8, 0.8);
  CAM.vy = constrain(CAM.vy, -0.8, 0.8);

  CAM.x = lerp(CAM.x, CAM.tx, 0.04);
  CAM.y = lerp(CAM.y, CAM.ty, 0.04);
  CAM.zoom = lerp(CAM.zoom, CAM.tzoom, 0.03);
  CAM.angle = lerp(CAM.angle, CAM.targetAngle, 0.02);
  CAM.shake *= 0.85;
}

function applyCamera() {
  translate(width/2, height/2);
  if (CAM.shake > 0.2) {
    translate(random(-CAM.shake, CAM.shake), random(-CAM.shake, CAM.shake));
  }
  scale(CAM.zoom);
  rotate(CAM.angle);
  translate(-CAM.x, -CAM.y);
}

// ─── TILES: dibujado ───────────────────────────────────────────────
function drawTiles() {
  for (const t of TILES) {
    if (t.active > 0) t.active -= 0.012;
    if (t.flash > 0)  t.flash  -= 0.035;

    const sx = t.wx - CAM.x;
    const sy = t.wy - CAM.y;
    const cullR = TILE_SIZE * 2.5;
    if (abs(sx) > width/CAM.zoom + cullR || abs(sy) > height/CAM.zoom + cullR) continue;

    push();
    translate(t.wx, t.wy);
    rotate(t.rot + frame * 0.0008 * (t.gx % 2 === 0 ? 1 : -1));
    scale(t.scale * (1 + t.active * 0.25));

    const col = Object.values(C)[t.colorIdx];
    const alpha = 18 + t.active * 120 + t.flash * 180;
    drawTileShape(t.type, col, alpha, t.phase + frame * 0.004, t.active);
    pop();
  }
}

function drawTileShape(type, col, alpha, phase, act) {
  const a = constrain(alpha, 0, 255);
  stroke(col[0], col[1], col[2], a);
  noFill();

  switch(type) {
    case 0:
      strokeWeight(0.8 + act*1.5);
      for (let r2 = 30; r2 <= 120; r2 += 30) {
        beginShape();
        for (let i = 0; i < 6; i++) {
          const ang = (i/6)*TWO_PI + phase;
          vertex(cos(ang)*r2, sin(ang)*r2);
        }
        endShape(CLOSE);
      }
      strokeWeight(0.4);
      for (let i = 0; i < 6; i++) {
        const ang = (i/6)*TWO_PI;
        line(0, 0, cos(ang)*120, sin(ang)*120);
      }
      break;

    case 1:
      strokeWeight(0.7 + act);
      for (let i = 0; i < 5; i++) {
        const s2 = 20 + i*22;
        push(); rotate(i*0.35 + phase); rect(0,0,s2,s2); pop();
      }
      strokeWeight(0.3);
      noStroke(); fill(col[0], col[1], col[2], a*0.08);
      rect(0,0,120,120);
      break;

    case 2:
      strokeWeight(0.8 + act);
      noFill();
      stroke(col[0], col[1], col[2], a);
      const R = 90 + sin(phase)*10;
      triangle(0,-R, cos(PI/6+TWO_PI/3)*R, sin(PI/6+TWO_PI/3)*R, cos(PI/6+2*TWO_PI/3)*R, sin(PI/6+2*TWO_PI/3)*R);
      triangle(0, R, cos(-PI/6+TWO_PI/3)*R, sin(-PI/6+TWO_PI/3)*R, cos(-PI/6+2*TWO_PI/3)*R, sin(-PI/6+2*TWO_PI/3)*R);
      strokeWeight(0.3);
      circle(0,0,60); circle(0,0,120);
      break;

    case 3:
      strokeWeight(0.5);
      for (let gx2 = -2; gx2 <= 2; gx2++) for (let gy2 = -2; gy2 <= 2; gy2++) {
        const px = gx2*40, py2 = gy2*40;
        const dist2 = sqrt(px*px+py2*py2);
        const sz = (1-dist2/140)*14 * (1+act);
        if (sz > 0) {
          stroke(col[0], col[1], col[2], a*(1-dist2/160));
          line(px-sz, py2, px+sz, py2);
          line(px, py2-sz, px, py2+sz);
        }
      }
      break;

    case 4:
      strokeWeight(0.6 + act);
      for (let i = 1; i <= 4; i++) {
        const r3 = i*28 + sin(phase+i)*5;
        stroke(col[0], col[1], col[2], a * (1-i*0.1));
        circle(0, 0, r3*2);
      }
      strokeWeight(0.3);
      stroke(col[0], col[1], col[2], a*0.4);
      for (let i = 0; i < 12; i++) {
        const ang2 = (i/12)*TWO_PI + phase*0.5;
        line(cos(ang2)*28, sin(ang2)*28, cos(ang2)*112, sin(ang2)*112);
      }
      break;

    case 5:
      strokeWeight(0.8 + act*1.2);
      const sz2 = 80 + sin(phase)*15;
      beginShape();
      vertex(0, -sz2); vertex(sz2*0.6, 0);
      vertex(0,  sz2); vertex(-sz2*0.6, 0);
      endShape(CLOSE);
      strokeWeight(0.4);
      for (let i = 1; i <= 3; i++) {
        const f = i/3;
        beginShape();
        vertex(0, -sz2*f); vertex(sz2*0.6*f, 0);
        vertex(0,  sz2*f); vertex(-sz2*0.6*f, 0);
        endShape(CLOSE);
      }
      line(-sz2*0.6, 0, sz2*0.6, 0);
      line(0, -sz2, 0, sz2);
      break;

    case 6:
      strokeWeight(0.7);
      noFill();
      beginShape();
      for (let ang3 = 0; ang3 < TWO_PI*4; ang3 += 0.08) {
        const r4 = ang3 * 8 + sin(phase)*5;
        if (r4 > 130) break;
        const sc3 = 1-r4/130;
        stroke(col[0], col[1], col[2], a*sc3);
        vertex(cos(ang3+phase)*r4, sin(ang3+phase)*r4);
      }
      endShape();
      break;

    case 7:
      strokeWeight(0.8 + act);
      beginShape();
      for (let i = 0; i <= 24; i++) {
        const ang4 = (i/24)*TWO_PI + phase*0.3;
        const rr = i%2===0 ? 90+sin(phase+i)*12 : 35+cos(phase*2+i)*8;
        vertex(cos(ang4)*rr, sin(ang4)*rr);
      }
      endShape(CLOSE);
      strokeWeight(0.3);
      circle(0, 0, 50);
      stroke(col[0], col[1], col[2], a*0.3);
      for (let i = 0; i < 8; i++) {
        const ang5 = (i/8)*TWO_PI;
        line(cos(ang5)*35, sin(ang5)*35, cos(ang5)*90, sin(ang5)*90);
      }
      break;
  }
}

// ─── PARTÍCULAS ────────────────────────────────────────────────────
function initParticles(n) {
  particles = [];
  for (let i = 0; i < n; i++) {
    const isC = random() < 0.4;
    particles.push({
      wx: random(-TILE_SIZE*4, TILE_SIZE*4),
      wy: random(-TILE_SIZE*4, TILE_SIZE*4),
      r: random(0.5, 2.2),
      vx: random(-0.4,0.4), vy: random(-0.4,0.4),
      alpha: random(25,90),
      color: isC ? C.cyan : C.purple,
    });
  }
}

function updateDrawParticles() {
  for (const p of particles) {
    p.wx += p.vx * (1+dropIntensity*2);
    p.wy += p.vy * (1+dropIntensity*2);
    const bound = TILE_SIZE*4.5;
    if (p.wx < -bound) p.wx = bound;
    if (p.wx >  bound) p.wx = -bound;
    if (p.wy < -bound) p.wy = bound;
    if (p.wy >  bound) p.wy = -bound;
  }
  noStroke();
  for (const p of particles) {
    fill(p.color[0], p.color[1], p.color[2], p.alpha);
    circle(p.wx, p.wy, p.r*2);
  }
  strokeWeight(0.3);
  for (let i = 0; i < particles.length; i++) {
    for (let j = i+1; j < particles.length; j++) {
      const dx = particles[i].wx-particles[j].wx;
      const dy = particles[i].wy-particles[j].wy;
      const d  = sqrt(dx*dx+dy*dy);
      if (d < 70+dropIntensity*30) {
        stroke(C.purple[0], C.purple[1], C.purple[2], (1-d/70)*22);
        line(particles[i].wx, particles[i].wy, particles[j].wx, particles[j].wy);
      }
    }
  }
}

// ─── PULSOS ────────────────────────────────────────────────────────
function drawPulses() {
  noFill();
  for (let i = pulseRings.length-1; i >= 0; i--) {
    const p = pulseRings[i];
    p.r    += p.speed || 6;
    p.alpha -= p.big ? 0.015 : 0.038;
    if (p.alpha <= 0) { pulseRings.splice(i,1); continue; }
    stroke(p.color[0], p.color[1], p.color[2], p.alpha*255);
    strokeWeight(p.big ? 1.5 : 0.8);
    circle(p.wx||0, p.wy||0, p.r*2);
  }
}

// ─── DROP STATE ────────────────────────────────────────────────────
function updateDropState() {
  const heavy = activeAnims.filter(a =>
    a.type==='bd'||a.type==='sd'||a.type==='sawtooth'||a.type==='square').length;
  if (heavy > 3) {
    if (!inDrop) inDrop = true;
    dropIntensity = min(1, dropIntensity+0.025);
  } else {
    dropIntensity = max(0, dropIntensity-0.01);
    if (dropIntensity < 0.05) inDrop = false;
  }
}

// ─── SPAWN ─────────────────────────────────────────────────────────
function spawnAnimation(ev) {
  const col = colorForSound(ev.sound);
  const now = Date.now();

  const wx = CAM.x + random(-300, 300);
  const wy = CAM.y + random(-300, 300);

  activeAnims.push({
    startTime: now,
    duration:  max(ev.delta*1000, 120),
    type: ev.sound, wx, wy, color: col, gain: ev.gain,
  });

  let closest = null, closestD = Infinity;
  for (const t of TILES) {
    const d = dist(wx, wy, t.wx, t.wy);
    if (d < closestD) { closestD = d; closest = t; }
  }
  if (closest) {
    closest.active = min(1, closest.active + ev.gain * 0.6 + 0.4);
    closest.flash  = 1;
    closest.colorIdx = Object.keys(C).indexOf(
      Object.keys(C)[Math.floor(random(Object.keys(C).length))]
    ) || 0;
  }

  if (ev.sound === 'bd') {
    CAM.shake = max(CAM.shake, 6 * ev.gain);
    CAM.tzoom = 0.88;
    pulseRings.push({wx, wy, r:20, color: col, alpha:0.75, big:true, speed:9});
    CAM.tx += random(-60, 60);
    CAM.ty += random(-60, 60);
  }
  if (ev.sound === 'sd') {
    CAM.shake = max(CAM.shake, 3 * ev.gain);
    CAM.targetAngle += random(-0.018, 0.018);
    pulseRings.push({wx, wy, r:10, color: col, alpha:0.6, big:false, speed:11});
  }
  if (['hh','oh'].includes(ev.sound)) {
    CAM.vx += random(-0.08, 0.08);
    CAM.vy += random(-0.08, 0.08);
  }
}

// ─── FORMAS DE SONIDO ─────────────────────────────────────────────
function drawActiveAnims() {
  const now = Date.now();
  for (let i = activeAnims.length-1; i >= 0; i--) {
    const a = activeAnims[i];
    const p = (now - a.startTime) / a.duration;
    if (p > 1) { activeAnims.splice(i,1); continue; }
    push();
    translate(a.wx, a.wy);
    const c = a.color;

    switch(a.type) {
      case 'bd':
        { const d = lerp(40, 260, p); const al = lerp(240, 0, p);
          noFill(); stroke(c[0],c[1],c[2],al); strokeWeight(lerp(3,0.3,p));
          circle(0, 0, d);
          const d2 = lerp(15, 130, pow(p,0.6)); const al2 = lerp(160, 0, p);
          stroke(c[0],c[1],c[2],al2); strokeWeight(0.8);
          circle(0, 0, d2);
          stroke(c[0],c[1],c[2],al2*0.5); strokeWeight(0.5);
          const half2 = d*0.45;
          line(-half2,0,half2,0); line(0,-half2,0,half2);
        }
        break;
      case 'sd':
        { const al = lerp(210, 0, p);
          noFill(); stroke(c[0],c[1],c[2],al); strokeWeight(1.2);
          const w2 = lerp(140, 0, p), h2 = lerp(8, 1, p);
          rect(0, 0, w2, h2);
          stroke(c[0],c[1],c[2],al*0.4); strokeWeight(0.4);
          for (let y2 = -20; y2 <= 20; y2 += 8)
            line(-70*(1-p), y2, 70*(1-p), y2);
        }
        break;
      case 'hh':
        { push(); rotate(p*TWO_PI*3);
          const sz2 = lerp(22, 1, p); const al = lerp(210, 0, p);
          noFill(); stroke(c[0],c[1],c[2],al); strokeWeight(0.9);
          rect(0,0,sz2,sz2);
          stroke(c[0],c[1],c[2],al*0.5); strokeWeight(0.4);
          rect(0,0,sz2*1.7,sz2*1.7);
          pop();
        }
        break;
      case 'oh':
        { const sz3 = lerp(10, 80, p); const al = lerp(190, 0, p);
          noFill(); stroke(c[0],c[1],c[2],al); strokeWeight(1);
          push(); rotate(p*PI);
          triangle(0,-sz3, sz3*0.85, sz3*0.5, -sz3*0.85, sz3*0.5);
          pop();
        }
        break;
      case 'white':
        { const al = lerp(90, 0, p); const r2 = lerp(5, 55, p);
          noStroke();
          for (let k = 0; k < 8; k++) {
            const ang = (k/8)*TWO_PI + p*PI;
            fill(c[0],c[1],c[2], al*(1-k*0.1));
            circle(cos(ang)*r2*0.5, sin(ang)*r2*0.5, r2*0.35);
          }
        }
        break;
      case 'sawtooth':
        { const al = lerp(180, 0, p); const sc4 = lerp(0.2, 1.4, p);
          noFill(); stroke(c[0],c[1],c[2],al); strokeWeight(1.1);
          beginShape();
          for (let k = 0; k < 8; k++) {
            vertex((-80+k*20)*sc4, k%2===0 ? -30*sc4 : 30*sc4);
          }
          endShape();
        }
        break;
      case 'triangle':
        { const al = lerp(200, 0, p);
          push(); rotate(p*PI*2);
          const sz4 = lerp(50, 5, p);
          noFill(); stroke(c[0],c[1],c[2],al); strokeWeight(1.2);
          triangle(0,-sz4, sz4*0.85, sz4*0.5, -sz4*0.85, sz4*0.5);
          stroke(c[0],c[1],c[2],al*0.3); strokeWeight(0.4);
          circle(0, 0, sz4*2.4);
          pop();
        }
        break;
      case 'square':
        { const al = lerp(190, 0, p);
          for (let k = 0; k < 3; k++) {
            const sc5 = (1-k*0.3) * lerp(1, 0.1, p);
            push(); rotate(p*PI*(k%2===0?1:-1));
            const side = (60+k*15)*sc5;
            noFill(); stroke(c[0],c[1],c[2], al*(1-k*0.2)); strokeWeight(1-k*0.2);
            rect(0, 0, side, side);
            pop();
          }
        }
        break;
      default:
        { const sz5 = lerp(55, 3, p); const al = lerp(190, 0, p);
          push(); rotate(p*TWO_PI*1.3);
          noFill(); stroke(c[0],c[1],c[2],al); strokeWeight(1.1);
          beginShape();
          vertex(0,-sz5); vertex(sz5*0.55,0);
          vertex(0,sz5); vertex(-sz5*0.55,0);
          endShape(CLOSE);
          strokeWeight(0.4); stroke(c[0],c[1],c[2],al*0.4);
          line(-sz5*0.55,0,sz5*0.55,0); line(0,-sz5,0,sz5);
          circle(0,0,sz5*0.5);
          pop();
        }
    }
    pop();
  }
}

// ─── GRID ──────────────────────────────────────────────────────────
function drawWorldGrid() {
  const alpha = 8 + dropIntensity*18;
  stroke(C.purple[0], C.purple[1], C.purple[2], alpha);
  strokeWeight(0.2);
  const spacing = TILE_SIZE;
  const range = 5;
  const cx = round(CAM.x / spacing) * spacing;
  const cy = round(CAM.y / spacing) * spacing;
  for (let i = -range; i <= range; i++) {
    line(cx+i*spacing, cy-range*spacing, cx+i*spacing, cy+range*spacing);
    line(cx-range*spacing, cy+i*spacing, cx+range*spacing, cy+i*spacing);
  }
}

// ─── COLOR POR SONIDO ─────────────────────────────────────────────
function colorForSound(s) {
  const map = {
    bd:'pink', sd:'cyan', hh:'gold', oh:'gold',
    white:'purple', sawtooth:'violet', triangle:'ice', square:'lime',
  };
  const key = map[s];
  if (key && C[key]) return C[key];
  let h = 0;
  for (let i = 0; i < s.length; i++) h = (h*31+s.charCodeAt(i))&0xFFFFFF;
  return [((h>>16)&0xFF)*0.6+80, ((h>>8)&0xFF)*0.6+80, (h&0xFF)*0.6+80];
}

// ─── DRAW PRINCIPAL ────────────────────────────────────────────────
function draw() {
  frame++;
  const trailAlpha = inDrop ? 14 : 22;
  background(0, 0, 8, trailAlpha);

  updateCamera();
  updateDropState();

  const now = Date.now();
  while (eventQueue.length > 0 && now >= eventQueue[0].timestamp) {
    spawnAnimation(eventQueue.shift());
  }

  push();
  applyCamera();

  drawWorldGrid();
  drawTiles();
  updateDrawParticles();
  drawActiveAnims();
  drawGloboAnims();   // NUEVO
  drawPulses();

  pop();

  // Flash de tecla
  if (keyFlash > 0) {
    const col = [C.purple, C.cyan, C.pink][floor(random(3))];
    fill(col[0], col[1], col[2], keyFlash*1.2);
    rect(width/2, height/2, width, height);
    keyFlash = max(0, keyFlash-6);
  }

  // NUEVO: timer del flash de nivel
  if (nivelFlashTimer > 0) {
    nivelFlashTimer--;
    if (nivelFlashTimer === 0) flashEl.classList.remove('show');
  }

  drawVignette();
}

function drawVignette() {
  noStroke();
  for (let i = 0; i < 6; i++) {
    const f = i/6;
    fill(0, 0, 8, (f*f) * 120);
    rect(width/2, height/2, width, height);
  }
}

// ─── TECLADO ──────────────────────────────────────────────────────
function onKeyPress(e) {
  keyFlash = 55;
  CAM.shake = max(CAM.shake, 12);
  CAM.tzoom = 0.85;
  const a = random(TWO_PI);
  CAM.tx += cos(a) * 80;
  CAM.ty += sin(a) * 80;
  CAM.targetAngle += random(-0.04, 0.04);

  const nearby = TILES
    .map(t => ({t, d: dist(CAM.x, CAM.y, t.wx, t.wy)}))
    .sort((a2,b) => a2.d - b.d)
    .slice(0, 5);
  for (const {t} of nearby) {
    t.active = min(1, t.active + 0.7);
    t.flash  = 1;
  }

  const col = [C.purple, C.cyan, C.pink][floor(random(3))];
  pulseRings.push({ wx: CAM.x, wy: CAM.y, r:15, color: col, alpha:0.7, big:true, speed:10 });
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
</script>
</body>
</html>
```
y un pequeño video de como funcionan



https://github.com/user-attachments/assets/91f0a442-2faf-4c90-a773-1e303f5ee86c



## Bitácora de reflexión

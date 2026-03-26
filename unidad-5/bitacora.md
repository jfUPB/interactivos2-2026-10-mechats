# Unidad 5
## Bitácora de proceso de aprendizaje

###¿Qué quieres comunicar o provocar con tu obra?

Quiero que sea una cancion muy technno, que active al publico de forma frenetica pero tambien que lo haga mover la cabeza por lo menos, con sonidos agresivos pero melodicos que hagan una combinación perfecta, quiero comunicar euforia, alegria, que la cancion les diga literalmente "parchate"

###¿En qué contexto se presentará?

Me gustaria que sea en una sala con poca luz, pero que cuente con luces que sumerjan al espectador en la sintonia punsante de la canción

###¿Cuál es la experiencia que deseas para el público? 

Quiero que sea inmersiva, que el usuario pueda interactuar con las cosas que pasan en las visuales

EJEMPLO: que en las visuales apaarezcan gotas o globos y cuando ellos los presionen en sus celulares exploten pintando la pantalla

### Rol del publico

El publico tendra un rol de participante, no modificaran las visuaes pero interactuaran con ellas

### Referentes artisticos

Amygdala
Daft Punk
gary vs david (un show mas)

### Referentes artisticos

https://www.youtube.com/watch?v=EaCUyNQWY2M

### Esta cancion usa sintetizadores muy buenos, que me van a ayudar a darme una idea de como quiero que sean los de mi canción

https://www.youtube.com/watch?v=Qccjcwxb0D0

Esta canción va a ser muy util para mi inicio y mi pre puente por sus bombos rapidos


### Proceso de creación

Al principio le queria meter voces, pero algo raro pasaba, y es que como eran audios tan largos se mezclaba con el ciclo de las otras voces, al final decidi quitarlos porque era dificil combinarlos con sonidos. Pero logre crear algo que en mi opinion esta bastante decente para mi objetivo

```




setcps(0.55)

// ============================================
// INSTRUMENTOS
// ============================================

// --- RUIDO BLANCO ---
const ruido = sound("white")
  .gain(sine.range(0, 0.6).slow(4))
  .lpf(sine.range(200, 4000).slow(4))

// --- BOMBO PRINCIPAL ---
const bombo = sound("bd")
  .gain(0.8)
  .struct("1 ~ 1 ~")
 // .fast(2)

// --- BOMBO SUAVE ---
const bombo_suave = sound("bd")
  .gain(sine.range(0.2, 0.5).slow(4))
  .struct("1 ~ ~ ~")
  .room(0.7)
  .size(0.6)
  .lpf(300)

// --- GUITARRA ELÉCTRICA GRAVE ---
const guitarra = note("a1 b1 c2 d2 e2 f2 g2 a2")
  .sound("sawtooth")
  .lpf(900)
  .distort(0.4)
  .room(0.3)
  .gain(0.6)
  .fast(2)

// --- PIANO GRAVE ---
const piano = note("a2 c3 e3 g3 a3 g3 e3 c3")
  .sound("triangle")
  .distort(0.15)
  .room(0.85)
  .size(0.8)
  .delay(0.3)
  .delaytime(0.25)
  .delayfeedback(0.4)
  .gain(0.55)
  .fast(2)

// --- PIANO MEDIO ---
const piano2 = note("a4 c5 e5 g5 a5 g5 e5 c5")
  .sound("triangle")
  .room(0.9)
  .size(0.85)
  .delay(0.35)
  .delaytime(0.2)
  .delayfeedback(0.45)
  .gain(0.4)
  .fast(2)

// --- PIANO AGUDO ---
const piano3 = note("a5 c6 e6 g6 a6 g6 e6 c6")
  .sound("triangle")
  .room(0.9)
  .size(0.85)
  .delay(0.35)
  .delaytime(0.2)
  .delayfeedback(0.45)
  .gain(0.4)
  .fast(2)

// --- PIANO MUY AGUDO ---
const piano4 = note("a6 c7 e7 g7 a7 g7 e7 c7")
  .sound("triangle")
  .room(0.9)
  .size(0.85)
  .delay(0.35)
  .delaytime(0.2)
  .delayfeedback(0.45)
  .gain(0.4)
  .fast(2)

// --- SAXOFÓN ---
const saxofon = note("c5")
  .sound("sawtooth")
  .lpf(sine.range(800, 2400).slow(4))
  .attack(0.15)
  .release(0.8)
  .distort(0.25)
  .vib(4)
  .vibmod(0.3)
  .room(0.4)
  .size(0.5)
  .gain(0.6)
  .slow(1)



// --- BATERÍA SUAVE ---
const caja = sound("sd")
  .struct("~ 1 ~ 1")
  .gain(0.35)
  .room(0.7)
  .size(0.6)
  .lpf(600)
  .fast(1)

const hihat = sound("hh")
  .struct("1 ~ 1 ~")
  .gain(0.18)
  .room(0.5)
  .fast(1)

// --- SINTETIZADOR TECHNO ---
const synth_techno = note("a2 ~ a2 ~ g2 ~ a2 ~")
  .sound("sawtooth")
  .lpf(sine.range(300, 1800).slow(2))
  .resonance(15)
  .attack(0.05)
  .release(0.3)
  .distort(0.35)
  .room(0.4)
  .size(0.4)
  .gain(0.55)

// --- BAJO PULSANTE ---
const bajo_techno = note("a1 ~ ~ a1 ~ ~ g1 ~")
  .sound("square")
  .lpf(400)
  .resonance(8)
  .attack(0.02)
  .release(0.2)
  .distort(0.2)
  .room(0.3)
  .gain(0.6)

// --- BUM ---
const bum = sound("bd")
  .gain(0.9)
  .struct("1 ~ ~ ~")
  .room(0.2)
  .size(0.2)
  .lpf(150)
  .fast(2)

// --- WHOOSH DEL PUENTE ---
const whoosh_puente = sound("white")
  .lpf(saw.range(120, 8000).slow(2))
  .hpf(150)
  .gain(saw.range(0.0, 0.5).slow(4))
  .room(0.98)
  .size(0.99)
  .pan(sine.range(-0.4, 0.4).slow(0.3))

// --- SINTETIZADOR DEL PUENTE ---
const synth_puente = note("a2 ~ a2 ~ g2 ~ a2 ~")
  .sound("sawtooth")
  .lpf(sine.range(300, 800).slow(4))
  .resonance(20)
  .attack(0.3)
  .release(1.0)
  .distort(0.2)
  .room(0.9)
  .size(0.9)
  .gain(0.3)
  .slow(2)

// --- BOMBO DEL PUENTE ---
const bombo_puente = sound("bd")
  .struct("1 ~ ~ ~")
  .gain(0.4)
  .room(0.6)
  .size(0.5)
  .lpf(200)

// --- GOLPE DE ENTRADA AL DROP ---
const golpe_drop = sound("bd")
  .struct("1 1 ~ ~")
  .gain(1.2)
  .room(0.05)
  .size(0.05)
  .lpf(150)
  .distort(0.5)

// ============================================
// INSTRUMENTOS DEL DROP
// ============================================

// --- SINTETIZADOR DEL DROP ---
const synth_techno1 = note("a2 ~ a2 ~ g2 ~ a2 ~")
  .sound("sawtooth")
  .lpf(sine.range(300, 1800).slow(2))
  .resonance(15)
  .attack(0.05)
  .release(0.3)
  .distort(0.35)
  .room(0.4)
  .size(0.4)
  .gain(0.55)

// --- BAJO DEL DROP ---
const bajo_techno1 = note("a1 ~ ~ a1 ~ ~ g1 ~")
  .sound("square")
  .lpf(400)
  .resonance(8)
  .attack(0.02)
  .release(0.2)
  .distort(0.2)
  .room(0.3)
  .gain(0.6)

// --- BUM DEL DROP ---
const bum_drop1 = sound("bd")
  .struct("1 ~ 1 ~")
  .gain(1.1)
  .room(0.05)
  .size(0.05)
  .lpf(180)
  .distort(0.6)

// --- CAJA DEL DROP ---
const caja_drop1 = sound("sd")
  .struct("~ 1 ~ 1")
  .gain(0.85)
  .room(0.15)
  .size(0.15)
  .distort(0.4)
  .lpf(4000)

// --- HIHAT DEL DROP ---
const hihat_drop1 = sound("hh")
  .struct("1 1 1 1 1 1 1 1")
  .gain(0.45)
  .room(0.1)
  .distort(0.2)

// --- RUIDO INDUSTRIAL DEL DROP ---
const ruido_drop1 = sound("white")
  .struct("~ 1 ~ 1")
  .gain(0.3)
  .lpf(3000)
  .hpf(800)
  .distort(0.6)
  .room(0.1)

// --- GUITARRA 1 DEL DROP ---
const guitarra_drop1 = note("a2 ~ a2 ~ g2 ~ a2 ~")
  .sound("sawtooth")
  .lpf(2000)
  .distort(0.8)
  .room(0.3)
  .size(0.3)
  .gain(0.9)

// --- GUITARRA 2 DEL DROP ---
const guitarra_drop2 = note("a2 ~ a2 ~ g2 ~ a2 ~")
  .sound("sawtooth")
  .lpf(1600)
  .distort(0.95)
  .room(0.2)
  .size(0.2)
  .pan(0.5)
  .gain(0.85)

// --- GUITARRA 3 DEL DROP ---
const guitarra_drop3 = note("a3 ~ a3 ~ g3 ~ a3 ~")
  .sound("sawtooth")
  .lpf(2400)
  .distort(0.7)
  .room(0.25)
  .size(0.25)
  .pan(-0.5)
  .gain(0.8)

// --- SAXOFÓN DEL DROP ---
const saxofon_drop1 = note("a4 ~ a4 ~ g4 ~ a4 ~")
  .sound("sawtooth")
  .lpf(sine.range(1200, 3500).slow(4))
  .attack(0.15)
  .release(0.8)
  .distort(0.4)
  .vib(4)
  .vibmod(0.3)
  .room(0.4)
  .size(0.5)
  .gain(0.85)

// --- SINTETIZADOR DEL CIERRE ---
const cierre_final = note("a2 ~ a2 ~ g2 ~ a2 ~")
  .sound("sawtooth")
  .lpf(sine.range(300, 800).slow(4))
  .resonance(15)
  .attack(0.05)
  .release(2.0)
  .distort(0.2)
  .room(0.98)
  .size(0.99)
  .gain(saw.range(0.55, 0.0).slow(4))

// ============================================
// SECCIONES
// ============================================

const fundido        = stack(ruido)
const inicio         = stack(bombo, guitarra)
const inicio_1       = stack(bombo, guitarra, piano)
const inicio_2       = stack(bombo, piano, piano2)
const inicio_3       = stack(bombo, piano, piano2, piano3)
const inicio_4       = stack(bombo, piano, piano2, piano3, piano4)
const transicion     = stack(bombo_suave, saxofon)
const bateria        = stack(caja, hihat)
const puente         = stack(bateria, synth_techno, bajo_techno)
const puente1        = stack(bateria, synth_techno, bajo_techno, bum)
const puente2        = stack(bateria, synth_techno, bajo_techno, bum)
const puentef        = stack(whoosh_puente, synth_puente, bombo_puente)
const entrada_drop   = stack(golpe_drop)
const drop1          = stack(synth_techno1, bajo_techno1, bum_drop1, caja_drop1, hihat_drop1, ruido_drop1, guitarra_drop1, guitarra_drop2, guitarra_drop3, saxofon_drop1)
const cierre_1       = stack(synth_techno1, bajo_techno1, bum_drop1, caja_drop1, hihat_drop1, ruido_drop1)
const cierre_2       = stack(synth_techno1, bajo_techno1, bum_drop1)
const cierre_3       = stack(synth_techno1, bajo_techno1)

// ============================================
// ARRANGE
// ============================================

arrange(
  [2,  fundido],
  [4,  inicio],
  [4,  inicio_1],
  [4,  inicio_2],
  [2,  inicio_3],
  [2,  inicio_4],
  [1,  saxofon],
  [2,  transicion],
  [4,  bateria],
  [2,  puente],
  [2,  puente1],
  [4,  puente2],
  [4,  puentef],
  [1,  entrada_drop],
  [1,  saxofon_drop1],
  [1,  guitarra_drop3],
  [8,  drop1],
  [2,  cierre_1],
  [2,  cierre_2],
  [2,  cierre_3],
  [4,  cierre_final],
)
```

## Bitácora de aplicación 


## Bitácora de reflexión

De las personas que les pregunte, si, logre el objetivo

<img width="1407" height="738" alt="image" src="https://github.com/user-attachments/assets/0a4967dd-5652-4ff9-bd1d-a5953d003d97" />


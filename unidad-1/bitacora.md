# Unidad 1

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

### Idea inicial: quiero hacer un ritmo algo parecido al technno (por que me gusta el technno) que contenga una bateria bastante pegadiza



Empece guiandome con este ejemplo de strudel porque me enseñaba como hacer que una serie de notas se repetieran y como yo podia controlar cuantas veces se repetia

<img width="1861" height="791" alt="image" src="https://github.com/user-attachments/assets/fa0a3acc-7afe-4001-acdf-c735fba66fc3" />

Con este ejemplo me di cuenta que puedo nombrar a un grupo de sonidos como una constante y asi mismo utilizarlos en un stack mucho mas armado y complejo. Tambien descubri la funcion arrange que me permite hacer que varias cosas entren despues.

Despues, como no se nada de teoria musical decidi buscar en los ejemplos de strudel y me encontre con un bass bajo que sonaba bastante bien, pero lo modifique para que al principio de mi canción sonara unas 3  escalas mas arriba (le sume 12 a todas las tonalidades)

<img width="718" height="258" alt="image" src="https://github.com/user-attachments/assets/badbc282-b338-42c3-8e57-48f74d031467" />

Un problema que tuve es que queria hacer que unos instrumentos sonaran mas tarde que otros, pero eso lo solucione con la funcion arrange, que me permitia controlar cuanto tiempo queria que una serie de instrumentos estuviera en silencio, logrando asi que todos entraran cuando yo queria. 


<img width="807" height="188" alt="image" src="https://github.com/user-attachments/assets/61b454c1-c557-4f6f-8887-9824a6830066" />


Por ultimo algo que hice mas por decision creativa que optima fue poner todos mis sonidos en stacks, asi fuese solo uno. Pues esto me permitia ponerlos en otros stacks sin que hubiese tanta linea de codigo, poderlos reutilizar de una forma mas rapida y aparte podia añadirles efectos, como es este caso, que un const llamado principal, que esta metido en otro stack. Le añadi un efecto de velocidad

<img width="538" height="425" alt="image" src="https://github.com/user-attachments/assets/1e220d7e-90ad-4654-bca2-de61b4854d88" />


### codigo final:

```javascript

setcps(0.5)


const inicio = stack (
 
note("<[74 86]*4 [72 84]*4 [70 82]*4 [68 80]*4>")
  .s("piano").room(0.3),
  
 
)

const principal = stack (note("<[50 62]*4 [48 60]*4 [46 58]*4 [44 56]*4>")
.sound("gm_acoustic_bass"),
inicio
 )

const preboom = stack(

  s("bd:5*4") 
    .gain(1.2),
  s("hh*8") 
    .gain(0.7), 

  s("~ cp") 
    .gain(0.8)
    .delay(0.2).delayfb(0.3), 

  s("~ [hh:4*2]") 
    .gain(0.9),

  principal.fast("1 1 2")
)

const puente = stack(

  s("cp:2")
    .gain(1.3)
    .room(1)    
    .sz(0.9)    
    .sustain(2),    


  s("noise")

    .room(0.5),



)
.slow(5) 


const final = stack(
preboom.room(0.7),

note("<[74 86]*4 [72 84]*4 [70 82]*4 [68 80]*4>")
    .s("square").gain(0.4),

  note("<[50 62]*4 [48 60]*4 [46 58]*4 [44 56]*4>")
    .s("triangle").gain(0.4)

  
)


  

  

  
  

stack(
  arrange([4,inicio], [18, silence]),
  arrange([4, silence], [4, principal], [4, preboom], [10, silence]),
  arrange([12, silence], [1, puente], [8, final])
  )

            

```












## Bitácora de reflexión



<img width="1576" height="737" alt="image" src="https://github.com/user-attachments/assets/5a670b71-db5e-4dd2-a14e-a2858dacfc58" />





# Unidad 8

## Bitácora de proceso de aprendizaje

### Diagrama del proyecto

<img width="708" height="499" alt="image" src="https://github.com/user-attachments/assets/2751ce24-e1d6-48d1-8279-95e54989610a" />

### Requisitos para reprodcuir el sistema

##### Requisitos de hardware
laptop o computador con al menos 8 GB de RAM
Tarjeta de sonido (integrada o externa)
proyector o pantalla conectado por HDMI
eed WiFi local (router o punto de acceso)
Uno o más smartphones con navegador web 

##### Requisitos de software

Node.js v18 o superior — nodejs.org
Google Chrome para las visuales y para Strudel
Strudel REPL  entorno de live coding, se ejecuta localmente

#### Como reproducirlo

Primero hay que isntalar el streudel Local, para que pueda enviar los datos al bridge, en una carpeta vamos a clonar el repositorio que lo contiene usando:

git clone https://github.com/tidalcycles/strudel.git (esto es un ejemplo, no es exactamente el mismo link del repositorio de strudel)
cd strudel
pnpm install
pnpm run dev

despues tenemos que ir a el repositorio que nos proporciono juanferfranco.github

https://github.com/juanferfranco/strudelP5-tests.git

y clonar el bridge e instalar las dependencias de node  y ws (websocket)

despues de esto, creamos la carpeta del proyecto donde vamos a poner las dos carpetas que acabamos de instalar (strudel local y brisge)

Nos vamos a ir al strudel local y en la carpeta website vamos a colocar el codigo de nuestras visuales como un html

### Importante

El bridge te va a dar el puerto en el que se esta transmitiendo, y podras abrir el strudel local donde tienes tu pieza musical y las visuales, pero se necesita tambien la interaccion con el publico. En mi proyecto se debe consultar la IP local para pasarle ese link al publico y que puedan entrar, entonces se deben realizar estos pasos en la consola de windows

ipconfig

y de ahi en la informaicon que nos aparece tenemos que buscar "Dirección IPv4" bajo WiFi. Esa va a ser la ip para poner en el link de los computadores del publico

Una vez ya se tenga todo eso se debe abrir el proyecto

##### En el bridge

se debe escribir en una terminal el comando node bridge.js

##### En el Strudel local

se debe poner el comando npm run dev

como pudimos ver en el diagrama el publico esta en el puerto 3000, entonces el link que hay que pasarles es mas o menos asi http://10.8.116.135:3000 (Hay que cambiarlo dependiendo de la IP)


#### como funciona cada componente del sistema
 Strudel es el entorno de live coding donde se compone y ejecuta la música en tiempo real y genera un evento por cada nota o sonido, con parámetros como el tipo de sonido y la duración,  los envía como JSON por WebSocket. El bridge es un servidor Node.js y recibe esos eventos de Strudel, los reenvía a las visuales, y al mismo tiempo gestiona la conexión con los móviles del público. Las visuales son un archivo HTML con p5.js que recibe los eventos del bridge y los traduce en formas geométricas animadas sobre un espacio infinito de mosaicos por el que una cámara navega continuamente y por ultimo el  móvil del público carga una página web servida por el mismo bridge y permite que cada persona explote un globo virtual, enviando un evento con la posición del toque que altera las visuales en tiempo real

 #### Justifica las decisiones técnicas: ¿Por qué esa herramienta?, ¿Por qué ese protocolo?, ¿Por qué esa arquitectura?

 Para iniciar hablemos de el bridge, y es que antes en clase usabamos dos bridge para visuales y audio con fin educativo. Pero con fin de eficencia era mejor usar uno que nos resolviera los dos problemas. Tambien use websocket en varios puertos porque por ejemplo encesitaba que las visuales enviaran y recibieran datos, algo que ese protocolo permite 


 #### Justifica las decisiones estéticas: ¿Por qué esos sonidos?, ¿Por qué esas visuales?, ¿Por qué esa forma de participación del público?
 
 Los sonidos son los bloques básicos del techno: percusivos, repetitivos y hipnoticos. La decisión de usarlos y de crear un techno va principalmente en el hecho de que compagina muy bien la idea de musica techno (electronica, sintetizada) con el tema del live coding. Ademas no es solo estilística sino estructural porque cada tipo de sonido activa una forma visual distinta, creando un lenguaje donde la música es legible en pantalla. Las visuales usan una paleta de púrpura, cian, rosa y dorado porque son los colores asociados a la noche y los clubes. Que refuerzan la atmósfera de trance que el techno produce. Ademas el sistema de cámara que navega infinitamente por un mundo de mosaicos geométricos responde a esa misma idea y es que  no hay un punto fijo, no hay reposo, el espacio siempre se mueve como en un estado alterado

 
#### Conexión con el concepto artístico
 parte de una pregunta sobre el poder dentro de una performance ¿quién controla lo que ocurre en escena? El live coding como práctica ya propone una respuesta parcial pero yo trate de que esta obra extiende esa lógica al público así como el artista interviene el sistema desde su teclado, el público lo interviene desde su teléfono, y paa darle mas independencia o autoridad al publico decidi que yo no iba a intervenir en las visuales. Solo ellos, y es por eso que  Los tres umbrales de globos definen una curva de caos creciente que el artista no controla

 #### Prueba de reproduccion

 Hubo solo una confusión y era que mi amiguita sofia pensaba que en unn solo repositorio estaban las dos cosas necesarias (brisge y strudel) asi que aclare que eran dos diferentes en la bitacora.


 


 




## Bitácora de aplicación 


## Bitácora de reflexión

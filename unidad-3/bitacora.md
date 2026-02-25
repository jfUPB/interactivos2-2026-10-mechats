# Unidad 3

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 


Lo primero que tenía que hacer para usar el Open Stage era lograr que este se conectara con las visuales, y a la vez el puente de las visuales transmitiera ambas cosas a  el server del strudel. Para esto tocaba crear otro puente. El problema principal era que el poenstage estaba configurado para lanzar datos desde el puerto 8082. Pero no se puede comunicar por un mismo "canal" desde dos puentes diferentes, por eso tocaba enviar los datos desde otro, en este caso el 9000. Ademas el open stage necesitaba un lugar donde alojarse para enbiarle sus datos al bridge. En este caso el 8083, todo eso lo configure desde el open stage

<img width="779" height="573" alt="image" src="https://github.com/user-attachments/assets/2367ff1d-a9e0-4e5c-a99d-499c150d0990" />

Para verifiar que si se estuviesen recibiendo los datos use la misma linea de codigo que usamos en el puente de strudel, pero esta vez en open stage

<img width="1090" height="491" alt="image" src="https://github.com/user-attachments/assets/57e4b2dd-931e-43b7-8d5c-9925effdf817" />








## Bitácora de reflexión

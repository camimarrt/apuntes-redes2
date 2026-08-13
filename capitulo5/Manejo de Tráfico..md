# 1. Congestión:

Cuando existen demasiados paquetes circulando en la red se producen retrasos y pérdidas de paquetes que degradan el rendimiento. Es la principal necesidad de manejo de tráfico.
La capa de red es la que determina que hacer con el exceso de paquetes. Son los operadores de la red quienes mitigan la congestión reduciendo la velocidad de envío o traficando en rutas nuevas. 
La congestión ocurre cuando el número de paquetes que los hosts envían a la red no está dentro de la capacidad de la red. A medida que la carga ofrecida se acerca a la capacidad de transporte, los buffers de los router se llenan ocasionando pérdida de paquetes ocasionando la caída del rendimiento.
-  *Gestión de tráfico/congestión:* asegurar que la red sea capaz de transportar el tráfico ofrecido. Realizada por los hosts o los emisores de tráfico.
- *Control de Flujo:* tráfico entre un emisor y un receptor concretos. Se ocupa de garantizar que el emisor no transmita más rápido de lo que el receptor pueda recibir.

## Enfoques de Manejo de Tráfico:

### 1. Network Provising - Aprovisionamiento:
La forma más sencilla de evitar la congestión es diseñando una red robusta preparada para una carga de tráfico. Se añaden recursos de reserva como routers, ancho de banda o líneas de enlace que son utilizadas en momentos de congestión grave.
Sin embargo, a veces no es posible aumentar la capacidad, por lo que la mejor forma de controlar la congestión es disminuyendo la carga.

### 2. Enrutamiento Consciente:
El objetivo es que al calcular las rutas teniendo en cuenta la carga para alejar el tráfico de los caminos congestionados. La forma más directa es haciendo que las rutas con menor peso sean favorecidas a las rutas con menor carga.

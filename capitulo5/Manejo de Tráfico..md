# 1. Congestión:

Cuando existen demasiados paquetes circulando en la red se producen retrasos y pérdidas de paquetes que degradan el rendimiento. Es la principal necesidad de manejo de tráfico.
La capa de red es la que determina qué hacer con el exceso de paquetes. Son los operadores de la red quienes mitigan la congestión reduciendo la velocidad de envío o traficando en rutas nuevas. 
La congestión ocurre cuando el número de paquetes que los hosts envían a la red no está dentro de la capacidad de la red. A medida que la carga ofrecida se acerca a la capacidad de transporte, los buffers de los router se llenan ocasionando pérdida de paquetes ocasionando la caída del rendimiento.

-  *Gestión de tráfico/Ingeniería de tráfico:* asegurar que la red sea capaz de transportar el tráfico ofrecido. Realizada por los hosts o los emisores de tráfico.
- *Gestión de Congestión:* afecta el comportamiento de los routers.
- *Control de Flujo:* tráfico entre un emisor y un receptor concretos. Se ocupa de garantizar que el emisor no transmita más rápido de lo que el receptor pueda recibir.

## Enfoques de Manejo de Tráfico:

### 1. Network Provising - Aprovisionamiento:
La forma más sencilla de evitar la congestión es diseñando una red robusta preparada para una carga de tráfico. Se añaden recursos de reserva como routers, ancho de banda o líneas de enlace que son utilizadas en momentos de congestión grave.
Sin embargo, a veces no es posible aumentar la capacidad, por lo que la mejor forma de controlar la congestión es disminuyendo la carga.

### 2. Enrutamiento Consciente:
El objetivo es que al calcular las rutas teniendo en cuenta la carga para alejar el tráfico de los caminos congestionados. 
Por ejemplo, las rutas pueden modificarse para alejar el tráfico de las rutas más utilizadas cambiando los pesos de la ruta más corta.
La forma más directa es haciendo que las rutas con menor carga sean favorecidas a las rutas con menor peso.
### 3. Control de Admisión:
A veces no es posible aumentar la capacidad de la red, por lo que en una red de circuito virtual se pueden rechazar nuevas conexiones si provocan congestión.
Este concepto niega la admisión a los remitentes si la red no puede soportar más carga. 
Se permite cargas nuevas solo si la red tiene suficiente capacidad, e.g., con circuitos virtuales. 

Dependiendo del tráfico, la red decide si admitir un nuevo circuito virtual o no. La red reserva suficiente capacidad en los trayectos del circuito para evitar congestión. Se basa en mediciones pasadas para estimar el numero de circuitos admisible para un mejor rendimiento con riesgos aceptables. 
Se puede combinar con *enrutamiento consciente* para considerar los puntos conflictivos de tráfico.
###### Modelado de Tráfico:




### 4. Regulación de Tráfico:

Es una técnica para regular la velocidad media de las ráfagas de un flujo de datos que entra a la red. El objetivo es que las aplicaciones de la red transmitan a una velocidad ajustada a sus necesidades, ni más ni menos, asegurando un rendimiento estable y una asignación de recursos justa para todos.
El cliente y el proveedor (usuario y red) deben de ponerse de acuerdo en un patrón de transmisión denominado **SLA( Service Level Agreement)**.
### 5. Desprendimiento de Carga:

Cuando los routers están inundados (búferes llenos) simplemente descartan los paquetes que ya no soportan.  Un router ahogado debe decidir qué paquetes descartar. Este proceso depende del servicio que se esté utilizando.
##### Primera política: VINO.
Para transferencias de datos, un paquete viejo es preferible antes uno nuevo debido a que no necesita los todos paquetes de una trama completo, no le sirve tener los paquetes más nuevos si no puede completar el mensaje que necesita.
##### Segunda política: LECHE.
Para servicios de tiempo real, se prioriza que el paquete llegue sin importar demasiado la calidad de servicio, por lo que se prefieren los paquetes nuevos antes que los viejos debido que los paquetes viejos atrasados se vuelven obsoletos durante la transmisión.
##### Política de descarte inteligente:
Existen paquetes que transportan información sobre el enrutamiento, por lo que si se pierden, se dascartan. Para aplicar un descarte inteligente, se definen "clases" que se denominan a los paquetes para indicar su importancia. Entonces, al momento de tener que rechazarlos, los routers descartan los de la clase menos importante, y así sucesivamente va descendiendo de clase.
#### Extras:
- Bufferboat: Los routers tienden a tener más memoria de lo que necesitan, que puede degradar el rendimiento de la red.
- Estrangulamiento: La red detecta una congestión inminente y pide a los remitentes disminuir la velocidad de transmisión o ralentizar el tráfico ella misma.
- Indicador de congestión: pérdida de paquetes.
- 
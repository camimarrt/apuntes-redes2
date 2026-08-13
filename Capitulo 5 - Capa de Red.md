Responsable del envío de paquetes entre dos puntos a través de múltiples enlaces. Para la transmisión es necesario conocer la topología de red (conjunto de enrutadores y enlaces). Su función principal es enrutar paquetes.
# 1. Conmutación de Paquetes de Almacenamiento y Reenvío.

- Cómo funciona? 
Los host envían paquetes a la red (routers) y los routers reenvían los paquetes. 
### Servicios a la Capa de Transporte:
Objetivos:
- Los servicios deben ser independientes de la tecnología del router.
- La capa de transporte debe estar protegida del número, tipo y topología de los routers presentes. 
- Las direcciones de red puestas a disposición de la capa de transporte deben utilizar un plan de numeración uniforme, incluso entre redes LAN y WAN.
Tipos de Servicios:

### Orientados a la Conexión - CIRCUITOS VIRTUALES:
Los circuitos virtuales es la ruta entre el router origen hasta el router destino. Son utilizados para evitar elegir rutas nuevas para cada paquete. Las rutas se establecen por adelantado durante la conexión y se almacenan en las tablas del router. Dicha ruta se mantiene para toda la conexión. Cada paquete lleva un identificador que indica el CV al que pertenece.

### NO orientados a la Conexión - DATAGRAMAS:
Los paquetes se envían a la red de forma individual y puede tomar caminos distintos unos de otros. No necesitan conexión previa, pero puede realizarse. Los paquetes se denominan Datagramas. 

![[Pasted image 20260810215003.png]]
# 2. Algoritmos de Enrutamiento.
Algoritmos que gestionan las tablas de enrutamiento y toma decisiones sobre el enrutamiento. Forman parte del software de la red encargada de enviar el paquete. Si es una red de datagramas, la decisión debe tomarse de nuevo en cada nodo. 
Minimizan la distancia a recorrer, mejoran el retardo, reducen el ancho de banda.
Pueden ser: 
- *Adaptivos (dinámicos):* Cambian sus decisiones de encaminamiento a la que cambia la red o el tráfico
- *No adaptivos (estáticos):* No basa su decisión en ninguna medición o estimación de topología/tráfico. La ruta se calcula de antemano y se descarga en los routers cuando se inicia la red. No responde a fallos pero es útil para redes donde los caminos son claros.
### Propiedades:
- Exactitud, sencillez, robustez, estabilidad, equidad, eficiencia.

Reenvío: Enviar cada paquete que llega por una línea de salida buscando el camino en las tablas de enrutamiento.
Ruteo: Rellenar y actualizar las tablas.

## 2.1. Principio de Optimización.

Si la red forma un árbol, se halla el árbol de expansión mínima. (Como un STP no contiene bucles, cada paquete se entrega en un numero finito de saltos). 
## 2.2. Algoritmo Dijkstra.

A diferencia de grafos, en redes de computadoras de calcula el camino más corto con Dijkstra desde un nodo terminal en vez de un nodo origen, debido a que cada nodo se etiqueta con su predecesor en lugar de sucesor. El camino se invierte. 
Cada enlace tiene un peso no negativo (costo) Puede ser: retardo, longitud, ancho de banda, tráfico promedio.
Si los pesos = 1 se computa el camino con menor salto
## 2.3. Inundación.

Cada paquete entrante se envía por todas las líneas de salida excepto por la que llegó. Para evitar paquetes duplicados es hacer a los routers llevar cuenta de los saltos, incluye un contador que cuando llega s cero es descartado. Normalmente inicializado con la longitud del camino destino. Si no se conoce, se inicializa con el diámetro de la red.
Una forma de lograr este objetivo es hacer que el router de origen ponga un número de secuencia en cada paquete que recibe de sus hosts. Cada router necesita entonces una lista por router de origen que indique qué números de secuencia originados en esa fuente ya han sido vistos. Si un paquete entrante está en la lista, no se inunda.

Es muy robusta, siempre encuentra el camino a su destino.
## 2.4. Enrutamiento por Bellman Ford (Vector Routing).

### Pasos:
1.  Conocer a sus vecinos y sus direcciones de red.
2. Calcular distancias a cada vecino:
3. Construir Paquetes.
4. Enviar y recibir paquetes.

## 2.5. Enrutamiento por Estado de Enlace.
## 2.6. Enrutamiento Jerárquico.

## 2.7. Enrutamiento de Difusión (Broadcasting).

## 2.8. Enrutamiento de Multidifusión.

## 2.9. Anycast Routing.


# 3. Manejo de Tráfico.
## 1. Congestión.

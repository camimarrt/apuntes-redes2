Responsable del envío de paquetes entre dos puntos a través de múltiples enlaces. Para la transmisión es necesario conocer la topología de red (conjunto de enrutadores y enlaces). Su función principal es enrutar paquetes.
# 1. Conmutación de Paquetes de Almacenamiento y Reenvío.

### Cómo funciona? 
Los host envían paquetes a la red (routers) y los routers reenvían los paquetes. 
## Servicios a la Capa de Transporte:
### Objetivos:
- Los servicios deben ser independientes de la tecnología del router.
- La capa de transporte debe estar protegida del número, tipo y topología de los routers presentes. 
- Las direcciones de red puestas a disposición de la capa de transporte deben utilizar un plan de numeración uniforme, incluso entre redes LAN y WAN.
Tipos de Servicios:

## Orientados a la Conexión - CIRCUITOS VIRTUALES:
Los circuitos virtuales es la ruta entre el router origen hasta el router destino. Son utilizados para evitar elegir rutas nuevas para cada paquete. Las rutas se establecen por adelantado durante la conexión y se almacenan en las tablas del router. Dicha ruta se mantiene para toda la conexión. Cada paquete lleva un identificador que indica el CV al que pertenece.

## NO orientados a la Conexión - DATAGRAMAS:
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

Mecanismo de supresión:
1. Limite de saltos: contador númerico si llega a 0 off.
2. Nros de secuencia: El nodo origen pone números al paquete. Los routers tienen un registro de paquetes recibidos con sus números. Si llegan muchos paquetes que están en el registro, lo descarta.

Es muy robusta, siempre encuentra el camino a su destino.

Cada nodo recibe un paquete y retransmite por todos sus nodos. Por probabilidad, llega al destino pero puede colapsar la red porque la transmisión de paquetes es exponencial. 
## 2.4. Enrutamiento por Bellman Ford (Vector Routing).

Algoritmo de Bellman Ford y RIP. Cada nodo solo conoce la distancia a los vecinos y mantienen una tabla de distancia a sus vecinos directos. Una tabla con sus mejores estimaciones a sus vecinos y se las pasa a sus vecinos. El vecino recibe la tabla y le suma el costo de enlace y actualiza su tabla si es necesario.

### Problema de "Conteo al infinito":
Solos los vecinos directos se enteran de los fallos y enlaces que dejan de funcionar. Otros nodos, al no conocer la tipología de red y solo mantienen una tabla con métricas de distancias y no se enteran nunca que su camino al nodo inalcanzable ya no sirve, entonces todos los otros calculan el nuevo camino (que no existe) y actualizan infinitamente su tabla.
#### Solución?
> Se define "infinito" como 16 saltos. Si llega a 16, el algoritmo descarta ese camino. 

> Para evitar llegar al conteo numero 16, existe un mecanismo ***Horizonte Dividido*** que prohíbe a un nodo decirle anunciar que tiene un camino a través de la interfaz por la cual la aprendió originalmente.

> ***Envenenamiento:*** Cuando un router percibe una caída de un enlace, anuncia dicha caída pero pone en la distancia a ese punto en 16 (infinito).

> ***Envenenamiento en Reversa:*** El router receptor envía la actualización a la interfaz por la que la aprendió dando un valor de “infinito”. 

## 2.5. Enrutamiento por Estado de Enlace.

Es un algoritmo robusto aunque requiere de mucho computo y memoria para almacenar las tablas de enrutamiento y recalcular dijkstra. Cada router tiene el mapa de la red y hace su propia tabla de enrutamiento con dijkstra.
### Pasos:
1.  ***Descubrimiento:*** Conoce a sus vecinos directos, envía un paquete especial HELLO en cada línea de enlace punto a punto, guarda sus direcciones de red. Un router designado en la LAN es seleccionado para jugar el papel de N en el protocolo de enrutamiento.
2. ***Medición:*** Crea un tráfico de prueba y calcula métricas a cada vecino. (Métricas se pude referir al retardo, ancho de banda o tráfico, distancia, etc.). Mide el tiempo de ida y vuelta y dividiéndolo por dos, el router emisor puede obtener una estimación.
3. ***Construir Paquetes:*** Cada enrutador construye un paquete "LSP (Link State Protocol)" que contenga todos los datos recabados (Remitente, numero de secuencia, antigüedad, lista de vecinos y el coste de cada uno) que contiene la lista de vecinos y los pesos de los enlaces.
4. ***Enviar y recibir paquetes:*** Todos los routers deben recibir los paquetes de estado de enlace de forma rápida y fiable. Inunda la red controladamente con los paquetes creados. Los routers mantienen un registro de los números de secuencia recibidos, por cada paquete nuevo se fija en el numero de secuencia y si este es menor que el mayor recibido, lo considera obsoleto y los descarta. Si no, reenvía en todas sus líneas menos por la que llegó. Para evitar números de secuencia duplicados, se utilizan 32 bits. 
5. ***Cálculo de rutas:*** Luego de recibir los paquetes cada router tiene suficiente información para crear el grafo completo de la red. Ejecuta Dijkstra para conocer los caminos más cortos a los demás routers.

#### **Protocolos que utilizan esta técnica:** 
 - **IS-IS (Intermediate Sys- tem-to-Intermediate System):** Muchos ISP utilizan este protocolo de estado de enlace. Puede transportar información sobre múltiples protocolos de capa de red al mismo tiempo (por ejemplo, IP, IPX y AppleTalk).
- **OSPF (Open Shortest Path First):** Diseñado por IETF. Incluyen un método auto estabilizador de inundación de actualizaciones de estado de enlace, el concepto de enrutador designado en una LAN y el método de cálculo y soporte de división de rutas y métricas múltiples.

## 2.6. Enrutamiento Jerárquico.

A veces, la red crece tanto que ya no es factible que cada router almacene una entrada para cada router por lo que se tiene una estrategia que gestiona el crecimiento de las redes mediante la división de los routers en "regiones" o "sistemas autónomos". 

No es necesario conocer los caminos, sino a hacia donde quedan los demás nodos. El enrutador guarda los detalles de sobre como encaminar los paquetes a las regiones cercanas, y para el resto solo necesitan saber la línea por donde salir para llegar a una región específica.

>[!danger] Desventaja!!
> Puede resultar en caminos largos porque se prioriza la gestión sobre obtener la mejor ruta.

 La formula óptima que define el número óptimo de niveles para una red con $N$ routers es:
$$ ln(N)$$
Total de entradas requeridas:
$$e*ln(N)$$
## 2.7. Enrutamiento de Difusión (Broadcasting).

Consiste en el envío simultaneo de paquetes a todos los nodos de la red.
Existen cinco métodos principales:

1. Enviar el mismo paquete a todas las direcciones de la red. Requiere que el router conozca la lista completa de nodos, y no es muy inteligente ya que consume mucho ancho de banda.
2. ***Enrutamiento Multi-destino:*** Enviar paquete que contienen una lista de destino o un mapa de bits. Cada router que recibe un paquete, comprueba si los destinos de la lista incluye una de sus salidas de enlace. En ese caso, genera una copia del paquete y lo envía conjunto a una lista de los destinos que se vayan a utilizar en esa línea. 
3. Inundación. (ya explicado arriba)
4. ***RPF - Reverse Path Forwarding:***  En este método, un enrutador solo retransmite un paquete de difusión si este fue recibido a través de la **línea preferida** que se utiliza normalmente para llegar al origen del mensaje, si es así, lo más probable es que sea la primera copia de paquete y que este viene por la mejor ruta posible. Si el paquete llega por cualquier otra interfaz, es descartado para evitar bucles. 
5. ***Uso de Spanning Trees:*** Cada enrutador copia un paquete entrante en todas las líneas que forman parte de su árbol de expansión, exceptuando la línea por la que llegó el mensaje. Genera el número mínimo de paquetes. Cada router debe tener un SPT.

## 2.8. Enrutamiento de Multidifusión.
Primero, *Multicasting* consiste en enviar un mismo paquete a todos los miembros de un subconjunto de nodos. Los routers conocen los grupos y sus integrantes. Generalmente cada grupo tiene su propia dirección de red. Este enrutamiento consiste en enviar un paquete a todos los miembros de un grupo.

Está basado en el esquema de enrutamiento de difusión por arboles sumideros. Obtener el mejor STP depende de que si el árbol es denso o disperso.

#### ***Qué ocurre con grupos densos?***
Se necesita de un STP generado por un algoritmo de difusión. Este árbol de expansión es "podado" de tal forma a dejar en él solo los nodos que pertenecen al grupo.
### Si se utiliza el Enrutamiento de estado de enlace:
Cada enrutador conoce la topología de la red y construye su propio STP podado para cada remitente al grupo en cuestión. 
Ej.: MOSPF (Multicast OSPF)
### Si se utiliza el Enrutamiento de Vector Distancia:
Para la poda se utiliza el algoritmos de RPF. Cada vez que un router sin hosts interesados en un grupo concreto y sin conexiones con otros routers recibe un mensaje multicast para ese grupo, responde con un mensaje PRUNE, indicando al vecino que envió el mensaje que no le envíe más multicasts del remitente para ese grupo. Cuando un router sin miembros de grupo entre sus propios hosts ha recibido tales mensajes en todas las líneas a las que envía el multicast, también puede responder con un mensaje PRUNE.

Ej.: DVMRP (Distance Vector Multicast Routing Protocol)
#### ***Para grupos dispersos:***
Un diseño alternativo es el árbol de expansión Core Base Tree (CBT), que emplea un solo árbol por grupo. 
Este árbol parte de un nodo central hacia otros miembros. Todos los routers se ponen de acuerdo en una raíz **núcleo** y construyen el árbol enviando un paquete desde cada miembro a la raíz. El árbol es la unión de los caminos trazados por estos paquetes. Para enviar a este grupo, un remitente envía un paquete al núcleo. Cuando el paquete llega al núcleo, es reenviado hacia abajo en el árbol.
Tan pronto como un paquete alcanza el árbol, puede ser reenviado hacia arriba, hacia la raíz, así como hacia abajo por todas las otras ramas.

*OBS.: Tener un árbol compartido no es óptimo para todas las fuentes. La ineficiencia depende de dónde se encuentren el núcleo y los remitentes, pero a menudo es razonable cuando el núcleo está en medio de los remitentes.*

Ej.: PIM (Protocol Independent Multicast).


> [!danger]
> DESVENTAJA: El router almacena mxn árboles distintos.
## 2.9. Anycast Routing.

El **enrutamiento Anycast** es un modelo de entrega donde un paquete se envía a **un solo miembro** de un grupo de nodos, seleccionando específicamente al que se encuentre **más cercano** al emisor según la métrica de la red.
En lugar de asignar una dirección única a un solo dispositivo, se le otorga la **misma dirección de red** a un grupo de nodos que ofrecen el mismo servicio en diferentes puntos del mundo 
El Anycast genera una **topología aparente de árbol sumidero** dirigida hacia un "nodo" que parece estar ubicado en múltiples lugares simultáneamente.

### - Qué es el Sink Tree?
Se define como el conjunto de todas las **rutas óptimas** desde cualquier origen posible en la red hacia un nodo de destino específico. 
Está basado en el ***Principio de la Optimalidad***, que indica que si un nodo X se encuentra en el camino óptimo de A a B, entonces la ruta de  X a A también es óptima.




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

Es un algoritmo robusto aunque requiere de mucho computo y memoria para almacenar las tablas de enrutamiento y recalcular dijkstra. Cada router tiene el mapa de la red y hace su tabla enrutamiento.
### Pasos:
1.  ***Descubrimiento:*** Conocer a sus vecinos directos y sus direcciones de red.
2. ***Medición:*** Crea tráfico de prueba y calcula métricas a cada vecino. (Métricas se pude referir a retardo, ancho de banda o tráfico).
3. ***Construir Paquetes:*** Empaqueta las mediciones en un paquete de estado de Enlace LSP que contiene la lista de vecinos y los pesos de los enlaces.
4. ***Enviar y recibir paquetes:*** Inunda la red controladamente.
5. ***Cálculo de rutas:*** Cada router tiene el grafo completo de la red y aplica Dijkstra para conocer los caminos más cortos.

**Protocolos que utilizan esta técnica:** IS-IS, OSPF.
## 2.6. Enrutamiento Jerárquico.

Tiene una estrategia que gestiona el crecimiento de las redes mediante la división de los routers en "regiones" o "sistemas autónomos". No es necesario conocer los caminos, sino a hacia donde quedan los demás nodos. El enrutador guarda los detalles de su región cercana, y para el resto solo necesitan saber la línea por donde salir para llegar a una región específica.

>[!danger] Desventaja!!
> Puede resultar en caminos largos porque se prioriza la gestión sobre obtener la mejor ruta.

 La formula óptima que define el número óptimo de niveles es:
$$ log(n)$$
## 2.7. Enrutamiento de Difusión (Broadcasting).

Consiste en enviar paquetes haciendo broadcast a todos los nodos de la red.
Existen cinco métodos principales:

1. Enviar el mismo paquete a todas las direcciones de la red. Requiere que el router conozca la lista completa de nodos, y no es muy inteligente ya que consume mucho ancho de banda.
2. ***Enrutamiento Multi-destino:*** Enviar paquete que contienen una lista de destino. Cada router:
	- chequea la lista, 
	- consulta su tabla,
	- reenvía las copias con una tabla de con destino que incluyan las líneas siguientes.
3. Inundación.
4. ***RPF - Reverse Path Forwarding:***  En este método, un enrutador solo retransmite un paquete de difusión si este fue recibido a través de la **línea preferida** que se utiliza normalmente para llegar al origen del mensaje. Si el paquete llega por cualquier otra interfaz, es descartado para evitar bucles.
5. ***Uso de Spanning Trees:*** Cada enrutador copia un paquete entrante en todas las líneas que forman parte de su árbol de expansión, exceptuando la línea por la que llegó el mensaje. Genera el número mínimo de paquetes. Cada router debe tener un SPT.

## 2.8. Enrutamiento de Multidifusión.

Primero, *Multicasting* consiste en enviar un mismo paquete a todos los miembros de un subconjunto de nodos. Los routers conocen los grupos y sus integrantes. Generalmente cada grupo tiene su propia dirección de red.

Este enrutamiento consiste en enviar un paquete a todos los miembros de un grupo.

>[!danger]
> ***Qué ocurre con grupos densos?***
>Se necesita de un STP generado por un algoritmo de difusión. El router almacena mxn árboles distintos. Fácil de implementar si ya se utilizan algoritmos de control de enlace (Estado de Enlace).
>En algoritmos de vector de distancia, se emplea RPF con mensajes PRUNE, que consisten en mensajes del router cuando no le interesan los paquetes.
>Ej.: DVMRP (distance vector multicast routing protocol)
>
>***Para grupos dispersos:***
>Se utiliza CBT (Core Base Tree), que emplea un solo árbol por grupo.  Este árbol parte de un nodo central hacia otros miembros, Ej.: PIM (Protocol Independent Multicast).

## 2.9. Anycast Routing.

*Anycast*, envía paquetes a UN miembro del grupo (el más cercano). 
Es un enrutamiento regular hacia un nodo que parece estar ubicado en **múltiples lugares** de la red de forma simultánea.

Desde el punto de vista del algoritmo, se genera una topología aparente de **árbol sumidero** (_sink tree_) dirigida hacia ese "nodo" distribuido, permitiendo que el tráfico se dirija siempre al destino más próximo según las métricas de la red.
# 3. Manejo de Tráfico.
## 1. Congestión.


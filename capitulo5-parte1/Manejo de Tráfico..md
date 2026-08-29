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

##### Leaky Bucket:

El algoritmo de **cubeta con goteo** (_leaky bucket_) es una técnica de modelado de tráfico diseñada para convertir un flujo de datos irregular y con ráfagas en un flujo de salida **constante y uniforme**,.

Su funcionamiento y características principales son:

- **Analogía técnica:** Se imagina como una cubeta que tiene un pequeño orificio en el fondo. Sin importar la velocidad a la que entre el agua (los paquetes), el flujo de salida es **estrictamente constante (tasa R)** mientras haya contenido en la cubeta,.
- **Gestión de ráfagas (Capacidad B):** La cubeta tiene una capacidad finita **B**. Si llega una ráfaga de paquetes que llena la cubeta, cualquier dato adicional que entre se desborda y se pierde (proceso conocido como **vigilancia de tráfico** o _policing_),.
- **Modelado vs. Vigilancia:**
    - **En el host (Shaping):** Se utiliza para retrasar los paquetes en una cola, enviándolos solo a la tasa permitida para que la red pueda procesarlos sin congestión,.
    - **En el router del proveedor (Policing):** Se usa para asegurar que el cliente no exceda la tasa contratada en el **SLA** (_Service Level Agreement_); si el tráfico excede la capacidad **B**, el hardware del proveedor simplemente descarta los paquetes sobrantes o reduce su prioridad,.
- **Diferencia con la cubeta con tokens:** A diferencia del algoritmo de _token bucket_, que permite que las ráfagas pasen a la velocidad máxima del enlace hasta que se agoten los tokens, la **cubeta con goteo es rígida** y suaviza completamente el tráfico, forzando una tasa de salida estable,.

En resumen, este mecanismo es un bloque fundamental para el **control de admisión**, ya que permite a la red estimar cuántos flujos puede aceptar garantizando que no se superen los límites de ancho de banda y evitando así el colapso por congestión,.
##### Token Bucket:

El algoritmo de **cubeta con fichas** (_token bucket_) es una técnica de modelado de tráfico diseñada para regular la tasa promedio de datos, permitiendo al mismo tiempo ráfagas controladas de tráfico. A diferencia de métodos más rígidos, este algoritmo ofrece flexibilidad para que las aplicaciones envíen picos de información sin ser penalizadas de inmediato, siempre que se mantengan dentro de los límites acordados.

### **Mecanismo de funcionamiento**

El sistema se visualiza como una cubeta que se llena de "fichas" (tokens) a una tasa constante de **\(R\) fichas por segundo**. La cubeta tiene una capacidad máxima de **\(B\) fichas**; si la cubeta se llena, las fichas nuevas que llegan se descartan y se pierden.

Para que un host pueda transmitir un paquete a la red, debe ser capaz de **sacar de la cubeta** una cantidad de fichas proporcional al tamaño del paquete (por ejemplo, una ficha por byte o por paquete). Si no hay suficientes fichas, el paquete debe esperar en una cola hasta que se generen más fichas, o bien es descartado o marcado con menor prioridad (lo que se conoce como _traffic policing_).

### **Gestión de ráfagas y fórmulas clave**

La principal ventaja de este algoritmo es que permite que ráfagas de hasta \(B\) bytes pasen a la velocidad máxima del enlace antes de estabilizarse en la tasa promedio \(R\). El material técnico proporciona fórmulas fundamentales para calcular su desempeño:

- **Duración de la ráfaga máxima (\(S\)):** Indica cuánto tiempo puede el host transmitir a su velocidad máxima (\(M\)) antes de agotar las fichas. Se calcula considerando que, mientras ocurre la ráfaga, siguen llegando fichas nuevas: 
$$S = \frac{B}{M - R} $$

Donde:
B: es la capacidad de la cubeta.
M: es la velocidad máxima de salida.
R: es la tasa de llenado.
-
- **Contenido total de la ráfaga:** La cantidad máxima de datos enviados en ese tiempo \(S\) es \(M \times S\), lo cual es equivalente a la ráfaga inicial más las fichas que llegaron durante el proceso $$(B + R \times S).$$
- **Garantía de retardo máximo (\(D\)):** Cuando se combina con algoritmos de encolamiento como **WFQ** (Weighted Fair Queueing), se puede garantizar que el retardo máximo en la cola no superará: $$D = \frac{B}{R}$$ _(Ignorando efectos de paquetización y asumiendo ancho de banda garantizado \(R\))._

### **Implementación y optimización**

En la práctica, el algoritmo no usa agua o fichas físicas, sino un **contador** que avanza en unidades de $$R/\Delta T$$ por cada tic de reloj $$\Delta T. $$Si los paquetes son de tamaño variable, el nivel de la cubeta se cuenta en bytes.

Para lograr un tráfico aún más fluido, se puede insertar un **segundo token bucket** en serie. Mientras que el primer cubo define la tasa media y permite ráfagas, el segundo cubo puede configurarse con una tasa mucho más alta pero capacidad cero (\(B=0\)) para **limitar la tasa pico absoluta** a la que los datos entran a la red, evitando que las ráfagas iniciales sean demasiado agresivas para el hardware.

### **Diferencia con la cubeta con goteo (_leaky bucket_)**

Aunque ambos sirven para modelar el tráfico, la **cubeta con goteo es rígida**: independientemente de la entrada, la salida es estrictamente constante. El **token bucket es más flexible**, ya que permite que el tráfico salga a la velocidad máxima de la interfaz hasta que se agoten las fichas ahorradas, momento en el cual se estabiliza a la tasa \(R\).
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
Existen paquetes que transportan información sobre el enrutamiento, por lo que si se pierden, se descartan. Para aplicar un descarte inteligente, se definen "clases" que se denominan a los paquetes para indicar su importancia. Entonces, al momento de tener que rechazarlos, los routers descartan los de la clase menos importante, y así sucesivamente va descendiendo de clase.

### 6. Requerimiento de QoS de la Aplicación.

Un flujo de paquetes de un punto origen a un punto destino se denomina *flujo*. Cada la calidad de servicio del flujo puede se caracteriza por el ancho de banda, retardo, fluctuación y pérdida. Diferentes aplicaciones requieren diferentes ancho de banda.
La variación del retardo o los tiempos de llegada se denomina *jitter*. 
Categorías de soporte de QoS:
- Velocidad binaria constante. (Telefonía).
- Velocidad binaria variable en tiempo real. (Videoconferencia comprimida).
- Velocidad binaria variable en tiempo no real. (Película streaming).
- Tasa de bits disponible. ( Transferencia de archivos).
##### Sobre-aprovisionamiento:
Una solución par ofrecer buena calidad de servicio es construir un red con suficiente capacidad para todo el tráfico que reciba. El problema es que resulta caro.
##### Cuestiones a abordar para QoS:
- Qué necesita la red?
- Cómo regular el tráfico que entra?
- Cómo se reservan recursos?
- La red puede soportar más tráfico?
### 7.  Programación de Paquetes.
Para garantizar un buen rendimiento se deben reservar recursos a lo largo de las rutas de la red. Los algoritmos que distribuyen los recursos del router se denominan *algoritmos de programación de paquetes*. Los recursos a reservar son: ancho de banda, retardo, ciclos del CPU.
##### - FIFO/FCFS:
Cada router almacena paquetes en una cola para cada línea de salida.  Los routers FIFO suelen descartar los paquetes recién llegados cuando la cola se llena, conocido como *"tail drop"*.  Cuando existen flujos agresivos cuyos paquetes llegan más rápido que otros, el router los procesa por orden de llegada y privan acceso de los paquetes de los demás flujos, causando retrasos. 
##### - Encolamiento Justo:
El algoritmo de colas justas tienen colas separadas para cada línea de salida determinada. 
Cuando la línea está inactiva, el enrutador escanea las colas de forma rotatoria, toma el primer paquete de la siguiente cola. De esta forma, con n hosts compitiendo por la línea de salida, cada host consigue enviar uno de cada n paquetes. Es justo en el sentido de que todos los flujos envían paquetes a la misma velocidad.
Da más ancho de banda a los hosts que utilizan paquetes grandes que a los que utilizan paquetes pequeños.
###### Mejora:
En 1990 sugirieron una mejora en la que el *round robin* se realiza de tal forma que simula un round byte a byte, en lugar de un round robin paquete a paquete. El truco consiste en calcular un tiempo virtual que es el número de la ronda en la que cada paquete terminaría de enviarse. Cada ronda vacía un byte de todas las colas que tienen datos que enviar.
Los paquetes se ordenan según su hora de finalización y se envían en ese orden. Si un paquete tiene una longitud L, la ronda en la terminará es simplemente L rondas después de la hora de inicio. El inconveniente es que todos los paquetes tienen la misma prioridad aunque a veces hay servicio que requieren más recursos.
##### - Encolamiento Justo Ponderado:
Este algoritmo se llama ***WFQ (Weighted Fair Queue)***. Requiere que los paquetes se inserten por hora de finalización en una cola ordenada. Con N flujos, esto es en el mejor de los casos una operación O(log N ) por paquete.

>[!tip] 
>$$Fi = max(Ai,F(i-1)) + Li/W $$

**Donde:**
- Fi: tiempo de finalización:
- A: Tiempo de llegada:
- Li : Longitud del paquete.
- W: Número de bytes por ronda.

Este algoritmo se llama **WFQ (Weighted Fair Queue)**.  Requiere que los paquetes se inserten por hora de finalización en una cola ordenada. Con N flujos, esto es en el mejor de los casos una operación O(log N ) por paquete.

##### - Puesta en común:
??????
### 8. Control de Admisión:
##### - Enrutamiento con QoS:
El **encaminamiento de QoS** (Calidad de Servicio) es una técnica de red que busca rutas no basándose solo en cuál es el camino más corto, sino en qué camino tiene los **recursos necesarios** para que una aplicación funcione correctamente. Mientras que el enrutamiento normal suele enviar todo el tráfico por la "mejor" ruta (la que tiene menos saltos), el enrutamiento de QoS es más flexible y puede elegir un camino alternativo si este tiene la capacidad sobrante que el usuario necesita. 

- **Especificación del Flujo:** El usuario debe decirle a la red qué es lo que necesita mediante un contrato llamado "especificación de flujo". Aquí se definen parámetros como la **velocidad máxima** de los datos y el **tamaño de las ráfagas** de tráfico que se enviarán.
- **Reserva de Recursos:** Una vez que se elige la ruta, la red debe "apartar" o **reservar los recursos** (ciclos de CPU, espacio en el buffer y ancho de banda) en cada router a lo largo del camino. Esto asegura que, cuando los paquetes empiecen a llegar, tengan su lugar asegurado y no sufran retrasos por culpa de otros usuarios.
- **Rutas Inteligentes:** Si la ruta principal está congestionada, el algoritmo de QoS busca **rutas alternativas** que tengan exceso de capacidad. Incluso puede dividir el tráfico entre varias rutas diferentes para aprovechar mejor los recursos de toda la red.
##### - Especificación del Flujo:

La **especificación de flujo** (_flow spec_) es un conjunto de parámetros técnicos que describen con precisión los requerimientos de un flujo de datos para que la red pueda garantizar su Calidad de Servicio (QoS). Dado que en una negociación de recursos pueden intervenir el emisor, el receptor y todos los enrutadores intermedios, estos parámetros sirven como un "contrato" o descripción exacta de lo que la aplicación necesita y lo que la red se compromete a entregar.

Los aspectos más importantes de la especificación de flujo, según las fuentes, son:

**Funcionamiento y Negociación**

- **Origen de la propuesta:** Normalmente, el emisor de los datos (como un servidor de vídeo) genera la especificación de flujo proponiendo los parámetros que desea utilizar.
- **Proceso de revisión:** A medida que esta especificación viaja por la ruta hacia el destino, **cada enrutador la examina y puede modificar los parámetros** según su capacidad disponible.
- **Sentido de la modificación:** Los enrutadores solo pueden modificar la especificación para **reducir el flujo** (por ejemplo, bajando la velocidad de datos), nunca para aumentarlo.

**Parámetros Principales**

Basándose en estándares como los de Servicios Integrados (RFC 2210 y 2211), una especificación de flujo típica incluye los siguientes parámetros:

- **Tasa y tamaño de la cubeta con tokens:** Definen la velocidad máxima sostenida a largo plazo y el tamaño de la ráfaga más grande que el emisor puede enviar en un intervalo corto.
- **Tasa pico de datos:** Es la velocidad máxima de transmisión que se tolera, incluso en ráfagas breves, y que el emisor nunca debe exceder.
- **Tamaño mínimo de paquete:** Útil porque procesar paquetes pequeños requiere más esfuerzo de CPU por parte del enrutador por cada byte transmitido.
- **Tamaño máximo de paquete:** Define el límite superior para evitar problemas con la Unidad de Transmisión Máxima (MTU) de los enlaces en la ruta.

**Importancia en la Red**

La especificación de flujo es la herramienta que permite al **Control de Admisión** decidir si la red tiene capacidad suficiente para transmitir un nuevo flujo sin congestionarse. Además, permite que cada enrutador en el camino realice una **reserva de recursos** (ancho de banda, memoria en el buffer y ciclos de CPU) para asegurar que se cumplan las garantías de rendimiento prometidas a la aplicación.
###### Formula:

La fórmula **\(T\)** en el control de admisión es una herramienta técnica basada en la **teoría de colas** que permite calcular el **retardo promedio** que experimentará un paquete al ser procesado por un enrutador. Este cálculo es fundamental para que la red decida si puede aceptar un nuevo flujo de datos garantizando que no se superen los límites de latencia permitidos por el contrato de Calidad de Servicio (QoS).

##### Componentes de la fórmula:

La ecuación se expresa técnicamente de la siguiente manera: 

$$T = \frac{1}{\mu} \times \frac{1}{1 - \lambda/\mu} = \frac{1}{\mu} \times \frac{1}{1 - \rho}$$

Sus variables representan conceptos críticos del rendimiento de la red:

- **\(T\):** Es el tiempo total de retardo medio, que incluye tanto el tiempo de espera en la cola como el tiempo de servicio efectivo.
- **\(\lambda\) (lambda):** Representa la tasa media de llegada de paquetes (medida en paquetes por segundo).
- **\(\mu\) (mu):** Es la capacidad media de servicio del enlace, es decir, cuántos paquetes por segundo puede enviar el enrutador.
- **\(\rho\) (rho):** Se define como el cociente \(\lambda/\mu\) y representa el nivel de **utilización del recurso** (CPU o ancho de banda); este valor debe ser menor a 1 para evitar que la cola crezca indefinidamente.

##### **Interpretación técnica y ejemplo**

La fórmula se divide en dos factores lógicos: el primero, **\(1/\mu\)**, representa el tiempo que tardaría el paquete en ser enviado si no hubiera ninguna competencia en la red. El segundo factor, **\(1/(1-\rho)\)**, es el "factor de ralentización", el cual aumenta de forma dramática a medida que la carga se acerca a la capacidad total del enlace.

Para ilustrarlo con un ejemplo práctico de las fuentes: supongamos un enrutador capaz de procesar un millón de paquetes por segundo (\(\mu = 1,000,000\)) y una carga de llegada de 950,000 paquetes por segundo (\(\lambda = 950,000\)). En este escenario, el uso de recursos es del 95% (\(\rho = 0.95\)). Aplicando la fórmula, el retardo promedio \(T\) resulta ser de **20 microsegundos**, lo cual es 20 veces superior al tiempo de servicio base de 1 microsegundo que se tendría en una red vacía.

##### **Uso en el Control de Admisión**

El algoritmo de control de admisión utiliza esta fórmula para traducir una **especificación de flujo** en una reserva real de recursos. Si al sumar un nuevo flujo el valor de \(\lambda\) aumenta tanto que el retardo \(T\) resultante sobrepasa los requerimientos de la aplicación (o satura el enrutador), la red simplemente **rechaza la conexión** para proteger la integridad del tráfico que ya está circulando. De esta forma, el control de admisión evita proactivamente que el sistema caiga en un estado de congestión severa.
**Uso en el Control de Admisión**

El algoritmo de control de admisión utiliza esta fórmula para traducir una **especificación de flujo** en una reserva real de recursos. Si al sumar un nuevo flujo el valor de λ aumenta tanto que el retardo T resultante sobrepasa los requerimientos de la aplicación (o satura el enrutador), la red simplemente **rechaza la conexión** para proteger la integridad del tráfico que ya está circulando. De esta forma, el control de admisión evita proactivamente que el sistema caiga en un estado de congestión severa.
### 9. Servicios Integrados:

Los **Servicios Integrados** (o _IntServ_) son un modelo de diseño de red creado específicamente para gestionar aplicaciones multimedia que necesitan un flujo constante de datos, como el vídeo en streaming o las llamadas de voz. Su idea central es que, para que estas aplicaciones funcionen bien, la red no puede simplemente "hacer su mejor esfuerzo", sino que debe dar **garantías individuales** para cada transmisión específica o "flujo".

**El contrato por cada flujo**
A diferencia de otros métodos que agrupan el tráfico por categorías (como "prioridad alta"), los Servicios Integrados se enfocan en cada flujo individual, como por ejemplo una videollamada de un usuario concreto. El sistema funciona mediante un "contrato": el usuario le dice a la red exactamente cuánto ancho de banda necesita y qué tan grandes serán sus ráfagas de datos, y la red decide si puede cumplir con esa promesa antes de dejarlo empezar.
##### RSVP - The ReSerVation Protocol.
**¿Cómo se aparta el lugar?**
Para que esto funcione, se utiliza una herramienta llamada **RSVP** (Protocolo de Reserva de Recursos). El proceso es curioso porque funciona "hacia atrás":
1. El emisor avisa que quiere transmitir, pero es el **receptor** quien envía la solicitud de reserva hacia el origen.
2. A medida que esta petición viaja por los enrutadores del camino, cada uno revisa sus recursos.
3. Si el enrutador tiene suficiente **ancho de banda, memoria y capacidad de procesamiento**, los "reserva" o aparta exclusivamente para ese usuario.
4. Si un solo enrutador en toda la ruta no puede cumplir, la reserva no se realiza para nadie.

**Eficiencia en grupos (Multicast)**
Este sistema es muy inteligente cuando se trata de enviar datos a muchas personas a la vez (multidifusión). Si varios receptores piden lo mismo, los enrutadores en los puntos de unión **mezclan las peticiones** en una sola reserva para no desperdiciar ancho de banda en los tramos compartidos del camino.

**¿Por qué no se usa en todo Internet?**
Aunque garantizan una calidad perfecta, tienen problemas graves de **escalabilidad**. Imagina que un enrutador en el centro de Internet tuviera que recordar y gestionar las reservas individuales de millones de personas; simplemente se saturaría. Además, si un enrutador se apaga o falla, todas las reservas que pasaban por él se pierden y hay que empezar de nuevo. Debido a esta complejidad, se han implementado muy poco en el Internet público y se han preferido soluciones más simples como los Servicios Diferenciados.

### 10. Servicios Diferenciados:

Los **Servicios Diferenciados** (o _DiffServ_) son un modelo de calidad de servicio (QoS) diseñado para ser mucho más simple y escalable que otros métodos más antiguos. A diferencia de los Servicios Integrados, que intentan reservar recursos para cada "flujo" individual (como una sola llamada de voz), DiffServ funciona agrupando el tráfico en **categorías o clases**, lo que lo hace ideal para redes masivas como Internet donde gestionar millones de flujos individuales sería imposible.

El funcionamiento es ingenioso por su sencillez: cuando un paquete entra en una red que usa DiffServ, se le asigna una marca en un campo específico de su encabezado (tanto en IPv4 como en IPv6) llamado **campo de servicios diferenciados**. Esta marca le indica a cada enrutador en el camino cómo debe tratar a ese paquete sin necesidad de haber hecho una reserva previa de ancho de banda; a este tratamiento local se le conoce técnicamente como **comportamiento por salto** (_Per-Hop Behavior_). Es como el correo postal: si pagas por un envío "urgente", cada oficina de correos que vea el sello le dará prioridad sin tener que consultar un contrato global para tu carta específica.

Existen dos tipos principales de clases de servicio ya estandarizadas para facilitar la comunicación entre distintos operadores:

- **Envío Expedito (Expedited Forwarding):** Es la clase de "lujo" diseñada para aplicaciones que no toleran retrasos, como la telefonía por Internet (VoIP). Los paquetes marcados así transitan por la red como si esta estuviera casi vacía, asegurando un **bajo retardo y mínima variación de llegada (_jitter_)**.
- **Reenvío Asegurado (Assured Forwarding):** Es un esquema más detallado que ofrece diferentes niveles de prioridad, a menudo comparados con categorías como **oro, plata y bronce**. Además de la prioridad, este sistema define qué tan probable es que un paquete sea descartado si hay congestión (baja, media o alta probabilidad), lo que suma un total de 12 clases de servicio distintas.

Para que todo esto funcione de forma justa, los enrutadores utilizan algoritmos como el **Encolamiento Justo Ponderado (WFQ)**, que asigna un "peso" o porcentaje de la capacidad del enlace a cada clase; por ejemplo, la clase oro podría recibir siempre el doble de ancho de banda que la plata. El administrador de la red también debe vigilar y "moldear" el tráfico en la entrada para asegurarse de que los usuarios no envíen más paquetes de alta prioridad de los que han contratado, descartando o bajando la categoría de los excesos. En resumen, DiffServ permite que Internet funcione de forma fluida para aplicaciones críticas al tratar a los paquetes según su importancia grupal, sin la pesada carga de gestionar reservas individuales.


---

#### Extras:
- Bufferboat: Los routers tienden a tener más memoria de lo que necesitan, que puede degradar el rendimiento de la red.
- Estrangulamiento: La red detecta una congestión inminente y pide a los remitentes disminuir la velocidad de transmisión o ralentizar el tráfico ella misma.
- Indicador de congestión: pérdida de paquetes.

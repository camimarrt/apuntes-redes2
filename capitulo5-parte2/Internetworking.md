# 1. Panorama General:
**Internetworking** es crear un conjunto de redes heterogéneas para formar una gran red lógica llamada *interred*. El propósito es unir todas las redes con el objetivo de permitir a los usuarios de cualquiera de ellas comunicarse con todos los usuarios de las demás redes. 

Las redes individuales que se conectan suelen diferir en tecnologías, arquitectura y mecanismos. Los problemas surgen cuando se conectan redes *heterogéneas* para formar una interred.

Bob Metcalfe postuló que el valor de una red con N nodos es proporcional a ***N^2***, por lo que las redes grandes son mucho más valiosas que las pequeñas porque permiten muchas más conexiones.
### Cómo difieren las redes?
Las redes individuales que se conectan suelen diferir en aspectos críticos como:
- Formatos de trama.
- Tecnología de red: Orientado a la conexión o No orientado a la conexión.
- Cada red tiene un tamaño de paquetes máximo (MTU, Maximum Transmission Unit).
- Esquemas de direccionamiento (tamaño y estructura).
- Calidad del servicio (QoS) (Retardo, ancho de banda diferente).

El internetworking se ocupa de "empapelar" y resolver estas disparidades.

![[Pasted image 20260826111015.png|467]]
### Conexión de redes heterogéneas.
Existen dos opciones básicas para conectar redes diferentes:

##### A. Gateway - Dispositivos de Traducción.
Consiste en construir dispositivos en los límites de las redes que traduzcan o conviertan los paquetes de cada tipo de red en paquetes para cada otra red.
Cuando un paquete pasa de una red a otra, el dispositivo de interconexión —tradicionalmente llamado **pasarela o gateway** — modifica el formato del paquete, adaptando aspectos como las direcciones o el control de flujo para que sea compatible con la red de destino.

**Desventajas:** En la práctica, realizar una traducción directa y completa es extremadamente difícil debido a las diferencias insalvables entre tecnologías (por ejemplo, intentar traducir prioridades de calidad de servicio o esquemas de seguridad de una red que no los soporta a otra que sí). Por ello, este método no escala bien para conectar una gran diversidad de redes heterogéneas.

##### B. Añadir una capa común superior (Capa de indirección).
Este enfoque propone resolver el problema añadiendo una capa de indirección, es decir, una capa común que se asienta por encima de todas las redes distintas.

- **Cómo funciona:** Se define un formato de paquete universal y un conjunto de reglas que todas las redes subyacentes deben transportar dentro de sus propios formatos locales. Los enrutadores (**routers**) operan en esta capa común (la **capa de red**) para conmutar los paquetes de manera uniforme sin importar la tecnología física o de enlace que haya debajo.
- **El ejemplo de Internet (IP):** Este es el modelo propuesto originalmente por Cerf y Kahn en 1974 que dio origen al protocolo **IP (Internet Protocol)**. IP actúa como esa capa común universal, ocultando de manera exitosa la heterogeneidad de las redes inferiores.
# 2. Conexión de puntos finales en redes heterogéneas.
Los hosts de origen y de destino se encuentran en redes del mismo tipo, pero para comunicarse deben atravesar una red intermedia que es completamente diferente.
Para que el tráfico fluya eficazmente entre las diferentes redes de una interred se requieren tres mecanismos esenciales:
### A) ¿Cómo funciona la tunelización (_tunneling_)?
Cuando el host de origen envía un paquete, este viaja por su red local con su formato original (por ejemplo, **IPv6**). Al llegar al límite con la red intermedia (que podría ser una red **IPv4**), se utiliza un **router multiprotocolo** que actúa como la entrada a un "túnel":

1. **Encapsulamiento:** El router toma el paquete original completo (con su cabecera y datos) y lo introduce como carga útil (_payload_) **dentro de un nuevo paquete compatible con la red intermedia**. Es decir, le pone una "envoltura" o cabecera de la red de tránsito.
2. **Tránsito:** El paquete viaja a través de la red intermedia de un extremo a otro. Para los routers de esa red intermedia, el paquete es simplemente un dato común y corriente; no necesitan entender el formato interno del paquete encapsulado.
3. **Desencapsulamiento:** Al llegar al router multiprotocolo del otro extremo (la salida del túnel), este remueve la cabecera externa y entrega el paquete original en la red de destino, donde continúa su camino de forma nativa. Los hosts de los extremos ni siquiera se enteran de que el paquete fue encapsulado en el camino.

El tunelado se utiliza ampliamente para conectar hosts y redes aislados utilizando otras redes. La red resultante se denomina superpuesta, ya que se ha superpuesto a la red de base.
##### Aplicaciones reales de esta técnica.
- **IPv6 sobre IPv4:** Permite conectar islas de redes modernas IPv6 utilizando la enorme infraestructura global IPv4 que aún predomina en muchas partes, facilitando una transición gradual.
- **Redes Superpuestas (****Overlay Networks****):** La red lógica creada por encima de la infraestructura de base mediante túneles se denomina red superpuesta.
- **VPN (Redes Privadas Virtuales):** Las VPN son un tipo de red superpuesta que utiliza la tunelización para encapsular y cifrar el tráfico de una organización de forma segura a través de Internet.

**Su principal limitación:** Un paquete que viaja dentro de un túnel no puede comunicarse con ningún host de la red intermedia por la que transita, ya que va "encerrado en su caja" y no puede escapar a mitad del camino.
### B) Enrutamiento a través de múltiples redes.

El **enrutamiento a través de múltiples redes** (o enrutamiento interred) es el mecanismo que permite que los paquetes de datos viajen de manera correcta cruzando diferentes redes independientes hasta llegar a su destino. Cuando un paquete de datos tiene que cruzar de una red a otra, surgen problemas complejos que no existen dentro de una sola red:

- **Diferentes algoritmos de enrutamiento:** Una red puede usar enrutamiento por _estado de enlace_ (que requiere conocer todo el mapa de la red) y otra red vecina puede usar _vector de distancia_ (que no tiene el mapa completo). Como usan "idiomas" diferentes, no es obvio cómo calcular el camino más corto entre ambas.
- **Métricas incompatibles:** Lo que para una red es un camino "barato" o "rápido" (basado, por ejemplo, en el retardo), para otra red puede medirse de forma distinta (por ejemplo, en cantidad de saltos o ancho de banda).
- **Falta de escalabilidad:** Si cada router de Internet tuviera que conocer detalladamente la ubicación de cada uno de los millones de dispositivos conectados en el mundo, las tablas de enrutamiento serían gigantescas y colapsarían.
#### La solución: Enrutamiento en dos niveles
Para resolver este rompecabezas, Internet utiliza una estrategia inteligente de **dos niveles de enrutamiento**. La gran red mundial se divide en **Sistemas Autónomos (AS)**, que son redes independientes gestionadas por una única organización (como un ISP, una universidad o una gran corporación). Gracias a esto, el enrutamiento se separa en:
##### Nivel 1: Enrutamiento Intradominio (Interior)
- **¿Dónde ocurre?:** Únicamente **dentro** de cada Sistema Autónomo (AS).
- **¿Cómo funciona?:** Se utiliza un protocolo llamado **IGP** (_Interior Gateway Protocol_). El operador de la red tiene total libertad para elegir el protocolo que prefiera para organizar el tráfico interno a su antojo.
- **Su ventaja:** El resto del mundo no necesita saber cómo está diseñada esa red por dentro; solo necesitan saber cómo llegar a sus fronteras.
##### Nivel 2: Enrutamiento Interdominio (Exterior)
- **¿Dónde ocurre?:** En las fronteras, para comunicar y pasar paquetes **entre** los distintos Sistemas Autónomos.
- **¿Cómo funciona?:** Se utiliza un protocolo de pasarela exterior. Aquí ya no hay libertad de elección: **todas las redes de Internet deben utilizar obligatoriamente el mismo protocolo** para poder entenderse. En Internet, este protocolo estándar es el **BGP** (_Border Gateway Protocol_).
###### Las políticas de enrutamiento: El negocio detrás del camino
A diferencia del enrutamiento interno (donde siempre se busca el camino físicamente más corto o rápido), en el enrutamiento interdominio con BGP mandan las **políticas comerciales**.

El camino que sigue tu paquete no siempre será el más corto; será el que determine la economía y los acuerdos entre compañías:
- **Tránsito:** Un ISP le paga a otro (un proveedor de tránsito) para que transporte su tráfico a cualquier lugar de Internet.
- **Peering (Interconexión de pares):** Dos redes acuerdan intercambiar tráfico de manera directa y gratuita porque les conviene a ambas, pero prohíben que la otra use ese enlace para enviar cosas a terceros.

Gracias a este esquema de dos niveles, Internet puede seguir creciendo de manera ilimitada (escalabilidad), protegiendo los secretos comerciales de cada empresa y permitiendo que fluyan los acuerdos económicos globales.

### C) Fragmentación de Paquetes:
El **fragmentación de paquetes** es un mecanismo fundamental de la capa de red que consiste en dividir un paquete de datos grande en trozos más pequeños (llamados fragmentos) para que puedan ser transmitidos a través de un enlace o red física que no admite su tamaño original.

#### ¿Por qué existe un límite de tamaño y qué es el MTU?
No existe un tamaño único universal para los paquetes porque cada tecnología de red se diseña con limitaciones propias. 
Por ejemplo, mientras que Ethernet limita su carga útil a **1,500 bytes**, las redes inalámbricas 802.11 admiten hasta **2,272 bytes** e IP permite teóricamente paquetes de hasta **65,515 bytes**.
El tamaño de paquete más pequeño que se tolera a lo largo de toda una ruta entre el origen y el destino se conoce como **MTU de ruta** (_Path MTU_ o Unidad de Transmisión Máxima de la Ruta).
#### Las dos estrategias de fragmentación:
Cuando un paquete grande debe atravesar una red con un MTU inferior a su tamaño, se pueden aplicar dos filosofías opuestas de diseño2more_horiz:

##### A) Fragmentación Transparente:
En esta estrategia, la fragmentación se mantiene oculta para el resto de las redes por las que viaja el paquete1213.
- **Cómo funciona:** Cuando un paquete grande llega al router de entrada ($G_1$) de una red con MTU pequeño, el router lo divide en fragmentos1213. Estos trozos viajan por esa red y, al llegar al router de salida de la misma ($G_2$), se vuelven a ensamblar para restaurar el paquete original antes de enviarlo a la siguiente red1213.
- **Desventajas:**
    - Requiere una gran carga de procesamiento y almacenamiento para los routers fronterizos, que deben guardar los fragmentos y decidir cuándo desecharlos si no llegan todos14more_horiz.
    - Restringe el enrutamiento: obliga a que todos los fragmentos salgan obligatoriamente por el mismo router de salida para poder reensamblarse, perdiendo la flexibilidad de usar rutas alternativas más eficientes7more_horiz.
##### B) Fragmentación No Transparente (La que usa el protocolo IP)
En este enfoque, una vez que un paquete es fragmentado por primera vez, los fragmentos no se vuelven a unir en ningún router intermedio17more_horiz.

- **Cómo funciona:** Los routers intermedios simplemente transmiten cada fragmento como si fuera un paquete independiente y completo1719. El **reensamblaje se realiza únicamente en el host de destino final**1719.
- **Mecanismo de control:** Para poder reconstruir el paquete original desordenado en el destino, IP añade información clave en la cabecera de cada fragmento1820:
    1. **Identificación (Número de paquete):** Para saber a qué paquete original pertenecen los fragmentos recibidos18more_horiz.
    2. **Desplazamiento de fragmento (****Offset****):** Indica la posición exacta en bytes que ocupa la carga útil del fragmento dentro del paquete original (en múltiplos de 8 bytes, que es la unidad elemental de fragmento)18more_horiz.
    3. **Bandera de Más Fragmentos (bit MF o** **More Fragments****):** Se activa en todos los fragmentos excepto en el último, indicando cuándo han llegado todas las piezas2122.
- **Desventajas:**
    - Genera una mayor sobrecarga (_overhead_) debido a que cada fragmento requiere transportar su propia cabecera IP sobre enlaces que tal vez no la necesitarían15more_horiz.
    - **La gran desventaja de rendimiento:** Si se pierde un solo fragmento en el camino, todo el paquete original se considera perdido y la capa de transporte se ve obligada a retransmitirlo por completo15more_horiz.

##### Extra: La solución moderna - Descubrimiento de la MTU de la ruta
Debido a que la fragmentación degrada notablemente el rendimiento de la red, la estrategia utilizada en la Internet moderna es evitar por completo la fragmentación dentro de la red utilizando el **Descubrimiento de la MTU de la ruta** (_Path MTU Discovery_)23more_horiz.
- **Cómo funciona:** El host de origen envía sus paquetes IP con el bit de cabecera **DF (Don't Fragment / No Fragmentar)** activado25more_horiz. Si un router intermedio recibe un paquete que supera la MTU de su siguiente enlace, no lo fragmenta; en su lugar, **lo descarta y devuelve un mensaje de error ICMP al host de origen**25more_horiz. Este mensaje le notifica al origen el tamaño máximo que puede manejar el router3032.
- **Adaptación del origen:** Al recibir el error, el emisor adapta el tamaño de sus paquetes futuros reduciendo el segmento de datos para que encajen perfectamente en la ruta, evitando la fragmentación intermedia30more_horiz.


#### El grave problema de la fragmentación
A pesar de su utilidad, los investigadores de red (como Kent y Mogul) demostraron que **la fragmentación degrada severamente el rendimiento**. La razón principal es que IP no tiene mecanismos de recuperación de errores individuales por fragmento. **Si se pierde un solo fragmento de un paquete grande, todo el paquete original se pierde** y el emisor (generalmente controlado por TCP) se ve obligado a retransmitir todo el paquete completo, multiplicando el tráfico de la red innecesariamente.


En el protocolo moderno **IPv6, los routers intermedios ya no tienen permitido fragmentar paquetes bajo ninguna circunstancia**31more_horiz. Si un paquete es demasiado grande, el router lo descarta directamente y envía un mensaje de error al origen para que este reduzca el tamaño del paquete desde el host emisor, lo cual simplifica el trabajo de los routers y hace que el enrutamiento sea más rápido31more_horiz.

Las principales causas de estos límites son:
1. **Hardware:** Las limitaciones de la tecnología física de transmisión (por ejemplo, el tamaño máximo de una trama Ethernet es de 1,500 bytes).
2. **Sistemas Operativos:** Restricciones de memoria (por ejemplo, búferes configurados a 512 bytes).
3. **Protocolos:** El tamaño asignado a los campos de longitud en las cabeceras de los protocolos.
4. **Estándares:** Cumplimiento de normas nacionales o internacionales.
5. **Control de Errores:** El deseo de reducir el impacto y las retransmisiones provocadas por errores en el canal.
6. **Uso del Canal:** Evitar que un solo paquete muy grande acapare la línea de comunicación durante demasiado tiempo.

---
Siguiente: [[SDN - Software Defined Networking.]] 

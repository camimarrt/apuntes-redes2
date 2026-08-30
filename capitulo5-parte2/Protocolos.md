# 1. Protocolos de Control de Red:

### A) ICMP- Internet Control Message Protocol:
Es un subprotocolo de la capar de red que asiste y ayuda el funcionamiento del protocolo IP actuando como un sistema de control y notificación de errores.
ICMP supervisa el estado de la comunicación y si un router detecta un problema inesperado genera un mensaje ICMP y lo envía al emisor original. 
Estos mensajes viajan de forma segura encapsulando dentro de un paquete IP. 
#### Mensajes comunes:
- **Destino Inalcanzable:** Se genera cuando un paquete no puede ser entregado a su destino final. Esto puede ocurrir si un router no encuentra una ruta hacia la dirección IP de destino, si el host está apagado, o si un paquete grande con la bandera de "no fragmentación" (bit DF) se topa con un enlace físico con un límite de tamaño inferior y no puede atravesarlo.
- **Tiempo Excedido:** Se envía cuando un paquete es descartado porque su contador de tiempo de vida (TTL o Hop Limit) llega a cero. 
- **Echo/ Echo Reply:** Es un mecanismo simple de pregunta y respuesta para comprobar de forma remota si una máquina está activa y comunicada.
- **Redireccionamiento:** Ocurre cuando un router detecta que un paquete está viajando por una ruta ineficiente. Le envía este mensaje al emisor original para indicarle que actualice sus tablas de enrutamiento con un camino directo mucho mejor.
- **Problema de Parámetros:** Informa al host emisor que su paquete IP tiene una cabecera ilegal o corrupta. Esto suele revelar la presencia de fallos en el software IP del host emisor o de algún router del trayecto.
##### Comandos:
- La herramienta **traceroute (o tracert)**_:_ Utiliza este mensaje de error de forma ingeniosa para averiguar la lista de routers de una ruta. Envía paquetes sucesivos al destino empezando con un TTL de 1, luego de 2, luego de 3, obligando a cada router del camino a descartar el paquete y responder obedientemente con un mensaje de "tiempo excedido", revelando así su dirección IP.
- La herramienta **ping**_:_ Es el comando de diagnóstico que todos usamos para comprobar la conectividad. Envía una solicitud de eco ICMP al host de destino y, si este se encuentra activo, queda obligado a responder con un mensaje de respuesta de eco.
##### Cabecera:
![[Pasted image 20260830000905.png]]
### B) ARP - Address Resolution Problem:
El  ARP es el puente que permite comunicar el mundo de las direcciones lógicas (IP) con el mundo de las direcciones físicas (MAC) dentro de una misma red local.
Para entenderlo de manera simple, debes saber que los sistemas operativos y los programas se comunican usando direcciones IP, pero las tarjetas de red físicas (NIC, como las de Ethernet) no entienden de IPs; ellas solo saben enviar y recibir datos utilizando su **dirección física MAC de 48 bits** grabada de fábrica. ARP es el traductor que se encarga de averiguar a qué dirección MAC corresponde una dirección IP determinada.

#### ¿Cómo funciona el proceso básico de ARP?

Cuando el Host A quiere enviar un paquete al Host B dentro de su misma red, ocurre lo siguiente:
1. **Pregunta en Broadcast:** El Host A conoce la dirección IP del Host B, pero no su MAC. Como no sabe a dónde enviarlo físicamente, manda un mensaje especial de broadcast a toda la red que dice: "¿Quién tiene la dirección IP X.X.X.X? Por favor, envíame tu dirección MAC". 
2. **La respuesta en Unicast:** Todos los equipos reciben la pregunta, pero la ignoran al ver que no es su IP. Únicamente el dueño de esa IP responde directamente al Host A diciéndole: "Yo tengo esa IP, mi dirección MAC es Y-Y-Y-Y".
3. **El envío:** Al recibir la respuesta, el Host A finalmente mete el paquete IP dentro de una trama física dirigida a esa MAC y la transmite.

#### Mecanismos especiales:
- **La Tabla ARP (o Caché ARP):** Cada computadora guarda en su memoria RAM una tabla temporal con los mapeos que ya ha aprendido. Así, la próxima vez que quiera enviar un paquete a esa misma máquina, simplemente consulta su caché y evita hacer una pregunta ruidosa a la red. Estas entradas expiran y se borran después de unos minutos en caso de que algún equipo cambie de IP.
- **¿Qué pasa si el destino está fuera de la red local?** Si el Host A ve que la IP de destino está en otra subred (por ejemplo, en Internet), sabe que no puede comunicarse directamente. Por lo tanto, en lugar de preguntar por la MAC del destino final, usa ARP para averiguar la dirección MAC de su puerta de enlace predeterminada y le envía el paquete a él.
- **ARP Gratuito:** Cuando una computadora se enciende o cambia de IP, emite una pregunta ARP buscando su propia dirección IP recién asignada. Sirve para dos cosas: comprobar que nadie más en la red esté usando su misma IP (evitando conflictos) y forzar a que todos los vecinos actualicen sus tablas ARP con su nueva dirección.
- **Proxy ARP:** Es un caso especial en el que un router responde a una solicitud ARP en nombre de un host que está en otra red distinta, entregando su propia dirección MAC para que el tráfico pase a través de él.

![[Pasted image 20260830002906.png]]
### C) DHCP - Dynamic Host Configuration Protocol:
El DHCP es el servicio encargado de asignar de forma automática y temporal direcciones IP y otros parámetros de configuración a los dispositivos que se conectan a una red.
Sin DHCP, un administrador tendría que configurar a mano cada computadora, teléfono o impresora de la oficina, lo cual resulta sumamente tedioso y propenso a cometer errores de digitación.

##### Datos que configura DHCP en un dispositivo:
Además de otorgar una **dirección IP** libre para que el dispositivo pueda comunicarse, el servidor DHCP le entrega automáticamente otros parámetros esenciales:
1. **Máscara de subred:** Indica el tamaño y límite de la red local.
2. **Puerta de enlace predeterminada:** La IP del router para poder salir hacia Internet.
3. **Servidores DNS:** Las direcciones de los servidores que traducen nombres de dominio (como `google.com`) a direcciones IP.
4. **Servidor de hora:** Para mantener el reloj del sistema sincronizado.

****Leasing:**** Si un dispositivo se desconecta definitivamente de la red o su dueño se va del edificio, esa IP se quedaría retenida e inutilizable para siempre. Gracias al leasing, si el dispositivo se retira, el tiempo expira y la IP vuelve al pool de direcciones libres para que la use otra máquina. Si el dispositivo sigue conectado, tiene la obligación de **solicitar una renovación de su arrendamiento DHCP** justo antes de que este expire. Si no lo hace o si el servidor deniega la solicitud, el dispositivo debe dejar de utilizar esa IP de inmediato.
##### Proceso de asignación dinámica:
1. **DHCP DISCOVER (Descubrimiento):** Como el cliente no tiene IP ni sabe dónde está el servidor, envía un mensaje de difusión (_broadcast_) a toda la red diciendo: _"¡Hola! Soy la tarjeta de red con dirección MAC tal, ¿hay algún servidor DHCP por aquí que me dé una IP?"_. (Si el servidor está en otra red distinta, los routers están preparados para recibir esta difusión y reenviarla directamente al servidor).
2. **DHCP OFFER (Oferta):** El servidor DHCP recibe la solicitud, consulta su lista de IPs libres, selecciona una y se la ofrece al cliente enviándole un mensaje que contiene la IP propuesta y los datos de red. Para que el mensaje llegue a su destino antes de que el cliente tenga una IP, el servidor lo identifica usando su dirección MAC física.
3. **DHCP REQUEST (Solicitud):** Como puede haber más de un servidor DHCP ofreciendo IPs al mismo tiempo, el cliente elige una de las ofertas recibidas y envía una confirmación en modo de difusión (_broadcast_). Este mensaje sirve para avisarle formalmente al servidor elegido: _"He revisado tu oferta y me gusta, me quedo con ella"_, y de paso permite que los demás servidores enterados retiren sus ofertas y liberen esas IPs para otros.
4. **DHCP ACK (Acuse de recibo):** El servidor elegido finaliza el proceso enviando un mensaje directo de aceptación (_unicast_) donde le dice: _"¡Estamos listos! Aquí tienes tu configuración de red completa, ya puedes navegar"_.

### D) MPLS - MultiProtocol Label Switching :
El protocolo MPLS es una tecnología utilizada principalmente por los proveedores de servicios de Internet (ISP) para mover el tráfico de datos a través de sus redes de manera sumamente rápida y eficiente.

En el enrutamiento IP tradicional, cuando un paquete llega a un router, este debe examinar la dirección IP de destino completa (de 32 o 128 bits), realizar búsquedas complejas en su tabla de enrutamiento y buscar la coincidencia de prefijo más larga para decidir a dónde enviarlo. Este proceso puede resultar lento cuando hay millones de rutas.

Agregan una **etiqueta** al paquete cuando entra a la red. A partir de ese momento, los routers internos de la red solo leen esa etiqueta. Como la etiqueta es simplemente un número entero pequeño, el router la usa como un índice directo en una tabla interna, encontrando la interfaz de salida de forma casi instantánea.

#### Componentes:
1. **LER (Label Edge Router - Router de borde):** Se encuentra en los límites de la red MPLS. Cuando un paquete IP común entra a la red, el LER analiza su dirección IP de destino, decide qué ruta debe seguir, le añade la cabecera con la **etiqueta** adecuada y lo introduce en la red MPLS. Cuando el paquete sale de la red MPLS hacia su destino final, otro LER le quita la etiqueta y lo entrega como un paquete IP normal.
2. **LSR (Label Switched Router - Router de conmutación):** Son los routers internos de la red. No miran las IPs; solo reciben el paquete, leen su etiqueta, consultan la tabla interna, reemplazan la etiqueta por una nueva (_label swapping_) y lo envían al siguiente router. Las etiquetas solo tienen un significado local entre routers vecinos.
3. **FEC (Forwarding Equivalence Class):** Es el grupo de paquetes que reciben el mismo trato y comparten la misma ruta dentro de la red MPLS. Los routers agrupan de forma inteligente múltiples flujos de datos bajo una única etiqueta para simplificar el proceso.
#### Cabecera:
![[Pasted image 20260830002838.png]]
Para que esto funcione, se inserta una cabecera MPLS de **4 bytes (32 bits)** justo antes de la cabecera IP. Esta cabecera contiene cuatro campos muy sencillos:
- **Etiqueta (20 bits):** Contiene el número de circuito o índice que sirve para identificar la ruta.
- **QoS (3 bits):** Indica la clase o prioridad del servicio para saber qué paquetes deben pasar primero si hay congestión.
- **S (1 bit):** Sirve para el **apilamiento de etiquetas** (_label stacking_). Permite meter etiquetas dentro de otras (como túneles dentro de túneles). Este bit se pone en `1` si es la etiqueta del fondo de la pila y en `0` para el resto.
- **TTL (8 bits):** Tiempo de vida (_Time to Live_). Se decrementa en 1 en cada router para evitar que un paquete dé vueltas infinitas en la red en caso de error.
##### ¿Por qué los ISP prefieren usar MPLS?
- **Velocidad de conmutación:** Buscar un índice simple en una tabla es mucho más rápido que procesar complejas cabeceras IP completas.
- **Ingeniería de tráfico:** Permite a los operadores diseñar rutas específicas, evitando que todos los paquetes usen el mismo enlace saturado y distribuyendo la carga de manera eficiente.
- **Soporte de VPNs y QoS:** Facilita la creación de redes privadas virtuales (VPN) transparentes y seguras para empresas, separando su tráfico del resto de Internet y garantizándoles un ancho de banda específico.
- **Es "Multiprotocolo":** Al ser independiente de las capas superior e inferior, puede transportar tanto paquetes IP como de otros protocolos antiguos o especializados.
### E) OSPF - Open Shortest Path First:
OSPF es un protocolo de enrutamiento dinámico de tipo estado de enlace. Se clasifica como un IGP (Interior Gateway Protocol), lo que significa que su función es organizar de la forma más eficiente posible el movimiento de paquetes de datos dentro de una misma red corporativa o Sistema Autónomo (AS) (como la red de una universidad, una empresa o un ISP). 
En OSPF, cada router tiene un mapa completo y detallado de toda la red. Este mapa se representa matemáticamente como un grafo dirigido. Con este mapa en su memoria, cada router calcula de forma independiente el camino más rápido y libre de fallos para enviar sus paquetes. Este proceso ayuda a balancear la carga y se conoce como ECMP (Multiruta de Igual Costo o Equal Cost MultiPath).
![[Pasted image 20260830003802.png]]

#### Tres características inteligentes de OSPF.
Para funcionar de manera eficiente en redes del mundo real, OSPF incorpora tres mecanismos clave:
- **Balanceo de carga (ECMP):** Si el algoritmo encuentra que existen dos o más caminos diferentes hacia un destino que tienen exactamente el mismo costo mínimo, OSPF no desperdicia ninguno. En su lugar, **divide el tráfico de forma equitativa entre todos esos caminos** para equilibrar la carga de la red (mecanismo conocido como _Equal Cost MultiPath_).
- **El Router Designado (DR):** Si tienes una red LAN (como Ethernet conmutada) con 10 routers conectados, hacer que todos hablen e intercambien mapas con todos generaría un tráfico caótico e innecesario. Para evitarlo, OSPF elige democráticamente a un **Router Designado (DR)** y a un sustituto de reserva (BDR). Todos los routers de la LAN solo le envían sus actualizaciones al DR, y este se encarga de redistribuirlas de forma ordenada a los demás.
- **Organización en Áreas (Jerarquía):** Cuando una red corporativa crece y se vuelve gigantesca, las tablas de enrutamiento pueden volverse demasiado grandes y consumir toda la memoria y procesador de los routers. Para solucionarlo, OSPF permite dividir la red en subgrupos llamados **áreas**.
    - Existe un área central obligatoria llamada **Área 0 (Backbone)**.
    - Todas las demás áreas (ej. Área 1, Área 2) se conectan al Área 0 a través de routers especiales llamados **ABR (Area Border Routers)**.
    - La ventaja es que los routers internos de un área no necesitan conocer la topología detallada de las otras áreas; solo necesitan un resumen de rutas que les proporciona el ABR, lo que simplifica enormemente sus cálculos de Dijkstra.

![[Pasted image 20260830004018.png]]
### F) BGP:
Es el protocolo de enrutamiento dinámico que actúa como el "pegamento" de Internet. Su función principal es interconectar y dirigir el tráfico entre los diferentes **Sistemas Autónomos (AS)**. Un Sistema Autónomo es una red independiente administrada por una sola organización (como un gran ISP corporativo, una universidad o una multinacional tecnológica).
Para entender BGP de manera sencilla y directa, es útil analizar sus conceptos fundamentales:
#### Enrutamiento basado en políticas (No en la ruta más rápida)
A diferencia de protocolos internos como OSPF (que solo buscan mover paquetes dentro de una red de la forma más rápida y con el menor costo posible), **BGP se enfoca en políticas de enrutamiento comerciales, políticas o de seguridad**.
A los operadores de Internet no les basta con saber cuál es el camino físico más corto; necesitan aplicar reglas específicas sobre por dónde puede o no viajar el tráfico. Por ejemplo, las políticas típicas en BGP pueden incluir:

- _"No transportar tráfico comercial de un tercero por la red académica gratis"_.
- _"El tráfico que se origina en Apple no debe transitar bajo ninguna circunstancia por los servidores de Google"_.
- _"Prefiere la ruta a través del proveedor TeliaSonera en lugar de Verizon porque es más económica"_.
#### ¿Cómo funciona técnicamente? (El Vector de Ruta)
BGP no calcula rutas basándose en "métricas de velocidad" o en "número de routers individuales". En su lugar, es un **protocolo de vector de ruta** (_path vector protocol_):
- **El camino de Sistemas Autónomos:** Cuando un router anuncia una ruta hacia un destino, este anuncio incluye una lista con los números de todos los AS que el tráfico tendrá que atravesar (por ejemplo: _"Para llegar a la red C, debes pasar por el AS2 y luego por el AS3"_).
- **Evita bucles de forma simple:** Cuando un router recibe un anuncio de ruta, revisa la lista de AS. Si ve su propio número de AS en la lista, sabe que el paquete daría vueltas en círculos; por lo tanto, **descarta el anuncio de inmediato**.
- **Conexión confiable:** Para intercambiar estas tablas de rutas de manera segura y sin errores, los routers que hablan BGP se comunican estableciendo conexiones estables y directas a través del protocolo **TCP**.
#### Las Relaciones Comerciales en BGP (Tránsito vs. Peering)
Dado que Internet es una red de redes privadas, el tráfico se mueve según acuerdos comerciales que se configuran en BGP:
- **Servicio de Tránsito (_Transit_):** Ocurre cuando una red cliente le paga a una red proveedora para que transporte sus paquetes a cualquier parte de Internet. El proveedor anuncia en BGP todas las rutas del mundo hacia su cliente.
- **Interconexión sin liquidación (_Peering_):** Ocurre cuando dos redes grandes acuerdan conectar sus routers directamente para intercambiar tráfico de manera gratuita porque a ambas les beneficia económicamente. Sin embargo, este acuerdo **no es transitivo**: si el AS-A tiene peering con el AS-B, y el AS-B tiene peering con el AS-C, el AS-A no puede usar al AS-B para enviarle datos de forma gratuita al AS-C.
#### BGP Externo (eBGP) vs. BGP Interno (iBGP)
El protocolo se divide en dos variantes según los límites de la red:
- **eBGP (External BGP):** Se utiliza para comunicar routers que pertenecen a **diferentes Sistemas Autónomos** (por ejemplo, entre tu proveedor local y una red troncal de Google).
- **iBGP (Internal BGP):** Se utiliza para que los routers que están en las fronteras de un **mismo Sistema Autónomo** compartan de forma consistente las rutas que han aprendido del exterior.
#### Enrutamiento de la "Patata Caliente" (_Hot-Potato Routing_)
Una de las estrategias más comunes configuradas en BGP es la **salida anticipada**. Si un paquete entra a la red de un ISP con destino a otra red lejana, el ISP no quiere gastar sus recursos transportándolo internamente. En su lugar, busca **deshacerse del paquete lo antes posible**, enviándolo al router de salida fronterizo más cercano. Esto tiene el curioso efecto de hacer que el enrutamiento de Internet sea asimétrico: el camino que siguen tus datos para ir a un servidor casi nunca es el mismo que toman para regresar.

---

# 2. Multicast en Internet:
La **multidifusión (multicast)** es un modelo de entrega de paquetes donde un único emisor transmite un mensaje de forma simultánea a un **grupo específico de receptores interesados**, en lugar de a un solo destino (_unicast_) o a absolutamente todos los dispositivos de la red (_broadcast_).
Es el método ideal para aplicaciones multimedia en tiempo real, como transmisiones de video en vivo (IPTV), videoconferencias o juegos multijugador, ya que evita que el emisor tenga que duplicar y enviar el mismo archivo miles de veces, ahorrando un inmenso ancho de banda en la red.
Para entender su funcionamiento en Internet de manera sencilla, el proceso se apoya en tres pilares:
#### 1. Las Direcciones de Grupo (Clase D)
Para que los receptores puedan identificarse como miembros de un grupo común, la multidifusión utiliza un rango especial de direcciones IP IPv4 conocidas como **direcciones de Clase D** (que abarcan desde la `224.0.0.0` hasta la `239.255.255.255`).
- **Ámbito local:** El rango `224.0.0.0/24` está reservado exclusivamente para la red local (LAN). Aquí no se requiere enrutamiento; por ejemplo, la IP `224.0.0.1` se usa para enviar un mensaje a todos los hosts de la LAN, y la `224.0.0.2` a todos los routers locales.
- **Ámbito global:** Las demás direcciones de Clase D permiten agrupar a miembros que se encuentran en diferentes redes a lo largo de Internet, lo cual sí requiere el trabajo de los routers para mover los paquetes entre redes.
#### 2. ¿Cómo se une un dispositivo a un grupo? (IGMP)
Cuando una aplicación en tu computadora quiere recibir el contenido de un grupo de multidifusión, el host debe avisarle a su router local que desea unirse. Esto lo hace a través del protocolo **IGMP (Internet Group Management Protocol)**.
- El router multicast envía consultas periódicas (aproximadamente una vez por minuto) a todos los dispositivos de su LAN para saber a qué grupos pertenecen.
- Los dispositivos interesados responden indicando las direcciones de Clase D que quieren escuchar.
- Si el último dispositivo de la LAN abandona un grupo, el router se entera y deja de solicitar ese tráfico para no saturar la red local de forma innecesaria.
#### 3. El enrutamiento de los paquetes por la red (PIM)
Una vez que los routers saben dónde están los usuarios interesados, deben construir de forma dinámica un camino eficiente (un árbol de expansión sin bucles) para hacerles llegar los paquetes desde el emisor original.
El protocolo estándar y más utilizado hoy en día dentro de las redes de los operadores es **PIM (Protocol Independent Multicast)**, el cual funciona principalmente en dos modos según cómo estén distribuidos los receptores:
- **Modo Denso (PIM-DM):** Se utiliza cuando los miembros del grupo están muy concentrados por toda la red (por ejemplo, al distribuir archivos a muchos servidores en un centro de datos). El router asume que todos quieren el paquete, lo envía a todas partes y luego "poda" (_prune_) recursivamente las ramas de la red donde los routers vecinos le avisen mediante mensajes `PRUNE` que no tienen hosts interesados.
- **Modo Disperso (PIM-SM):** Se utiliza cuando los miembros están muy separados geográficamente en Internet (como los abonados de televisión de un ISP). En lugar de inundar la red, se utiliza un router central como **núcleo** (_core_ o punto de encuentro). Los routers de los usuarios interesados deben pedirle explícitamente al núcleo unirse al árbol, de modo que el tráfico solo viaja por las ramas que realmente lo solicitaron.
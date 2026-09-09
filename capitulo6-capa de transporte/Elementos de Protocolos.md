El servicio de transporte se implementa mediante protocolos de transporte, usados entre dos entidades de transporte. Son las reglas y mecanismos que utilizan las entidades de transporte para implementar un servicio de transmisión de datos eficiente y confiable entre procesos de diferentes máquinas.
Aunque estos protocolos comparten similitudes con los de la capa de enlace de datos (como el control de errores, la secuenciación y el control de flujo), el entorno en el que operan es radicalmente diferente. En la capa de enlace, dos enrutadores se comunican mediante un canal físico directo, mientras que en la de transporte, este enlace es reemplazado por toda una subred compleja que puede retrasar, duplicar o perder paquetes.
#### Diferencias entre Protocolos de Datos vs Protocolos de Transporte:
1. **Medio de comunicación:** En enlaces punto a punto no es requerido el direccionamiento. En la capa de transporte, al operar sobre toda una red compleja, requiere un **direccionamiento explícito**. Utiliza direcciones lógicas de transporte (llamadas **TSAP o números de puerto**)
2. **Establecimiento de conexión:** Sencillo en la capa de enlace de datos, como ambas máquinas están físicamente unidas por un enlace directo. Complejo en transporte, al viajar a través de múltiples enrutadores de una red que puede retrasar y duplicar paquetes, se debe implementar protocolos sofisticados para evitar que paquetes viejos o duplicados con retraso inicien conexiones accidentales.
3. **Almacenamiento en la red:** Las tramas llegan inmediatamente o se pierden por el camino, pero nunca se estancan en el camino. La red interna es capaz de almacenar un registro de paquetes para controlar errores como retrasos, duplicados o pérdidas de mensajes en la capa de transporte.
4. **Control de flujo y buffers:** En la capa de enlace se manejan pocas cantidades de enlaces. En la capa de transporte, una sola máquina puede gestionar miles de conexiones y requiere esquemas de control de flujo.
## 1.1. Direccionamiento:
Son los mecanismos para identificar y ubicar explícitamente el proceso de aplicación de destino al que se deben enviar los datos. La capa de transporte requiere un direccionamiento explícito. Cuando un programa cliente quiere comunicarse con un servidor remoto, no le basta con conocer la máquina física; necesita especificar **a qué proceso específico** dentro del equipo remoto debe conectarse. 
1. **TSAP (Transport Service Access Point):**
    - Es la dirección lógica correspondiente a la **capa de transporte**.
    - Son conocidos comúnmente como **números de puerto** (por ejemplo, el puerto TCP 80 para servidores web o el puerto TCP 25 para correo).
    - Su propósito principal es distinguir entre los múltiples procesos o aplicaciones que se están ejecutando simultáneamente en una misma computadora.
2. **NSAP (Network Service Access Point / Punto de Acceso al Servicio de Red):**
    - Es la dirección correspondiente a la **capa de red**.
    - En las redes modernas de Internet, los NSAP son representados por las **direcciones IP**.
    - Su función es identificar de manera única a la máquina o _host_ dentro de toda la red global.
#### La analogía para entenderlo de forma simple
Para comprender la relación lógica entre ambos conceptos, el libro ilustra que la conexión de transporte se realiza a través de estos dos puntos de acceso:
- El **NSAP (dirección IP)** es como la **dirección física de un edificio de oficinas**. Te indica a qué edificio del mundo debe llegar la correspondencia.
- El **TSAP (número de puerto)** es como el **número de oficina o el nombre del destinatario** dentro de ese edificio. Aunque el edificio (host) tenga una sola dirección de entrada (IP), necesitas el número de oficina (puerto) para que el paquete de datos se le entregue a la persona correcta (el proceso de aplicación, como el servidor de correo o el navegador).
#### Proceso de envío:
**En el host origen:**
- **Inicio de la solicitud:** Un proceso de aplicación en el origen (ej. un navegador) quiere establecer conexión con un proceso remoto y especifica el **TSAP de destino** (puerto remoto, ej. 80) y el **NSAP de destino** (dirección IP del servidor).
- **Encapsulamiento en Transporte:** La entidad de transporte de origen asigna un **TSAP de origen** (puerto efímero del cliente). Construye el segmento de transporte incluyendo el TSAP origen y el TSAP destino en la cabecera.
- **Encapsulamiento en Red:** El segmento se pasa a la capa de red. Esta añade la cabecera IP con el **NSAP de origen** y el **NSAP de destino**, creando el paquete o datagrama.
- **Baja a capas inferiores:** El paquete pasa a la capa de enlace de datos (donde se añaden direcciones MAC si es un medio compartido) y finalmente se transmite en forma de bits por el medio físico.

**En el host destino:**
- **Desencapsulamiento en Red:** La interfaz de red del host receptor captura la trama, y la capa de red verifica que el **NSAP de destino** coincida con su propia dirección de red. Extrae el paquete y lo entrega a la capa de transporte.
- **Identificación por TSAP (Demultiplexación):** La entidad de transporte lee el **TSAP de destino** contenido en la cabecera del segmento. Con este valor (número de puerto), localiza el proceso de aplicación que está escuchando en ese puerto específico y le entrega la carga útil de los datos.
### Cómo se conocen los números TSAP de los servidores?
#### Portmapper:
Un **Portmapper** (también conocido como _mapeador de puertos_ o servidor de nombres de transporte) es un **servicio auxiliar** en un sistema operativo que actúa como si fuera la **«central telefónica» o las «páginas amarillas» de los puertos (TSAPs)** dentro de ese equipo.
1. **El problema:** Existen servicios bien conocidos que siempre usan el mismo número de puerto (por ejemplo, la web usa siempre el puerto `80` o `443`). Sin embargo, existen muchas aplicaciones cuyos puertos **no son fijos**, sino que cambian o se asignan de forma **dinámica** cada vez que el servidor se enciende o se ejecuta. Si un cliente quiere conectarse a uno de estos servicios, no tiene forma de saber de antemano a qué número de puerto llamar.
2. **La solución (Portmapper):** En lugar de adivinar el puerto, el `portmapper` siempre se ejecuta en un **puerto fijo y conocido** (por ejemplo, el puerto `111` en sistemas UNIX). Funciona como un intermediario que le dice a los clientes a qué puerto específico deben conectarse.
##### ¿Cómo funciona en la práctica? (Paso a paso)
Imagina que un cliente quiere usar el servicio de archivos en red **NFS** o un procedimiento **RPC** en un servidor remoto:
- **Paso 1 (Registro del servicio):** Cuando el servidor de archivos arranca, obtiene un puerto dinámico (por ejemplo, el puerto `4005`). Inmediatamente le avisa al Portmapper local: _«Hola, soy el servicio de archivos y hoy estoy escuchando en el puerto 4005»_.
- **Paso 2 (Consulta del cliente):** El cliente quiere conectarse al servicio de archivos, pero no sabe el puerto. En lugar de llamar directo al servicio, llama al **Portmapper** (que sabe que siempre está en el puerto `111`) y le pregunta: _«¿En qué puerto está corriendo el servicio de archivos?»_.
- **Paso 3 (Respuesta):** El Portmapper busca en su tabla interna y le responde al cliente: _«Está en el puerto 4005»_.
- **Paso 4 (Conexión directa):** El cliente cierra la comunicación con el Portmapper y abre su conexión directa hacia el puerto `4005` del servidor.
#### ICP (Initial Connection Protocol):
El **ICP (Initial Connection Protocol)** es un conjunto de reglas estándar o un **mecanismo de "apretón de manos" inicial** que utilizan dos entidades de transporte para **ponerse de acuerdo y establecer una conexión formal** antes de empezar a transferir datos reales.
En la analogía telefónica, el ICP equivale al momento en que llamas a alguien, la otra persona atiende, ambos dicen _"¿Hola?", "¿Me escuchas bien?"_ y confirman que están listos para hablar antes de iniciar la conversación.
- **El problema:** Las redes no son perfectas: los paquetes pueden **retardarse, duplicarse, perderse o llegar fuera de orden**. Si simplemente empezáramos a enviar datos sin un acuerdo previo, podríamos encontrarnos con situaciones caóticas como:
    - Que el servidor reciba un mensaje viejo duplicado de una conexión anterior que ya cerró y piense que es una conexión nueva.
    - Que los dos equipos inicien la comunicación usando números de secuencia distintos para contar los datos, provocando que la información se mezcle o desordene.
- **La solución (ICP):**
    El ICP establece una secuencia ordenada de pasos para garantizar que **ambos extremos estén listos, de acuerdo con los números de inicio (números de secuencia) y que no se estén confundiendo con conexiones fantasmas o antiguas**.

##### ¿Cómo funciona en la práctica? (El "Handshake" de 3 Vías)
El ejemplo más famoso y práctico de un ICP en las redes modernas es el **Acuerdo en Tres Pasos (Three-Way Handshake)** de TCP:

![[Pasted image 20260908145450.png]]

- **Paso 1 (SYN - Solicitud de inicio):** El cliente envía un segmento especial al servidor indicando: _«Quiero conectarme a tu puerto X (TSAP). Mi número de inicio para contar datos será el 100»_.
- **Paso 2 (SYN + ACK - Confirmación y respuesta):** El servidor recibe la solicitud, la acepta y le responde: _«Recibido tu número 100 (ACK). Acepto la conexión y mi propio número de inicio para enviarte datos será el 300 (SYN)»_.
- **Paso 3 (ACK - Confirmación final):** El cliente confirma la recepción de la respuesta del servidor: _«Perfecto, recibí tu número 300. ¡Conexión lista!»_.

A partir de este punto del ICP, la conexión queda en estado **ESTABLECIDA** y recién ahí se comienzan a enviar los datos de la aplicación.
## 1.2. Establecimiento de la Conexión:
### **Problema:**
El principal motivo por el que existe esta sección es resolver qué sucede cuando un paquete o segmento se retarda en la red. El establecimiento de una conexión es uno de los problemas más complejos en redes debido a que **la red subyacente no es confiable**.
- Imagina que un cliente intenta conectarse, pero el mensaje de conexión se queda atascado en un router lento. El cliente da por perdida la conexión y se rinde.
- Tiempo después, ese paquete retrasado finalmente llega al servidor. Si el servidor simplemente aceptara la conexión, estaría creando una **conexión fantasma** pensando que el cliente quiere hablar con él, malgastando recursos.
- Peor aún, si ese paquete duplicado contiene datos de una transferencia vieja, los datos nuevos podrían mezclarse y corromperse.
### Soluciones:
#### - Direcciones de Transporte desechables:
Es un método en el que las direcciones de transporte (los puntos de acceso al servicio de transporte o **TSAP**) se vuelven temporales. En lugar de usar direcciones fijas, cada conexión de transporte utiliza una dirección nueva y exclusiva que se destruye al terminar la comunicación.
El mecanismo funciona de la siguiente manera:
1. **Generación dinámica:** Cada vez que una entidad de transporte necesita iniciar una conexión, genera una dirección de transporte (TSAP) completamente nueva.
2. **Uso exclusivo:** La comunicación de datos se realiza únicamente a través de esa dirección específica.
3. **Descarte definitivo:** En cuanto la conexión se libera (se cierra), la dirección **se descarta y no se vuelve a utilizar jamás**.
Al operar así, si un paquete duplicado que se había quedado retenido temporalmente en los routers de la red reaparece mucho tiempo después de que la sesión terminó, intentará buscar la dirección original. Como esa dirección ya fue desechada y ya no existe, el paquete se descarta de forma segura, evitando que llegue a un proceso de transporte activo y **previniendo que cause daños o conexiones falsas accidentales**.
##### Su limitación técnica
Aunque es un método efectivo para neutralizar duplicados, el material advierte que **no es una solución muy satisfactoria**. El principal inconveniente es que, al cambiar constantemente de TSAP, **se dificulta notablemente el proceso de conexión inicial**. Al no haber una dirección fija o predecible, los clientes tienen problemas para saber a qué dirección exacta deben apuntar para encontrar un proceso o servicio en primer lugar.
#### - Identificadores únicos de conexión:
Consiste en asignar a cada nueva conexión establecida un **identificador único** (generalmente un número de secuencia que se incrementa de forma secuencial). Este identificador viaja dentro de cada segmento transmitido para distinguir de forma inequívoca una sesión de otra.
El protocolo ejecuta este proceso en tres pasos clave:
1. **Asignación en el origen:** El host que inicia la comunicación elige un identificador único secuencial y lo coloca en cada uno de los segmentos que envía, incluyendo la solicitud de conexión inicial.
2. **Registro de obsolescencia:** Cuando una conexión termina y se libera, cada **entidad de transporte** involucrada registra esa conexión finalizada en una tabla local. La guarda estructurada como un par: `(entidad de transporte homóloga, identificador de conexión)`.
3. **Verificación y descarte:** Cada vez que llega una nueva solicitud de conexión, el receptor busca el identificador en su tabla interna de conexiones obsoletas. Si encuentra una coincidencia, sabe de inmediato que se trata de un duplicado antiguo retrasado en la red y lo rechaza de forma segura.
##### Su limitación técnica
A pesar de su lógica clara, este esquema presenta un **defecto de diseño insalvable** que lo hace poco práctico en escenarios reales: **obliga a las entidades de transporte a almacenar información histórica de manera indefinida**.
Esta tabla debe persistir de forma ilimitada tanto en el origen como en el destino. Si uno de los hosts sufre una caída repentina (_crash_) o se reinicia, **pierde por completo el estado de su memoria RAM**. Al volver a encenderse, el dispositivo no tendrá ningún registro de las conexiones obsoletas ni de los identificadores que ya se utilizaron con sus pares, dejando la red completamente vulnerable a que los paquetes duplicados antiguos provoquen conexiones falsas accidentales.
#### - Limitar la vida de paquetes:
La solución de **limitar la vida de los paquetes** es el tercer método propuesto para resolver de raíz el problema de los **paquetes duplicados con retardo**.

A diferencia de las soluciones anteriores (que intentaban recordar identificadores para siempre), esta estrategia cambia el enfoque: en lugar de dejar que los paquetes viejos naveguen indefinidamente por la red, diseña un mecanismo para garantizar que cualquier paquete "muera" después de un tiempo máximo conocido.

Es una técnica de diseño que asegura que ningún paquete (ni su acuse de recibo o **ACK**) permanezca activo en la red más allá de un límite de tiempo estrictamente definido. Al ponerle una "fecha de caducidad" obligatoria a los datos, el problema de los duplicados se vuelve completamente manejable.

Para restringir y garantizar la muerte de un paquete, el protocolo de red utiliza una (o varias) de las siguientes técnicas físicas y lógicas:
1. **Diseño de red restringido:** Consiste en estructurar la topología de la red para evitar de forma estricta los ciclos o bucles de enrutamiento, estimando y limitando el retardo en el camino más largo posible.
2. **Contador de saltos (_Hop counter_):** Se introduce un contador numérico en cada paquete (como el TTL en IP) que disminuye en una unidad cada vez que el paquete pasa por un router. Si el contador llega a cero, el router descarta el paquete inmediatamente.
3. **Marca de tiempo (_Timestamping_):** Cada paquete se etiqueta con la hora exacta en la que fue creado. Los routers de la red, que deben tener sus relojes sincronizados, leen esta marca y descartan de inmediato cualquier paquete cuya antigüedad supere el límite permitido.
##### El Período de Seguridad (\(T\)):
Para que este sistema funcione de forma infalible en la capa de transporte, no basta con que muera el paquete; también deben haber desaparecido todos sus acuses de recibo (**ACK**).
Por ello, se define un **período de seguridad \(T\)**, que es un pequeño múltiplo del tiempo de vida máximo real del paquete (en Internet, este tiempo máximo se toma de forma conservadora como **120 segundos**). Si la entidad de transporte espera estos \(T\) segundos, puede estar absolutamente segura de que cualquier rastro del paquete viejo ha desaparecido de la red y que es totalmente seguro volver a utilizar ese número de secuencia.
#### - Acuerdo de las tres vías:
Es un proceso de verificación mutua en tres pasos que realizan dos máquinas antes de empezar a transmitirse datos. Su propósito fundamental es **diferenciar los paquetes legítimos de los paquetes duplicados con retardo**. Evita que un paquete viejo que se quedó atrapado temporalmente en la red reaparezca de la nada y engañe al receptor para abrir una conexión falsa o repetir una transacción por error.
##### ¿Cómo lo hace? (Los 3 pasos clave)
Imagina que el **Host 1** (cliente) quiere conectarse con el **Host 2** (servidor):

```
  Host 1                                     Host 2
    |                                          |
    | ---- 1. CONNECTION REQUEST (seq=x) ----> |
    |                                          |
    | <--- 2. ACK (seq=y, ACK=x) ------------- |
    |                                          |
    | ---- 3. DATA/ACK (seq=x, ACK=y) -------> |
    v                                          v
```

1. **Paso 1: Solicitud de conexión (Cliente \(\rightarrow\) Servidor)**  
    El Host 1 elige un número de secuencia inicial (\(x\)) de manera independiente y envía un segmento de solicitud de conexión (`CONNECTION REQUEST` o `CR`) que contiene dicho número.
2. **Paso 2: Acuse de recibo y propuesta (Servidor \(\rightarrow\) Cliente)**  
    El Host 2 recibe el segmento, lo acepta y responde enviando un acuse de recibo (`ACK`) que confirma haber recibido \(x\). Al mismo tiempo, propone su propio número de secuencia inicial (\(y\)) para el tráfico de vuelta.
3. **Paso 3: Confirmación final (Cliente \(\rightarrow\) Servidor)**  
    El Host 1 recibe la respuesta y confirma la elección de la secuencia \(y\) del Host 2. Esta confirmación final se envía de forma muy eficiente dentro del primer segmento de datos reales que el cliente le manda al servidor.
##### ¿Por qué es infalible contra duplicados viejos?
- **Si aparece una solicitud antigua duplicada:**  
    Si un paquete de solicitud (`CR`) de una conexión vieja y cerrada que estaba retenido en la red aparece de repente en el Host 2, este reaccionará enviando un `ACK` de vuelta para verificarlo (proponiendo un número \(y\)). Cuando el Host 1 reciba este acuse de recibo, se dará cuenta de que él no ha iniciado ninguna comunicación con ese número de secuencia antiguo, por lo que **rechazará el intento de inmediato** (`REJECT`). El Host 2 abandonará la conexión sin haber procesado ningún dato erróneo.
- **Si tanto la solicitud como el acuse de recibo son duplicados antiguos:**  
    Incluso en el peor de los casos, cuando reaparecen tanto un `CR` viejo como un `ACK` viejo, el Host 2 se dará cuenta de que el acuse de recibo que le llega no coincide con el número \(y\) que él acaba de proponer, identificándolo como un duplicado obsoleto y cancelando la sesión de inmediato.

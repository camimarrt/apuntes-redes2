IP: Es una dirección lógica de la capa de red. Su función es definir de manera única la conexión de un dispositivo a una red. Permite que los datos lleguen al destino correcto dentro de una Interred.
IPv4: Es la versión 4 del IP. Es el estándar más utilizado.

# 1. Estructura del Encabezado IPv4:

![[Pasted image 20260829192417.png|700]]

- **Versión**: Indica la versión del protocolo IP utilizada (valor 4).
- **IHL**: Indica la longitud de la cabecera en palabras de 32 bits, con un valor mínimo de 5 y un máximo de 15.
- **Servicios diferenciados**: Utilizado para clasificar el tráfico y aplicar calidad de servicio (QoS).
- **Longitud total**: Especifica el tamaño completo del datagrama IP, incluyendo la cabecera y los datos.
- **Identificación**: Ayuda a identificar los fragmentos individuales que pertenecen a un mismo datagrama original.
- **Indicadores (Flags - DF/MF)**: Banderas que controlan las reglas de fragmentación del paquete.
- **Desplazamiento de fragmentos**: Indica la posición exacta del fragmento dentro del datagrama completo.
- **Tiempo de vida (TTL)**: Se decrementa en 1 en cada salto y se desecha cuando llega al límite.
- **Protocolo**: Indica el protocolo de capa superior encapsulado (como TCP, UDP o ICMP) al que se debe entregar el paquete.
- **Suma de comprobación de la cabecera**: Permite detectar errores en la cabecera y se recalcula en cada salto.
- **Dirección de origen**: Especifica la dirección IP de la interfaz de red del emisor.
- **Dirección de destino**: Especifica la dirección IP de la interfaz de red del receptor.
- **Opciones**: Campo de longitud variable que permite añadir funciones adicionales o de prueba, comenzando con un código de 1 byte. 

Campos para *opciones*:
![[Pasted image 20260829193240.png|700]]

# 2. Direccionamiento:
Es una dirección lógica única e universal. Cada dirección IP es asignada por ICANN (Internet Corporation for Assignd Names and Numbers). 

**Una dirección IP está compuesta por dos partes:**
1. Porción de Red: Este segmento identifica a la red a la que pertenece el dispositivo.
2. Porción de Host:  Identifica el dispositivo específico dentro de esa red.

Tiene una longitud de 32 bits, que se separan de a 8 bits que se expresan en decimal.

![[Pasted image 20260829193529.png|700]]

Un block de direcciones puede ser definida como **x.y.z.t /n** en la cua*l x.y.z.t* define una de las direcciones y */n* define la máscara.

**Direcciones Especiales:**
![[Pasted image 20260829193759.png]]

- **Dirección de red:** Cada segmento de red tiene una dirección IP de red que contiene solo “0” en el segmento correspondiente al host y no puede ser asignado a ningún host.
- **Dirección de broadcast:** Dirección de difusión local. No puede ser asignada a ningún host.
- **Puerta de Enlace:** Cuando un host necesita enviar un paquete a un destino que está fuera de su subred local, no puede comunicarse directamente. Por lo tanto, debe dirigir el paquete a la puerta de enlace predeterminada, que es la interfaz física de un router conectada a ese segmento de red.

>[!tip] 
Maximo numero de hosts en un bloque (red o subred) Host=2k−2 k =( 32 - prefijo)
# 3. Subnetting -División en subredes:

La **división en subredes** es la práctica de dividir un bloque grande de direcciones IP asignado a una organización en múltiples partes o segmentos lógicos más pequeños y autónomos llamados **subredes**. Todas estas subredes deben estar contenidas dentro del bloque de direcciones IP principal que define a la red original.

El propósito fundamental de dividir una red en subredes es **reducir el tráfico de _broadcast_ (transmisión por difusión)**. Al segmentar la infraestructura, las transmisiones de difusión quedan confinadas a su subred de origen y no inundan toda la red corporativa. Además, presenta la ventaja de que **las redes externas siguen viendo toda la estructura interna como si fuera una sola red grande**, simplificando el enrutamiento desde el exterior.

# 4. CIDR: Classless Inter-Domain Routing.

Es el sistema estándar actual para la asignación de direcciones IP y el enrutamiento de paquetes en Internet.
###### ¿Cómo funciona la notación CIDR?
En lugar de depender exclusivamente de las máscaras de subred escritas en decimal, CIDR simplifica la representación de una red utilizando el formato $x.y.z.t /n$ :

- $x.y.z.t$ representa una de las direcciones IP pertenecientes al bloque.
- $/n$ (conocido como el **prefijo**) define la máscara de subred e indica cuántos bits (de los 32 bits que componen una dirección IPv4) están reservados para identificar la porción de la red.
- **Flexibilidad:** Permite que un bloque de direcciones se divida o combine de forma mucho más eficiente, adaptándose a las necesidades reales de cada organización.

##### A) Agregación de rutas:
El objetivo fundamental de CIDR es frenar el crecimiento explosivo de las **tablas de enrutamiento** en los routers troncales de Internet.
- Para lograrlo, utiliza la **agregación de rutas**, que permite juntar múltiples prefijos IP pequeños en uno solo más grande (a veces llamado **superred**).
- De esta forma, un router distante solo necesita una entrada en su tabla para representar a miles de hosts o redes pequeñas.
##### B) Prefijo más largo coincidente:
Debido a que con CIDR un destino puede estar contenido en varios prefijos de diferentes tamaños dentro de la misma tabla de enrutamiento, los routers aplican la regla del **prefijo más largo coincidente** (_longest matching prefix_).

- Esto significa que si un paquete coincide con una entrada de máscara `/20` y otra de `/24`, el router elegirá la entrada `/24` por ser la más específica para ese destino.
##### C) Direccionamiento por clases:
El **direccionamiento por clase** (_classful addressing_) fue el diseño original para la asignación de direcciones IP que precedió al sistema actual (CIDR). En este esquema, el espacio de direcciones IP se dividía en cinco categorías fijas, identificadas por los bits de mayor orden de la dirección.
![[Pasted image 20260829214451.png]]

##### Direcciones privadas en Internet:
Direcciones que no están asignadas en la Internet. Pueden ser utilizadas por las subredes que usan traducción de dirección de red (NAT), para conectarse a Internet (usando una o pocas direcciones IP públicas) o, por los hosts que no se conectan a Internet.
![[Pasted image 20260829214553.png]]

# 5. Protocolo NAT: Network Address Translation.

Es un mecanismo que cambia las direcciones IP y/o puertos de origen y destino de los paquetes de datos cuando estos entran o salen de una red. Su objetivo principal es **paliar la escasez de direcciones IPv4**.  Gracias a NAT, una red local puede utilizar un rango de direcciones IP privadas internamente y conectarse a Internet global utilizando una dirección IP pública.
### Tipos de NAT:

#### A) NAT Estática:
Se realiza un mapeo fijo de uno a uno, donde una dirección IP privada siempre se traduce a una dirección IP pública específica.
#### B) NAT Dinámica:
Se dispone de un conjunto (_pool_) de direcciones IP públicas. Cuando un host interno requiere conexión, el router le asigna dinámicamente una dirección pública que no esté siendo utilizada en ese momento. Cuando la conexión finaliza, la IP pública regresa al conjunto para que otro dispositivo pueda usarla.
#### C) PAT: Port Address Translation o NAT con sobrecarga:
Es el tipo más común (usado en hogares). Permite mapear múltiples direcciones privadas a través de una **única dirección pública** utilizando los números de **puerto TCP/UDP** para distinguir las conexiones. El router guarda una tabla que asocia la IP privada y el puerto de origen con la IP pública y un puerto asignado al azar. El router registra esta relación en una tabla para saber exactamente a qué dispositivo interno debe entregarle la respuesta cuando esta regrese desde Internet.
##### Limitaciones:
- **Viola el modelo arquitectónico de IP:** El principio fundamental de Internet exige que cada dirección IP identifique de manera única y global a una sola máquina en todo el mundo. NAT quiebra este principio al ocultar múltiples dispositivos detrás de un solo punto de entrada.
- **Transforma Internet en una red "orientada a la conexión":** Convierte un modelo sin conexión en uno dependiente del estado (_stateful_), obligando a la caja NAT a almacenar el mapeo de cada sesión. Si la tabla de mapeo se pierde por una falla o reinicio del router, todas las conexiones TCP activas se destruyen inmediatamente.
- **Ruptura de la independencia de capas:** Rompe la separación de capas (modelo OSI/TCP-IP), ya que la capa $k$ (capa de red / IP) no debería hacer suposiciones ni manipular la información de carga útil de la capa $k+1$ (capa de transporte / puertos TCP y UDP).
- **Dependencia de protocolos específicos:** Asume arbitrariamente que los paquetes transportan TCP o UDP. Cualquier otro protocolo de capa de transporte que no utilice puertos estándar enfrenta problemas graves de enrutamiento.
- **Incompatibilidad con aplicaciones de capa superior:** Aplicaciones que insertan direcciones IP dentro del cuerpo del texto o en los datos del protocolo de aplicación (como FTP, SIP o juegos) se rompen a menos que se usen técnicas especiales como _NAT Trasversal_.
- **Restricciones de conexiones entrantes y mapeo:** Dado que las entradas en la tabla NAT se generan únicamente mediante tráfico saliente, un usuario remoto no puede iniciar directamente una conexión hacia un equipo dentro de la red doméstica sin una configuración previa (como reenvío de puertos). Además, existe una restricción física en el número de asignaciones simultáneas basada en la cantidad de puertos disponibles.
# 6. Routers.
Son dispositivos de conmutación de capa 3. Su propósito es conectar dos o más redes físicas independientes y heterogéneas, actuando como una pasarela. 
### A) Componentes:
- **Memoria de Trabajo o RAM:** Es volátil y se encarga de guardar la **tabla de enrutamiento**, la configuración actual en ejecución (_running configuration_), las colas de paquetes en espera y el sistema operativo mientras el equipo está encendido.
- **NVRAM (Non-volatile RAM):** Almacena la **configuración de inicio** (_start-up configuration_), la cual se carga en la memoria RAM cada vez que el equipo arranca. Generalmente una memoria flash de poca capacidad.
- **Memoria Flash:** Es una memoria no volátil de mayor capacidad que guarda la **imagen del Sistema Operativo (IOS)**. Puede retener varias versiones del software.
- **Memoria ROM:** Contiene el programa de arranque o **bootstrap**, el software básico de inicio, pruebas de diagnóstico de hardware y rutinas de recuperación.
- **Interfaces de LAN:** Utilizadas para conectar el router a redes locales (típicamente puertos Ethernet).
- **Interfaces de WAN:** Permiten la conexión con redes de área amplia proporcionadas por los ISP (como enlaces seriales, fibra óptica o DSL).
- **Puertos de Administración:** Incluyen el **Puerto de Consola** (puerto serial no destinado al tráfico de red) y el puerto AUX para configurar el equipo de forma local o remota.
### B) Configuración Serial:

Para que dos dispositivos se comuniquen correctamente a través de una línea serial, ambos deben estar configurados con los mismos parámetros. Según las fuentes, los parámetros principales son:

> [!info]
> 
> 1. **Tasa de datos (velocidad):** Se mide en bits por segundo (bps) o baudios. Los valores comunes son **4800, 9600 o 19200 bps**..
> 2. **Bits por carácter:** Define cuántos bits se utilizan para representar un carácter ASCII, siendo habituales **5, 7 u 8 bits**.
> 3. **Bits de parada:** Son bits adicionales que indican el final de la transmisión de un carácter. Pueden configurarse **1, 1.5 o 2 bits**
> 4. **Paridad:** Un mecanismo básico de detección de errores. Las opciones son **Sin paridad (No), Par o Impar**.
> 5. **Control de flujo:** Evita que el receptor se sature. Puede ser:
> 
> - **Por Software:** Utiliza caracteres ASCII especiales como **XON** (carácter 17) y **XOFF** (carácter 19).
> - **Por Hardware:** Utiliza señales físicas como **RTS/CTS** (Request To Send / Clear To Send).

Esta configuración es la que se utiliza típicamente para acceder a la **consola de un router** o configurar **interfaces de WAN**.

### C) Transmisión asíncrona:
Dado que es una comunicación **asíncrona**, no existe una señal de reloj física compartida entre la computadora y el router para mantenerlos sincronizados constantemente. En su lugar, la sincronización se realiza carácter por carácter en la propia línea de datos, estructurando cada carácter de la siguiente manera:
1. **Estado inactivo:** Cuando no se están transmitiendo datos, la línea se mantiene en un nivel de voltaje constante (alto).
2. **Bit de inicio:** Es un bit de transición física (la señal cae a nivel bajo) que avisa al receptor que un carácter está a punto de llegar, permitiéndole sincronizar su reloj interno para leer los bits siguientes.
3. **Bits de datos:** El bloque de datos reales que representa al carácter (típicamente de **5 a 8 bits** utilizando el código ASCII).
4. **Bit de paridad :** Un bit opcional de control de errores para verificar que los bits no se hayan corrompido en el cable.
5. **Elemento de parada:** Uno o más bits de nivel alto (**1, 1.5 o 2 bits**) que marcan el final del carácter y devuelven la línea al estado inactivo, preparándola para la llegada del siguiente bloque.

### D) Tabla de Enrutamiento:
La **tabla de enrutamiento** es una base de datos interna almacenada en la memoria RAM del router. Su función principal es servir de guía para que el software de enrutamiento determine por cuál de sus interfaces físicas de salida debe enviar un paquete recibido.
Cada paquete de datos contiene una dirección IP de destino. El router extrae esa dirección y busca una coincidencia en su tabla. 

Cada entrada de la tabla consta principalmente de:
- La **red o subred de destino** (representada mediante su dirección IP y máscara de subred o prefijo CIDR).
- La **línea o interfaz de salida** a utilizar para alcanzar dicho destino.
- A menudo, una **métrica** (el costo o la distancia del trayecto).
#### Tipos de entradas en una tabla de enrutamiento:
Por defecto, una tabla de enrutamiento contiene tres tipos de rutas:
1. **Subredes directamente conectadas:** Redes que están unidas físicamente a las interfaces del propio router.
2. **Ruta por defecto (Default Gateway):** Es la regla que utiliza el router para enviar paquetes cuando no encuentra una ruta específica para el destino en su tabla.
3. **Rutas estáticas o dinámicas:** Rutas hacia redes remotas que el router ha aprendido de forma manual o automática.

#### Enrutamiento estático:
Se basa en algoritmos **no adaptativos**. Un administrador de red calcula los caminos de antemano de forma manual y configura directamente las rutas en la tabla del router.
- **Características:**
    - **Rígido:** El router no cambia sus decisiones de envío por sí solo. Si un enlace en el camino falla o la red se congestiona, el router estático seguirá intentando enviar los paquetes por la ruta configurada, lo que provocará la pérdida de datos.
    - **Simple y eficiente:** No consume ancho de banda de la red ni capacidad de procesamiento del router en calcular rutas, ya que no se intercambia información con otros routers.
    - **Cuándo se utiliza:** Es sumamente útil en topologías simples donde la elección del camino es obvia o no hay rutas alternativas. Por ejemplo, en una red de extremo (_stub network_) donde solo hay un único router de salida hacia el ISP.

#### Enrutamiento Dinámico:
Se basa en algoritmos **adaptativos**.
- **Cómo funciona:** Los routers ejecutan protocolos de enrutamiento dinámico (como OSPF, RIP, IS-IS o BGP) mediante los cuales **intercambian información de forma automática** con los routers vecinos.
- **Características:**
    - **Flexible y tolerante a fallos:** Si un enlace se cae o se satura de tráfico, los routers se comunican el cambio de inmediato, recalculan los caminos óptimos y actualizan sus tablas de enrutamiento de manera autónoma.
    - **Complejo:** Requiere más procesamiento de CPU, memoria en el router y ancho de banda en la red para intercambiar los mensajes de control.

Ejemplos :
1. Vector de Distancia (ej. RIP)
2. Estado de Enlace (ej. OSPF, IS-IS):
3. IGRP

---
Siguiente: [[IPv6]] 











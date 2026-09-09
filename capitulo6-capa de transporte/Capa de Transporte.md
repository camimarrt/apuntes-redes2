Es la cuarta capa del Modelo OSI. Su función principal es **proporcionar un transporte de datos eficiente, fiable y rentable** desde un proceso en una máquina de origen a un proceso en una máquina de destino, de manera independiente de las redes físicas en uso.
Es la frontera entre los proveedores de servicio (capas 1 a 4) y el usuario (capas superiores). Ofrece las abstracciones necesarias para que los programas utilicen la red sin preocuparse por la complejidad subyacente.
Esta capa es el cimiento sobre el cual se construyen aplicaciones de red robustas, facilitando la comunicación proceso a proceso en sistemas distribuidos.
# 1. Servicios de la Capa de Transporte:

## 1.1. Servicios a Capas Superiores:
El objetivo último de la capa de transporte es proporcionar un servicio de transmisión de datos fiable y eficiente para sus usuarios, que son los procesos de la capa de aplicación. 
### - **La entidad de transporte:** 
La **entidad de transporte** es el componente de hardware o software responsable de realizar el trabajo de la capa de transporte y gestionar la transmisión de datos. Su función clave es actuar como intermediaria: toma los datos de los procesos de la capa de aplicación y utiliza los servicios de la capa de red para garantizar una entrega eficiente, fiable y económica de extremo a extremo.
Puede residir en diversas ubicaciones dentro de un sistema: 
- En el kernel del sistema operativo. 
- Como parte de una biblioteca de aplicaciones de red. 
- En un proceso de usuario separado. 
- Directamente en la tarjeta de interfaz de red
### - Escudo Protector:
- Actúa como un escudo protector, **aislando la tecnología** e imperfecciones de la red subyacente de las aplicaciones. 
- **Mitiga los errores,** si la capa de red es deficiente y pierde paquetes o se interrumpe de forma abrupta, la entidad de transporte lo detecta y lo compensa de manera invisible mediante la retransmisión de segmentos o el restablecimiento de las conexiones. Gracias a esto, los programadores de aplicaciones pueden asumir que están trabajando sobre una "tubería" de comunicación perfecta, ordenada y libre de errores.
- **Control del usuario frente al control del operador:** El código de la capa de transporte se ejecuta completamente en las **máquinas de los usuarios**, mientras que la capa de red se ejecuta en gran medida en los routers, los cuales son administrados por los operadores de red.
### - Direccionamiento y Multiplexación:
Dado que una computadora suele tener asignada una única dirección de red (dirección IP o NSAP) pero ejecuta múltiples aplicaciones a la vez, la capa de transporte ofrece puntos de acceso llamados **puertos o TSAPs** (Transport Service Access Points). Esto permite realizar la **multiplexación**, es decir, que varias conversaciones de diferentes procesos de aplicación compartan de manera simultánea la misma conexión o dirección de red al host remoto.
### - Tipos de Servicios de Transporte:
Al igual que en la capa de red, ofrece principalmente dos modalidades de servicio según las necesidades de la aplicación:
- **Servicio orientado a la conexión (fiable):** Está diseñado para actuar como una **"tubería de bits fiable"** (o flujo de bytes fiable como en TCP). El emisor introduce datos en un extremo y el receptor los obtiene en el otro extremo de forma libre de errores, sin duplicados y en el mismo orden físico en el que fueron transmitidos. Para lograr esta fiabilidad, la entidad de transporte gestiona de forma invisible los acuses de recibo, los temporizadores y las retransmisiones. Este servicio consta de tres fases bien definidas: establecimiento de la conexión, transferencia de datos y liberación.
- **Servicio sin conexión (no fiable / datagramas):** Permite enviar mensajes de manera independiente y directa sin necesidad de configurar previamente una conexión. No garantiza que los paquetes lleguen ni que lo hagan en orden, pero ofrece menor sobrecarga y es ideal para aplicaciones de consulta rápida (como DNS) o de streaming multimedia en tiempo real.
## 1.2. Primitivas de Servicio:
Las **primitivas del servicio de transporte** son llamadas a procedimiento (procedimientos de biblioteca o llamadas al sistema) que permiten a los programas de aplicación (capas superiores) interactuar directamente con la capa de transporte para comunicarse a través de la red. Actúan como una interfaz unificada que aísla las complejidades, imperfecciones y diferencias tecnológicas de la red física subyacente de cara al programador de la aplicación.
#### Objetivos Principales de las Primitivas de Transporte:
A diferencia de las primitivas de la capa de red (que reflejan las imperfecciones de las redes físicas, como la pérdida de paquetes), las primitivas de transporte orientadas a la conexión tienen como objetivos fundamentales:
- **Ocultar los errores de la red**.
- **Proveer un flujo de bits (o bytes) fiable y libre de errores** para las aplicaciones, de modo que estas puedan asumir una conexión de red perfecta.
- **Ser cómodas, convenientes y fáciles de usar** para los programadores de software.
#### Primitivas de un servicio simple:

| Primitiva      | Paquete Enviado    | Significado y Comportamiento                                                                                                         |
| -------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| **LISTEN**     | _(Ninguno)_        | **Bloqueo pasivo.** El servidor se bloquea y queda en espera pasiva hasta que un proceso cliente remoto intente establecer contacto. |
| **CONNECT**    | `CONEXIÓN REQ.`    | **Establecimiento activo.** El cliente solicita activamente una conexión enviando un segmento de solicitud de conexión al servidor.  |
| **SEND**       | `DATOS`            | **Transmisión de datos.** Envía de forma bidireccional la información de la aplicación una vez establecida la conexión.              |
| **RECEIVE**    | _(Ninguno)_        | **Espera de datos.** Bloquea al proceso receptor en espera de que llegue un paquete de datos (`DATA`) desde el otro extremo.         |
| **DISCONNECT** | `DESCONEXIÓN REQ.` | **Liberación de conexión.** Solicita formalmente finalizar la comunicación y liberar los recursos asociados.                         |
**Flujo de Interacción:
1. **Establecimiento de la Conexión:** El servidor ejecuta `LISTEN` para prepararse. El cliente ejecuta `CONNECT`, enviando el paquete `CONEXIÓN REQ.`. Si el servidor está listo, se desbloquea y envía un acuse de recibo de conexión aceptada; al recibirlo, el cliente también se desbloquea y se inicia la sesión.
2. **Transferencia de Datos:** El emisor llama a `SEND` para mandar datos y el receptor llama a `RECEIVE`, quedando bloqueado temporalmente hasta que la información llega de forma íntegra a su búfer.
3. **Liberación de la Conexión:** Se llama a `DISCONNECT` cuando el canal ya no es necesario. Esta desconexión puede ser **asimétrica** (se cierra de golpe y puede provocar pérdida de datos si el otro extremo seguía transmitiendo) o **simétrica** (cada sentido de transmisión se cierra de forma independiente cuando no le quedan datos por mandar, evitando pérdidas).

## 1.3. API Estándar de TCP/IP:
#### Primitivas de un servicio de Tom Berkeley:
Primitivas como las de sockets de Berkeley proporcionan una API estándar y fácil de usar para que los programadores interactúen con la red

| Primitiva   | Rol Principal      | Significado y Comportamiento                                                                                                                                                                                      |
| ----------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SOCKET**  | Servidor / Cliente | **Crea un nuevo punto final (endpoint) de comunicación** y le asigna un espacio de tabla dentro de la entidad de transporte. Devuelve un descriptor de archivo para llamadas sucesivas.                           |
| **BIND**    | Servidor           | **Asocia una dirección local con un socket**. Permite asignar una dirección IP y un número de puerto específicos para que los clientes remotos puedan conectarse a él.                                            |
| **LISTEN**  | Servidor           | **Anuncia la disposición de aceptar conexiones entrantes** e indica el tamaño máximo de la cola para almacenar solicitudes simultáneas de clientes. Es una llamada no bloqueante.                                 |
| **ACCEPT**  | Servidor           | **Establece de forma pasiva una conexión entrante**. Bloquea al servidor en espera de una solicitud. Al recibirla, crea un nuevo socket con las mismas propiedades y devuelve un descriptor de archivo para este. |
| **CONNECT** | Cliente            | **Intenta establecer activamente una conexión**. Bloquea al proceso cliente y envía el segmento de solicitud de conexión (`SYN`) al servidor remoto.                                                              |
| **SEND**    | Servidor / Cliente | **Envía datos a través de la conexión establecida**. Se utiliza para la transmisión de información en modo full-duplex.                                                                                           |
| **RECEIVE** | Servidor / Cliente | **Recibe datos de la conexión**. Permite extraer los bytes de información que han llegado a través de la sesión full-duplex.                                                                                      |
| **CLOSE**   | Servidor / Cliente | **Libera y cierra la conexión**. Finaliza la comunicación y destruye los puntos finales liberando los recursos asociados en el sistema.                                                                           |
## Versatilidad y Futuro
- La API de sockets se utiliza comúnmente con el protocolo TCP para ofrecer un servicio orientado a conexión, conocido como flujo confiable de bytes
- Cualquier aplicación la puede utilizar para otros servicios de transporte. Por ejemplo, se pueden usar sockets con un servicio de transporte sin conexión.
- Los sockets también se pueden usar con protocolos de transporte que proporcionen un flujo continuo de mensajes en vez de un flujo continuo de bytes, y que dispongan o no de un control de congestión. Por ejemplo, **DCCP (Datagram Congestion Controlled Protocol)** es una versión de UDP con control de congestión.
- Se han desarrollado protocolos e interfaces más recientes que soportan grupos de flujos relacionados de una forma más efectiva y simple para la aplicación. Dos ejemplos son **SCTP (Stream Control Transmission Protocol ), y SST (Structured Stream Transport)**. Estos protocolos deben modificar un poco la API para soportar grupos de flujos, múltiples rutas y mezclas de tráfico.

Siguiente: [[Elementos de Protocolos]]


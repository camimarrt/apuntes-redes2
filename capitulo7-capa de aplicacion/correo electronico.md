## Arquitectura y servicios

#### **Subsistemas Principales**

- **Agentes de Usuario (_User Agents_ - UA):** Programas con los que interactúa directamente el usuario (mediante interfaz gráfica o de comandos) para redactar, leer, responder y organizar mensajes. También pueden ejecutar tareas automáticas como filtrar _spam_ o enviar respuestas automáticas.
- **Agentes de Transferencia de Mensajes (_Message Transfer Agents_ - MTA / Servidores de correo):** Procesos de sistema en segundo plano que están siempre activos en los servidores. Su función es mover automáticamente los mensajes a través de la red desde el origen hasta el destino utilizando el protocolo **SMTP**.
- **Buzones de correo (_Mailboxes_):** Almacenados y mantenidos por los servidores de correo, guardan los mensajes recibidos para que los _User Agents_ puedan consultarlos, gestionarlos o descargarlos (paso de entrega final).
#### ** Estructura del Mensaje (Sobre vs. Contenido)**

Para la transmisión de correo, se aplica una distinción fundamental entre el sobre y el contenido:
- **El sobre (_Envelope_):** Encapsula el mensaje y contiene la información que los MTA necesitan para el transporte y enrutamiento (como dirección de destino, prioridad o nivel de seguridad).
- **El mensaje:** Se divide en dos componentes:
    - **Cabecera (_Header_):** Contiene información de control utilizada por los _User Agents_ (de quién procede, hacia quién va, asunto, etc.).
    - **Cuerpo (_Body_):** El texto o contenido destinado exclusivamente al receptor humano, transmitido de forma transparente por los MTA.

## El agente de usuario


El **agente de usuario** (o lector de correo) es el programa ejecutable que proporciona la interfaz directa para que las personas puedan redactar, recibir, leer, responder y organizar sus mensajes de correo electrónico.

- **Función principal:** Acepta comandos del usuario para interactuar con el sistema de correo y manipular los buzones almacenados en los servidores (crear, enviar, borrar o archivar mensajes).
#### **Interfaz y Visualización del Buzón**

- **Resumen y ordenación:** Al iniciarse, presenta un resumen clasificado de los mensajes del buzón, generalmente ordenados por fecha y hora de recepción (los más recientes primero).
- **Campos clave:** Extrae y da formato legible a los datos de la cabecera y sobre, destacando quién envía el mensaje (_De_), el tema (_Asunto_) y el momento en que llegó (_Recibido/Fecha_).
- **Indicadores visuales y vistas previas:** Utiliza iconos para identificar el estado del mensaje (un sobre para correo no leído, un clip para archivos adjuntos y un signo de exclamación para prioridad alta) y ofrece vistas previas breves del cuerpo.
#### **Disposición del Mensaje y Organización**

- **Acciones del usuario (_Message Disposition_):** Una vez leído un correo, el usuario decide su destino final: eliminarlo, responderlo, reenviarlo a otro destinatario o guardarlo para consulta posterior.
- **Carpetas y reglas personalizadas:** Facilita la organización mediante carpetas (como _Bandeja de entrada_ y _Correo basura_) y permite definir reglas automáticas compuestas por una **condición y una acción** (por ejemplo, mover inmediatamente los correos del jefe a una carpeta de lectura prioritaria).
#### **Filtrado Inteligente y Funciones Automáticas**

- **Detección de _Spam_:** Analiza los campos y el contenido del correo entrante para identificar _spam_ (procedente habitualmente de _botnets_), apoyándose en listas de _spammers_ conocidos gestionadas por la empresa o ISP.
- **Procesamiento desatendido:** Incluye capacidades avanzadas automáticas, como la generación de respuestas automáticas de "fuera de la oficina" durante periodos vacacionales.

## Formatos de mensajes

#### **RFC 5322: El Formato de Mensajes de Internet ASCII**

- **Estructura general:** Un mensaje se compone de líneas de **cabecera ASCII**, una **línea en blanco** y el **cuerpo del mensaje**.
- **Cabeceras de transporte principales:**
    - **`Para:` (`To:`) / `Cc:` / `CCO:` (`Bcc:`):** Indican destinatarios principales, secundarios y ocultos. La línea `CCO:` (`Bcc:`) se elimina automáticamente de todas las copias enviadas para preservar la privacidad del destinatario oculto.
    - **`De:` (`From:`) vs. `Remitente:` (`Sender:`):** `De:` identifica al autor del mensaje, mientras que `Remitente:` especifica la dirección de la persona o proceso que realmente lo transmitió (por ejemplo, un asistente).
    - **`Recibido:` (`Received:`):** Cada agente de transferencia (MTA) que procesa el mensaje añade una línea con su identidad y marca de tiempo, dejando un registro para depurar el enrutamiento.
- **Cabeceras de gestión e hilos:**
    - **`Message-Id:`:** Identificador único generado para referenciar el mensaje en respuestas (`In-Reply-To:`) y evitar entregas duplicadas.
    - **`Respuesta a:` (`Reply-To:`):** Especifica a qué dirección deben dirigirse las respuestas si difiere de la del remitente.
- **Cabeceras personalizadas (`X-`):** Permite incluir campos privados o experimentales creados por el usuario, obligando a usar el prefijo `X-` para evitar colisiones con futuras cabeceras oficiales.

---

#### **MIME: Extensiones de Correo de Internet Multiuso**

- **Propósito:** Dado que RFC 5322 estaba limitado a texto plano en ASCII, MIME (_Multipurpose Internet Mail Extensions_) añade estructura al cuerpo y reglas de codificación para transmitir imágenes, audio, video, archivos binarios y textos con alfabetos no latinos sin romper la infraestructura SMTP existente.
- **Las 5 Nuevas Cabeceras de MIME:**
    1. **`MIME-Version:`** Indica la versión del estándar (por ejemplo, `1.0`).
    2. **`Content-Description:`** Cadena ASCII descriptiva que informa sobre el contenido del mensaje antes de descargarlo.
    3. **`Content-Id:`** Identificador único para el contenido encapsulado.
    4. **`Content-Transfer-Encoding:`** Especifica cómo se empañeta el cuerpo para la transmisión. El esquema **`base64`** convierte bloques binarios de 8 bits en caracteres ASCII de 7 bits para garantizar su paso seguro por servidores de correo tradicionales sin exceder los límites de longitud por línea.
    5. **`Tipo de contenido` (`Content-Type:`):** Define la naturaleza exacta del archivo mediante una pareja _tipo/subtipo_ (por ejemplo, `text/html`, `image/jpeg`, `application/pdf`). Incluye el tipo **`multipart`**, que permite combinar múltiples archivos o elementos adjuntos de diferente formato en un único mensaje.

## Transferencia de mensajes

#### **Usos de SMTP y Conexión de Transporte**
- **Dos funciones principales:** SMTP se utiliza tanto para el **envío de correo** (desde el agente de usuario hacia el servidor de correo local) como para la **transferencia de mensajes** entre servidores de correo (MTA a MTA en un solo salto).
- **Puerto y transporte:** El emisor establece una conexión TCP con el **puerto 25** del servidor receptor. Es un protocolo basado en texto **ASCII**, lo que facilita su desarrollo, prueba y depuración (pudiendo probarse manualmente mediante `telnet`).

#### **Diálogo Básico SMTP (RFC 5321)**
El protocolo sigue un modelo cliente/servidor donde el servidor toma la iniciativa respondiendo primero:
- **`HELO`:** Comando de 4 caracteres con el que el cliente se identifica e inicia el diálogo.
- **`MAIL FROM:`** Especifica la dirección de correo del remitente.
- **`RCPT TO:`** Indica la dirección del destinatario; permite enviar un mensaje a múltiples receptores y cada uno se acepta o rechaza de forma individual.
- **`DATA`:** Transmite el contenido del mensaje (cabeceras y cuerpo), finalizando con una línea que contiene un único punto `"."`.
- **Códigos de respuesta:** El servidor responde con códigos numéricos de 3 dígitos (como `220`, `250`, `354`), acompañados de un texto descriptivo flexible.

#### **Enrutamiento Físico mediante Registros MX**
- Para saber a qué servidor entregar un mensaje destinado a un dominio remoto (por ejemplo, `bob@ee.uwa.edu.au`), el MTA remitente realiza una consulta DNS solicitando los registros **MX** (_Mail Exchange_) del dominio.
- La consulta devuelve una lista ordenada de servidores de correo autorizados para recibir mensajes en ese dominio.
#### **Limitaciones de SMTP y Extensiones ESMTP**
El SMTP clásico presenta tres limitaciones severas: carece de autenticación (facilidad para enviar _spam_ suplantando el `MAIL FROM`), solo transmite texto ASCII de 7 bits y envía los datos en texto claro sin cifrado. Para resolverlo, se creó **ESMTP** (_Extended SMTP_):
- **`EHLO`:** El cliente inicia el saludo con `EHLO` en lugar de `HELO`; si el servidor lo acepta, devuelve la lista de extensiones soportadas.
- **Extensiones clave:**
    - **`AUTH`:** Exige autenticación de usuario y contraseña para el envío de correo (frecuentemente en el **puerto 587**), evitando _relays_ abiertos que aprovechan los _spammers_.
    - **`STARTTLS`:** Conmuta la transmisión a un canal seguro y cifrado con TLS.
    - **`SIZE` (`TALLA`):** Comprueba el tamaño del mensaje antes del envío.
    - **`BINARYMIME` / `CHUNKING`:** Permite la transferencia de datos binarios directos o en trozos sin depender de la ineficiente codificación `base64`.
    - **`UTF8SMTP`:** Admite direcciones de correo internacionalizadas.

## Entrega final

#### **El problema de la entrega final y por qué no se usa SMTP**
- **Incompatibilidad de SMTP:** SMTP es un protocolo orientado a empujar tráfico (_push_). No sirve para la entrega final al usuario porque los dispositivos del destinatario (laptops, teléfonos) no están conectados de forma continua a la red con una dirección IP fija ni ejecutando servidores a la escucha.
- **Requisito del buzón:** El buzón de correo debe permanecer almacenado centralmente en el servidor de correo del ISP o la empresa, requiriendo protocolos de tracción (_pull_) para acceder a él o manipularlo a distancia.
#### **IMAP (_Internet Message Access Protocol_ - RFC 3501)**
- **Acceso remoto sincronizado:** Funciona sobre el puerto 143 (o mediante conexiones cifradas con TLS).
- **Manipulación en el servidor:** A diferencia de otros protocolos, mantiene los mensajes y la estructura de carpetas directamente en el servidor de correo.
- **Multidispositivo y granular:** Permite al cliente consultar resúmenes, buscar mensajes por atributos, descargar solo partes específicas de un correo y sincronizar el estado del buzón entre múltiples dispositivos simultáneamente.
#### **POP3 (_Post Office Protocol_, versión 3 - RFC 1939)**
- **Descarga local:** Es un protocolo más antiguo y sencillo que descarga habitualmente los correos al almacenamiento local del dispositivo del usuario (borrándolos del servidor).
- **Inconvenientes:** Dificulta el acceso al mismo buzón desde varios ordenadores y genera riesgo de pérdida permanente de información si el equipo del usuario sufre una avería.
#### **Correo Web (_Webmail_)**
- **El navegador como Agente de Usuario:** Servicios como _Gmail, [suspicious link removed]_ o _Yahoo! Mail_ integran la interfaz del lector dentro del propio navegador Web utilizando HTTP/HTTPS.
- **Procesamiento híbrido:** Utiliza programas en JavaScript del lado del cliente para ofrecer una interfaz fluida y realizar cargas y descargas en segundo plano, mientras el servidor genera dinámicamente el contenido del buzón y retransmite los correos salientes al sistema SMTP tradicional.

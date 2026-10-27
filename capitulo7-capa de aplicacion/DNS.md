# 7.1 DNS: Domain Name System

DNS es una base de datos que mapea nombres de host a direcciones IP. Es un esquema jerárquico de nombres. Fue creado en 1983.
- Guardar direcciones IP ilegibles para el humano es incomodo.
- Mucho sistemas son distribuidos, DNS decide a qué IP enviar.
- De otro modo, habría conflictos de nombre de hosts constantemente.
## Proceso de búsqueda.

DNS funciona de la siguiente manera:
1. Un programa pasa el nombre de dominio a una biblioteca *gethostbyname*, que actua como `stub resolver`.
2. El stub envía una consulta **recursiva** al `resolver local`. En este modo recursivo, el cliente delega la tarea por completo y solo espera la respuesta final. (NO se encuentra en la máquina normalmente, solo es el más cercano al cliente).
3. El resolver local consulta jerarquicamente a los `servidores autoritativos` haciendo consultas iterativas. Cada servidor responder según su nivel de jerarquía y da una respuesta parcial.
4. El `local resolver` recibe y reenvía la respuesta con la dirección IP al `stub resolver`. 
**Servidor DNS Autoritario:** Es el que tiene una respuesta definitiva para un dominio particular. 

![[Pasted image 20261025171707.png]]

Las respuestas parciales se almacenan en caché. Las entradas del caché tienen un tiempo de vida para asegurar que las respuestas parciales estén actualizadas.

Anteriormente DNS realizaba consultas sobre UDP por su rapidez y simplicidad. Sin embargo, debido a que DNS es vunerable a ataques DoS y de envenenamiento de caché, se ha empezado a utilizar TCP para aprovechar su seguridad de cifrado.  Ej.: DNS over TLS / DNS over HTTPS.

**Minimización de QNAME:**
El resolvedor local solo envía una consulta parcial para cada nivel de jerarquía. Evita revelar el nombre de dominio completo. 

#### **Consultas DNS:**
**Formato de las Consultas DNS:**
- **QNAME:** El nombre consultado.
- **ID**: Asigna consultas a respuestas. Son de 16bits.
- **Tipo de consulta:** Habitualmente es una consulta de registro A, que solicita una asignación de un nombre de dominio a una dirección IP.
##### **DNSBL**
Son *listas negras* que se mantienen para verificar spammers o malware. Un cliente puede enviar una consulta DNS A-record a un servidor DNS especial, como pbl.spamhaus.org. Si la dirección IP no figura en la lista, el servidor DNS lo indicaría respondiendo con la correspondiente respuesta `NXDOMAIN`, correspondiente a ''no existe tal dominio''.

#### **Mejoras de DNS:**

##### **EDNS Client Subnet:**
Es una extensión donde el resolutor local transmite la subred IP del cliente al servidor autoritativo. Esto permite al servidor autoritativo determinar la ubicación geográfica o topológica del cliente y responder con la dirección IP de un servidor cercano (como un nodo CDN), resolviendo la falta de precisión cuando se usan resolutores públicos como 8.8.8.8 o 1.1.1.1.
##### **Codificación 0x20:**
Aprovechando que los nombres de dominio no distinguen mayúsculas de minúsculas, los resolutores alternan arbitrariamente entre mayúsculas y minúsculas en el nombre consultado (_QNAME_, ej. `uCHicaGO.EDu`). Esta técnica agrega bits de aleatoriedad al identificador de transacción original de 16 bits, mitigando ataques de envenenamiento de caché (como el ataque Kaminsky) sin romper la compatibilidad con servidores existentes.

#### **Respuestas DNS:**
Cuando un resolver envía un nombre de dominio al DNS, lo que obtiene de vuelta son los registros de recursos asociados a ese nombre. Que constituyen las entradas de la base de datos distribuida de DNS. Se estructuran formalmente como una **5-tupla**:

| `Nombre` | `TTL` | `Clase` | `Tipo` | `Valor` |
| -------- | ----- | ------- | ------ | ------- |

1. **Nombre de dominio:** Clave principal de búsqueda en la base de datos.
2. **Tiempo a vivo (TTL):** Indica en segundos cuánto tiempo se puede guardar el registro en caché antes de expirar.
3. **Clase:** Para el tráfico estándar de Internet, su valor siempre es `IN.
4. **Tipo:** Especifica la naturaleza del registro
5. **Valor:** El dato o contenido concreto (un entero de 32/128 bits, un nombre de dominio o texto ASCII), cuya interpretación depende del tipo.
##### **Principales Tipos de Registros:**
- **A/AAAA:** Asocian un nombre con una dirección IPv4 de 32 bits[16] o IPv6 de 128 bits (_quad A_)
- **SOA:** Proporciona parámetros sobre la zona (administrador, versión/número de serie y tiempos de expiración).
- **NS:** Identifica el nombre del servidor de nombres autorizado para un dominio o subdominio
- **MX:** Especifica el servidor de correo designado para el dominio y su nivel de prioridad.
- **CNAME:** Asigna un alias hacia el nombre de dominio canónico principal.
- **PTR:** Puntero utilizado en búsquedas inversas para asociar una dirección IP con su nombre de dominio.
- **SRV:** Identifica el host que presta un servicio específico en un dominio (generaliza el concepto de MX para otros servicios).
- **SPF/TXT:** El registro SPF especifica qué servidores tienen permitido enviar correo en nombre del dominio (para prevenir _spam_ y suplantación); se suele codificar en registros de tipo TXT.
#### **Registros DNSSEC:**
Los registros DNSSEC permiten que las respuestas de los servidores de nombres DNS lleven firmas digitales, que el resolver local o stub puede verificar posteriormente para asegurarse de que los registros DNS no han sido modificados o manipulados. Cada DNS calcula un hash. 
RRSET (Resource Record Set): conjunto de registros de recursos del mismo tipo, con sus claves criptográficas privadas.

- El `RRSIG` corresponde a una firma sobre el RRSET (resource record set), firmado con la clave privada del servidor de nombres autoritativo correspondiente. 
- El registro `DNSKEY`, que es la clave pública para el RRSET correspondiente, que está firmado por la clave privada del padre.
#### **Zonas DNS:**
DNS se divide en zonas delegadas dentro de la jerarquía de dominios que no se superponen. Esto permite que distintas organizaciones o suborganizaciones gestionen sus propios registros, manteniendo el sistema distribuido, escalable y tolerante a fallos.
Una **zona DNS** (o archivo de zona) es una porción delimitada del árbol del espacio de nombres de Internet sobre la cual una entidad tiene control administrativo directo.
- Evita sobrecargas al descentralizar los nombres de servidores.
- Evita que los nombres de dominio no se solapen entre sí.

Un dominio superior puede administrar varios subdominios directamente dentro de su misma zona o delegar subdominios específicos a una **zona separada** con sus propios servidores (por ejemplo, `chicago.edu` puede administrar `cs.uchicago.edu` dentro de su zona o separar `eng.uchicago.edu` en una zona independiente si ese departamento desea gestionar su propio servidor).

##### **3. Servidores Primarios y Secundarios**
Cada zona está atendida por uno o varios servidores de nombres autoritativos que almacenan la base de datos de esa parte del árbol:
- **Servidor de nombres primario:** Carga la base de datos de la zona directamente desde un archivo almacenado en su disco local.
- **Servidores de nombres secundarios:** Obtienen y actualizan su información copiándola desde el servidor primario.
- **Redundancia:** Para mejorar la fiabilidad, es habitual ubicar servidores secundarios fuera de la zona física o de red.
##### **4. El Registro SOA (_Start of Authority_)**
- La base de datos de cada zona inicia formalmente con un registro de tipo **SOA**.
- Este registro define los parámetros fundamentales de la zona, incluyendo la fuente principal de información autoritativa, el correo electrónico del administrador, el número de serie de la base de datos para la sincronización y varios temporizadores de expiración.
## Espaciado de Nombres y Jerarquía:

#### **Árbol de Dominios:**
Cada dominio se divide en subdominios, que a su vez también se dividen en subdominios. Esto constituye una jerarquía de dominios que puede ser representada por un árbol invertido.
- Las hojas son dominios de nivel superior, no tienen subdominios.
- Cada dominio se denomina por la ruta ascendente desde él hasta la raíz (sin nombre).
- Un dominio con nombre se refiere a un nodo específico del árbol y a todos los nodos que se encuentran bajo él.
- Los dominios pueden insertarse tanto en el dominio genérico como en el de país.
- El “dominio raíz” se representa por un punto (.).
- No está prohibido figurar en varios TLD (ej. sony.com y sony.net)
- Los nombres de dominio pueden ser absolutos o relativos. Un nombre de dominio absoluto siempre termina con un punto (por ejemplo, eng.cisco.com.), mientras que uno relativo no.
- Los nombres de componentes pueden hasta 63 caracteres, y los nombres de rutas completas no deben superar los 255 caracteres.
- DNS no distingue mayúsculas y minúsculas.
- Cada dominio controla cómo asigna los dominios que están bajo él.
- Para crear un nuevo dominio, se requiere el permiso del dominio en el que se incluirá.
- Una vez que se ha creado y registrado un nuevo dominio, éste puede crear subdominios.
- Los nombres siguen los límites organizativos, no las redes físicas.
#### **Tipos de TLD:**
Internet se divide en más de 250 “dominios de nivel superior” o TLD (top level domain).
Los dominios de primer nivel pueden ser del tipo:
- **gTLD (Generic Top Level Domain)**: Dominios genéricos originales como edu, com, gov, org, etc. 
- **ccTLD (Country Code Top Level Domain):** Dominios de dos letras por país: py, ar, us, br, etc.
#### **Niveles de Autoridad:**
Usuario → Registrar → Registry → ICANN
- •ICANN: Es la organización internacional que supervisa el sistema de nombres de dominio y delega quién puede administrar cada TLD
- **Registro:** Es la entidad que administra un TLD.
Por ejemplo, Verisign administra el TLD `.com` , mientras que NIC Paraguay administra el ccTLD `.py`.
- **Registrador acreditado:** Son empresas autorizadas por ICANN (o por el registry de un país) para vender dominios al público.
## Resolución de Nombre

Cada zona está asociada a uno o varios servidores de nombres. El proceso de buscar un nombre y encontrar una dirección se denomina *resolución de nombres*. 
Cuando un resolver tiene una consulta sobre un nombre de dominio,
- pasa la consulta a un servidor de nombres local. 
- Si el dominio buscado está en el servidor de nombres, devuelve los registros recursos autoritativos.
- Si no hay información almacenada en caché sobre el dominio disponible localmente, el servidor de nombres inicia una consulta remota.

##### **Ejemplo:**
 `flits.cs.vu.nl` solicita la dirección IP de un dominio ubicado en otra organización (`cs.uchicago.edu`) sin disponer de información guardada en la caché local.
La secuencia de pasos especificada en la búsqueda es la siguiente:

1. **Paso 1:** El host cliente (_stub resolver_) envía la consulta inicial que contiene el nombre de dominio buscado (`cs.uchicago.edu`), el tipo (`A`) y la clase (`IN`) a su **servidor de nombres local** (`cs.vu.nl`).
2. **Paso 2:** El servidor de nombres local envía la consulta al **servidor de nombres raíz** (`a.root-servers.net`).
3. **Paso 3:** El servidor raíz responde devolviendo la referencia (nombre e IP) del **servidor de nombres del TLD correspondiente** (`.edu`).
4. **Paso 4:** El servidor de nombres local envía la consulta completa al **servidor de nombres de `.edu`** (`a.edu-servers.net`).
5. **Paso 5:** El servidor de nombres `.edu` responde devolviendo la referencia del **servidor de nombres de `uchicago.edu`**.
6. **Paso 6:** El servidor de nombres local envía la consulta al **servidor de nombres de `uchicago.edu`**.
7. **Paso 7:** El servidor de `uchicago.edu` responde con la referencia del **servidor de nombres del departamento de Informática** (`cs.uchicago.edu`).
8. **Paso 8:** El servidor de nombres local envía la consulta al **servidor de nombres autoritativo de `cs.uchicago.edu`**.
9. **Paso 9:** El servidor autoritativo de `cs.uchicago.edu` devuelve la **respuesta final con la dirección IP solicitada** (`128.135.24.19`).
## Privacidad

Históricamente, las consultas y respuestas DNS no se han cifrado. Como resultado, cualquier otro dispositivo podría observar el tráfico DNS de un usuario y determinar información sobre ese usuario.

**Mejoras Propuestas: DNS oblivious y DoH oblivious:**
**ODoH** (_DNS olvidadizo_/ DNS over HTTPS) separa la información. El stub resolver cifra la consulta original al resolver recursivo local, que a su vez envía la consulta cifrada a un servicio de nombres autoritativo que puede descifrar y resolver la consulta, pero no conoce la identidad o la dirección IP del stub resolver que inició la consulta

--- 
- **FQDN:** Full QNAME Domain Name.
# 1
## 2
### 3
#### 4
##### 5
###### 6

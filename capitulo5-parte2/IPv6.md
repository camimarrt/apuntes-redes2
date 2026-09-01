El protocolo IPv6 fue crear debido al agotamiento de las direcciones IPv4. Los sistemas operativos modernos lo soportan plenamente.
Utiliza direcciones de **128 bits**, que equivale a 2^128 direcciones totales. 

#### Ventajas de IPv6:
- **Espacio de direcciones masivo:** IPv6 utiliza direcciones de **128 bits (16 bytes)**. Esto equivale a $2^{128}$ direcciones totales.
- **Encabezado simple:** La cabecera base de IPv6 tiene un tamaño fijo de **40 bytes** y se redujo a solo **7 campos obligatorios**. Se reduce la sobrecarga de procesamiento en los routers, permitiendo que conmuten y reenvíen paquetes de manera mucho más rápida6more_horiz.
- Eliminación de la suma de comprobación (Layer 3 Checksum).
- **Gestión de fragmentación descentralizada:** En IPv6, los routers intermedios tienen estrictamente prohibido fragmentar paquetes. El host de origen es el único responsable de determinar dinámicamente el tamaño adecuado de sus paquetes utilizando el mecanismo de **descubrimiento de la MTU de la ruta (Path MTU Discovery)**. Si un router recibe un paquete que supera el MTU de su enlace, simplemente lo descarta y devuelve un mensaje de error ICMPv6 de vuelta al host de origen para que este lo fragmente en el emisor antes de retransmitir. Además, la MTU mínima obligatoria para los enlaces IPv6 se incrementó a **1280 bytes**.
- **Calidad de Servicio (QoS) mejorada:** Introduce el campo **Flow label (Etiqueta de flujo)**. Este identificador de 20 bits permite marcar paquetes de un mismo flujo origen-destino para que reciban un tratamiento especial rápido (como reserva de ancho de banda o garantías de retardo) en las tablas de búsqueda de los routers sin que estos tengan que realizar una inspección profunda de datos6more_horiz.
- **Seguridad nativa integrada:** El diseño original de IPv6 incluyó de forma obligatoria mecanismos criptográficos integrados de autenticación y privacidad (encriptación de extremo a extremo)2021, aunque estas especificaciones de IPsec se adaptaron más tarde también a IPv42021.

# 1. Encabezado:

![[Pasted image 20260829233025.png]]

1. **Versión (4 bits):** Siempre contiene el valor binario `0110` (representando el número 6).
2. **Servicios Diferenciados (8 bits):** Clasifica la prioridad del tráfico de la misma forma que en IPv4.
3. **Etiqueta de flujo (Flow label - 20 bits):** Utilizado para identificar flujos de paquetes que requieren tratamiento de QoS en tiempo real.
4. **Longitud de carga útil (Payload length - 16 bits):** Indica el tamaño en bytes de los datos que siguen a la cabecera fija de 40 bytes (a diferencia de IPv4, no incluye los 40 bytes de la cabecera en el conteo).
5. **Siguiente encabezado (Next header - 8 bits):** Reemplaza al campo "Protocolo" de IPv4. Especifica el tipo de cabecera de extensión que sigue inmediatamente o, si es la última, el protocolo de la capa de transporte encapsulado (como TCP o UDP).
6. **Límite de saltos (Hop limit - 8 bits):** Reemplaza al campo "Time to Live" de IPv4. Es un contador que se decrementa en 1 en cada router; si llega a cero, el paquete se descarta para evitar bucles infinitos.
7. **Dirección de origen (16 bytes / 128 bits):** Dirección IP del host emisor.
8. **Dirección de destino (16 bytes / 128 bits):** Dirección IP del host receptor final.

##### **Soporte eficiente de opciones (Cabeceras de extensión):** 
En lugar de incorporar campos opcionales dentro de la cabecera fija, se introduce el concepto de **cabeceras de extensión opcionales**. Si un paquete requiere funciones especiales, estas cabeceras se colocan justo después de la cabecera fija en un orden predefinido.

![[Pasted image 20260829234337.png]]

1. _Opciones salto a salto (hop-by-hop):_ Información que deben examinar todos los routers de la ruta (como el soporte de _jumboframes_ de más de 64 KB).
2. _Opciones de destino:_ Información adicional exclusiva para el host receptor final.
3. _Enrutamiento:_ Lista suelta de routers específicos a visitar en el camino.
4. _Fragmentación:_ Datos para gestionar la fragmentación del datagrama por el host de origen.
5. _Autenticación:_ Verificación criptográfica de la identidad del remitente.
6. _Carga útil de seguridad cifrada (ESP):_ Información sobre el contenido encriptado del paquete.

# 2. Direccionamiento:

Las direcciones de IPv6 son de 16 bytes, se utiliza la notación decimal estructurada en ocho grupos de cuatro dígitos hexadecimales separados por dos puntos.
$$\text{Ejemplo: } \mathbf{8000:0000:0000:0000:0123:4567:89AB:CDEF}$$ **Reglas de Optimización:**

- Se omiten los ceros a la izquierda.
$$\text{Ejemplo: } \mathbf{8000:0000:0000:0000:123:4567:89AB:CDEF}$$
- Compresión de ceros consecutivos:

$$\text{Ejemplo: } \mathbf{8000::0123:4567:89AB:CDEF}$$
- Compatibilidad IPv4. Se pueden escribir las direcciones IPv4 colocando un doble dos puntos seguido de la direccion decimal con puntos habitual.

$$\text{Ejemplo:    } \mathbf{::192.31.20.46}$$---
Siguiente: [[Protocolos]]

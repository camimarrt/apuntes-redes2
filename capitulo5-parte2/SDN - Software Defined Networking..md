Históricamente, el software que ejecutaba los algoritmos de enrutamiento e implementaba la lógica de reenvío de paquetes estaba integrado con el hardware físico del fabricante. Si un operador adquiría estos equipos, no podía modificar su comportamiento interno ni adaptar a su gusto protocolos básicos .

Las **Redes Definidas por Software** representan una arquitectura moderna que revoluciona la gestión de las redes de computadoras. Su principio fundamental consiste en **separar por completo el plano de control del plano de datos** de los dispositivos de red (como routers y conmutadores).

![[Pasted image 20260829145520.png|559]]
### 1. **El Plano de Control:**
Es el software que se encarga de seleccionar las rutas, decidir qué hacer con el tráfico e actualizar de forma remota las tablas de reenvío en los dispositivos de red.  
La lógica del enrutamiento está separada de los routers, dicho software de nominado *controlador* está centralizado en un servidor remoto. 

Para que el controlador pueda darle órdenes a los dispositivos de hardware (plano de datos), necesita hablar un idioma que estos entiendan. Existen varios protocolos y estándares para este canal de comunicación:

- **OpenFlow (El pionero):** Es un protocolo estándar que permite al controlador modificar de forma remota sus tablas de enrutamiento mediante **tablas de coincidencia-acción**. 
- **RCP (Routing Control Platform):** Fue una de las primeras encarnaciones. Utilizaba el protocolo BGP clásico para que una plataforma externa calculara directamente las rutas de tránsito en las redes troncales, ayudando a equilibrar el tráfico y defenderse de ataques DDoS.

Al no estar limitados por el procesador de un router individual, el plano de control en SDN puede ejecutar algoritmos de enrutamiento y optimización de tráfico sumamente complejos y exigentes computacionalmente. Como los switches físicos ya no necesitan procesar protocolos de enrutamiento complejos (solo tienen que obedecer al controlador), el hardware se vuelve mucho más simple y económico, permitiendo el uso de equipos genéricos.
### 2. **El Plano de Datos (El "Ejecutor"):** 
Es el hardware que reside en los conmutadores y routers físicos de la red. Su única responsabilidad es procesar rápidamente los paquetes entrantes, realizar búsquedas en sus tablas y ejecutar la acción física dictada por el controlador (como reenviar el paquete por un puerto específico, duplicarlo o descartarlo). 
Los enfoques modernos permiten programar el hardware físico de manera independiente de los protocolos estándar:
##### Arquitectura RMT: 
Permite que el procesamiento dentro del chip del conmutador se divide en tres etapas programables principales: 
1. **Analizador Programable:** El hardware recibe el paquete entrante y extrae los campos de las cabeceras. Permite al programador definir formatos de cabecera de paquetes completamente personalizados.
2. **Etapas de Coincidencia y Acción:** Consiste en múltiples etapas dispuestas en un conducto (_pipeline_) tanto de entrada (_ingress_) como de salida (_egress_). En cada etapa se pueden realizar operaciones aritméticas simples (como sumas), modificar valores y actualizar metadatos del paquete (por ejemplo, registrar la latencia o la congestión de la cola).
3. **Desanalizador Programable (Deparser):** Escribe de manera ordenada todos los campos y valores modificados de regreso en el paquete antes de que este salga físicamente del dispositivo.

### 3. Telemetría de red programable (Hardware Programable)
SDN no solo centraliza el control, sino que también ha hecho programable el propio hardware de reenvío.

- Los conmutadores modernos pueden procesar paquetes de forma personalizada e independiente de los protocolos estándar gracias a la arquitectura de canalización **RMT**.
- Esto habilita aplicaciones avanzadas como la **telemetría de red programable** en banda (INT - _In-band Network Telemetry_), donde los propios paquetes transportan de manera dinámica metadatos de la red, como la latencia acumulada o la profundidad de las colas en cada router del trayecto.
### 4. Beneficios clave de implementar SDN:
La adopción de SDN aporta múltiples ventajas estratégicas y operativas sobre las arquitecturas tradicionales:

- **Innovación a la velocidad del software:** Elimina la lentitud de innovación propia del hardware cerrado, permitiendo a los operadores probar, desarrollar y lanzar nuevos servicios o protocolos de red de forma sumamente ágil.
- **Optimización de costos de hardware:** Permite simplificar drásticamente el equipamiento físico de red y abre la puerta al uso de conmutadores básicos más económicos y desagregados (_white-box_).
- **Capacidad de cálculo superior:** Posibilita que el plano de control lógicamente centralizado ejecute algoritmos de enrutamiento y balanceo de tráfico mucho más complejos y exigentes computacionalmente de lo que sería viable en routers individuales.

### 5. Aplicaciones en el mundo real:

- **Centros de datos (Datacenters):** Las infraestructuras de nube de proveedores gigantescos requieren un alto grado de personalización, automatización y reconfiguración ágil, convirtiéndose en el principal ecosistema de despliegue para SDN.
- **SD-WAN (Redes de Área Extensa Definidas por Software):** Permite conectar oficinas dispersas reprogramando el tráfico en tiempo real sobre una combinación de enlaces costosos dedicados y conexiones a Internet de bajo costo, optimizando rendimiento y tarifas.
- **Segmentación en redes móviles (Network Slicing en 5G):** Junto con la virtualización de funciones de red (NFV), SDN ayuda a los operadores móviles a crear múltiples redes virtuales sobre la misma infraestructura física física, dedicando recursos específicos a clientes que requieren características particulares de rendimiento o baja latencia.

---
Extras:
##### El modelo inicial de coincidencia-acción (_Match-Action_)
La primera tecnología estándar para comunicar el plano de control con el plano de datos fue el protocolo **OpenFlow**. Este protocolo introdujo un mecanismo de procesamiento muy directo:

- **Tablas de Coincidencia-Acción (_Match-Action Tables_):** El hardware del conmutador almacena una tabla en una memoria especializada (TCAM).
- **Coincidencia (_Match_):** Cuando llega un paquete, el conmutador compara campos de su cabecera (como la dirección IP de origen/destino o la dirección MAC) con las reglas de la tabla.
- **Acción (_Action_):** Si hay una coincidencia, el hardware ejecuta de forma inmediata la acción asociada (por ejemplo: _"enviar por la interfaz física 3"_, _"descartar"_ o _"enviar al controlador externo"_).

---
Siguiente: [[IPv4]] 
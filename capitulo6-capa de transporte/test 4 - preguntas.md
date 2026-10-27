# **6.5 **
1. como se pueden representar las conexiones tcp? 
	Se pueden representar con un diagrama llamado la Maquina de Estados Finitos que muestra los 11 estados que pueden tomar un cliente o servidor durante el ciclo de vida de una conexión. 
2. que son considerados eventos en tcp? 
	Son las llamadas al sistema como CLOSE, CONNECT, LISTEN, SEND o llegada de segmentos como ACK, SYN, FIN, RST.
3. que segmentos de control se tienen en tcp? 
	ACK: acknoledgement,
	SYN: sycronize
	FIN: finalize
	RST: reset
4. describe el flujo y los 11 estados de tcp 

	Para el cliente:
	1. CLOSED: No existe una conexión antes de enviar SYN
	2. SEND SYN: Envía el segmento solicitando una conexión
	3. ESTABLISHED: Luego de haber finalizado el paso de mensajes del acuerdo de tres vías, la conexión está establecida.
	4. FIN WAIT 1: / SEND FIN (cierre activo): Cuando la aplicación finaliza el paso de datos, solicita cerrar la conexión con el segmento FIN. Este estado cierra un lado de la conexión.
	5. CLOSING: Ocurre el cierre simultaneo de las máquinas cliente y servidor.
	6. FIN WAIT 2: Es igual que FIN WAIT 1 pero para el otro lado de la conexión.
	7. TIME WAIT: Espera un tiempo igual al doble de tiempo de vida de un paquete para evitar que queden paquetes retrasados o duplicados.
	8. CLOSED: Se termina la conexión y se borra del registro.
	Para el servidor:
	9. CLOSED: No existe una conexión sin un servidor activo.
	10. LISTEN: El servidor ejecuta la llamada LISTEN.
	11. SYN RCVD: El servidor recibe la solicitud de conexión de un cliente.
	12. ESTABLISHED: Se establece la conexión luego con el acuerdo de tres vías.
	13. CLOSE WAIT: El servidor recibe un segmento FIN y espera que la aplicación termine su cierre.
	14. LAST ACK: Recibe la confirmación del cliente que ha cerrado.
	15. CLOSED: Se termina la conexión y se borra del registro.
	
5. que particularidad tiene tcp y la ventana deslizante que utiliza? 
	TCP separa la confirmación de recepción de segmentos y la asignación de búfer.
6. como puede ocurrir un deadlock en la comunicación tcp? 
	Si un receptor que tenía su tamaño de ventana en cero (WIN = 0) libera espacio en su búfer, envía un segmento de actualización notificando al emisor que tiene espacio disponible pero este se pierde. El emisor sigue creyendo que no puede enviar más segmentos, entonces ambos se quedan esperando.
	Se soluciona con un temporizador de persistencia de parte del emisor quien envía segmentos de "sondeo de ventana".
7. cuales son algunos problemas que se presentan en tcp? cite 
	- Sobrecarga para establecer conexión si un servidor recibe muchas solicitudes. El acuerdo de tres vías requiere el paso de muchos segmentos, y si un servidor recibe muchas solicitudes, se sobrecarga.
	- Ineficiente en aplicaciones interactivas: Se desperdicia ancho banda envían segmentos donde las cabecera es más grande que los datos (en bytes).
8. que pasa con las conexiones interactivas y tcp? 
	Se desperdicia ancho de banda debido a que las cabeceras IP/TCP (de 20 bytes cada una) son más grande que los mismo datos.
9. muestre algun ejemplo de conexión interactiva 
	Conexión a una terminal remota a través de SSH o telnet.
	Juegos en línea.
10. describa los enfoques de optimización para las conexiones interactivas 
	UTILIZANDO EL ALGORITMO DE NAGLE!!!
	El algoritmo de nagle permite enviar varios paquetes juntos en un solo segmento.
	Primero, se envía un segmento de 1 solo byte.
	Mientras que se envía ese segmento, la aplicación sigue descargando datos en el búfer del emisor.
	Al recibir el ACK, el emisor envía todos los datos acumulados en su búfer.
	Si los datos acumulados llegaron al tamaño de un MSS antes de que llegue el ACK, también envía los datos acumulados en ráfaga.
	OJO: Puede generar retrasos de ACK.
11. que problema muestra el síndrome de la ventana tonta?
	Un receptor que procesa lentamente los datos recibidos, por ejemplo cada 1 byte. Al terminar de procesar 1 byte, notifica al emisor que se liberó espacio. Este vuelve a enviar segmento haciendo que el búfer se vuelva a llenar. Este comportamiento se repite y genera retardos.
	La Solución de Clark hace que el receptor solo avise al emisor cuando tiene espacio disponible decente, como por ejemplo 1 MSS o la mitad del tamaño del búfer.
	El emisor también puede ayudar enviando datos de tamaño pequeño.
12. describa temporizadores de tcp 
	- RTO:
	Retransmission Timeout. Es el temporizador de retransmisión. Si la confirmación llega antes de que expire, se resetea. Si no, retransmite.
	Definir el tiempo de espera se realiza con el Algoritmo de Jacobson
		- Tiempo RTT suavizado: R es el retardo promedio, a = 7/8,
			`SRTT = aSRTT + (1-a)R`
		- Variación de RTT: Mide la variabilidad del retardo.
			`RTTV = B*RTTV + (1-B)*|SRTT - RTT|`
		- Calculo final de RTO:
				`RTO = SRTT + 4RTTV`
	Valor mínimo siempre 1 segundo.
	- Temporizador de Persistencia:
	Es un temporizador utilizado por el emisor para enviar enviar segmentos de "sondeo de ventana" por si haya ocurrido un deadlock.
	- Temporizador de Seguir con Vida:
	Existen conexiones que quedan inactivas por un largo tiempo. Cuando expira este temporizador, las máquinas envían un segmento para confirmar si la otra sigue activa. En caso de no recibir una confirmación, se desconecta la conexión. Si se pierde el segmento de confirmación de actividad y la conexión termina por error, puede terminar con conexión sanas.
13. explica la utilidad y en que consiste el algoritmo de Jacobson 
	Es utiliza para definir los tiempos RTT. Este algoritmo tiene en cuenta el estado de la red como el retardo, la congestión, el tiempo de ida y vuelta de cada paquete para poder converger al valor óptimo del temporizador de retransmisión.
14. que reglas pone el algoritmo de karn? 
	No actualiza las métricas del retardo y aproximaciones de RTT si no que cuando llegan ack's repetidos solo duplica el tiempo RTO.
15. distingue las responsabilidades de la capa de red y la capa de transporte en control de congestión 
	La congestión se produce en los routers de capa 3, quienes detectan la congestión.
	La capa de transporte debe regular la velocidad a la que inyecta paquetes en la red.
16. que es la cwnd? 
	Congestion Window
	Es la ventana gestionada por el emisor, su tamaño cambia dinámicamente y determina la cantidad de datos que puede enviar/insertar en la red.
17. que tasa de transmisión usa el emisor aproximadamente teniendo en cuenta la ventana efectiva de la conexión? 
	La tasa de transmisión es cwnd/RTT.
	La ventana efectiva siempre es el menor entre la ventana del receptor y la ventana de congestión.
18. que usa la capa de transporte como clock? 
	Usa la confirmacion de recepcion -> llegadas de ACKs.
19. que ley de control implementa TCP? 
	La ley AIMD (Additive Increase Multiplicative Decrease).

A continuación... explicación de algoritmos de control de congestión.
20. indique el funcionamiento del algoritmo de inicio lento
	Objetivo: descubrir la capacidad de red el inicio.
	La ventana de congestion empieza con un tamaño de 1 a 4 segmentos.
	Por cada ACK que recibe el emisor, cwnd aumenta en +1. 
	Crece de forma exponencial.
	Llega un momento en donde el tamaño de la ventana es tan grande que genera congestión. Se establece un umbral de segmentos en vuelo llamado ssthresh.
	Si expira un temporizador antes de que llegue un ack o si se supera el umbral: se reduce cwnd a su valor inicial, se disminuye el umbral a la mitad y se vuelve a empezar.
21. indique el funcionamiento del algoritmo de incremento aditivo 
	Se activa cuando cwnd supera el umbral. Crece de forma lineal. Por cada RTT completo, aumenta en +1. 
22. indique el funcionamiento del algoritmo de retransmisión rápida 
	Objetivo: Detectar la pérdida de paquetes antes de que expire el RTO.
	Se asumen que un paquete se perdió si llegan al menos 3 acks duplicados.
	Al recibir el tercero, el emisor retransmite.
	Reduce el umbral a la mitad.
	Reinicia cwnd en 1.
23. que hace tcp Tahoe? 
	Es un ejemplo de retransmisión rápida. Esta versión utiliza este algoritmo.
24. indique el funcionamiento del algoritmo de recuperación rápida
	Tras retransmitir al recibir 4 acks duplicados, se asume que el reloj ack sigue funcionando.
	El umbral se reduce a cwnd/2 y cwnd se restablece a cwnd/2 + 3 MSS.
	Cuando llega el ACK que faltaba, se establece cwnd = ssthresh y continúa con incremento aditivo.
25. que hace tcp reno? 
	Version de TCP que utiliza el algoritmos recuperación rápida.
26. describa tres mejoras de tcp
	- SACK - Selective ACK: El receptor notifica al emisor los bytes que datos sí recibió.
	Ej.: "Recibí los bytes 1-4 y 8" Entonces el emisor reenvía el bloque de 5-7
	- ECN: Explicit Congestion Notification. Los routers activan con un 1 en este campo del encabezado TCP para indicar que hay congestión.
	- BBR: Bottleneck Bandwidth 

# **6.6**

1. que dos limitaciones presenta TCP en redes modernas? 
	- Limita el tamaño de los mensajes?
2. que tres cosas hace quic? 
	todo lq sé de QUIC:
    - Arquitectura multiplexada sobre UDP orientada a reducir la latencia de establecimiento.
    - Manejo independiente de múltiples flujos (_streams_) para evitar bloqueos en línea, cifrado integral y su papel como base de **HTTP/3**.
    Qué es?
    Es un protocolo de transporte cuto objetivo es hacer más rápidos los protocolos de aplicación y el traspaso de red.
    Por qué?
    En TCP, el establecimiento de conexión genera sobrecarga debido al intercambio de muchos mensajes entre el cliente y servidor.
    Dos, cuando un cliente solicita datos de un servidor, pero se pierde un bloque de datos, la transferencia se detiene hasta que se retransmita todo ese bloque. -> GENERA LATENCIA
    Cómo soluciona?
    Crea varios flujos de datos en una sola conexión QUIC. Dentro del canal TCP-QUIC, existen varios flujos independientes, que pueden ser diferente solicitudes/respuestas entre las máquinas. Si se pierde tan solo un byte de la transmision, no es necesario parar todo, sino sólo ESE flujo. Así las demás respuestas/solicitudes del cliente o servidor no se ven afectadas y se retransmite solo ESE flujo.
    Extras de QUIC!! También hace:
    - Cifrado de datos de nivel de aplicación y transporte.
    - El cifrado es individual, para que la perdida de un paquete no afecte al resto.
    - Mejora la velocidad de traspaso de red usando un identificador de conexión.

3. que tres optimizaciones implementa quic? 
	- Encripta paquetes de forma independiente, si uno se pierde no afecta a los demás.
	- Los flujos independientes lo que significa que si un paquete se pierde, solo afecta ese flujo.
	- Facilita el trapaso de redes utilizando un Identificador para los estados cambiantes de las redes.
4. menciona un protocolo que implemente quic
	TCP
5. que significa BBR? 
	Bottleneck Bandwidth Round Trip propagation time
6. que problema presenta como solucion el uso de bbr? 
	El bufferboat, que ocurre cuando las colas de un receptor se llenan antes de que se detecte congestión. Un emisor que envía datos grandes y un receptor que tiene un búfer con mucho espacio. Un segmento ocupa gran parte del búfer, por lo que los segmentos se encolan y genera latencia antes de que la congestion sea detectable.
7. como bbr soluciona el problema para el que esta hecho? 
	Mide los valores del ancho de banda de cuello de botella y el tiempo de ida y vuelta.
8. que representa el producto ancho de banda - retardo ?
	Representa el punto en donde los búferes se empiezan a llenar.
	Cuando se llenan, la tasa de entrega no varía pero sí el tiempo RTT.
	BBR usa las aproximaciones calculadas para converger la velocidad de transmisión y reducir el retardo.

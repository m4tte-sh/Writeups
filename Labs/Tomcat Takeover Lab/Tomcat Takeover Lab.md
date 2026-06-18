Network Forensics

## Escenario
>	El equipo del SOC ha detectado actividad sospechosa en un servidor web de la intranet de la empresa. Para comprender mejor la situación, han capturado el tráfico de red para analizarlo. El archivo PCAP puede contener pruebas de actividades maliciosas que provocaron el compromiso del servidor web Apache Tomcat. Tu tarea consiste en analizar el archivo PCAP para comprender el alcance del ataque.

Q1 Given the suspicious activity detected on the web server, the PCAP file reveals a series of requests across various ports, indicating potential scanning behavior. Can you identify the source IP address responsible for initiating these requests on our server?

Para iniciar el análisis de la captura de paquetes en Wireshark, se examinaron los flujos globales mediante la opción del menú `Statistics ➝ Endpoints` (pestaña IPv4/TCP), con el objetivo de identificar qué dirección IP presentaba el mayor volumen de paquetes y conexiones establecidas.

![[Captura de pantalla 2026-06-17 172030 1.png]]

Se detectó la actividad de 7 direcciones IP distintas, de las cuales se destacó de manera anómala el comportamiento de 14.0.0.120. Para aislar su actividad, se aplicó el siguiente filtro de visualización: `ip.src == 14.0.0.120`.

![[Captura de pantalla 2026-06-17 172103.png]]

Tras aplicar el filtro, se constató que esta IP envió una ráfaga masiva de paquetes con el flag SYN activo hacia múltiples puertos de una máquina de la red local. Este patrón de tráfico confirma una fase activa de enumeración y escaneo de puertos.

Q2. Based on the identified IP address associated with the attacker, can you identify the country from which the attacker's activities originated?

Con el fin de determinar el origen geográfico de la actividad maliciosa, se procedió a consultar la dirección IP identificada (14.0.0.120) en un servicio público de inteligencia de amenazas y geolocalización IP.
![[Captura de pantalla 2026-06-17 172226.png]]

Origen de la Ip: Guangzhou, China.

Q3. From the PCAP file, multiple open ports were detected as a result of the attacker's active scan. Which of these ports provides access to the web server admin panel?

Para determinar cuál de los puertos escaneados correspondía al servicio web de administración, se aisló el tráfico de la capa de aplicación filtrando por el protocolo HTTP en co-relación con la IP del atacante mediante el siguiente filtro:`http and ip.src == 14.0.0120`.

![[Pasted image 20260618011149.png]]

Al inspeccionar el Panel de Detalles del Paquete (Packet Details Pane), se analizó la capa de transporte (Transmission Control Protocol), la cual reveló que las peticiones web se dirigían específicamente hacia el puerto de destino 8080 (puerto por defecto del servicio Apache Tomcat).

Q4. Following the discovery of open ports on our server, it appears that the attacker attempted to enumerate and uncover directories and files on our web server. Which tools can you identify from the analysis that assisted the attacker in this enumeration process?

Manteniendo el análisis sobre el mismo flujo de paquetes HTTP, se procedió a desglosar el protocolo en la capa de aplicación buscando firmas o huellas digitales (fingerprinting) en las solicitudes del atacante.

![[Pasted image 20260618012221.png]]

Dentro de la estructura de las cabeceras HTTP, se inspeccionó el campo User-Agent. Este parámetro expuso la firma explícita de la herramienta automatizada de fuzzing utilizada por el atacante para el descubrimiento de directorios: gobuster/3.6.

Q5. After the effort to enumerate directories on our web server, the attacker made numerous requests to identify administrative interfaces. Which specific directory related to the admin panel did the attacker uncover?

Tras verificar el uso de Gobuster, se analizó cuáles de las rutas web solicitadas existían realmente en el servidor comprometido. Para identificar los hallazgos exitosos del atacante, se filtraron las respuestas del servidor web Apache Tomcat (10.0.0.112) omitiendo los códigos de error genéricos 404 Not Found:

Filtro: `http.response.code != 404 && ip.dst == 14.0.0.120`

![[Pasted image 20260618013445.png]]

El análisis demostró que mientras la herramienta fallaba en rutas genéricas como /admin, las solicitudes dirigidas al directorio /manager rompieron el patrón de errores devolviendo un desafío de autenticación activa. Esto confirmó que dicho directorio era el punto de entrada real expuesto en el servidor.

Q6. After accessing the admin panel, the attacker tried to brute-force the login credentials. Can you determine the correct username and password that the attacker successfully used for login?

Sabiendo que el panel /manager de Tomcat emplea Autenticación Básica sobre HTTP (donde las credenciales viajan en las cabeceras de solicitudes GET), se aplicó un filtro basado en texto plano para aislar de forma agnóstica cualquier intento de validación de identidad, eliminando el ruido del tráfico restante:

Filtro: `http.request.line contains "Authorization"`

![[Pasted image 20260618021705.png]]

Al examinar cronológicamente los resultados en el Panel de Detalles del Paquete (específicamente en el paquete descriptivo 20571), se observó la directiva Authorization: Basic. Wireshark decodificó automáticamente el token en Base64, dejando en evidencia las credenciales válidas comprometidas que otorgaron acceso al atacante: admin:tomcat.
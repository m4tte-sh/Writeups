Network Forensics
Escenario:
>Durante un incidente de seguridad reciente, un atacante aprovechó con éxito una vulnerabilidad en nuestro servidor web, lo que le permitió subir webshells y obtener control total sobre el sistema. El atacante utilizó el servidor web comprometido como punto de partida para llevar a cabo otras actividades maliciosas, incluida la manipulación de datos. 
>Como parte de la investigación, se le proporciona una captura de paquetes (PCAP) del tráfico de red durante el ataque para reconstruir la cronología del mismo e identificar los métodos utilizados por el atacante. El objetivo es determinar el punto de entrada inicial, las herramientas y técnicas del atacante, y el alcance de la vulneración.

Siguiendo el escenario, empezaremos analizando el protocolo http en la Captura de paquetes proporcionada con Wireshark, (dar un porqué), y utilizar `statistics ➝ endpoints`Identificar las IPs implicadas en la comunicación HTTP.

![[Pasted image 20260615181147.png]]

Se inició el análisis sobre la primera dirección IP identificada. Con el objetivo de optimizar la revisión del tráfico capturado, se filtraron los paquetes asociados a dicha dirección y se restringieron los resultados a comunicaciones HTTP. A continuación, se buscaron solicitudes que incluyeran la cadena **"upload"** en el cuerpo de la petición, ya que la vulnerabilidad explotada correspondía a un mecanismo de carga de archivos. Este procedimiento permitió aislar las peticiones relevantes y facilitar la identificación de la actividad maliciosa.




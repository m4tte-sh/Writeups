Investigate IcedID malware using VirusTotal and threat intelligence platforms to identify IOCs, associated threat actors, and execution mechanisms.

---
### Escenario
>Se ha identificado un grupo de ciberamenazas por poner en marcha campañas de phishing a gran escala con el fin de distribuir otras cargas maliciosas. Las cargas más frecuentes eran de tipo IcedID. Se te ha facilitado un hash de una muestra de IcedID para que analices y supervises las actividades de este grupo de amenaza persistente avanzada (APT).

----
El hash que nos proporciona el lab es el siguiente: `191eda0c539d284b29efe556abb05cd75a9077a0`

Primeramente, analizaremos el hash en la página **VirusTotal**.
![Análisis Inicial en VirusTotal](img/Pasted%20image%2020260623173043.png)



Q1. What is the name of the file associated with the given hash?

El análisis comienza con la ingesta del hash SHA-1 en la plataforma VirusTotal para examinar sus propiedades estáticas y metadatos. Al inspeccionar la sección de Names dentro de la pestaña Details, se identifican los nombres con los que el archivo fue reportado en la telemetría global.
![Nombres del Archivo en VirusTotal](img/Pasted%20image%2020260623173217.png)

En nuestro caso nos interesa el último: `document-1982481273.xlsm`

**Nota Técnica**: La extensión .xlsm indica que se trata de un libro de Microsoft Excel habilitado para macros. Los atacantes suelen camuflar sus vectores de ataque en estos formatos para aprovechar el motor de ejecución de Visual Basic for Applications (VBA) o macros de Excel 4.0, logrando así evadir los filtros de correo electrónico que bloquean archivos ejecutables directos (como .exe).

Q2. Can you identify the filename of the **GIF** file that was deployed?

El siguiente paso consiste en determinar si la ejecución del vector inicial realiza la descarga o creación de componentes adicionales en el sistema comprometido (fase de Installation o Actions on Objectives). Para ello, nos dirigimos a la pestaña Relations y examinamos el apartado de Dropped Files (archivos depositados en disco).

![Apartado Dropped Files](img/Pasted%20image%2020260623173823.png)
En esta sección se detecta la creación del siguiente archivo:
`3003.gif`

Q3. How many domains does the malware look to download the additional payload file in **Q2**?

Para comprender el alcance de la infraestructura de red del atacante, se analiza el comportamiento de conexión de la muestra. En la misma pestaña Relations, nos desplazamos hacia la sección de Contacted URLs con el fin de auditar las peticiones HTTP/HTTPS salientes generadas por el malware para extraer la carga útil identificada en el paso anterior.

El análisis de las conexiones revela que el malware interactúa exactamente con:
5 dominios independientes

![Sección Contacted URLs](img/Pasted%20image%2020260623174705.png)

Q4. From the domains mentioned in **Q3**, a DNS registrar was predominantly used by the threat actor to host their harmful content, enabling the malware's functionality. Can you specify the Registrar INC?

Una vez aislada la infraestructura maliciosa, el siguiente paso es auditar la propiedad y gestión de los dominios utilizados por el adversario. Cruzando los indicadores de red con la tabla de Contacted Domains de VirusTotal, se procede a examinar la información de Whois de los servidores activos.

Al filtrar y descartar los dominios legítimos de infraestructura en la nube (como Microsoft o Amazon), el análisis determina que el registrador comercial (DNS Registrar) utilizado predominantemente por el atacante para alojar su contenido dañino es: **NameCheap**.
![Información Whois del Registrador](img/Pasted%20image%2020260623180515.png)

Q5. Could you specify the threat actor linked to the sample provided?

Con los Indicadores de Compromiso (IoCs) recopilados (hash, nombres de archivo, dominios y estructura de URLs), se avanza hacia la fase de atribución táctica utilizando plataformas globales de Threat Intelligence. Se realiza una consulta avanzada cruzando estos datos con el marco de trabajo MITRE ATT&CK y los perfiles de indexación de familias de malware en Malpedia.

La correlación de datos vincula de forma directa esta actividad maliciosa con el siguiente grupo de amenazas:
TA551 o conocido como GOLD CABIN
![Atribución del Threat Actor TA551](img/Pasted%20image%2020260623183631.png)
Este grupo es conocido por utilizar IcedID en sus campañas maliciosas, proporcionando información sobre el origen del malware y los objetivos típicos. El perfil de Malpedia confirma esta atribución, mostrando descripciones detalladas de las tácticas, técnicas y procedimientos de GOLD CABIN.
![Perfil de Gold Cabin en Malpedia](img/Pasted%20image%2020260623183819.png)

Q6. In the **Execution** phase, what function does the malware employ to fetch extra payloads onto the system?

Finalmente, para validar el mecanismo técnico exacto que emplea el malware durante su fase de Execution para traer las cargas secundarias al sistema, se analiza la muestra mediante análisis dinámico en una Sandbox (en este caso, Recorded Future Triage), monitoreando las llamadas al sistema en tiempo real.

El monitoreo de la ejecución revela que la macro del documento abusa directamente de la interfaz de programación de aplicaciones de Windows (Windows API) invocando la función: `URLDownloadToFileA`
![Monitoreo de la API URLDownloadToFileA en Triage](img/Pasted%20image%2020260623184759.png)

Esta función permite la descarga de archivos desde Internet, lo que permite que el malware obtenga y ejecute más componentes maliciosos, ampliando así su funcionalidad y persistencia dentro del entorno infectado. Las capturas de pantalla de las herramientas de monitoreo de API confirman el uso de `URLDownloadToFileA`Y destacar las llamadas relacionadas a dominios sospechosos.






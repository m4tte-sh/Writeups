Analice un kit de phishing de criptomonedas para identificar métodos de exfiltración, extraer COI críticos y recopilar inteligencia de actores de amenazas utilizando registros locales y API de Telegram.

----
### Escenario
>Una plataforma de finanzas descentralizadas (DeFi) informó recientemente múltiples quejas de los usuarios sobre retiros de fondos no autorizados. Una revisión forense descubrió un sitio de phishing que se hizo pasar por el intercambio legítimo de PancakeSwap, atrayendo a las víctimas a ingresar sus frases de semillas de billetera. El kit de phishing se alojó en un servidor comprometido y se exfiltró credenciales a través de un bot de Telegram.
>Su tarea es realizar análisis de inteligencia de amenazas en la infraestructura de phishing, identificar indicadores de compromiso (IoC) y rastrear la presencia en línea del atacante, incluidos los alias y los identificadores de Telegram, para comprender sus tácticas, técnicas y procedimientos (TTP).

----

Q1. Which wallet is used for asking the seed phrase?

Dentro de los archivos, una carpeta contiene el nombre de la wallet. El kit de phishing está diseñado específicamente para apuntar a usuarios de MetaMask, simulando la interfaz de dicha extensión para engañar a las víctimas.

![Q1-Wallet](img/Pasted%20image%2020260622164956.png)

Q2. What is the file name that has the code for the phishing kit?

El código principal encargado del procesamiento de credenciales, persistencia local y la lógica de exfiltración se encuentra en el archivo backend principal.

![Q2-Code1](img/Pasted%20image%2020260622165314.png)
![Q2-Code2](img/Pasted%20image%2020260622165419.png)

Q3. In which language was the kit written?

Como se observa en el análisis del código fuente, el kit está completamente escrito en PHP (Hypertext Preprocessor), ejecutándose del lado del servidor.

Q4. What service does the kit use to retrieve the victim's machine information?

SypexGeo.
El kit utiliza el servicio externo de Sypex Geo (sypexgeo.net).
Este servicio en particular se especializa en geolocalización por IP. En este script, el atacante lo usa para averiguar en qué parte del mundo está la víctima (obteniendo datos como el país y la ciudad) a partir de su dirección IP mediante la variable global `$_SERVER['REMOTE_ADDR']`. Adicionalmente, recolecta la firma del navegador a través de `$_SERVER['HTTP_USER_AGENT']`.
![Q4-SypexGeo](img/Pasted%20image%2020260622170557.png)

Q5. How many seed phrases were already collected?

Al ver las últimas líneas del código, el script guarda la frase robada en un archivo de texto llamado log.txt dentro de una carpeta oculta (/log/) en el propio servidor web como mecanismo de respaldo. Tras inspeccionar el archivo, se confirma que contiene 3 frases robadas.
![Q5-Log1](img/Pasted%20image%2020260622171106.png)
![Q5-Log2](img/Pasted%20image%2020260622171254.png)

Q6. Could you please provide the seed phrase associated with the most recent phishing incident?

Extrayendo la tercera y última entrada registrada en el archivo de log local, la frase semilla más reciente es:
`father also recycle embody balance concert mechanic believe owner pair muffin hockey`

Q7. Which medium was used for credential dumping?

Telegram. El script utiliza las credenciales de un Bot de Telegram `($token)` y un identificador de chat específico `($id)` dentro de una función dedicada llamada `sendTel()`. Envía el reporte con la frase semilla robada, la IP, el User-Agent y la ubicación de la víctima directamente al chat del atacante utilizando la API oficial de Telegram mediante peticiones HTTP de tipo GET.

Q8.What is the token for accessing the channel?

Dentro del script, está declarado explícitamente el token de acceso del bot de Telegram:
`5457463144:AAG8t4k7e2ew3tTi0IBShcWbSia0Irvxm10`.

![Q8-Token](img/Pasted%20image%2020260622172944.png)

Q9. What is the Chat ID for the phisher's channel?

Dentro del script, está declarado el identificador numérico del chat receptor:
`5442785564`.

![Q9-ChatID](img/Pasted%20image%2020260622172532.png)

Q10. What are the allies of the phish kit developer?

Observamos en los comentarios integrados en el encabezado del código la firma y el alias del creador del script: `j1j1b1s@m3r0`.

![Q10-Developer](img/Pasted%20image%2020260622172753.png)

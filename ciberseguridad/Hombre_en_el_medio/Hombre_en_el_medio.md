# Man in the middle (Práctica 3)

> Ibai Guillén Pacho
> 

> Ing. Informática + TDE
> 

# Objetivo de la práctica

Realizar un ataque de **Man in the middle** mediante **ARP poisoning**.

# Herramientas

### Whireshark

Analizador de protocolos, utilizado para realizar análisis, solucionar problemas en redes de comunicaciones y para el análisis de datos y protocolos.

[Wireshark · Go Deep.](https://www.wireshark.org)

### Ettercap

Ettercap es un interceptor/sniffer/registrador para LANs con switch. Sirve en redes LAN conmutadas, aunque es utilizado para auditorías en distintos tipos de redes. Soporta direcciones activas y pasivas de varios protocolos.

[Ettercap Home Page](https://www.ettercap-project.org/)

### DRIFTNET

Driftnet observa el tráfico de la red y selecciona y muestra JPEG, GIF y otros formatos de imagen para su visualización. También puede extraer datos de audio MPEG de la red y reproducir.

[deiv/driftnet](https://github.com/deiv/driftnet)

# Red objetivo

En este caso el objetivo de este ataque es una red en la que dos ordenadores están comunicándose entre sí. La tarea a realizar es interceptar esa comunicación, escucharla y enviarla para que continúe su camino, de forma que seamos conocedores de toda la información que fluctúa entre estos dos ordenadores.

![recursos/Untitled.png](recursos/Untitled.png)

# ARP posioning

[Dispositivos en la red](https://www.notion.so/794d5d40c6cc47b6990c7bcf591bfc7f)

## Comunicación normal

Windows Server 1 haciendo ping a Windows Server 2:

![recursos/Untitled%201.png](recursos/Untitled%201.png)

Pregunta por la MAC detrás de la IP 192.168.145.130, el broadcast le devuelve la MAC original del ordenador.

## Intercepción de la comunicación

### Configuración de Ettercap

Para realizar este ataque se usará la herramienta ettercap, la cual permite mediante su interfaz gráfica realizar este ataque.

Para poder utilizarlo primero deberemos abrir la aplicación, ir a Hosts y hacer click en Host List:

![recursos/Untitled%202.png](recursos/Untitled%202.png)

Después añadiremos la IP del Windows Server 1 como Objetivo 1.

![recursos/Untitled%203.png](recursos/Untitled%203.png)

Y la dirección IP del Windows Server 2 como Objetivo 2.

![recursos/Untitled%204.png](recursos/Untitled%204.png)

Una vez estén ambos ordenadores seleccionados iremos a la pestaña Targets, y haremos clic en Current Targets.

![recursos/Untitled%205.png](recursos/Untitled%205.png)

En este apartado podremos ver la conexión que vamos a interceptar.

![recursos/Untitled%206.png](recursos/Untitled%206.png)

Si todo está bien podremos ir a la pestaña Mitm y seleccionar el tipo de ataque, en este caso ARP poisoning.

![recursos/Untitled%207.png](recursos/Untitled%207.png)

Una vez seleccionado el tipo de ataque nos dejará parametrizarlo, de forma que podamos interceptar solo el tráfico unidireccional o que podamos escuchar también las conexiones remotas. En este caso marcaremos esta última opción.

![recursos/Untitled%208.png](recursos/Untitled%208.png)

Para ejecutar este ataque solo quedaría ir a la pestaña start y darle a start sniffing.

![recursos/Untitled%209.png](recursos/Untitled%209.png)

### Tablas ARP de las víctimas

Para poder ver si este ataque está consolidado se podría acceder a la tabla ARP de las víctimas para comprobar si la dirección MAC del otro ordenador en la red tiene la MAC del atacante.

De esta forma la tabla ARP del Windows Server 1 es:

![recursos/Untitled%2010.png](recursos/Untitled%2010.png)

De esta forma la tabla ARP del Windows Server 2 es:

![recursos/Untitled%2011.png](recursos/Untitled%2011.png)

Como se puede ver ambas tablas ARP de los diferentes ordenadores apuntan al ordenador del atacante.

### Comunicación

Repitiendo el proceso de la comunicación normal del anterior punto, podemos ver como para hacer ping los paquetes se han duplicado y la MAC de destino para el Windows Server 1 (realizador del ping) es la del atacante. De esta forma los paquetes primero llegan al atacante y después al destino real.

![recursos/Untitled%2012.png](recursos/Untitled%2012.png)

<aside>
ℹ️ Se han añadido dos columnas para [mostrar la dirección MAC en wireshark](https://osqa-ask.wireshark.org/questions/23080/source-and-destination-columns#:~:text=If%20you%20want%20to%20show,Hardware%20dest%20addr"%2C%20respectively.).

</aside>

El proceso ya no es tan sencillo como un ping normal:

1. Emitir desde el Windows Server 1
2. Recibir en Windows Server 2
3. Emitir desde el Windows Server 2
4. Recibir en Windows Server 1

Ahora el proceso se ve intervenido por el atacante, de forma que el proceso es más largo:

1. Emitir desde Windows Server 1
2. Recibir en Atacante
3. Emitir desde Atacante
4. Recibir en Windows Server 2
5. Emitir desde Windows Server 2
6. Recibir en Atacante
7. Emitir desde Atacante
8. Recibir en Windows Server 1

Este proceso degrada bastante la calidad de la conexión, puesto que el número de paquetes para realizar la misma función se ven duplicados y esto incide bastante en la velocidad para realizar dichas tareas, pudiendo detectarse dicho ataque.

### Problemas de seguridad

Este ataque puede ocasionar grandes problemas de seguridad especialmente cuando ambos ordenadores intercambian información sensible, de forma que esta información podrá ser accedida desde el atacante.

Un ejemplo de lo anterior es la situación en la que el Windows Server 1 accede al servidor ftp de Windows Server 2, quedando expuesta las credenciales de acceso puesto que en este protocolo no existe encriptación.

Inicio de conexión:

- Víctima:
    
    ![recursos/Untitled%2013.png](recursos/Untitled%2013.png)
    
- Paquetes de la red:
    
    ![recursos/Untitled%2014.png](recursos/Untitled%2014.png)
    

Usuario metido:

- Víctima:
    
    ![recursos/Untitled%2015.png](recursos/Untitled%2015.png)
    
- Paquetes de la red:
    
    ![recursos/Untitled%2016.png](recursos/Untitled%2016.png)
    

Contraseña metida:

- Víctima:
    
    ![recursos/Untitled%2017.png](recursos/Untitled%2017.png)
    
- Paquetes de la red:
    
    ![recursos/Untitled%2018.png](recursos/Untitled%2018.png)
    

Si seguimos el flujo TCP de esas trazas podremos observar lo ocurrido con más detalle:

![recursos/Untitled%2019.png](recursos/Untitled%2019.png)

Los paquetes que han participado en este proceso han sido todos estos:

![recursos/Untitled%2020.png](recursos/Untitled%2020.png)

Como la contraseña viaja sin ningún tipo de encriptación es leíble en texto plano y fácilmente reconocible, en este momento se hubiera conseguido las credenciales de acceso al ordenador de la víctima con simplemente leer los paquetes que se pasan entre ellos.

### Capturar imágenes

Para este tipo de secuestros de conexión se suele aprovechar y usar una herramienta de captura de imágenes, como puede ser driftnet, que nos permite mostrar por pantalla los recursos del servidor http a los que se accedan. Para usarla primero habrá que instalarla en la máquina `sudo apt-get install driftnet` y después ejecutarla mediante `driftnet -i eth0`.

<aside>
⚠️ La máquina virtual provista para las prácticas ocasiona un error a la hora de instalar driftnet, la solución se encuentra en el siguiente enlace.

</aside>

[Invalid signature for Kali Linux repositories : "The following signatures were invalid: EXPKEYSIG ED444FF07D8D0BF6 Kali Linux Repository"](https://unix.stackexchange.com/questions/421821/invalid-signature-for-kali-linux-repositories-the-following-signatures-were-i)

Los recursos del servidor http son los siguientes:

![recursos/Untitled%2021.png](recursos/Untitled%2021.png)

![recursos/Untitled%2022.png](recursos/Untitled%2022.png)

Para poder interceptar cualquier recurso al que se acceda habrá que abierta la ventana de driftnet que aparecerá en negro:

![recursos/Untitled%2023.png](recursos/Untitled%2023.png)

Sin embargo, nada más el usuario acceda a algún archivo o a algún elemento podremos ver lo que le aparecería por pantalla:

El usuario si accede a la imagen Sample.jpg que se ha introducido en la carpeta images la verá en el navegador.

![recursos/Untitled%2024.png](recursos/Untitled%2024.png)

Mientras que el atacante la podrá ver en driftnet.

![recursos/Untitled%2025.png](recursos/Untitled%2025.png)

# Documentación

Documentación trabajada en Notion, link al formato original:

[Man in the middle (Práctica 3)](https://www.notion.so/Man-in-the-middle-Pr-ctica-3-6516ab019e29429993eea30d28baea73)

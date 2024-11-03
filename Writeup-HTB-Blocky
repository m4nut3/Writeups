# HACK THE BOX - BLOCKY

### Sistema Operativo -> Linux
### Dificultad -> Easy

IP -> 10.10.10.37

### Enumeración con nmap

En primer lugar se va a realizar un escaneo con la herramienta *nmap* para ver puertos abiertos de la máquina **objetivo**:
```shell
sudo nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn 10.10.10.37 -oN nmap_enum
```
```shell
-p- #Escanear todo el rango de puertos (65535)
--open #Solo mostrar los puertos abiertos
-sS #Usar el método de escaneo SYN scan
--min-rate 5000 #No enviar menos de 5000 paquetes por segundo (acelerar el escaneo)
-n #No aplicar resolución DNS
-Pn #No aplicar descubrimiento de hosts
-vvv #Mostrar información en tiempo real de los puertos que se vayan encontrando
-oN <file> #Guardar el output en formato nmap
```
![escaneo](/capturas/2024-11-03_15-38.png)

En segundo lugar se va a usar también *nmap* para realizar un escaneo exhaustivo de los puertos que sabemos que están abiertos:
```shell
nmap -sCV -p21,22,80,25565 10.10.10.37 -oN nmap_versions
```
```shell
-p<port> #Escanear solo los puertos indicados
-sC #Ejecutar los scripts por defecto
-sV #Identificar la versión del servicio
```

![escaneo](/capturas/2024-11-03_15-40.png)

### Pruebas FTP

Al tener identificadas las versiones de los servicios, se va a empezar con la enumeración de vulnerabilidades y exploits conocidos para las versiones encontradas.

Por ejemplo, al ver que está abierto el puerto **21 FTP** con la versión **ProFTPD 1.3.5a** se va a probar con el *login* anónimo

![pruebas](/capturas/2024-11-03_16-07_1.png)

Al no resultar efectivo se va a buscar un exploit conocido con la herramienta *searchsploit* 
```shell
searchsploit ProFTPD 1.3.5a
```

![pruebas](/capturas/2024-11-03_16-07.png)

### Examinación de la Web

Al no tener éxito en las pruebas realizadas al protocolo **FTP** se va a examinar el servicio Web que está corriendo por el puerto **80 HTTP**

![examinacionweb](/capturas/2024-11-03_16-47.png)

Se ve que al acceder al servicio **HTTP** a través del navegador se nos redirige al dominio *blocky.htb* el cual no se nos resuelve por lo que se tiene que añadir al archivo `/etc/hosts`

```shell
sudo nano /etc/hosts
```

![examinacionweb](/capturas/2024-11-03_16-19.png)

Al anadirlo ya se encuentra el dominio *blocky.htb*

![examinacionweb](/capturas/2024-11-03_16-53.png)

Para ver a qué se está haciendo frente, se va a utilizar la herramiente `whatweb` para realizar un reconocimiento de las tecnologías que usa el servicio Web

```shell
whatweb http://blocky.htb
```

![examinacionweb](/capturas/2024-11-03_18-17.png)

Se obtiene como resultado que la Web está usando el gestor de contenido WordPress por lo que para enumerar información sobre usuarios, contraseñas, directorios vulnerables y plugins y temas vulnerables se va a hacer uso de la herramienta `wpscan`

```shell
wpscan --url http://blocky.htb -e vt,vp,u
```
```shell
-e #enumerar
vt #templates vulnerables
vp #plugins vulnerables
u #usuarios
```

![examinacionweb](/capturas/2024-11-03_18-18.png)

![examinacionweb](/capturas/2024-11-03_18-19.png)

Tras el escaneo se encuentra el usuario **notch** pero no se encuentra ninguna información adicional que nos ayude a obtener acceso al sistema

### Fuzzing Web

Para continuar con el reconocimiento de la Web se realiza un *fuzzing* al puerto **HTTP** para descubrir directorios que nos permitan obtener información o que sean vulnerables para encontrar una vía de acceso a la máquina por este puerto utilizando la herramienta `gobuster`

```shell
gobuster dir -u http://blocky.htb -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

![fuzzingweb](/capturas/2024-11-03_16-55.png)

### Examinación de directorios

Tras revisar el contenido de los distintos directorios obtenidos con el fuzzing se encuentra uno que llama la atención porque contiene dos archivos `.jar` descargables

![examinaciondirectorios](/capturas/2024-11-03_16-55_1.png)

### Examinación de ficheros

Se descargan los dos archivos y se procede con la decompilación de los mismos para analizar su contenido con la herramienta `jd-gui`

![examinacionficheros](/capturas/2024-11-03_16-56.png)

![examinacionficheros](/capturas/2024-11-03_16-57.png)

Tras analizar el contenido de los dos archivos se descubre un usuario **root** almacenado en una variable llamada *sqlUser* y una contraseña **8YsqfCTnvxAUeduzjNSXe22** almacenada en una variable llamada *sqlPass*, lo que nos informa de que son credenciales de acceso al servicio `mysql` del sistema

### Acceso a la máquina víctima

Una vez habiendo descubierto dos usuarios y una contraseña, lo primero que se debe de probar es el acceso al sistema mediante el protocolo `ssh` ya que es la manera más cómoda de tener acceso a un sistema ya que no es necesario la explotación de ninguna vulnerabilidad

Tras probar con los dos usuarios se obtiene acceso con el usuario **notch** lo que nos indica que este usuario es el administrador del servicio `mysql` y ha reutilizado la contraseña

```shell
ssh notch@blocky.htb
```
![accesomaquina](/capturas/2024-11-03_16-58.png)

### Escalada de privilegios

Al tener la contraseña del usuario, se puede probar a comprobar si el usuario **notch** tiene capacidad de ejecutar algún archivo o comando como `sudo`

```shell
sudo -l
```
![escaladaprivilegios](/capturas/2024-11-03_16-58_1.png)

Se identifica que nuestro usuario puede ejecutar cualquier comando como `root` usando `sudo` por lo que se puede invocar una terminal interactiva para ganar privilegios de `root`

```shell
sudo /bin/bash -p
whoami
```
![escaladaprivilegios](/capturas/2024-11-03_16-59.png)
# ROOTED!

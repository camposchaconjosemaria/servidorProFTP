# Instalación y configuración del servidor FTP ProFTPd en AWS
<img src="https://github.com/camposchaconjosemaria/servidorProFTP/assets/114906855/f8a553b4-8a82-4a8b-9dcc-88e51d2b5b50" width="900px">

## Índice

1. [Instalación del servidor ProFTP](#1-instalación-del-servidor-proftp)
2. [Configuración del servidor ProFTP](#2-configuración-del-servidor-proftp)
3. [Otras configuraciones](#3-otras-configuraciones)
4. [Referencias](#4-referencias)
5. [Licencia](#5-licencia)

---

## 1. Instalación del servidor ProFTP

En primer lugar, instalaremos el paquete correspondiente del servidor realizando una actualización de los repositorios anteriormente y paquetes del sistema.

```bash
sudo apt update ; apt upgrade -y
sudo apt-get install proftpd
```
Y comprobamos su estado:

```bash
sudo systemctl status proftpd
```
Podríamos Iniciar/Parar/Reiniciar/Habilitar/Recargar el servicio con las siguientes opciones del comando systemctl:

```bash
sudo systemctl "start/stop/restart/enable/reload" proftpd
```

## 2. Configuración del servidor proftp

Una vez instalado el servicio dispondremos de un fichero de configuración situado en ***/etc/proftpd/proftpd.conf***

### Paso 1. Comprobar que el servidor está configurado como standalone y asignarle un nombre para personalizarlo.

```bash
#[...]
ServerName                      "cachacoFTP"
ServerType                      standalone
DeferWelcome                    off
#[...]
```

### Paso 2. Enjaular a los usuarios en sus respectivos directorios ***/home*** si quisiéramos. 

```bash
# Use this to jail all users in their homes 
 DefaultRoot ~  #Bastaría con descomentar esta línea
 === Acceso restringido a un directorio específico ===
 DefaultRoot ~/ftp #Para que se pueda efectuar, debe existir dicha carpeta en cada usuario
```

**Siempre es necesario reiniciar el servicio para que se actualicen las configuraciones realizadas.**

### Paso 3. Creación de usuario y prueba de conexión
Para realizar una conexión debemos hacerlo con algún usuario del servidor, sino prodríamos crealo:
```bash
sudo adduser cachaco1
```
Para poder establecer la conexión, bastaría con utilizar el siguiente comando, con la IP pública de nuestro servidor de AWS (con los puertos 20 y 21 habilitados del server), además de tener el paquete ftp instalado en el cliente (#apt install ftp)

```bash
ftp 54.146.221.198
```
Con la insersión del usuario creado, ya podríamos acceder al servidor:

![image](https://github.com/camposchaconjosemaria/servidorProFTP/assets/114906855/4b8c3e12-b9a8-4a71-8420-505d410a5830)

Para conocer los comandos que permite utilizar FTP puedes introducir el comando **help** o el comando **?**:

<img src="https://github.com/camposchaconjosemaria/servidorProFTP/assets/114906855/ae24892b-7d17-461c-a913-7eea27496f33" width="450px">

## 3. Otras configuraciones

### Opcion 1. Mensajes de bienvenida y/o error personalizados:

Puedes cambiar el mensaje de bienvenida y el de error al conectar editando ***/etc/proftpd/proftpd.conf***, para ello debes añadir al final del fichero estas dos líneas:

```bash
AccessGrantMsg "Mensaje de bienvenida"
AccessDenyMsg "Mensaje de error"
```
También puedes hacer que ProFTPD lance un fichero ASCII como mensaje antes del login y después, imagina que guardamos los ficheros welcome1.msg y welcome2.msg en el directorio de ProFTPD, en el fichero de configuración escribiríamos:

```bash
DisplayConnect /etc/proftpd/welcome1.msg
DisplayLogin /etc/proftpd/welcome2.msg
```
Y obtendríamos:

<img src="https://github.com/camposchaconjosemaria/servidorProFTP/assets/114906855/65ff8bdb-d12a-4cec-acd0-1d86423f2fd6" width="550px">


































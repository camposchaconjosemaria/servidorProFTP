# Instalación y configuración del servidor FTP ProFTPd en AWS
<img src="https://github.com/camposchaconjosemaria/servidorProFTP/assets/114906855/f8a553b4-8a82-4a8b-9dcc-88e51d2b5b50" width="900px">

## Índice

1. [Instalación del servidor ProFTP](#1-instalación-del-servidor-proftp)
2. [Configuración del servidor ProFTP](#2-configuración-del-servidor-proftp)
3. [Otras configuraciones](#3-otras-configuraciones)
4. [Acceso de usuario anónimo](#4-acceso-de-usuario-anónimo)
5. [Referencias](#5-referencias)
6. [Licencia](#6-licencia)

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
También puedes hacer que ProFTPD lance un fichero ASCII como mensaje antes del login y después. Imagina que guardamos los ficheros welcome1.msg y welcome2.msg en el directorio de ProFTPD, en el fichero de configuración escribiríamos:


DisplayConnect /etc/proftpd/welcome1.msg
DisplayLogin /etc/proftpd/welcome2.msg
Y obtendríamos:

<img src="https://github.com/camposchaconjosemaria/servidorProFTP/assets/114906855/65ff8bdb-d12a-4cec-acd0-1d86423f2fd6" width="550px">

### Opción 2. Limitar el acceso al servidor a usuarios específicos:
El servidor es capaz de permitir el acceso a algunos usuarios y bloquear a otros, para ello deberemos escribir unas directivas en el fichero de configuración:

```bash
<Limit LOGIN>
    AllowUser usuario1
    DenyUser usuario2
</Limit>
```

En este ejemplo se permite el acceso al usuario1 y se denega el acceso al usuario2.

Las directrices que puedes introducir aquí para permitir o denegar el acceso son:

 - AllowUser usuario - Permite el login al ftp al usuario.
 - DenyUser usuario - Impide el login al ftp al usuario.
 - AllowAll - Permite el login al ftp a todos los usuarios.
 - DenyAll - Impide el login al ftp a todos los usuarios que no tengan una directiva AllowUser específica.

### Opción 3. Limitar el número de sesiones simultáneas por usuario:
Puedes limitar la cantidad de sesiones simultáneas que un usuario puede tener abiertas con el servidor FTP con la directiva MaxClientsPerUser en el fichero de configuración:

```bash
MaxClientsPerUser  1 
```
Hará que un usuario solo pueda tener una sesión abierta con el servidor a la vez.

### Opción 4. Crear cuotas de transferencia a los usuarios:

Una manera de limitar el uso que se haga del servidor FTP por los usuarios es el de crear cuotas, que son límites de transferencia de ficheros. Limitando la cantidad de ficheros que puede subir o descargar o limitando la cantidad de datos (bytes) subidos o descargados.

#### 4.1. Configurar el uso de cuotas:

Por defecto las cuotas en ProFTPD están desactivadas, así que lo primero será activarlas en el fichero de configuración:

```bash
<IfModule mod_quotatab.c>
#Activación de las quotas
QuotaEngine on
#Creación de fichero log de quotas:
QuotaLog /var/log/proftpd/quota.log
<IfModule mod_quotatab_file.c>
#Creación de ficheros con la tabla de cuotas:
     QuotaLimitTable file:/etc/proftpd/ftpquota.limittab
     QuotaTallyTable file:/etc/proftpd/ftpquota.tallytab
</IfModule>
</IfModule
```

Las cuotas utilizan módulos adicionales de ProFTPD (mod_quotatab y mod_quotatab_file) así que debes indicarle a ProFTPD en el fichero de configuración que incluya el código de estos módulos descomentando o añadiendo la siguiente línea.

```bash
Include /etc/proftpd/modules.conf
```
Y pasaríamos al reinicio del servicio para la actualización de la configuración.

#### 4.2. Crear tabla de cuotas:
Para poder utilizar las cuotas de usuario primero has de crear una tabla donde se guardarán las cuotas que vayamos generando, para ello utilizarás el terminal:

```bash
sudo ftpquota --create-table --type=limit --table-path=/etc/proftpd/ftpquota.limittab
sudo ftpquota --create-table --type=tally --table-path=/etc/proftpd/ftpquota.tallytab
```

#### 4.3. Crear una cuota de usuario:

Los ficheros que has creado antes /etc/proftpd/ftpquota.limittab son unas tablas con las cuotas de usuarios que generes, cada cuota de usuario será un registro de las tablas, vamos a generar un registro de ejemplo:

Registro de ejemplo --> Usuario: cachaco1 / Tipo de cuota: user, podría ser de grupo también. / Cantidad de información que puede subir al servidor: 2. / Cantidad de información que puede descargar del servidor: 40. / Especificamos las unidades en Mb. (2Mb de subida y 40Mb de descarga). / Cantidad de ficheros que puede subir al servidor: 15. / Cantidad de ficheros que puede descargar del servidor: 50.

```bash
sudo ftpquota --add-record --type=limit --name=cachaco1 --quota-type=user --bytes-upload=2 --bytes-download=40 --units=Mb --files-upload=15 --files-download=50 --table-path=/etc/proftpd/ftpquota.limittab
```

Como puedes ver has añadido el registro, por lo que usas --add-record, también puedes modificar o borrar un registro:

* --add-record: Añadir registro de cuota.
* --update-record: Cambiar registro de cuota ya existente.
* --delete-record: Borrar registro de cuota.
* --show-record: Muestra los registros (necesitas poner el parámetro --type=limit o --type=tally, según qwuieras ver los límites por tamaño o por cantudad de ficheros).
  
La cuota es aplicada a un usuario y se acumula entre sesiones de FTP, si quisiers que los límites se aplicasen solamente a una sesión podrías utilizar la opción --per-session que indica que los límites son solo para la sesión y se reinician en la siguiente sesión.

Ver las cuotas creadas:
Podremos verlas con el siguiente comando, ejecutandolo en la carpeta donde se encuentran las tablas. En este caso en ***/etc/proftpd***

```bash
sudo ftpquota --show-records --type=limit
```

Siguiendo el ejemplo de cachaco1, vemos las tablas con las siguientes instrucciones:

![image](https://github.com/camposchaconjosemaria/servidorProFTP/assets/114906855/48b48ec3-da4e-4d2b-802b-dfe639ba9b8a)

**Si en algún momento excedes la cuota lo podrás ver en el fichero de log /var/log/proftpd/proftpd/quota.log**

#### 4.4. Limitar el ancho de banda de la transferencia:

Una opción importante para tu servidor, sobre todo si el ancho de banda es pequeño, es el limitar el ancho de banda que tiene un grupo de usuarios o, incluso, usuarios a nivel particular.

Para ello debes introducir en el fichero de configuración la directiva TransferRate:
```bash
TransferRate RETR 50 user usuario1
```

http://www.proftpd.org/docs/contrib/ftpquota.html

### 5. Acceso de usuario anónimo

Una función que permiten los servidores FTP es dejar a usuarios anónimos conectarse a ellos.

Esto es algo que puede poner en riesgo tu equipo servidor por el mero hecho de que estarás dejando subir ficheros a perfectos desconocidos... A cambio puede ser un sistema de distribución de ficheros rápido si se impide que estos usuarios anónimos puedan subir ficheros al servidor.

Cosas a tener en cuenta:

El usuario anónimo es en realidad un usuario que por defecto viene creado en el sistema del servidor, con nombre ftp (si no es así, pasaríamos a crearlo). Pero al cual se le dará el alias anonymous después en la configuración de ProFTPD.

Se debe crear un directorio que será donde se conecten los usuarios anónimos, ese directorio será propiedad del usuario ftp.
```bash
sudo mkdir -p /var/ftp
sudo chown -R ftp /var/ftp
```
Descomentamos las lineas necesarias para una configuración básica de la conexión y reiniciamos el servicio:

![image](https://github.com/camposchaconjosemaria/servidorProFTP/assets/114906855/519f0ede-c52b-47bb-bc0c-e03f2f1cb0f2)

![image](https://github.com/camposchaconjosemaria/servidorProFTP/assets/114906855/c60b692d-9cf4-405e-9a65-11ffdab45de1)












Nota: 2017-03-30.
Los directorios 32 y 64 tenian muchos archivos en identicos.
Esto fueron copiados al directorio common. Al momento de instalar
combinar el directorio common con el 32 o 64.
Fin Nota.



1. ESTRUCTURA DEL ARCHIVO

# directorio principal del archiving

arch_cli->..
           .
           .
           .
# archivos y directorios relacionados con la instalacion
# de exim    
           .
           /exim->..
                   .
                   .
                   .
# librerias de la cuales depende exim para
# ser portable.
                   .
                   /exim-lib 
                   .
                   .
                   .
# archivo que define los dominios que tienen archiving
# con la informacion adicional
                   .
                   archiving_domain_data
                   .
                   .
                   .
# plantilla de arranque de exim
                   .
                   exim-init.template
                   .
                   .
                   .
# plantilla de configuracion de exim
                   .
                   exim.conf.template
                   .
                   .
                   .
# binario de exim compilado en 32bits
                   .
                   exim4
                   .
                   .
                   .
# header generator, es un script para generar los headers
# del correo enviado a los servidores de archiving segun
# la informacion definida en achiving_domain_data
                   header_generator.template
           .
           .
           .
# directorio con la implementacion del milter
# que sirve como conector con el exim4
           .
           /milter->..
                     .
                     .
                     .
# implementacion del milter
                     .
                     mailforward.c
                     .
                     .
                     .
# script de arranque del milter
                     .
                     milter-init.template
           .
           .
           .
# script de arranque de exim y miter, internamente llama a
# a cada uno, exim-init, milter-init
           .
           arch-init.template
           .
           .
           .
# Las definiciones del sistema de cliente del archiving
# define varias de las opciones de configuracion importantes
# que deben ser revisadas y cambiadas segun haga falta.
# Ver seccion 2.
           .
           Makefile.defs 



2. CONFIGURACION DEL INSTALADOR
	Toda la configuracion para el cliente de archiving
es manejada en el archivo Makefile.defs. Este define varias
macros que son usadas tando por la utilidad "make" como para
instanciar los archivos de template antes de su instalacion

	Las optiones que pueden definirse son las siguientes.

  ARCH_CLI_SYSTEM_USER - aca se define el usuario del sistema
operativo a nombre del cual va a ejecutar el servicio de 
exim. Ese usuario tambien sera el propietario de los archivos 
que son creados por el proceso, por ejemplo "mainlog", 
"paniclog" a parte de los archivos temporales de la cola de 
correos.

  ARCH_DIR - El directorio raiz donde se instalaran los 
componentes del archiving.

  EXIM_SPOOL_DIR - El directorio donde sera alojada la cola de 
los correos.

  EXIM_CONF_DIR - Directorio donde estara alojada toda la 
configuracion de exim.

  EXIM_LOG_DIR - Directorio donde estaran alojados los logs que
va generand el exim.

  EXIM_PORT - El puerto en el cual escuchara el servicio de exim.

  EXIM_LIB - Edirectorio donde estaran almacenadas las labrerias
que son necesarias para la ejecucion de exim.

  EXIM_INTERFACE - Especificar la direccion ipv4 en la cual exim
escuchara.

  MILTER_DIR - Directorio donde se instalara el binario del 
milter.

  MILTER_SOCKET - Ruta absoluta donde sera creado el archivo
socket el cual sera usado para pasar informacion desde postfix al
milter, el cual a su vez transferira el correo al exim.

  MILTER_USER - El usuario del sistema operativo. Es el propietario
del archivo socket definido por MILTER_SOCKET. Tiene que ser el 
mismo usuario que es usado por el proceso postfix. En caso 
contrario postfix no podra pasar informacion al milter.


3. PASOS DE LA INSTALACION.

	Todos los pasos de la instalacion deben ejecutarse como 
root. Los pasos asumen que estan instalados los siguentes
componentes.

a) libmilter-dev (v 8.14.3 o superior)
b) make
c) linux-headers (para el kernel actual)
d) make


- Asegurarse que esta creado el usuario postfix, deberia estarlo
si esta instalado.

- Crear el usuario para archiving
  se tiene que crear el usuario del sistema tal cual es definido en 
  Makefile.defs por medio de la macro ARCH_CLI_SYSTEM_USER
# adduser archiving

- Descomprimir el contenido del archivo arch_cli.zip en un directorio
de trabajo, por ejemplo /usr/local/archiving

# mkdir -p /usr/local/archiving
# cd /usr/local/archiving
# unzip arch_cli.zip .

- Compilar e instanciar los templates.
# make

- Instalar el cliente de archiving. Esto lo instalara segun los 
parametros especificados en el archivo Makefile.defs, ver punto 2.
# make install


4. CONFIGURACION DE LOS DOMINIOS PARA ARCHIVADO

	Toda la informacion relacionada a los dominios afectados 
para el archivado es mantenida en el archivo 
"archiving_domain_data" el cual se instalara en el directorio
definido por EXIM_CONF_DIR.

	La estructura del archivo es la siguiente. En cada linea
se configura un dominio para el cual se activa el archivado de 
correo. Los datos para el dominio tiene que estar separados por :.
La semantica es.
DOMAIN:AUTH_USER:AUTH_PASS:HOST:PORT:LOCAL_PART

DOMAIN - es el dominio afectado por el archivado.

AUTH_USER - es el nombre del usuario con el cual exim va intentar
autenticarse. Usando alguno de los siguientes metodos: plain, login,
cram-md5. Si el servidor destino definido por HOST tiene activado TLS
se va a iniciar la negociacion TLS antes de la autenticacion.

AUTH_PASS - es el password para el usuario AUTH_USER.

HOST - nombre o ip del servidor donde sera enviado el correo para su
archivado

PORT - numero del puerto en el cual se va hacer la conexion usando 
nombre o direccion ip definida por HOST.

LOCAL_PART - parte local del comando MAIL FROM emitida por exim durante
el flujo SMTP, la direccion final sera de la forma LOCAL_PART@DOMAIN.

Una vez definido ese archivo se tiene que reiniciar el cliente de archiving
para que exim tire los datos que pueda tener en la cache y lea los nuevos.
Esto se logra parando y arrancando lo nuevamente. Ver el punto 4, para saber
como hacerlo.

Cuando se tiene la necesidad de agregar/eliminar nuevos dominios a ser 
archivados se tiene que reeditar el archivo de dominios mencionado arriba, 
y reiniciar el cliente de archiving nuevamente.


5. CONFIGURACION DE POSTFIX
	Una vez instalado el cliente de archiving y configurados los dominios
a archivar se tiene que configurarse el postfix del cliente. A continuacion se
detalla como hacerlo.

	Modificar el archivo de configuracion de postfix:
/etc/postfix/main.cf y agregar la siguiente linea:

smtpd_milters = unix:/var/run/mailf.sock

	Tener en cuenta lo siguiente, los servicios configurados en 
postfix puden ejecutarse en lo que se llama una jaula (chroot). En caso
de que el servicio corra en la jaula las rutas que son especificadas en 
smtp_milters son relativas al directorio spool de postfix. El directorio
spool por lo general es /var/spool/postfix, y esto significa que la ruta
hacia el arvhico socket es /var/spool/postfix/var/run/mailf.sock.
	Si el servicio NO se ejecuta en una jaula entonces el servicio
postfix usara la siguiente ruta para el socket /var/run/mailf.sock
	Esto es importante saberlo para configurar correctamente la macro
MILTER_SOCKET en Makefile.defs

Observacion para RedHat+Zimbra, el archivo de configuracion de postfix se 
encuentra en “/opt/zimbra/postfix/conf/main.cf”. En este caso, tambien 
debe ser agregada la siguiente opcion de configuracion. 
milter_protocol = 2

La regla en general a seguir es la siguiente
* Si version(Postfix) ? 2.6 entonces milter_protocol = 6
* Si 2.3 ? version(Postfix) ? 2.5 entonces milter_protocol = 2

Por defecto la version del protocolo que es usada para dialogar con los
milters es 6, en este caso se puede directamente omitir la misma.
En caso de otras distribuciones hay que obtener la version del postfix y
en base alla definir la version del protocolo a usar.

	Crear el directorio "var/run" dentro del directorio spool de postfix
SI Y SOLO SI el servicio smtpd del mismo corre en una jaula, en caso contrario
crearlo en la raiz del sistema de arvhivos. Para el directorio creado establecer el
propietario a postfix.root y permisos 755.

# chown -R postfix.root <spool>/var ; chmod 755 -R <spool>/var
Reemplazando <spool> por la ruta que corresponda. Por ejemplo 
/var/spool/postfix para el caso de Debian.

Si el servicio NO corre en una jaula (chroot) el paso de la creacion de "var/run"
debe ser omitido.


5.1 CONFIGURACION DE SENDMAIL

	La configuracion de sendmail por lo general se encuentra en el sig.
directorio /etc/mail y consta de varios archivos. En nuestro caso tenemos que 
editar el archivo sendmail.mc agregando el siguiente contenido al final del mismo.

INPUT_MAIL_FILTER(`mailforward', `S=unix:/var/lib/sendmail/mailf.sock, T=S:1m;R:1m;E:3m')dnl

Despues de la edicion del archivo reiniciar el servicio de sendmail.

 /etc/init.d/sendmail restart

Prestart atencion a la ruta donde se encuentra el archivo socket. En el caso de Debian 
es /var/lib/sendmail. Ademas de la ruta los duenos del mismo son "smmta.smmsp". Esto podria
variar en otras distribuciones, la informacion relevante es 

 * "donde esta el directorio spool de sendmail?" 
 * "como son los duenos para los archivos ubicados en ese directorio?"

Una vez determinada esa informacion tenemos que modificar el archivo Makefile.defs
y establecer las macros "MILTER_SOCKET", "MILTER_USER" y dejar los valores que corresponda.

Para el caso de Debian son como sigue.

MILTER_SOCKET=/var/lib/sendmail/mailf.sock
MILTER_USER=smmta

Luego de la edicion del archivo. Ejecutamos 
make clean install


6. USO DEL CLIENTE DE ARCHIVING
	Luego de la instalacion del arch_cli los servicios son arrancados
y parados por el script arch-init. Ubicado en el directorio ARCH_DIR 
definido en el archivo Makefile.defs (ver punto 2). A modo de ejemplo
/opt/arch_cli/arch-init.

/opt/arch_cli/arch-init start - arrancara los servicios
/opt/arch_cli/arch-init stop - parara los servicios


7. LOGS
	Los logs que se disponen para ver la operacion del cliente de
archivado son los logs de exim, los cuales se encuentran en el directorio
definido por EXIM_LOG_DIR, por defecto es /opt/arch_cli/var/log/exim

                

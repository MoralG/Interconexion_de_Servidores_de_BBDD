# Interconexión de Servidores de Bases de Datos

#### Las interconexiones de servidores de bases de datos son operaciones que pueden ser muy útiles en diferentes contextos. Básicamente, se trata de acceder a datos que no están almacenados en nuestra base de datos, pudiendo combinarlos con los que ya tenemos.

#### En esta práctica veremos varias formas de crear un enlace entre distintos servidores de bases de datos.

#### Los servidores enlazados siempre tendrán que estar instalados en máquinas diferentes.

### 1. Enlace entre Servidor ORACLE y Servidor ORACLE
-----------------------------------------------------------------
#### Realizar un enlace entre dos servidores de bases de datos ORACLE, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.
  
###### Para enlazar un Cliente Oracle con dirección '172.22.8.142' a un Servidor Oracle con dirección '172.22.7.59/16', tenemos que realizar unas configuraciones en cada una de las partes para que realice el enlace. Estas configuraciones se explicarán a continuación:

##### Configuración Servidor Oracle

###### En el Servidor 2 tendremos que crear un usuario con los privilegios que deseemos, ya que accedemos a través de dicho usuario, desde el cliente, a la base de datos. En este caso utilizaremos el usuario 'paloma' que tiene privilegios para ver sus tablas creadas.

##### Configuración Cliente Oracle

###### Vamos a modificar el fichero '/opt/oracle/product/12.2.0.1/dbhome_1/network/admin/tnsnames.ora'

> * Si no esta en la dirección indicada, tendremos que crearlo nosostros.

~~~
sudo nano /opt/oracle/product/12.2.0.1/dbhome_1/network/admin/tnsnames.ora
~~~

###### Añadimos las siguientes lineas en dicho fichero con las caracteristicas que hacen referencia al Servidor Oracle al que nos vamos a enlazar y añadimos la información refente a nuestro servidor si no teniamos el fichero creado.

~~~
LISTENER_ORCL =
 (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))

ORCL =
 (DESCRIPTION = Mi Servidor Oracle
    (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
    (CONNECT_DATA =
        (SERVER = DEDICATED)
        (SERVICE_NAME = orcl)
    )
 )

OraclePaloma =
 (DESCRIPTION = Servidor Oracle de Paloma
    (ADDRESS = (PROTOCOL = TCP)(HOST = 172.22.7.59)(PORT = 1521))
    (CONNECT_DATA =
        (SERVER = DEDICATED)
        (SERVICE_NAME = orcl)
    )
 )
~~~

> * ADDRESS: Especificamos el protocolo, la dirección y el puerto de la máquina que queremos conectarnos.
>
> * SERVER = DEDICATED: Para crear un proceso del servidor para atender solo a la conexión indicada 
>
> * SERVICE_NAME: Especificamos el nombre del sevicio de la máquina que queremos conectarnos. Este se especifica en el parámetro CONNECT_DATA.

###### Ahora tenemos que reiniciar el servicio para que se apliquen los cambios:

~~~
lsnrctl stop
lsnrctl start
~~~

###### Ya solo queda crear un enlace en nuestro Servidor Oracle para que pueda enlazarse al Servidor Oracle 2. Para crear el link se realiza de la siguiente forma:

~~~
create database link ConexionPaloma
connect to paloma
identified by paloma
using 'orcl';
~~~

> * Indicamos el nombre del link 'ConexionPaloma' para que se conecte al usuario 'paloma', con la contraseña 'paloma' usando el nombre del servicio 'orcl'

###### Comprobamos el enlace, realizando un select a una de la tablas de paloma:

~~~
SQL> select * from aspectos@ConexionPaloma;

    COD DESCRIPCION 				       IMPORTAN
    --- ---------------------------------- --------
    SAB Sabor.					           Muy alta
    COL Color					           Baja
    TEX Textura					           Alta
    VOL Volumen					           Media
    CAN Cantidad					       Alta
    PRE Presentacion				       Alta
    TEC Tecnica					           Media
    ORI Originalidad				       media

    8 filas seleccionadas.
~~~

> * La consulta nos muestra todos los campos de la tabla 'aspectos' al hacer la conexión por el enlace 'ConexionPaloma'

### 2. Enlace entre Servidor POSTGRES y Servidor POSTGRES
------------------------------------------------------------------

#### Realizar un enlace entre dos servidores de bases de datos Postgres, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.

###### Vamos a realizar un enlace entre un Servidor de Postgres con la dirección '192.168.43.141/24' y un Cliente de Postgres con la dirección '192.168.43.66/24' 

##### Configuración del Servidor Postgres

###### Modificamos el fichero '/etc/postgresql/11/main/postgresql.conf', descomentando una línea y añadiendo la dirección del cliente que quiere crear el enlace para que pueda escucharlo.

~~~
listen_addresses = '192.168.43.66, localhost'
~~~

###### Reiniciamos el servicio del servidor:

~~~
sudo systemctl restart postgresql.service
~~~

##### Configuración del Cliente Postgres

###### Vamos a intalar el paquete 'postgresql-contrib' para poder utilizar los módulos con 'CREATE EXTENSION'

~~~
sudo apt install postgresql-contrib 
~~~

###### Ahora vamos a añadir un nuevo registro de autentificación en el fichero '/etc/postgresql/11/main/pg_hba.conf' indicandole la dirección del servidor y el tipo de autentificación 'md5' para todas las base de datos y usuarios.

###### La linea que hay que aladir es:

~~~
# TYPE  DATABASE        USER            ADDRESS                 METHOD

host    all             all             192.168.43.66/24        md5
~~~

###### Reiniciamos el servicio del cliente:

~~~
sudo systemctl restart postgresql.service
~~~

###### Para crear el enlace vamos a utilizar el módulo 'dblink'

~~~
CREATE EXTENSION dblink;
    CREATE EXTENSION
~~~

> * Solo los usuarios con superusuarios pueden crear extensiones. Este privilegio se asigna de con 'ALTER ROLE <name_role> WITH superuser;'

###### Ahora podemos realizar una consulta a una base de datos del Servidor de Postgres con lo siguiente:

~~~
SELECT * FROM dblink('dbname=restaurante host=192.168.43.66 user=paloma password=paloma', 
                     'select * from aspectos LIMIT 3') as aspectos (codigo varchar, descripcion varchar, importancia varchar);

     codigo | descripcion  | importancia
    --------+--------------+-------------
     COL    | Color        | Baja
     TEX    | Textura      | Alta
     VOL    | Volumen      | Media

    (3 rows)
~~~

> * dblink: Tenemos que indicarle, para realizar el enlace, varios parámetros:
> >
> > * dbname: Nombre de la base de datos del servidor.
> >
> > * host: Direción del servidor.
> > 
> > * user: Usuario con el cual nos queremos conectar al servidor.
> >
> > * select .....: Indicar una consulta.

> * Además tenemos que indicar el tipo da datos de las columnas, ya que es obligatorio.

###### Podemos utilizar dblink_connect y le indicamos los parámetros del servidor para realizar una conexión persistente con el servidor y no tener que indicar mas, durante la sesión, dichos parámetros.

~~~
SELECT dblink_connect('ConexionPaloma', 'dbname=restaurante host=192.168.43.66 user=paloma password=paloma');
~~~

> * ConexionPaloma: Nombre que se le asigna a la conexión persistente.

###### Ahora realizamos una consulta con lo indicado anteriormente:

~~~
SELECT * FROM dblink('ConexionPaloma', 'select * from aspectos LIMIT 3') as aspectos (codigo varchar, descripcion varchar, importancia varchar);

     codigo | descripcion  | importancia
    --------+--------------+-------------
     COL    | Color        | Baja
     TEX    | Textura      | Alta
     VOL    | Volumen      | Media

    (3 rows)
~~~

### 3. Enlace entre Servidor ORACLE y Servidor POSTGRES
------------------------------------------------------------------

#### Realizar un enlace entre un servidor ORACLE y otro Postgres, empleando Heterogeneus Services, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.



### 4. Enlace entre Servidor ORACLE y Servidor MySQL
------------------------------------------------------------------

#### Realizar un enlace entre un servidor ORACLE y otro MySQL, empleando Heterogeneus Services, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.
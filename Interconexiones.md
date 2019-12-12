# Interconexión de Servidores de Bases de Datos

#### Las interconexiones de servidores de bases de datos son operaciones que pueden ser muy útiles en diferentes contextos. Básicamente, se trata de acceder a datos que no están almacenados en nuestra base de datos, pudiendo combinarlos con los que ya tenemos.

#### En esta práctica veremos varias formas de crear un enlace entre distintos servidores de bases de datos.

#### Los servidores enlazados siempre tendrán que estar instalados en máquinas diferentes.

### 1. Enlace entre Servidor ORACLE y Servidor ORACLE
-----------------------------------------------------------------
#### Realizar un enlace entre dos servidores de bases de datos ORACLE, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.
  
##### Para enlazar un Servidor Oracle a otro Servidor Oracle tenemos que realizar unas configuraciones en cada una de ellas para que realice el enlace. Estas configuraciones se explicarán a continuación:

##### Configuración Servidor Oracle 1

###### Lo primero que vamos a hacer es modificar el fichero '/opt/oracle/product/12.1.0.2/dbhome_1/network/admin/listener.ora' para determinar el Host y el Puerto.

###### Modificación del fichero 'listener.ora' 
~~~
LISTENER =
 (DESCRIPTION_LIST =
 (DESCRIPTION =
 (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))
 (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
 )
 )
~~~

> En el fichero anterior podemos ver como es nombrado nuestro Servidor Oracle en la linea de 'GLOBAL_DBNAME = OracleAlejandro'. Si queremos cambiar dicho parametro, tendremos que hacer lo siguiente:

~~~
SQL> update global_name set global_name = 'OracleAlejandro';

SQL> select * from global_name;

    GLOBAL_NAME
    --------------------------------------------------------------------------------
    OracleAlejandro
~~~

###### Ahora vamos a modificar el fichero '/opt/oracle/product/12.1.0.2/dbhome_1/network/admin/tnsnames.ora'

> Si no esta en la dirección indicada, tendremos que crearlo nosostros con el mismo nombre.

~~~
sudo nano /opt/oracle/product/12.2.0.1/dbhome_1/network/admin/tnsnames.ora
~~~

###### Añadimos las siguientes lineas en dicho fichero con las caracteristicas que hacen referencia al Servidor Oracle al que nos vamos a enlazar y añadimos nuestro Servidor Oracle si no teniamos el fichero creado.

~~~
LISTENER_OracleAlejandro =
 (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))

ORCL =
 (DESCRIPTION = Mi Servidor Oracle
 (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
 (CONNECT_DATA =
 (SERVER = DEDICATED)
 (SERVICE_NAME = orcl)
 )
 )

ORCL =
 (DESCRIPTION = Servidor Oracle de Paloma
 (ADDRESS = (PROTOCOL = TCP)(HOST = 172.22.0.244)(PORT = 1521))
 (CONNECT_DATA =
 (SERVER = DEDICATED)
 (SERVICE_NAME = orcl)
 )
 )
~~~

###### Ahora tenemos que reiniciar el servicio para que se apliquen los cambios:

~~~
lsnrctl stop
lsnrctl start
~~~

##### Configuración Servidor Oracle 2

###### Ahora tenemos que añadir las mismas lineas al fichero '/opt/oracle/product/12.1.0.2/dbhome_1/network/admin/listener.ora' del Servidor Oracle al que queremos enlazar el nuestro

~~~
LISTENER =
 (DESCRIPTION_LIST =
 (DESCRIPTION =
 (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))
 (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
 )
 )
~~~

###### Reiniciamos el sevicio en en el segundo servidor:

~~~
lsnrctl stop
lsnrctl start
~~~

##### Crear enlace en Servidor Oracle 1

###### Ya solo queda crear un enlace en nuestro Servidor Oracle para que pueda enlazarse al Servidor Oracle 2. Para realizar esto, utilizamos el siguiente comando:

~~~
create database link ConexionPaloma1
connect to paloma
identified by paloma
using 'orcl';
~~~

create database link ConexionPaloma2
connect to paloma
identified by paloma
using 'mydba';

create database link ConexionPaloma3
connect to paloma
identified by paloma
using 'mydb';

sql paloma/paloma@172.22.0.244/oraDB

select * from cat@ConexionPaloma1;

### 2. Enlace entre Servidor POSTGRES y Servidor POSTGRES
------------------------------------------------------------------

#### Realizar un enlace entre dos servidores de bases de datos Postgres, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.



### 3. Enlace entre Servidor ORACLE y Servidor POSTGRES
------------------------------------------------------------------

#### Realizar un enlace entre un servidor ORACLE y otro Postgres, empleando Heterogeneus Services, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.



### 4. Enlace entre Servidor ORACLE y Servidor MySQL
------------------------------------------------------------------

#### Realizar un enlace entre un servidor ORACLE y otro MySQL, empleando Heterogeneus Services, explicando la configuración necesaria en ambos extremos y demostrando su funcionamiento.


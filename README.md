# Practica 18 - Internet Of Things

![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/diagrama.png)

Necesitaremos una máquina con mínimo 4GB de memoria con los puertos 8888, 3000, y 1883 y 8086 (UDP y TCP)abiertos.
Necesitaremos tener instalado Docker y Docker Compose para realizar esta práctica, si no lo tenemos hacemos lo siguiente:
```
- sudo apt update
- sudo apt install docker docker-compose -y
- sudo systemctl enable docker
- sudo systemctl start docker
- sudo usermod -aG docker $USER
- newgrp docker
```

# 1. MQTT Broker
MQTT (MQ Telemetry Transport) es un protocolo de mensajería estándar utilizado en las aplicaciones de Internet de las cosas (IoT). Se trata de un protocolo de mensajería muy ligero basado en el patrón publish/subscribe, donde los mensajes son publicados en un topic de un MQTT Broker que se encargará de distribuirlos a todos los suscriptores que se hayan suscrito a dicho topic.

Eclipse Mosquitto será el MQTT Broker que vamos a utilizar en este proyecto.

## 1.1 Descripción del servicio en `docker-compose.yml`
### `docker-compose.yml`
```
  mosquitto: (1)
    image: eclipse-mosquitto:2 (2)
    ports:
      - 1883:1883 (3)
    volumes:
      - ./mosquitto/mosquitto.conf:/mosquitto/config/mosquitto.conf (4)
      - mosquitto_data:/mosquitto/data (5)
      - mosquitto_log:/mosquitto/log (6)
```
1. Nombre del servicio dentro del archivo `docker-compose.yml`.
2. Vamos a utilizar la versión 2 de la imagen oficial eclipse-mosquitto.
3. El broker MQTT estará aceptando peticiones en el puerto 1883.
4. Creamos un volumen de tipo bind mount para enlazar el archivo de configuración `mosquitto.conf` que tenemos en nuestro directorio local `mosquitto` con el directorio `/mosquitto/config/mosquitto.conf` del broker MQTT.
5. Creamos un volumen para almacenar los mensajes que se reciben.
6. Creamos un volumen para almacenar los mensajes de log.


## 1.2 Archivo de configuración mosquitto.conf
Este archivo estará almacenado en el directorio `./mosquitto/mosquitto.conf` de nuestra máquina local. Como vamos a utilizar la versión 2 del broker MQTT eclipse-mosquitto vamos a necesitar crear un archivo de configuración llamado `mosquitto.conf` que incluya las siguientes opciones de configuración.

### `mosquitto.conf`
```
listener 1883 (1)
allow_anonymous true (2)
```
1. En esta línea estamos configurando el listener para que acepte conexiones desde cualquiera de sus interfaces de red, es equivalente a poner `listener 1883 0.0.0.0.` Si sólo quisiéramos recibir conexiones por una interfaz de red específica se podría configurar aquí de forma manual.
2. A partir de la versión 2 es necesario indicar un método de autenticación para el *listener*. Con esta configuración vamos a permitir conexiones desde cualquier cliente. Aquí sería posible definir un método de autenticación basado en contraseña para restringir el acceso al broker.

En este punto, el contenido de nuestro directorio de trabajo debe tener los siguientes archivos:

![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/mosquitto.PNG)

## 1.3 Cliente MQTT para publicar en un topic (*publish*)
En esta sección vamos a explicar cómo podemos hacer uso de un cliente MQTT para publicar mensajes de prueba en el broker MQTT que acabamos de crear.

Esta comprobación deberíamos realizarla antes de empezar a trabajar directamente con los sensores sobre el broker MQTT. Una vez que hayamos comprobado que podemos publicar y suscribirnos a los mensajes del broker con este cliente, ya podríamos realizar el siguiente paso para publicar mensajes con los sensores y recibirlos con el agente de Telegraf.

**Ejemplo:**
```
docker run --init -it --rm efrecon/mqtt-client sub -h test.mosquitto.org -t "iescelia/aula22/temperature"
```

Explicación de los parámetros utilizados:
```
docker run --init -it --rm efrecon/mqtt-client \ (1)
pub \ (2)
-h test.mosquitto.org \ (3)
-p 1883 \ (4)
-t "iescelia/aula22/temperature" \ (5)
-m 30 (6)
```
1. Utilizamos la imagen Docker efrecon/mqtt-client que contiene el cliente MQTT (`mosquitto_pub`) para publicar mensajes en un topic de un broker MQTT.
2. Utilizamos el comando `pub` para publicar un mensaje en el broker MQTT.
3. Con el parámetro `-h` indicamos el hosts (broker MQTT) con el que queremos conectarnos. En este ejemplo estamos utilizando el broker de prueba de `test.mosquitto.org`, pero **tendremos que cambiar este broker por la dirección IP de nuestro broker MQTT.**
4. Con el parámetro `-p` indicamos el puerto, que por defecto, será el puerto `1883`.
5. Con el parámetro `-t` indicamos el topic que vamos a utilizar para publicar nuestro mensaje. En este ejemplo, estamos utilizando el topic `iescelia/aula22/temperature`.
6. Con el parámetro -m indicamos el contenido del mensaje que queremos publicar. En este ejemplo, estamos publicando el mensaje 30.

En mi caso sería: `sudo docker run --init -it --rm efrecon/mqtt-client pub -h 34.232.80.233 -p 1883 -t "iescelia/aula22/co2" -m 30`

![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/pub.PNG)

## 1.4 Cliente MQTT para suscribirse a un topic (*subscribe*)
En esta sección vamos a explicar cómo podemos hacer uso de un cliente MQTT para publicar mensajes de prueba en el broker MQTT que acabamos de crear.

Esta comprobación deberíamos realizarla antes de empezar a trabajar directamente con los sensores sobre el broker MQTT. Una vez que hayamos comprobado que podemos publicar y suscribirnos a los mensajes del broker con este cliente, ya podríamos realizar el siguiente paso para publicar mensajes con los sensores y recibirlos con el agente de Telegraf.

**Ejemplo:**
```
docker run --init -it --rm efrecon/mqtt-client sub -h test.mosquitto.org -t "iescelia/aula22/temperature"
```

Explicación de los parámetros utilizados:
```
docker run --init -it --rm efrecon/mqtt-client \ (1)
sub \ (2)
-h test.mosquitto.org \ (3)
-t "iescelia/aula22/temperature" (4)
```
1. Utilizamos la imagen Docker efrecon/mqtt-client que contiene el cliente MQTT (`mosquitto_sub`) para suscribirse a un topic de un broker MQTT.
2. Utilizamos el comando ``sub` para suscribirnos a un topic del broker MQTT.
3. Con el parámetro `-h` indicamos el hosts (broker MQTT) con el que queremos conectarnos. En este ejemplo estamos utilizando el broker de prueba de `test.mosquitto.org`, pero **tendremos que cambiar este broker por la dirección IP de nuestro broker MQTT.**
4. Con el parámetro `-t` indicamos el topic al que vamos a suscribirnos. En este ejemplo, estamos utilizando el topic `iescelia/`. Con el *wildcard* estamos indicando que queremos suscribirnos a todos los topics que existan dentro de `iescelia/`, es decir, todos los mensajes que se publiquen en cada una de las aulas. También sería posible suscribirnos al topic específico de un aula, por ejemplo: `iescelia/aula22/temperature`.

En mi caso sería:
```
sudo docker run --init -it --rm efrecon/mqtt-client sub -h 34.232.80.233 -t "iescelia/aula22/co2"
```

![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/sub.PNG)

# 2. Telegraf
Telegraf es un agente que nos permite recopilar y reportar métricas. Las métricas recogidas se pueden enviar a almacenes de datos, colas de mensajes o servicios como: InfluxDB, Graphite, OpenTSDB, Datadog, Kafka, MQTT, NSQ, entre otros.

## 2.1 Descripción del servicio en `docker-compose.yml`
### `docker-compose-yml`
```
  telegraf: (1)
    image: telegraf (2)
      volumes:
        - ./telegraf/telegraf.conf:/etc/telegraf/telegraf.conf (3)
      depends_on: (4)
        - influxdb
```
1. Nombre del servicio dentro del archivo `docker-compose.yml`.
2. Utilizamos la imagen Docker telegraf.
3. Creamos un volumen de tipo bind mount para enlazar el archivo de configuración `telegraf.conf` que tenemos en nuestro directorio local `telegraf` con el archivo `/etc/telegraf/telegraf.conf` del contenedor.
4. Indicamos que este servicio depende del servicio influxdb y que no podrá iniciarse hasta que el servicio de influxdb se haya iniciado.

## 2.2 Creación del archivo de configuración `telegraf.conf`
En primer lugar tenemos que crear el archivo de configuración `telegraf.conf` en nuestra máquina local. Para crear el archivo de configuración haremos uso del comando `telegraf config` dentro del contenedor de Telegraf.
```
docker run --rm telegraf telegraf config > telegraf.conf
```
Una vez que hemos creado el archivo telegraf.conf lo movemos al directorio telegraf de nuestro proyecto. El contenido de nuestro directorio de trabajo debe tener los siguientes archivos.

![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/telegraf.PNG)

## 2.3 Configuración del archivo `telegraf.conf` para suscribirnos a un topic MQTT (`inputs.mqtt_consumer`)
Tendremos que buscar la sección `inputs.mqtt_consumer` dentro del archivo `telegraf.conf` y configurar los siguientes valores:
- servers
- topics
- data_format

Existen más directivas de configuración, pero en este proyecto sólo vamos a utilizar los valores que hemos indicado anteriormente.

### 2.3.1 servers
![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/servers.PNG)

En esta directiva de configuración indicamos la URL del broker MQTT al que queremos conectarnos. En nuestro caso pondremos el nombre del servicio `mosquitto` que es como lo hemos definido en nuestro archivo `docker-compose.yml`.


### 2.3.2 topics
![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/topics.PNG)

En esta directiva indicamos los topics a los que queremos suscribirnos. En nuestro caso pondremos el topic `iescelia/#`. El carácter `#` al final del topic indica que nos vamos a suscribir a cualquier topic que exista después de la cadena `iescelia/`.

### 2.3.3 data_format
![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/format.PNG)

En esta directiva indicamos cómo es el formato de los datos que vamos a recibir por MQTT.

Telegraf contiene muchos plugins de propósito general que permiten parsear datos de entrada para convertirlos en el modelo de datos de métricas que utiliza InfluxDB.

Los mensajes de entrada que recibe Telegraf pueden estar en los siguientes formatos:
- Value
- InfluxDB Line Protocol
- Collectd
- CSV
- Dropwizard
- Graphite
- Grok
- JSON
- Logfmt
- Nagios
- Wavefront

En este proyecto sólo vamos a estudiar Value.

## 2.4 Formato: `Value`
El formato value permite convertir valores simples en métricas de Telegraf.

Es necesario indicar el tipo de dato al que queremos convertir el valor leído, utilizando la opción de configuración data_type. Los tipos de datos disponibles son:
- integer
- float o long
- string
- boolean

Si los mensajes que vamos a recibir por MQTT sólo contienen valores numéricos de tipo real que representan valores de temperatura, tendríamos que indicar:
- data_format = "value"
- data_type = "float"

Una posible configuración para la sección`inputs.mqtt_consumer` del archivo `telegraf.conf` podría ser la siguiente.
```
 [[inputs.mqtt_consumer]]
   servers = ["tcp://mosquitto:1883"] (1)

topics = [
"iescelia/#" (2)
]

data_format = "value" (3)
data_type = "float" (4)
```

1. Indicamos la URL del broker MQTT al que queremos conectarnos. En nuestro caso pondremos el nombre del servicio `mosquitto` que es como lo hemos definido en nuestro archivo `docker-compose.yml`.
2. Indicamos los topics a los que queremos suscribirnos. El carácter `#` al final del topic `iescelia/#` indica que nos vamos a suscribir a cualquier topic que haya después de `iescelia/`.
3. Indicamos cuál es el formato de los datos que vamos a recibir por MQTT. En este caso indicamos el formato `value`, porque en el mensaje MQTT sólo vamos a recibir un valor numérico.
4. Indicamos el tipo de dato del valor numérico que vamos a recibir por MQTT.

## 2.5 Configuración del archivo `telegraf.conf` para almacenar los datos en InfluxDB (`outputs.influxdb`)
Una posible configuración de la sección outputs.influxdb podría ser la siguiente:
```
[[outputs.influxdb]]
  urls = ["http://influxdb:8086"] (1)

database = "iesceliadb" (2)

skip_database_creation = true (3)

username = "root" (4)
password = "root"
```

1. Indica la URL de la instancia de InfluxDB. En nuestro caso indicamos el nombre `influxdb` que es el nombre que le hemos puesto al servicio en el archivo `docker-compose.yml`
2. Indica el nombre de la base de datos donde vamos a almacenar las métricas.
3. Si  está a `true` no se crearán bases de datos en InfluxDB. Se recomienda configurarlo a `true` cuando estemos trabajando con usuarios que no tienen permisos para crear bases de datos o cuando la base de datos ya exista.
4. Los parámetros de autenticación para conectarnos a InfluxDB.

# 3. InfluxDB
InfluxDB es un sistema gestor de bases de datos diseñado para almacenar bases de datos de series temporales (TSBD - Time Series Databases). Estas bases de datos se suelen utilizar en aplicaciones de monitorización, donde es necesario almacenar y analizar grandes cantidades de datos con marcas de tiempo, como pueden ser datos de uso de cpu, uso memoria, datos de sensores de IoT, etc.

## 3.1 Descripción del servicio en `docker-compose.yml`
### `docker-compose.yml`
```
  influxdb: (1)
    image: influxdb (2)
    ports:
      - 8086:8086 (3)
    volumes:
      - influxdb_data:/var/lib/influxdb (4)
    environment:
      - INFLUXDB_DB=iescelia_db (5)
      - INFLUXDB_ADMIN_USER=root (6)
      - INFLUXDB_ADMIN_PASSWORD=root (7)
```

1. Nombre del servicio dentro del archivo `docker-compose.yml`.
2. Utilizamos la imagen Docker influxdb.
3. Este servicio utilizará el puerto `8086` de nuestra máquina local para enlazarlo con el puerto `8086` el contenedor.
4. Creamos un volumen con el nombre `influxdb_data` que estará enlazado con el directorio `/var/lib/influxdb` del contenedor.
5. Nombre de la base de datos.
6. Usuario de la base de datos.
7. Contraseña del usuario de la base de datos.
8. Habilitamos la autenticación básica HTTP.

## 3.2 Conectar a InfluxDB desde un terminal
Podemos conectarnos al contenedor de InfluxDB desde un terminal para comprobar si los datos que estamos recogiendo de los sensores se están insertando de forma correcta en la base de datos.

En primer lugar necesitamos obtener el `ID` del contenedor que se está ejecutando con InfluxDB. Para obtener el listado de todos los contenedores que están en ejecución podemos ejecutar el siguiente comando:
```
docker ps
```
Ahora sólo tenemos que buscar el `ID` del contenedor con InfluxDB entre la lista de contenedores. En el ejemplo que se muestra a continuación sería el valor `27b06d552719`.
```
CONTAINER ID        IMAGE                 COMMAND                  ...
27b06d552719        influxdb              "/entrypoint.sh infl…"   ...
...
```

Una vez que tenemos el `ID` del contenedor de InfluxDB, creamos un nuevo proceso con el comando `/bin/bash` dentro del contenedor para obtener un terminal que nos permita interactuar con el contenedor.
```
docker exec -it 27b06d552719 /bin/bash
```

Una vez que tenemos un terminal en el contenedor de InfluxDB, utilizamos el cliente `influx` para conectarnos al sistema gestor de bases de datos con el usuario y contraseña que hemos creado en el archivo `docker-compose.yml`.
```
influx -username root -password root
```

Después de autenticarnos, tendremos acceso al shell de InfluxDB donde podemos interaccionar con la base de datos con sentencias InfluxQL (Influx Query Language), que tienen una sintaxis muy similar a SQL.

A continuación, se muestra una secuencia de sentencias InfluxQL (Influx Query Language) que podemos utilizar para comprobar si los datos de los sensores se están almacenando en la base de datos.

**Consultar el listado de bases de datos.**
```
> show databases

name
----
iescelia_db
```

**Seleccionar la base de datos `iescelia_db`**
```
> use iescelia_db
```

**Mostrar las tablas que existen para la base de datos seleccionada.**
```
> show measurements

name
----
cpu
disk
diskio
kernel
mem
mqtt_consumer
processes
swap
system
```

**Mostrar todos los datos que existen en la tabla `mqtt_consumer`.**
```
> select * from mqtt_consumer

time                host         topic                       value
----                ----         -----                       -----
1612791727450709588 3f4be32bd18b iescelia/aula22/co2         30
1612791734718611922 3f4be32bd18b iescelia/aula22/co2         30
```

# 4. Grafana
Grafana es un servicio web que nos permite visualizar en un panel de control los datos almacenados en InfluxDB y otros sistemas gestores de bases de datos de series temporales.

## 4.1 Descripción del servicio en `docker-compose.yml`
### `docker-compose.yml`
```
grafana: (1)
    image: grafana/grafana:5.4.3 (2)
    ports:
      - 3000:3000 (3)
    volumes:
      - grafana_data:/var/lib/grafana (4)
    depends_on: (5)
      - influxdb
```

![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/grfana1.PNG)

1. Nombre del servicio dentro del archivo `docker-compose.yml.`
2. Utilizamos la imagen Docker grafana.
3. Este servicio utilizará el puerto `3000` de nuestra máquina local para enlazarlo con el puerto `3000` el contenedor.
4. Creamos un volumen con el nombre `grafana_data` que estará enlazado con el directorio `/var/lib/grafana` del contenedor.
5. Indicamos que este servicio depende del servicio `influxdb` y que no podrá iniciarse hasta que el servicio de `influxdb` se haya iniciado.

## 4.2 Configuración de un data source
Grafana permite utilizar diferentes data sources, en nuestro proyecto utilizaremos InfluxDB pero también es posible utilizar AWS CloudWatch, Elasticsearch, MySQL o PostgreSQl entre otros.

### 4.2.1 Configuración de forma manual
Antes de crear un dashboard es necesario crear un data source. Sólo los usuarios que tengan rol de administrador podrán crear un data source.

Para crear un data source debe seguir los siguientes pasos:
1. Accede al menu lateral haciendo clien en el icono de Grafana del encabezado superior.
2. Accede a la sección "Data Sources".
3. Añada un nuevo data source.
4. Seleccione InfluxDB en el desplegable donde se indica el tipo de data source.
5. Configure los parámetros url, database, user y password de su servicio InfluxDB.

![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/datasource.PNG)

### 4.2.2 Configuración con aprovisionamiento automático
Es posible configurar Grafana para que utilize un archivo de configuración de forma automática para evitar tener que realizar la configuración de forma manual.

En nuestro proyecto, vamos a crear un archivo con el nombre `datasource.yml` dentro de la siguiente estructura de directorios:
![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/grafanaestruc.PNG)

El contenido del archivo `datasource.yml` será el siguiente:
```
apiVersion: 1
datasources:
  - name: InfluxDB
    type: influxdb
    access: proxy
    database: iesceliabd (1)
    user: root (2)
    password: root (3)
    url: http://influxdb:8086 (4)
    isDefault: true
    editable: true
```

1. Nombre de la base de datos InfluxDB.
2. Nombre de usuario para acceder a la base de datos.
3. Contraseña del usuario para acceder a la base de datos.
4. URL del servicio InfluxDB.

Para que el aprovisionamiento se realice forma correcta, tenemos que definir un nuevo volumen en el servicio de Grafana en el archivo `docker-compose.yml`.
### `docker-compose.yml`
```
grafana: (1)
    image: grafana/grafana:5.4.3 (2)
    ports:
      - 3000:3000 (3)
    volumes:
      - grafana_data:/var/lib/grafana (4)
      - ./grafana-provisioning/:/etc/grafana/provisioning (5)
    depends_on: (6)
      - influxdb
```
1. Nombre del servicio dentro del archivo `docker-compose.yml`.
2. Utilizamos la imagen Docker grafana.
3. Este servicio utilizará el puerto `3000` de nuestra máquina local para enlazarlo con el puerto `3000` el contenedor.
4. Creamos un volumen con el nombre `grafana_data` que estará enlazado con el directorio `/var/lib/grafana` del contenedor.
5. Creamos un volumen de tipo bind mount entre el directorio local de nuestra máquina `./grafana-provisioning/` y el directorio `/etc/grafana/provisioning` del contenedor.
6. Indicamos que este servicio depende del servicio `influxdb` y que no podrá iniciarse hasta que el servicio de `influxdb` se haya iniciado.

## 4.3 Configuración de un dashboard
A continuación se muestra una imagen de cómo se podría configurar un dashboard para Grafana.

![](https://raw.githubusercontent.com/joseean29/Practica18-IOT/main/images/grafana.PNG)

# 5. Mi sitio
**[URL DEL SITIO](http://34.232.80.233:3000)**  

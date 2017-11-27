# arduino-temperature-streaming-demo

El presente proyecto cubre el proceso de despliegue de una arquitectura simple para el procesamiento en tiempo real y batch de lecturas de un sensor de temperatura sobre arduino y con tecnologías open source del ecosistema Big Data. El propósito de la solución es ejemplificar el flujo de datos a través de las distintas herramientas, desde su captación hasta su tranformación y generación de Insights.

## Introducción

Arduino es una plataforma open source enfocada a promover y producir componentes de hardware y software faciles de usar. Las múltiples placas de Arduino son capaces de leer distintos tipos de señales de entrada, procesarlos y generar algun tipo de señal de salida. Para ello, es necesario desarrollar y enviar un conjunto de instrucciones (bajo un lenguaje de programación especifico) al microcontrolador de la placa o componente en uso mediante el IDE oficial de Arduino.

Mediante el uso de la placa de Arduino es posible realizar lecturas e interactuar con distinto sensores. En este proyecto es usado un sensor de temperatura y humedad de la familia DHT. Este tipo de sensores son bastante lentos y básicos pero cumplen con su propósito. Estan compuentos por dos componentes principales: un sensor capacitivo de humedad  un termistor. Tambien son capaces de realizar conversiones analógicas a digitales y separar la señales entre la temperatura y la humedad.

A su vez, dependiendo de la placa de Arduino que se utilice, se pueden integrar otro tipo de componentes para ampliar las funcionalidades de la misma. Por ejemplo, con respecto al envío de datos por internet se puede hacer uso de un chip de la familia ESP2866, los cuales son alternativas de bajo costo que soportan el procolo TCP/IP y poseen tanto una unidad de memoria pequeña como un Microcontrolador propio, lo cual permite que tambien se puedan cargar instrucciones en el componente.

Muchos pueden ser lo clientes que puedan recibir o consultar la información generada por una placa de Arduino. Una de las opciones mas usadas por las comunidades de desarrollo consiste en untilizar Mosquitto. Mosquitto es una implementacion open source de un broker sobre el protocolo MQTT (MQ Telemetry Transport) bastante ligera. El proyecto de Mosquitto fue desarrollado por la Eclipse Foundation e incluye librerias escritas en C y C++, ademas de las utilidades mosquitto_pub y mosquitto_sub para la publicación de mensajes y la suscripción a "tópicos". 

Sin embargo, Mosquitto que carece de muchas funciones que pueden ser de gran relevancia para un proyecto en especifico, como por ejemplo, ofrecer facilidades para almacenar y consultar la persistencia de lo tópicos. Para ello, puede optarse por redirigir los mensajes a una solución mas robusta como Apache Kafka, el cual es una proyecto open source escrito en Scala y desarrollado por la Apache Software Fundation. Kafka implementa un modelo de publicacion y subscripción para la intermediación de mensajes por medio de canales o "tópicos", ofrenciendo mas opciones a nivel de seguridad, almacenamiento, y manipulación de los mensajes y tópicos.

Para hacer la redirección de mensajes entre un servicio de publicación y otro es viable utilizar una herramienta como Apache NiFi, la cual permite armar flujos de de datos llamados "templates", los cuales se constituyen por bloques de instrucciones de (relativamente) bajo nivel llamados "procesadores" , que permiten pueden captar y manipular los datos en tiempo real, tranformarlos y redirigirlos a otros medio de persistencia como archivos en HDFS, bases de datos ú otros servicios.

Cabe destacar que resulta beneficio enviar la información a un medio de persistencia que permita realizar procesamiento en batch de la información de una forma mas tradicional. Bajo el mismo orden de ideas de herramientas open source, Apache Hive se presenta como una opción atractiva. Hive es un datawarehouse relacional que puede ser manipulado bajo una lenguage SQL-like y posee funcionalidades que lo hacen sobresalir en ambientes clusterizados y/o distribuidos, facilitando el escalamiento horitzontal del datawarehouse, la cual es una caracteristica importante en proyectos donde se podrían recibir grandes volúmenes de información.

Gran parte de este tipo de proyectos requieren la necesidad de realizar exploraciones moderadamente sencillas, interactivas y rapidas de implementar. Bajo esta premisa, las consolas o shells interactivos basados en la creación de notebooks cobran mucha utilidad. Apache Zeppelin es un proyecto en incubación que consiste en una implementación de la idea/concepto de web noteboook, el cual fue introducido por primera vez por IPython. Zeppelin esta enfocado en el desarrollo de procesos analíticos e interactivos de datos mediante tecnologías y lenguajes como Shell, SQL, Spark (Scala), Hive, Elasticsearch, R y demás.

## Equipos usados

* Arduino UNO R3 Clone - (USB Chip CH340) + Cable USB
* Sensor de humedad y tempreatura DHT11
* WiFi Shield ESP 8266 ESP-01
* Protoboard MB-102
* Resitencia 1K (DHT)
* Resistencia 10K
* Cableado

## Versiones de herramientas

* Arduino IDE 1.8.5
* Hive 1.2.1
* Kafka 0.10.0
* Spark 1.6.2
* Zeppelin Notebook 0.6.0

## Flujo de datos

![Alt text](/images/architecture.png?raw=true "Architecture Diagram")

### 1 - Generación de datos a partir del sensor de temperatura y humedad.
* El código cargado en la plataforma Arduino realiza lecturas a través del sensor de temperatura cada 3 segundos, captando:
   * Porcentaje de humedad en el ambiente.
   * Temperatura en grados Celsius (°C)
   * Temperatura en grados Fahrenheit (°F)
* Se calcula el Índice de Calor en Celsius  y Fahrenheit, el cual determina como las personas perciben la temperatura de acuerdo a la humedad del ambiente.
* Se realiza una petición a un servicio web externo para determinar la hora de la lectura de acuerdo a una zona horaria predefinida.

### 2 - Publicación de datos al servidor MQTT.
* Se construye el mensaje o “payload” que será enviado al servidor MQTT:
   * El payload será construido en formato JSON.
   * Contiene los datos capturados por el sensor, la información calculada, la fecha/hora de la lectura, la cantidad de microsegundos desde que la plataforma Arduino fue encendida y un identificador único para el cliente emisor.
* Se realizan verificaciones de conexión a internet y al servidor MQTT.
* Se realiza la publicación del payload al servidor MQTT.
   * La publicación se realiza sobre un tópico especifico bajo un nombre de usuario y contraseña predefinidos.
   * El servidor MQTT posee una lista de permisos donde define cuales usuarios pueden publicar información sobre los tópicos existentes.

### 3, 4 y 5 - Captura de datos del servidor MQTT en tiempo real .
* El servicio Apache NiFi posee conjuntos organizados de instrucciones que orquestan el flujo de datos a medida que son captados:
   * NiFi se conecta o “suscribe” al tópico del Mosquitto y captura los mensajes que llegan en tiempo real.
   * NiFi se complementa el mensaje recibido (cadena JSON) definiendo nuevos campos fuera de la cadena, relacionados a aspectos técnicos del mensaje y del servidor MQTT.
   * NiFi inserta la cadena JSON y los nuevos campos en el almacén de datos Hive.
   * NiFi publica el mensaje exactamente igual en Kafka.
* Hive y Kafka guardan de forma persistente los datos:
   * Hive hace posible realizar procesamiento en batch de los datos almacenados de forma histórica.
   * Kafka permite realizar procesamiento en tiempo real de los datos enviados por la palca Arduino.

### 6 y 7 - Procesamiento de datos.
* Zeppelin ejecuta bloques de código (en Scala y SQL):
   * Es posible consultar los datos en el almacén de datos.
   * Es posible suscribirse en tiempo real al tópico de Kafka para monitorear y procesar los datos recibidos en tiempo real bajo distintas ventanas de tiempo
* El código es ejecutado sobre el motor de consultas Spark.
* Los datos obtenidos en cada ventana de tiempo son transformados y almacenados en tablas en Hive.
* Se calcula la media de las temperaturas medidas en Farenheit sobre las ventanas de tiempo y se almacenan en Hive.

## Instalación

### Arduino IDE
El software necesario para cargar secuencias de instrucciones a la placa de Arduino se llama Arduino IDE y puede ser descargado desde la web [oficial de arduino](https://www.arduino.cc/en/Main/Software]). Puede ser decargado para Windows, Linux y Mac OS X.

#### ESP8266
Para poder manipular el shield WiFi ESP8266 es necesario instalar componentes adicionales sobre el Arduino IDE. Esto permititrá configurar las opciones de compilación del codigo que se enviará al componente y utilizar los metodos y librerías para manipular las conexiones. 
El proceso de instalación se encuentra detallado en el (repositorio oficial de ESP8266 para Arduino)[https://github.com/esp8266/Arduino.git], pero básicamente, la instalación puede llevarse a cabo mediante dos métodos:

##### Usando el Administrador de Placas de Arduino IDE
1. Iniciar el Arduino IDE y abrir la ventana de Preferencias bajo las pestaña Archivo.
2. Ingresar el valor http://arduino.esp8266.com/stable/package_esp8266com_index.json en el campo Gestor de URLs Adicionales de Tarjetas.
3. Abrir el Gestor de Tarjetas desde Herramientas > Placa e instalar la placa esp8266.

##### Usando git
1. Dirigirse al directorio de instalación de Arduino.
2. Clonar el repositorio https://github.com/esp8266/Arduino.git en hardware/esp8266com :
```bash
cd hardware
mkdir esp8266com
cd esp8266com
git clone https://github.com/esp8266/Arduino.git esp8266
```
3. Descargar los archivos binarios con Python (2.7) :
```bash
cd esp8266/tools
python get.py
```
4. Reiniciar Arduino

#### Librerías
Para manipular los sensores, realizar los procesos de publicación de información y entre otras funciones es necesario instalar librerías adicionales.

Para instalar una librería adicional es posible usar el Gestor de Librerías del Arduino IDE bajo la pestaña Programas > Incluír librería.

Tambien es posible incluir librerías descargandolas de forma local en formato .ZIP y añadiendolas desde la pestaña Programas > Incluír librería > Añadir librería .ZIP

Las adicionales usadas en este proyecto son:
* Adafruit Unified Sensor (1.0.2)
* DHT sensor library (1.3.0)
* PubSubClient (2.6.0)
* Time (1.5.0)
* NTPClient (3.1.0)

## Montaje

Antes de presentar el diagrama del montaje de componentes, es necesario determinar cuales son las entradas y salidas de cada componente y como se identifican dentro del diagrama.

### Diagrama Pinout ESP8266

* TX - Verde
* GND - Negro
* CH_PD - Naranja
* GPIO2 - Azul
* GPIO0 - Blanco
* VCC - Rojo
* RX - Amarillo
![Alt text](/images/esp8266_pinout.png?raw=true "ESP8266 Pinout Diagram")

### Diagrama Pinout DHT11

* GND - Blanco
* Data - Azul
* VCC - Naranja
![Alt text](/images/dht11_pinout.png?raw=true "ESP8266 Pinout Diagram")

### Diagrama Pinout del demo

La sección superior del protoboard se encuentra dedicada a los pines del módulo ESP8266, el cual se alimenta con voltaje de 3.3V y usa una resistencia de 10k. El flujo de voltaje al módulo WiFi es controlado por el pin verde conectado al protoboard en la ultima columna de la fila de carga positiva.

La sección inferior del protoboard esta casi completamente dedicada al sensor de temperatura DHT. Este sensor trabaja con una voltaje de 5V y una resistencia de 1k. El pin azul transfiere las señales de salida del sensor de temperatura, las cuales son capturadas por el módulo WiFi a través del pin GPIO2 (azul).

![Alt text](images/demo_pinout_fm.png?raw=true "Demo Diagram Flash Mode")

En este proyecto no se cargan instrucciones a la placa de Arduino, sino al módulo ESP8266, ya que es éste quien enviará los datos y es quien se encargará de la manipulación y transformación de los mismos. 

Para cargar instrucciones al modulo WiFi es necesario que éste entre en Flash Mode al momento de inicio, lo cual se logra a través de la configuración de pines mostrada.

## Configuración

Debido a que la carga de las intrucciones no estará dirigida a la placa de arduino sino al shield WiFi, hay que seleccionar la placa ESP8266 y ajustar las opciones de compilación. Para ello seleccionamos la placa desde Herramientas > Placa > Generic ESP8266 Module.

La pestaña Herramientas tendrá nuevas ocpiones, las cuales se configurarán de la siguiente forma:
* Flash Mode: "DIO"
* Flash Size: "512K (64 SPIFFS)"
* Debug port: "Disabled"
* Debug level: "Ninguno"
* Reset Method: "ck"
* Crystal Frequency: "26 MHz"
* Flash Frequency: "40 MHz"
* CPU Frequency: "80 MHz"
* Upload Speed: "115200"
* Programador: "AVRISP mkII"

**Dependiendo del modelo del componente ESP8266 usado**, puede que sea necesario cambiar el parámetro **Upload Speed** a 9600. Por defecto, la mayoria de los modelos trabajan bajo la velocidad de baudios 115200.

## Sketch

Dentro de la comunidad de Arduino, el conjunto de instrucciones que se cargan en una placa son llamados "Sketch". El Sketch que utilizaremos se encuentra en la carpeta "sketch" del repositorio.

Dentro del sketch es necesario editar la constantes declaradas al inicio con los datos pertinentes a nuestro ecosistema.

El sketch se encargará de: 
* Inicializar el monitor de serie del Arduino IDE, mediante el cual podremos visualizar las salidas del programa y monitorear el estado de ejecución.
* Inicializar el sensor de temperatura
* Establecer una conexion con la red WiFi definida.
* Consultar la fecha/hora a un servidor externo de acuerdo a la zona horaria definida.
* Definir el nombre del dispositivo emisor de datos (shield WiFi en nuestro caso)
* Establecer conexión con el servidor MQTT.
* Realizar lecturas del sensor de temperatura.
* Estructurar el payload en formato JSON.
* Enviar el payload al servidor MQTT.
* Realizar verificaciones de conexión y lecturas.
* Imprimir mensajes de control e información en el monitor de serie a la velocidad de baudios definida (en nuestro caso, 115200)

## Compilación, carga y ejecución

Es posible compilar el código antes de cargarlo a alguna placa haciendo clic en el botón "Verificar" de la interfaz del Arduino IDE.

Para cargar el código al módulo ESP8266, es necesario conectar la placa Arduino al ordenador mediante el puerto USB. De esta forma, será posible definir el puerto de comunicación con la placa bajo la pestaña Herramientas > Puerto, donde seleccionaremos el puerto disponible.

Podemos abrir el monitor de serie del Arduino IDE para verificar la ejecución del código cuando se realice la carga. Se recomiendan las siguientes opciones sobre el monitor para visualizar la información correcta:

* Autoscroll
* Ambos NL & CR
* 115200 baudio

Una vez que el modulo ESP8266 reciba energía entrará en el Flash Mode, en el cual podremos cargarle las instrucciones del skecth.

Luego de que las instrucciones han sido cargadas, hay que desconectar de tierra el pin GPIO0 (blanco) del modulo ESP8266 y conectarlo al voltaje bajo la resistencia. De esta forma, el módulo ESP8266 no entrará en el Flash Mode la proxima vez que la plataforma arduino sea iniciada, permitiendo que ejecute el codigo cargado apenas reciba energía. La configuración de pines quedaría de la siguiente forma:

![Alt text](images/demo_pinout_bm.png?raw=true "Demo Diagram Boot Mode")

En el monitor de serie podemos observar el proceso de conexión, captura y publicación de mensajes.

**// imagen de monitor de serie**

## Transferencia de publicaciones

Si nos suscribimos al tópico de Mosquitto podremos ver en tiempo real como los mensajes son publicados por la placa de Arduino.

**//imagen mosquitto**

En la carpeta templates se encuentra la plantilla de NiFi utilizada para capturar los datos de Mosquitto y enviarlos tanto a Kafka como a Hive. Cabe destacar que es necesario modificar los parametros de conexión a cada uno de estos servicios como direcciones, tópicos, nombres de tablas, entre otros. 

![Alt text](images/nifi_template.png?raw=true "Demo Diagram Boot Mode")

Los mensajes capturados son publicados exactamente igual en Kafka y en Hive. En este último, se registran campos adicionales en la tabla, relacionados al servidor MQTT desde el cual se captura la información.

Una vez iniciada la plantilla, si nos suscribimos al tópico de Kafka al cual redirigimos los mensajes, podremos observar como los mensajes son publicados prácticamente al instante en que son recibidos en Mosquitto.

**//imagen mosquitto y kafka**

Por otro lado, si consultamos la tabla de Hive periodicamente, notaremos que el numero de registros aumenta de acuerdo a los mensajes que son capturados por NiFi.

**//imagen hive**

## Procesamiento en streaming

El notebook desarrollado con Scala se encuentra en la carpeta notebooks en formato json y puede ser importado en Zeppelin.

Las tablas creadas y utilizadas son las siguientes:

* **mqtt_message_temp**: Tabla temporal que almacenará los mensajes obtenidos en una ventana.
* **kafka_message**: Tabla que contendrá todos los datos obtenidos de cada una de las ventanas de consulta ejecutadas.
* **mean_fahrenheit_temp**: Tabla temporal que almacenará la media calculada de las temperaturas (°F) obtenidas en una ventana.
* **kafka_means_fahrenheit**:Tabla que contendrá todas las medias calculadas de cada una de las ventanas de consulta ejectuadas.
* **gen_test_data**: Tabla que contiene datos generados aleatoriamente para probar el modelo de clasificación Kmedias. Los datos pueden encontrarse en la carpeta test_data. Los datos fueron generados bajo las siguientes especificaciones:
   * **id**: Entero autoincremental (unidad) desde valor 1.
   * **fecha**: String en formato yyyy-MM-dd'T'HH:mm:ssZ entre 2016-11-10 y 2018-10-31.
   * **timestamp**: String de marca de tiempo en formato Unix desde 1478965467 hasta 1541592359.
   * **fahrenheit**: String númerico aleatorio entre 15 y 125.
   * **humedad**: String númerico aleatorio entre 25 y 85.

El notebook contiene los siguientes 7 parrafos:

1. **Setup**
   * Importación de paquetes y librerías usadas en el resto de los parrafos.
   * Definicion intervalo de segundos para el StreamingContext (batchIntervalSeconds).
   * Definición de duración de las ventanas de consulta (windowIntervalSeconds).
   * Definción de parámetros de configuración para la conexión con Kafka bajo su antiguo API (kafkaConf).
2. **Captura de datos**
   * Verificaciones sobre contextos creados y/o creación de nuevos contextos.
   * Establecimiento de conexión con Kafka.
   * Ejecución de ventana de consulta sobre Kafka.
   * Calculo de la media de las temperaturas (°F) recibidas en la ventana.
   
![Alt text](/images/notebook_streaming.png?raw=true "Kafka messages in real time")

3. **Asociación de marcas de tiempo a medias** //probar notebook unificando calculo de medias en un mismo parrafo <-
   * Asociación de la media calculada con marca de tiempo de generación de datos.
4. **Creación y entrenamiento de modelo Kmedias**
   * Transformación de datos históricos de ventanas como entrada al modelo.
   * Entrenamiendo del modelo Kmedias en base a la temperatura (°F) y la humedad.
   * Almacenamiento de modelo en HDFS.
   // Explicar la suma de cuadrados.
5. **Clasificación**
   * Carga de modelo previamente entrenado y almacenado en HDFS.
   * Clasificación de puntos obtenidos en la ultima ventana consultada.
   * Visualización de resultados.
6. **Clasificación - Datos aleatorios**
   * Carga de modelo previamente entrenado y almacenado en HDFS.
   * Clasificación de puntos aleatoriamente generados bajo rangos específicos.
   * Visualización de resultados.
![Alt text](/images/notebook_kmeans.png?raw=true "Kmeas prediction over random data")
7. **Seguimiendo de datos**
   * Visualización de la evolución de la humedad y la temperatura (°C y °F) a lo largo del tiempo sobre los datos hitóricos de ventanas.
![Alt text](/images/notebook_kmeans.png?raw=true "Humidity and temperature overview over time")

## Trabajos futuros

//Explicación plug&play de la plataforma arduino. Mas sensores de temperatura. Multiplexación. Indicadores de control. Boton de reseteo.

## Referencias

## Creditos

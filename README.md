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
*

## Instalación

### Arduino IDE

#### ESP8266

#### Librerías

## Configuración

## Montaje

## Sketch

## Ejecución

## Flujo de datos

## Procesamiento en streaming

## Trabajos futuros

## Referencias

## Creditos

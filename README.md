# arduino-temperature-streaming-demo

El presente proyecto cubre el proceso de despliegue de una arquitectura simple de proposito general para el procesamiento en tiempo real y batch de lecturas de un sensor de temperatura sobre arduino y con tecnologías open source del ecosistema Big Data.

## Introducción

Arduino es una plataforma open source enfocada a promover y producir componentes de hardware y software faciles de usar. Las múltiples placas de Arduino son capaces de leer distintos tipos de señales de entrada, procesarlos y generar algun tipo de señal de salida. Para ello, es necesario desarrollar y enviar un conjunto de instrucciones (bajo un lenguaje de programación especifico) al microcontrolador de la placa o componente en uso mediante el IDE oficial de Arduino.

Mediante el uso de la placa de Arduino es posible realizar lecturas e interactuar con distinto sensores. En este proyecto es usado un sensor de temperatura y humedad de la familia DHT. Este tipo de sensores son bastante lentos y básicos pero cumplen con su propósito. Estan compuentos por dos componentes principales: un sensor capacitivo de humedad  un termistor. Tambien son capaces de realizar conversiones analógicas a digitales y separar la señales entre la temperatue y humedad.

A su vez, dependiendo de la placa de Arduino que se utilice, se pueden integrar otro tipo de componentes para ampliar las funcionalidades de la misma. Por ejemplo, con respecto al envío de datos por internet se puede hacer uso de un chip de la familia ESP2866, los cuales son alternativas de bajo costo que soportan el procolo TCP/IP y poseen tanto una unidad de memoria pequeña como un Microcontrolador propio, lo cual permite que tambien se cargar instrucciones en el componente.



# Sensor DHT22 con ULTRASONICO y con NODE-RED
Este repositorio muestra como podemos programar una ESP32 con SR- y ocupando la plataforma node-red y ahi poder la distancia y almismo tiempo graficar los resultados obtenidos.


### Descripción

La ```Esp32``` la utilizamos en un entorno de adquision de datos, lo cual en esta practica ocuparemos un sensor (```HC-SR04```) para medir la distancia y un sensor ```DHT22``` para medir de temperatura y humedad. Cabe aclarar que esta practica se usara en el simulador llamado [WOKWI](https://https://wokwi.com/). y ```localhost:1880```


## Material Necesario

Para realizar esta practica necesitas lo siguiente

- [WOKWI](https://https://wokwi.com/)
- Tarjeta ESP 32
- HC-SR04
- DHT22
- NODE-red (http://localhost:1880/#flow/214928d358894902)


## Instrucciones

### Requisitos previos

# Instalar Node-Red

## Pasos para instalación

1. Entrar a la pagina  https://nodejs.org/en
2. Descargar el archivo **18.16.0 LTS** como se muestra en la siguente imagen.

![](https://github.com/DiegoJm10/Node-red-instalcacion/blob/main/Node.js%20-%20Google%20Chrome%2014_06_2023%2005_04_00%20p.%20m..png?raw=true)

3. Abrir el archivo e instalar el programa [node.js](https://nodejs.org/en)


4. Nos vamos al inicio y buscamos ```cmd``` cOmo se muestra en la imagen y damos clic en "ejecutar como administrador" y escribir lo siguente:
![](https://github.com/Omarcollado23/PRACTICA-7-CON-ULTRASONICO/blob/main/CMD.png?raw=true)

```
npm install -g --unsafe-perm node-red
```
![](https://github.com/Omarcollado23/PRACTICA-7-CON-ULTRASONICO/blob/main/npm%20instal.png?raw=true)

5. Despues comprobamos que funcione node-red con el siguente codigo: (con este mismo codigo podemos arrancar el programa siempre que lo necesitemos)

```
node-red
```
 ![](https://github.com/Omarcollado23/PRACTICA-7-CON-ULTRASONICO/blob/main/node-red.png?raw=true)


 ## Arranque de programa

Para abrir la aplicación nos vamos algun explorador y colocamos el siguente link:    ```localhost:1880```


## Instalación de Dashboard

1. Abrimos la pestaña de opciones y elegimos ```Manage palette``` 

![](https://github.com/DiegoJm10/Node-red-instalcacion/blob/main/Node.js%20-%20Google%20Chrome%2014_06_2023%2005_06_26%20p.%20m..png?raw=true)

2. Seleccionamos **Install* y buscamos ```node-red-dashboard```.
3. Seleccionamos ```node-red-dashboard```.
![](https://github.com/DiegoJm10/Node-red-instalcacion/blob/main/Node.js%20-%20Google%20Chrome%2014_06_2023%2005_06_17%20p.%20m..png?raw=true)


### Instrucciones de preparación de entorno 

1. Abrir la terminal de programación y colocar la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const int Trigger = 26;   //Pin digital 2 para el Trigger del sensor
const int Echo = 25;   //Pin digital 3 para el Echo del sensor

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="Alexcollado";
String password_mqtt="190323";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {


delay(1000);
long t; //timepo que demora en llegar el eco
long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
TempAndHumidity  data = dhtSensor.getTempAndHumidity();

  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
        //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
    doc["DISTANCIA"] = d;
    
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("OmarDAI&M", output.c_str());

  Serial.print("Distancia: ");
  Serial.print(d);      //Enviamos serialmente el valor de la distancia
  Serial.print("cm");
  Serial.println();
  delay(2000);          //Hacemos una pausa de 100ms
   
  }
}

```
2. Instalamos la libreria de **DHT sensor library for ESPx**, **ArdiunoJson** y **PubSubClient** como se muestra en la siguente imagen, dando clic en (Library Manager) y despues en el simbolo de (+)

![](https://github.com/Omarcollado23/PRACTICA-7-CON-ULTRASONICO/blob/main/libreria.png?raw=true)

3. Hacemos las conexiones el **ESP32** del sensor **HC-SR04**  y el **DHT22** como se muestra en la siguiente imagen

![](https://github.com/Omarcollado23/PRACTICA-8-DHT22-CON-ULTRASONICO/blob/main/CONEXIONES.png?raw=true)

### Instrucciónes de operación

1. Iniciamos el simulador.
2. Visualizar los datos en el monitor serial.
3. Colocar la distancia dando *doble click* al sensor **HC-SR04** 

### Despues de que el programa corrio bien sin errores, ahora nos vamos a las parte de conexion con NODE-RED

# Instrucciones para hacer la conexión con NODE-RED

Abrimos una nueva pestaña en el navegador que utilizas e insertamos en la barra de navegación el suiguiente link (localhost:1880)

![](https://github.com/Omarcollado23/PRACTICA-7-CON-ULTRASONICO/blob/main/localhost1.png?raw=true)

1. Colocamos bloque ```mqqtt in```.

![](https://github.com/DiegoJm10/dht22-con-node-red/blob/main/bloquemqtt.png?raw=true)

2. Configurar el bloque con el puerto mqtt con el ip ```44.195.202.69:1883``` como se muestra en la imagen.

![](https://github.com/Omarcollado23/PRACTICA-7-CON-ULTRASONICO/blob/main/%23servidor.png?raw=true)

3. Colocar el bloque json y configurarlo como se muestra en la imagen.

![](https://github.com/DiegoJm10/dht22-con-node-red/blob/main/JSON.png?raw=true)

Colocamos tres bloques function y lo configuramos con el siguente codigo.

```
msg.payload = msg.payload.TEMPERATURA;
msg.topic = "TEMPERATURA";
return msg;
```

![](https://github.com/Omarcollado23/PRACTICA-6-DHT22-CON-NODE-RED/blob/main/conf%20temp.png?raw=true)


```
msg.payload = msg.payload.HUMEDAD;
msg.topic = "HUMEDAD";
return msg;
```

![](https://github.com/Omarcollado23/PRACTICA-6-DHT22-CON-NODE-RED/blob/main/conf%20hum.png?raw=true)

```
msg.payload = msg.payload.DISTANCIA;
msg.topic = "DISTANCIA";
return msg;
```
![](https://github.com/Omarcollado23/PRACTICA-7-CON-ULTRASONICO/blob/main/code%20distancia.png?raw=true)

5. Colocamos los bloques de chart y gauge.
Los configuramos de la siguiente manera como se muestra en las siguientes imagenes.

![](https://github.com/Omarcollado23/PRACTICA-8-DHT22-CON-ULTRASONICO/blob/main/conf%20temp.png?raw=true)

![](https://github.com/Omarcollado23/PRACTICA-8-DHT22-CON-ULTRASONICO/blob/main/conf%20hum.png?raw=true)

![](https://github.com/Omarcollado23/PRACTICA-8-DHT22-CON-ULTRASONICO/blob/main/conf%20dist.png?raw=true)

![](https://github.com/Omarcollado23/PRACTICA-8-DHT22-CON-ULTRASONICO/blob/main/graficos.png?raw=true)


## Resultados

Cuando haya funcionado, la información obtenida del sensor **DHT22** Y **HC-SR04** los mandara a servidor cuando se haga conexión y los observaremos por medio del dashboard. 

Al final el diagrama nos queda de la siguiente manera:

![](https://github.com/Omarcollado23/PRACTICA-8-DHT22-CON-ULTRASONICO/blob/main/esquema.png?raw=true)

Despues damos Clic en el boton donde dice ```Deploy``` para cargar el programa y despues oprimos la pestaña con una flechita señalnado hacia arriba en diagonal, como se muestra en la imagen. 

![](https://github.com/Omarcollado23/PRACTICA-7-CON-ULTRASONICO/blob/main/deploy.png?raw=true)

Y veremos los resultados que manda el sensor al servidor como se muestra en la imagen.

![](https://github.com/Omarcollado23/PRACTICA-8-DHT22-CON-ULTRASONICO/blob/main/result%201.png?raw=true)

![](https://github.com/Omarcollado23/PRACTICA-8-DHT22-CON-ULTRASONICO/blob/main/result2.png?raw=true)

# Créditos

Desarrollado por Ing. Omar Alejandro Collado Carriola

- [GitHub](https://github.com/Omarcollado23)
# Proyecto llenado de un tinaco 

## Introducción

### Descripción

El proyecto consiste en un sistema de llenado, donde una cisterna suministra agua hacia un tinaco, es por ello que, se utiliza un sensor ultrasónico de distancia para determinar el nivel en el que se encuentra cada uno de estos, si el tinaco se encuentra vacío se enciende la bomba de la cisterna hasta cierta distancia y porterior a ello se apaga el motor.

## Material Necesario

Para realizar este proyecto se ocuparon las siguientes herramientas y componentes:

- [Wokwi](https://wokwi.com/)
- [Node-red](https://nodejs.org/en)
- Relay
- Tarjeta **ESP32**
- Sensor ultrasónico de distancia


## Instrucciones

### Arranque de Node-red 

1. Dentro de Node-red, se colocaron dos bloques ```mqtt in```, donde se hará la conexión mediante una IP.

2. Copiar la IP ```3.82.39.163```, porteriormente dirigirse al ```localhost:1880```, hacer *doble click* en **mqtt in**, en el apartado *Server*, hacer click en el ícono de lápiz y pegar en el *Server*. Cambiar el nombre de un bloque en *Topic*, en este caso se utilizó el nombre de "Axel2". Este paso se repite para el segundo bloque, cambiando el *Topic* a "Axel3".

3. Añadir cuatro bloques _function_, dos de ellos serán para la cisterna y los otros para el tinaco. Uno de los bloques se utilizará para mandar mensajes y el otro para visualizar las gráficas e indicadores.

4. Para los bloques de texto, escribir los siguientes códigos:
```
msg.payload = msg.payload.CISTERNA;
msg.topic = "CISTERNA";
if (msg.payload >= '200') {
    msg.payload = 'CISTERNA ESTA LLENA';
}
else {
    msg.payload = 'CISTERNA ESTA VACIA.';
}
return msg;
```
```
msg.payload = msg.payload.TINACO;
msg.topic = "TINACO";
if (msg.payload>= '200') {
    msg.payload = 'TINACO ESTA LLENO';
}
else {
    msg.payload = 'TINACO ESTÁ VACÍO';
}
return msg;
```

6. Para los bloques de gráficas se escribe el siguiente código:
```
msg.payload = msg.payload.CISTERNA;
msg.topic = "CISTERNA";
return msg;
```
```
msg.payload = msg.payload.TINACO;
msg.topic = "TINACO";
return msg;
```

7. En la esquina superior derecha (debajo del ícono de propiedades), se encuentra un ícono de triángulo, hacer click en el apartado _dashboard_, seleccionar _tab_>_group_>_spacer_ (crear 3 grupos). Cambiar el nombre a _Proyecto_ del _Tab_ y en los grupos uno será para el _texto_, otro para las _gráficas_ y por último uno de _indicadores_. 

8. Una vez hecho, abrir dos pestañas de [WOKWI](https://wokwi.com/) se selecciona la tarjeta **ESP32**, un sensor ultrasónico y un relay (en ambas pestañas se hace lo mismo).

9. En una de las pestañas de Wokwi, añadir el siguiente código para la cisterna:
```
//librerias a internet
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
// Se declaran las variables a utilizar
const int Trigger = 12; //cisterna
const int Echo = 13; //cisterna
int ledPin = 14; // Pin del LED

//conexion a nuestro wifi

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "3.82.39.163";
String username_mqtt = "Axel2";
String password_mqtt = "1234";

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
  Serial.println("WiFi conectado");
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
      Serial.println("conectado");
      // Once connected, publish an announcement...
      client.publish("outTopic", "Hola mundo");
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

  // Se define el modo del pin a utilizar
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  pinMode(ledPin, OUTPUT);
  pinMode (Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT); //pin como entrada
  digitalWrite (Trigger, LOW);//Inicializamos el pin con 0
  digitalWrite(ledPin,0);


}

void loop() {

  long t; //tiempo que demora en llegar el hecho
  long d; //distancia en centimetros

  // Tiempo se envio de señal

  //Para el primer sensor

  digitalWrite (Trigger, HIGH);
  delayMicroseconds(10); //Enviamos un pulso de 10us
  digitalWrite (Trigger, LOW);

  t= pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t / 59; //escalamos el tiempo a una distancia en cm

  // si la cisterna esta vacia y el tinaco vacio no se prende la bomba
  // 0| 0 =0
  if (d >= 200) {
    digitalWrite (ledPin , 1);
  }
  else{
    digitalWrite(ledPin, 0);
  }
  
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
    doc["CISTERNA"] = d;
   
    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("Axel2", output.c_str());
  }
}
```
Para el tinaco sería el siguiente código:
```
//librerias a internet
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>


// Se declaran las variables a utilizar
const int Trig_tinaco = 12; //cisterna
const int Echo_tinaco = 13; //cisterna
int ledPin = 14; // Pin del LED

//conexion a nuestro wifi

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "3.82.39.163";
String username_mqtt = "Axel3";
String password_mqtt = "1234";

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
  Serial.print("Conectando  ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi conectado");
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
      Serial.println("conectado");
      // Once connected, publish an announcement...
      client.publish("outTopic", "Hola mundo");
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

  // Se define el modo del pin a utilizar
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  pinMode(ledPin, OUTPUT);
  pinMode (Trig_tinaco, OUTPUT); //pin como salida
  pinMode(Echo_tinaco, INPUT); //pin como entrada
  digitalWrite (Trig_tinaco, LOW);//Inicializamos el pin con 0
  digitalWrite(ledPin,0);



}

void loop() {

  long t_tinaco; //tiempo que demora en llegar el hecho
  long d_tinaco; //distancia en centimetros

  // Tiempo se envio de señal

  //Para el primer sensor

  digitalWrite (Trig_tinaco, HIGH);
  delayMicroseconds(10); //Enviamos un pulso de 10us
  digitalWrite (Trig_tinaco, LOW);

  t_tinaco = pulseIn(Echo_tinaco, HIGH); //obtenemos el ancho del pulso
  d_tinaco = t_tinaco / 59; //escalamos el tiempo a una distancia en cm

    // si la sisterna esta vacia y el tinaco esta lleno no se prende la bomba
  // 0|1 = 0
  if (d_tinaco >= 200) {
    digitalWrite (ledPin , 0);
  }
  else{
    digitalWrite(ledPin, 1);
  }
  
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
    doc["TINACO"] = d_tinaco;
   
    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("Axel3", output.c_str());
  }
}
```
7. En la pestaña de *Library Manager*, instalar las librerías de **ArduinoJson** y **PubSubClient** como se muestra en la siguente imagen.

![](https://github.com/Michellecg/Node-red_con_sensor_de_distancia/blob/main/Lib_Ult.PNG)

8. Hacer la conexión de los **sensores ultrasónicos de distancia** a la tarjeta **ESP32** como se muestra en la siguente imagen.

![](https://github.com/Michellecg/Proyecto_Llenado-de-tanque/blob/main/Conex_1.PNG)

9. En Node-red, se debe observar de la siguiente manera el diagrama:

![](https://github.com/Michellecg/Proyecto_Llenado-de-tanque/blob/main/Diagrama_nodered.PNG)

## Resultados

El dashboard muestra un mensaje dependiendo el estado en el que se encuentre la cisterna y el tinaco, además de sus indicadores de nivel.

![](https://github.com/Michellecg/Proyecto_Llenado-de-tanque/blob/main/Com2_Sens.PNG)

![](https://github.com/Michellecg/Proyecto_Llenado-de-tanque/blob/main/Resultados_p.PNG)

# Créditos

Desarrollado por Michelle Cuatlapantzi González

- [GitHub](https://github.com/Michellecg/)
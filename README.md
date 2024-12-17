# NODE-RED-CON-DOS-SENSORES
DHT22, ULTRASONICO

# Introducción

La Esp32 la utilizamos en un entorno de adquision de datos, lo cual en esta practica ocuparemos un sensor (DTH11) para adquirir temperatura y humedad del entorno; Cabe aclarar que esta practica se usara un simulador llamado WOKWI.

# Material Necesario

Para realizar esta practica necesitas lo siguiente

- WOKWI
- Tarjeta ESP 32
- Sensor DHT22
- Sensor ultrasonico
- Plataforma Node-red

COMENZAREMOS COPIANDO EL SIGUIENTE CODIGO EN WOWKI:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.
const int Echo = 2;   //Pin digital 2 para el Echo del sensor
const int Trigger = 4; //pin del echo

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "35.172.255.228";
String username_mqtt="IVAN ZAGAL";
String password_mqtt="1996";

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

long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm

delay(1000);
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
    doc["DISTANCIA"] = String(d);
   doc["NOMBRE"] = "EDGAR IVAN ZAGAL";

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("TKMEX", output.c_str());
  }
}

 ```

CONFIGURAMOS LA IP DE LA LINEA 14 LA DIRECCION IP ES: 35.172.255.228


AHORA CARGAMOS LAS SIGUIENTES LIBRERIAS:

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/LIBRERIAS.PNG)

INSERTAMOS UN DHT22 Y SENSOR ULTRASONICO:

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/DHT22.PNG
)

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/ULTRASONICO.PNG)

REALIZAMOS LA SIGUIENTE CONEXION:

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/CONEXION.PNG)

 AHORA VAMOS DESCARGAR E INSTALANDO EL NODE-RED EN EL SIGUIENTE LINK:
 https://nodejs.org/en

 UNA VEZ INSTALADO ABRIREMOS EL CMD EN MODO ADMINISTRADOR DE TAREAS:

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/CMD%20ADMIN.png)

A CONTINUACION PEGAREMOS LO SIGUIENTE Y DAREMOS ENTER

 ```
npm install -g --unsafe-perm node-red
 ```
comprobamos que funcione node-red con el siguente codigo: (con este mismo codigo podemos arrancar el programa siempre que lo necesitemos)

 ```
node-red
 ```
Quedandonos asi:

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/CMD%20NODERED.PNG)

Con esta ventana abierta abriremos nuestro navegador y pegamos este link

 ```
localhost:1880
 ```
Se abrira esta pagina y ahora instalaremos el DASHBOARD:

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/INSTALACION%20DE%20DASHBOARD.PNG)

VAMOS A MANAGE PALETTE

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/MANAGE%20PALLETE.PNG
)

EN LA PESTAÑA INSTALL BUSCAMOS "node-red-dashboard" Y SELECCIONAMOS INSTALL

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/INSTALACION%20FIN%20DSAHBOARD.PNG)

LISTO YA PODEMOS EMPEZAR A TRABAJAR

DEBEMOS RELIZAR EL SIGUIENTE ESQUEMA:

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/DIGRAMA%20NODE%20RED.PNG)

Como primer paso insertaremos el mqtt in, y lo configuramos como se muestra a continuacion

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/INSERTAR%20MQTTBIN.PNG)

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/EDIT%20MQTT%20IN.PNG)

insertaremos un json y editaremos como se muestra

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/insertar%20json.PNG)

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/edit%20json.PNG)

Acontinuacion insertareos un debug

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/insertar%20debug.PNG)

Insertaremos 3 funciones una para temperatura, humedad y distancia.

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/insertar%20fuctions.PNG)

Ahora para cada funcion insertaremos los indicadores y las graficas (gauge y charts).

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/insertar%20gauge%20y%20charts.PNG)

Creamos los grupos (Graficas e indicadores)

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/agregar%20grupos%20en%20dashboard.PNG)

Editamos la funcion de tempertura

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/edit%20f%20temperatura.PNG)

Editamos la funcion de humedad

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/edit%20fhumedad.PNG)

Editamos la funcion de distancia

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/edit%20f%20distancia.PNG)

Editamos el gauge de tempertura

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/grupo%20indicadores.PNG)

Editamos el gauge de humedad

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/grupo%20indicadores%20humedad.PNG)

Editamos el gauge de distancia

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/grupo%20indicadores%20distancia.PNG)

Editamos el chart de tempertura

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/grupo%20graficas.PNG)

Editamos el chart de humedad

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/grupo%20graficas%20humedad.PNG)

Editamos el chart de distancia

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/grupo%20fraficas%20distancia.PNG)

Cargamos y damos en el simbolo de catarina para visualizar los resultados de nuestra programacion de la ESP 32

![]()

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/visualizar%20catarina.PNG)

 Damos click es Deploy y y en el siguiente simbolo:
 
![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/DEPLOY%20IMAGE.png)
![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/URLL%20SIMBOL.png)
 
 Nos redigira a otra pagina en donde observaremos nuesttros indicadores y graficas en tiempo real.
 Resultados DE LOS INDICADORES Y GRAFICAS EN TIEMPO REAL

![](https://github.com/IVANZAGAL996/NODE-RED-CON-DOS-SENSORES/blob/main/GRAFICAS%20E%20INDICADORES.PNG)


# CREDITOS
Elaborado por:
EDGAR IVAN PROSPERO ZAGAL PONCE
[GitHub]()





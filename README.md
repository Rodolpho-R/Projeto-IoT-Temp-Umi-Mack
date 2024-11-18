# Projeto-IoT-Temp-Umi-Mack
Medidor de temperatura e umidade

# Sistema de Monitoramento de Temperatura e Umidade

Este projeto utiliza um ESP32 e sensores DHT22 para monitorar a temperatura e a umidade de um ambiente. Os dados são enviados via protocolo MQTT para um servidor de nuvem, onde são exibidos em uma interface gráfica. O sistema também controla um relé para ligar ou desligar um dispositivo com base nos parâmetros de temperatura e umidade.

## Como usar
1. Configure o Wi-Fi e os parâmetros MQTT no código.
2. Carregue o código no ESP32.
3. O sistema irá monitorar a temperatura e umidade e enviar os dados para o broker MQTT.
4. O relé será acionado caso os parâmetros estejam fora da faixa definida.

##Código utilizado:

'''ino
//Bibliotecas utilizadas
#include <WiFi.h>
#include <PubSubClient.h>
#include <WiFiClientSecure.h>
#include <DHT.h>

// Definição de pinos e tipo de sensor
#define DHTPIN 13
#define DHTTYPE DHT22
#define RELAY_PIN 0

DHT dht(DHTPIN, DHTTYPE);

// Credenciais WiFi, insira suas credenciais de Wifi
const char* ssid = "*****";
const char* password = "*****";

// Configuração do servidor MQTT (HiveMQ Cloud)
const char* mqtt_server = "75572bc692324ba2a542276c9824b306.s1.eu.hivemq.cloud";
const int mqtt_port = 8883;
const char* mqtt_user = "hivemq.webclient.1731803195170";
const char* mqtt_password = "#Lk0*S8sXyMHpl9wU.1!";

// Tópicos MQTT
const char* master_topic = "home/mode";
const char* lamp_topic = "home/lamp";
const char* status_topic = "home/status";

// Cliente seguro para comunicação MQTT
WiFiClientSecure espClient;
PubSubClient client(espClient);

// Definindo valores mínimo e máximo para temperatura e umidade
const float TEMP_MIN = 18.0; // Temperatura mínima permitida
const float TEMP_MAX = 32.0; // Temperatura máxima permitida
const float HUM_MIN = 30.0;  // Umidade mínima permitida
const float HUM_MAX = 90.0;  // Umidade máxima permitida

// Variáveis de controle
bool modo_manual = false;
bool lampada_status = false;

// Função para conectar ao WiFi
void setup_wifi() {
  Serial.print("Conectando ao WiFi: ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi conectado!");
  Serial.print("Endereço IP: ");
  Serial.println(WiFi.localIP());
}

// Função para controlar a lâmpada
void controla_lampada(bool estado) {
  digitalWrite(RELAY_PIN, estado ? HIGH : LOW);
  lampada_status = estado;

  // Publica o status da lâmpada
  client.publish(status_topic, estado ? "Luz de Alerta Ligado: Temperatura/Umidade fora dos padrões!" : "Luz de Alerta Desligado: Temperatura/Umidade dentro dos padrões.");
  Serial.print("Lâmpada ");
  Serial.println(estado ? "Luz de Alerta Ligado: Temperatura/Umidade fora dos padrões!" : "Luz de Alerta Desligado: Temperatura/Umidade dentro dos padrões.");
}

// Função de callback para tratar mensagens recebidas
void callback(char* topic, byte* payload, unsigned int length) {
  String message;
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }

  Serial.print("Mensagem recebida no tópico: ");
  Serial.println(topic);
  Serial.print("Mensagem: ");
  Serial.println(message);

  // Controle do modo manual
  if (String(topic) == master_topic) {
    if (message == "ON") {
      modo_manual = true;
      Serial.println("Modo manual ativado.");
    } else if (message == "OFF") {
      modo_manual = false;
      Serial.println("Modo automático ativado.");
    }
  }

  // Controle manual da lâmpada
  if (String(topic) == lamp_topic && modo_manual) {
    if (message == "ON") {
      controla_lampada(true); // Liga a lâmpada
    } else if (message == "OFF") {
      controla_lampada(false); // Desliga a lâmpada
    }
  }
}

// Função para reconectar ao servidor MQTT
void reconnect() {
  while (!client.connected()) {
    Serial.print("Tentando conectar ao MQTT...");
    if (client.connect("ESP32Client", mqtt_user, mqtt_password)) {
      Serial.println("Conectado ao MQTT!");
      client.subscribe(master_topic);
      client.subscribe(lamp_topic);
    } else {
      Serial.print("Falha na conexão. Código de erro: ");
      Serial.println(client.state());
      delay(5000);
    }
  }
}

// Configuração inicial
void setup() {
  Serial.begin(115200);

  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, LOW); // Inicialmente desligada

  dht.begin();
  setup_wifi();

  client.setServer(mqtt_server, mqtt_port);
  espClient.setInsecure(); // Necessário para HiveMQ Cloud
  client.setCallback(callback);
}

// Loop principal
void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Lógica do modo automático
  if (!modo_manual) {
    float t = dht.readTemperature();
    float h = dht.readHumidity();

  if (isnan(t) || isnan(h)) {
      Serial.println("Erro na leitura do sensor DHT22!");
      return;
    }
    
  // Publicando a temperatura no tópico de temperatura
  char msg[50];
  snprintf(msg, sizeof(msg), "%.2f", t);
  Serial.print("Temperatura: ");
  Serial.println(msg);
  client.publish("home/temperatura", msg);

  // Publicando a umidade no tópico de umidade
  snprintf(msg, sizeof(msg), "%.2f", h);
  Serial.print("Umidade: ");
  Serial.println(msg);
  client.publish("home/umidade", msg);

  // Condições para ligar/desligar a lâmpada automaticamente
    if (t < TEMP_MIN || t > TEMP_MAX || h < HUM_MIN || h > HUM_MAX) {
      controla_lampada(true);
    } else {
      controla_lampada(false);
    }

  delay(5000); // Intervalo para próxima leitura
  }
}

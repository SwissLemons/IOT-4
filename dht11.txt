#include <WiFi.h>
#include <WebServer.h>
#include "DHT.h"
// Remova o comentário de uma das linhas abaixo para qualquer tipo de sensor DHT que você está usando!
//#define DHTTYPE DHT11   // DHT 11
//#define DHTTYPE DHT21   // DHT 21 (AM2301)
#define DHTTYPE DHT11     // DHT 22  (AM2302), AM2321
// * Coloque seu SSID e senha * /
const char* ssid = "ai minha canela";  // Entre SSID aqui
const char* password = "alpaka slayer";  // Insira a senha aqui
WebServer server(80);
// Sensor DHT
uint8_t DHTPin = 4; 
int pinoPotenciometro = 32;
int pinoBotao = 33;
               
// Inicializa o sensor DHT.
DHT dht(DHTPin, DHTTYPE);                
float Temperature;
float Humidity;
 
void setup() {
  Serial.begin(115200);
  delay(100);
  
  pinMode(DHTPin, INPUT);
  pinMode(pinoBotao, INPUT);
  dht.begin();              
  Serial.println("Conectando à ");
  Serial.println(ssid);
  // conecte-se à sua rede wi-fi local
  WiFi.begin(ssid, password);
  // verifique se o wi-fi está conectado à rede wi-fi
  while (WiFi.status() != WL_CONNECTED) {
     delay(1000);
     Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi conectado..!");
  Serial.print("IP obtido: ");  
  Serial.println(WiFi.localIP());
  server.on("/", handle_OnConnect);
  server.onNotFound(handle_NotFound);
  server.begin();
  Serial.println("Servidor HTTP iniciado");
}
void loop() {
  server.handleClient();
}
void handle_OnConnect() {
 Temperature = dht.readTemperature(); // Obtém os valores da temperatura
  Humidity = dht.readHumidity(); // Obtém os valores da umidade
  int valorPotenciometro = analogRead(pinoPotenciometro);
  bool valorBotao = digitalRead(pinoBotao);
  
  server.send(200, "text/html", SendHTML(Temperature,Humidity,valorPotenciometro, valorBotao)); 
}
void handle_NotFound(){
  server.send(404, "text/plain", "Não encontrado");
}

  String SendHTML(float temp, float hum, int pot, bool btn) {
  // R"rawliteral(...)" encapsula todo o texto como está, sem precisar de escape
  const char html[] = R"rawliteral(
<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <meta
    name="viewport"
    content="width=device-width, initial-scale=1.0, user-scalable=no"
  >
  <title>Boletim Meteorológico</title>
  <link 
    href="https://fonts.googleapis.com/css?family=Open+Sans:300,400,600" 
    rel="stylesheet"
  >
  <style>
    /* Container principal usando CSS Grid */
    .container {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
      gap: 1rem;
      padding: 1rem;
      max-width: 600px;
      margin: 0 auto;
    }
    /* Cartão para cada métrica */
    .card {
      background: #fff;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      padding: 1rem;
      text-align: center;
      font-family: 'Open Sans', sans-serif;
    }
    .card h2 {
      margin: 0.5rem 0;
      font-weight: 300;
      font-size: 2rem;
    }
    .card .label {
      font-size: 0.9rem;
      font-weight: 600;
      color: #555;
    }
    /* ícones podem ser SVG inline ou via font-icon */
    .icon {
      width: 32px;
      height: 32px;
      margin-bottom: 0.5rem;
    }
  </style>
  <meta http-equiv="refresh" content="2">
</head>
<body>
  <h1 style="text-align:center;margin-top:1rem;">Boletim Meteorológico ESP32</h1>
  <div class="container">
    <div class="card">
      <!-- Exemplo de SVG incorporado -->
      <svg class="icon" viewBox="0 0 24 24">
        <!-- (coloque aqui o path do ícone de temperatura) -->
      </svg>
      <div class="label">Temperatura</div>
      <h2>%TEMPC%°C</h2>
    </div>
    <div class="card">
      <svg class="icon" viewBox="0 0 24 24">
        <!-- Ícone de umidade -->
      </svg>
      <div class="label">Umidade</div>
      <h2>%HUM%&#37;</h2>
    </div>
    <div class="card">
      <svg class="icon" viewBox="0 0 24 24">
        <!-- Ícone de potência -->
      </svg>
      <div class="label">Potência</div>
      <h2>%POT%&#37;</h2>
    </div>
    <div class="card">
      <svg class="icon" viewBox="0 0 24 24">
        <!-- Ícone de botão -->
      </svg>
      <div class="label">Botão</div>
      <h2>%BTN%</h2>
    </div>
  </div>
</body>
</html>
)rawliteral";

  String s = html;
  s.replace("%TEMPC%",    String((int)temp));
  s.replace("%HUM%",      String((int)hum));
  s.replace("%POT%",      String(pot));
  s.replace("%BTN%",      (btn ? "Pressionado" : "Solto"));
  return s;
}

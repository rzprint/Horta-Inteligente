# Horta-Inteligente

Horta-Inteligente
│
├── Firmware
│   ├── Horta_Inteligente_v3.ino
│   ├── README.md
│   └── Bibliotecas.md
│
├── Blynk
│   ├── Dashboard.md
│   └── Widgets.md
│
├── Esquema
│   ├── Ligacao.png
│   └── Pinagem.md
│
├── Imagens
│
└── Documentacao
    ├── Changelog.md
    └── Melhorias.md

/*
 HORTA INTELIGENTE v3.0 - ENTREGA 2
 Base com integração Blynk IoT

 ===== BIBLIOTECAS NECESSÁRIAS =====
 - Adafruit GFX
 - Adafruit SSD1306
 - ESP8266 Boards
 - Blynk

 ===== CONFIGURE =====
#define BLYNK_TEMPLATE_ID "SEU_TEMPLATE_ID"
#define BLYNK_TEMPLATE_NAME "Horta Inteligente"
#define BLYNK_AUTH_TOKEN "SEU_TOKEN"

const char ssid[] = "SEU_WIFI";
const char pass[] = "SUA_SENHA";
*/

#define BLYNK_TEMPLATE_ID "SEU_TEMPLATE_ID"
#define BLYNK_TEMPLATE_NAME "Horta Inteligente"
#define BLYNK_AUTH_TOKEN "SEU_TOKEN"

#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

char auth[] = BLYNK_AUTH_TOKEN;
char ssid[] = "Rizing";
char pass[] = "Luna#2101";

#define OLED_SDA 14
#define OLED_SCL 12
#define SENSOR A0
#define RELE D7

Adafruit_SSD1306 display(128,64,&Wire,-1);
BlynkTimer timer;

const int seco=792;
const int molhado=294;

int limite=40;
int tempoRega=50;
bool automatico=true;
bool irrigando=false;

int leituraBruta=0;
int umidade=0;

int lerUmidade(){
  long soma=0;
  for(int i=0;i<20;i++){
    soma+=analogRead(SENSOR);
    delay(5);
  }
  leituraBruta=soma/20;
  int u=map(leituraBruta,seco,molhado,0,100);
  return constrain(u,0,100);
}

void tela(){
  display.clearDisplay();
  display.setTextSize(1);
  display.setCursor(0,0);
  display.println("HORTA INTELIGENTE");

  display.setTextSize(3);
  display.setCursor(8,16);
  display.print(umidade);
  display.print("%");

  display.drawRect(0,56,128,8,SSD1306_WHITE);
  display.fillRect(1,57,map(umidade,0,100,0,126),6,SSD1306_WHITE);

  display.setTextSize(1);
  display.setCursor(78,0);
  display.print(WiFi.status()==WL_CONNECTED?"WiFi OK":"Sem WiFi");

  display.display();
}

void regar(){
  irrigando=true;
  digitalWrite(RELE,LOW);
  Blynk.virtualWrite(V5,1);

  for(int i=tempoRega;i>=0;i--){
    display.clearDisplay();
    display.setTextSize(2);
    display.setCursor(0,0);
    display.println("REGANDO");
    display.setTextSize(2);
    display.setCursor(0,30);
    display.print(i);
    display.print(" s");
    display.display();
    Blynk.run();
    delay(1000);
  }

  digitalWrite(RELE,HIGH);
  irrigando=false;
  Blynk.virtualWrite(V5,0);
}

void atualizar(){
  umidade=lerUmidade();

  tela();

  Blynk.virtualWrite(V0,umidade);
  Blynk.virtualWrite(V1,leituraBruta);
  Blynk.virtualWrite(V2,automatico);
  Blynk.virtualWrite(V3,limite);
  Blynk.virtualWrite(V4,tempoRega);

  if(automatico && umidade<limite)
    regar();
}

BLYNK_WRITE(V2){
  automatico=param.asInt();
}

BLYNK_WRITE(V3){
  limite=param.asInt();
}

BLYNK_WRITE(V4){
  tempoRega=param.asInt();
}

BLYNK_WRITE(V6){
  if(param.asInt())
    regar();
}

void setup(){

  pinMode(RELE,OUTPUT);
  digitalWrite(RELE,HIGH);

  Serial.begin(115200);

  Wire.begin(OLED_SDA,OLED_SCL);
  display.begin(SSD1306_SWITCHCAPVCC,0x3C);

  display.clearDisplay();
  display.setTextSize(2);
  display.setCursor(0,18);
  display.println("INICIANDO");
  display.display();

  WiFi.begin(ssid,pass);

  Blynk.config(auth);
  Blynk.connect();

  timer.setInterval(1000L,atualizar);
}

void loop(){
  Blynk.run();
  timer.run();
}

/*
===== DASHBOARD BLYNK =====

V0 Gauge        Umidade (%)
V1 Gauge        Leitura Sensor
V2 Switch       Automatico
V3 Slider       Limite Umidade
V4 Slider       Tempo Rega
V5 LED          Estado Rele
V6 Botao        Irrigar Agora

*/

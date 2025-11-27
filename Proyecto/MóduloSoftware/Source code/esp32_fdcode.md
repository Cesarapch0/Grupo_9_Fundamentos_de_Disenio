#include <LiquidCrystal.h>

#include <HardwareSerial.h>

#define RS 14

#define EN 27

#define D4 26

#define D5 25

#define D6 33

#define D7 32

LiquidCrystal lcd(RS, EN, D4, D5, D6, D7);

HardwareSerial PMSerial(2);

#define PMS\_RX 16

#define PMS\_TX 17

float pm25\_suave = 0.0;   // valor filtrado

float alpha = 0.2;        // suavizado

void setup() {

Serial.begin(115200);

PMSerial.begin(9600, SERIAL\_8N1, PMS\_RX, PMS\_TX);

lcd.begin(16, 2);

lcd.print("PM2.5 Sensor");

delay(2000);

lcd.clear();

}

void loop() {

if (PMSerial.available() < 4) return;

if (PMSerial.read() != 0xA5) return;

uint8\_t zero = PMSerial.read();

uint8\_t low  = PMSerial.read();

uint8\_t high = PMSerial.read();

// Fórmula modo simple

int pm25 = low + (high - 0x2F);

if(pm25 < 0) pm25 = 0;

// -----------------------

// Aplicar filtro exponencial

// -----------------------

if(pm25\_suave == 0) pm25\_suave = pm25; // inicializar

pm25\_suave = alpha \* pm25 + (1 - alpha) \* pm25\_suave;

int pm25\_filtrado = (int)pm25\_suave;

// Serial monitor

Serial.print("PM2.5 = ");

Serial.print(pm25\_filtrado);

Serial.println(" ug/m3");

// LCD

lcd.setCursor(0,0);

lcd.print("                "); // limpiar línea

lcd.setCursor(0,0);

lcd.print("PM2.5: ");

lcd.print(pm25\_filtrado);

lcd.print(" ug/m3");

delay(1000); // actualizar cada segundo

}

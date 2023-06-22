# Smart-Energy-Meter
Iot Smart Energy Meter

S.N.	 COMPONENTS	       QUANTITY	
1)	   ESP32 WiFi Module	 1,	
2)	   ZMPT101B AC Voltage Sensor Module	1,	
3)	   SCT-013-030 Non-invasive AC Current Sensor	1,
4)	    16x2 LCD Display	1	,
5)	   Potentiometer 10K	1	,
6)	   Resistor 10K	2	,
7)	   Resistor 100ohm	1,
8)    	Capacitor 10uF	1	,
9)    Connecting Wires	10,	
10)   	Breadboard	1	


Code-
#include <LiquidCrystal.h>
LiquidCrystal lcd(13, 12, 14, 27, 26, 25);
#define BLYNK_PRINT Serial
#include "EmonLib.h"
#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>
 
EnergyMonitor emon;
 
#define vCalibration 83.3
#define currCalibration 0.50
 
BlynkTimer timer;
 
char auth[] = "hsYG_5da4gdP9jZkL18O5RNcJSrBT-Ou";
 
char ssid[] = "Alexahome";
char pass[] = "loranthus";
 
float kWh = 0;
unsigned long lastmillis = millis();
 
void myTimerEvent()
{
  emon.calcVI(20, 2000);
  kWh = kWh + emon.apparentPower * (millis() - lastmillis) / 3600000000.0;
  yield();
  Serial.print("Vrms: ");
  Serial.print(emon.Vrms, 2);
  Serial.print("V");
 
  Serial.print("\tIrms: ");
  Serial.print(emon.Irms, 4);
  Serial.print("A");
 
  Serial.print("\tPower: ");
  Serial.print(emon.apparentPower, 4);
  Serial.print("W");
 
  Serial.print("\tkWh: ");
  Serial.print(kWh, 5);
  Serial.println("kWh");
  
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Vrms:");
  lcd.print(emon.Vrms, 2);
  lcd.print("V");
  lcd.setCursor(0, 1);
  lcd.print("Irms:");
  lcd.print(emon.Irms, 4);
  lcd.print("A");
  delay(2500);
  
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Power:");
  lcd.print(emon.apparentPower, 4);
  lcd.print("W");
  lcd.setCursor(0, 1);
  lcd.print("kWh:");
  lcd.print(kWh, 4);
  lcd.print("W");
  delay(2500);
 
  lastmillis = millis();
 
  Blynk.virtualWrite(V0, emon.Vrms);
  Blynk.virtualWrite(V1, emon.Irms);
  Blynk.virtualWrite(V2, emon.apparentPower);
  Blynk.virtualWrite(V3, kWh);
}
 
void setup()
{
  Serial.begin(9600);
  Blynk.begin(auth, ssid, pass);
  lcd.begin(16, 2);
 
  emon.voltage(35, vCalibration, 1.7); // Voltage: input pin, calibration, phase_shift
  emon.current(34, currCalibration); // Current: input pin, calibration.
 
  timer.setInterval(5000L, myTimerEvent);
  lcd.setCursor(3, 0);
  lcd.print("IoT Energy");
  lcd.setCursor(5, 1);
  lcd.print("Meter");
  delay(3000);
  lcd.clear();
}
 
void loop()
{
  Blynk.run();
  timer.run();
}

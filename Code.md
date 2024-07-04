#include <Wire.h>

#include <Adafruit_Sensor.h>

#include <Adafruit_ADXL345_U.h>

#include <OneWire.h>

#include <LiquidCrystal.h> 

#include <SoftwareSerial.h>

SoftwareSerial gps = SoftwareSerial(2,3); // RX, TX

Adafruit_ADXL345_Unified accel = Adafruit_ADXL345_Unified(12345);

LiquidCrystal lcd(8,9,4,5,6,7);

int x=0;

char ch;

char str[70];

String gpsString="";

char *test="$GNGGA"; 

String latitude="No Range ";

String longitude="No Range ";

float latr=0;

float lonr=0;

int i;
boolean gps_status=0;

String stat="";

int distance=0;

unsigned long int duration=0;

int motor=10;

int buz=11;

void setup(void)

{

 lcd.begin(16,2);

 gps.begin(9600);

 Serial.begin(9600);

 pinMode(motor,OUTPUT);

 pinMode(buz,OUTPUT);

 digitalWrite(motor,LOW);

 digitalWrite(buz,LOW);

 lcd.setCursor(0,0);

 lcd.print("Accident Rescue");

 lcd.setCursor(0,1);

 lcd.print(" Detection ");

 delay(2000);

 lcd.clear();

 lcd.print("Waiting for GPS");
lcd.setCursor(0,1);

 lcd.print(" Signal ");

 lcd.clear();

 lcd.print("Lat :");

 lcd.setCursor(0,1);

 lcd.print("Lon :");

 get_gps();

 get_gps();

 delay(3000);

 lcd.clear();

 lcd.setCursor(0,0);

 lcd.print(" GSM Loading!..");

 delay(5000);

 if (!accel.begin()) 

 {

 lcd.begin(16, 2);

 lcd.setCursor(0, 0);

 lcd.print("ADXL345 not found");

 while (1); 

 }

 lcd.clear();

 lcd.setCursor(0,0);
lcd.print("Mems: ");

 digitalWrite(motor,HIGH); 

}

void loop(void)

{ 

 sensors_event_t event;

 accel.getEvent(&event);

 x = event.acceleration.x;

 int acceleration_magnitude = sqrt(pow(event.acceleration.x, 2));

 delay(1000);

 lcd.setCursor(5, 0);

 lcd.print(" ");

 if (x <= 0)

 x = 0;

 lcd.setCursor(5, 0);

 lcd.print(x);

 if (acceleration_magnitude >= 8) 

 {

 delay(2000);

 lcd.clear(); 

 lcd.print("Lat :");

 lcd.setCursor(0,1);
lcd.print("Lon :");

 get_gps();

 get_gps();

 lcd.clear(); 

 lcd.print("Lat :");

 lcd.setCursor(0,1);

 lcd.print("Lon :");

 lcd.clear();

 lcd.setCursor(0,0);

 lcd.print(" Accident ");

 lcd.setCursor(0,1);

 lcd.print(" Detection ");

 digitalWrite(motor,LOW);

 digitalWrite(buz,HIGH);

 delay(5000);

 digitalWrite(buz,LOW);

 sendGPS("Vehicle Accident Detected !!!");

 while(1);

 }

delay(500); 

}
void sendGPS(String msg)

{

 lcd.setCursor(15, 1);

 lcd.print("S");

 Serial.println("AT+\r");

 delay(1000);

 Serial.println("AT+CMGF=1\r");

 delay(2000);

 Serial.println("AT+CMGS=\"phone number\"\r");

 delay(1000);

 Serial.println(msg);

 delay(1000);

Serial.print("https://maps.google.com/maps?q=");

Serial.print(latr,6);

Serial.print(",");

Serial.print (lonr,6);

Serial.print("&hl=es;z=14&amp;output=embed\r");

 delay(1000);

 Serial.write(26);

 lcd.setCursor(15,1);

 lcd.print(" ");

 }
void gpsEvent()

{

 gpsString="";

 while(1)

 {

 while (Serial.available()>0) //checking serial data from GPS

 {

 char inChar = (char)Serial.read();

 gpsString+= inChar; //store data from GPS into gpsString

 i++;

 if (i < 7) 

 {

 if(gpsString[i-1] != test[i-1]) //checking for $GNGGA sentence

 {

 i=0;

 gpsString="";

 }

 }

 }

 if(gps_status)

 break;

 }
}

void get_gps()

{

 gps_status=0;

 int x=0;

 while(gps_status==0)

 {

 gpsEvent();

 int str_lenth=i;

 latitude="";

 longitude="";

 int comma=0;

 while(x<str_lenth)

 {

 if(gpsString[x]==',')

 comma+=1;

 if(comma==2) //extract latitude from string

 latitude+=gpsString[x+1]; 

 else if(comma==4) //extract longitude from string

 longitude+=gpsString[x+1];

 x++;

 }
Serial.print(longitude);

 Serial.print(latitude);

 int l1=latitude.length();

 latitude[l1-1]=' ';

 l1=longitude.length();

 longitude[l1-1]=' ';

 convert(latitude,longitude);

 lcd.setCursor(5,0);

 lcd.print(latitude);

 lcd.setCursor(5,1);

 lcd.print(longitude);

 i=0;x=0;

 str_lenth=0;

 delay(2000);

 }

}

/*
 * GccApplication20.c
 *
 * Created: 1/18/2024 11:13:37 AM
 * Author : Dell
 */ 

#include <SPI.h>
#include <SD.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SH110X.h>

#define LMS  A4
#define LS   A3
#define CS   A2
#define RS   A1
#define RMS  A0
#define enA  10
#define enB  11
#define MotorAip1  4
#define MotorAip2  5
#define MotorBip1  3
#define MotorBip2  17

#define i2c_Address  0x3c
#define screen_width  128
#define screen_height  64
#define OLED_RESET     -1

#define Current_sensor A6
#define Voltage_sensor A7

#define right_Encoder  27
#define left_Encoder   26

#define Ultra_Trigger  7
#define Ultra_Echo     6

const int chipSelect = 53;
float current = 0.00;
float voltage = 0.00;
int Obstacle_distance = 0;
int rpm_right = 0;
int rpm_left = 0;
int RPM = 0;

Adafruit_SH1106G display = Adafruit_SH1106G(screen_width,screen_height,&Wire,OLED_RESET);
void setup()
{
	display.begin(i2c_Address,true);
	display.clearDisplay();
	display.setTextSize(1);
	display.setTextColor(SH110X_WHITE);
	display.setCursor(0, 10);
	display.println("Initializing SD card.");
	display.display();
	delay(1000);
	if (!SD.begin(chipSelect))
	{
		display.println("Card failed, or not present");
		display.display();
		delay(2000);
	}
	else
	{
		display.println("Card initialized.");
		display.display();
		delay(3000);
	}
	
	pinMode(enA, OUTPUT);
	pinMode(enB, OUTPUT);
	pinMode(LMS, INPUT);
	pinMode(LS,  INPUT);
	pinMode(CS,  INPUT);
	pinMode(RS,  INPUT);
	pinMode(RMS, INPUT);
	pinMode(MotorAip1,OUTPUT);
	pinMode(MotorAip2,OUTPUT);
	pinMode(MotorBip1,OUTPUT);
	pinMode(MotorBip2,OUTPUT);
	analogWrite(enA, 100);
	analogWrite(enB, 100);

	pinMode(Current_sensor, INPUT);
	pinMode(Voltage_sensor, INPUT);
	pinMode(Ultra_Echo, INPUT);
	pinMode(Ultra_Trigger, OUTPUT);
	pinMode(right_Encoder, INPUT_PULLUP);
	pinMode(left_Encoder, INPUT_PULLUP);
	
	display.clearDisplay();
	display.setTextSize(1.8);
	display.setTextColor(SH110X_WHITE);
	display.setCursor(20, 15);
	display.println("Ready to Go..!!!");
	display.display();
}

void loop()
{
	if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==0 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	forward();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==0 && digitalRead(CS)==0 && digitalRead(RS)==0 && digitalRead(RMS)==1)
	forward();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==0 && digitalRead(CS)==1 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	left();
	else if(digitalRead(LMS)==0 && digitalRead(LS)==1 && digitalRead(CS)==1 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	left();
	else if(digitalRead(LMS)==0 && digitalRead(LS)==0 && digitalRead(CS)==1 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	left();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==0 && digitalRead(CS)==0 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	left();
	else if(digitalRead(LMS)==0 && digitalRead(LS)==0 && digitalRead(CS)==0 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	left();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==1 && digitalRead(RS)==0 && digitalRead(RMS)==1)
	right();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==1 && digitalRead(RS)==1 && digitalRead(RMS)==0)
	right();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==1 && digitalRead(RS)==0 && digitalRead(RMS)==0)
	right();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==0 && digitalRead(RS)==0 && digitalRead(RMS)==1)
	right();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==0 && digitalRead(RS)==0 && digitalRead(RMS)==0)
	right();
	else
	stop();
}

void  Line_follower()
{
	if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==0 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	forward();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==0 && digitalRead(CS)==0 && digitalRead(RS)==0 && digitalRead(RMS)==1)
	forward();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==0 && digitalRead(CS)==1 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	left();
	else if(digitalRead(LMS)==0 && digitalRead(LS)==1 && digitalRead(CS)==1 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	left();
	else if(digitalRead(LMS)==0 && digitalRead(LS)==0 && digitalRead(CS)==1 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	left();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==0 && digitalRead(CS)==0 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	left();
	else if(digitalRead(LMS)==0 && digitalRead(LS)==0 && digitalRead(CS)==0 && digitalRead(RS)==1 && digitalRead(RMS)==1)
	left();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==1 && digitalRead(RS)==0 && digitalRead(RMS)==1)
	right();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==1 && digitalRead(RS)==1 && digitalRead(RMS)==0)
	right();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==1 && digitalRead(RS)==0 && digitalRead(RMS)==0)
	right();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==0 && digitalRead(RS)==0 && digitalRead(RMS)==1)
	right();
	else if(digitalRead(LMS)==1 && digitalRead(LS)==1 && digitalRead(CS)==0 && digitalRead(RS)==0 && digitalRead(RMS)==0)
	right();
	else
	stop();
}

void forward()
{
	
	digitalWrite(MotorAip1, HIGH);
	digitalWrite(MotorAip2, LOW);
	digitalWrite(MotorBip1, LOW);
	digitalWrite(MotorBip2, HIGH);
}

void left()
{
	
	
	digitalWrite(MotorAip1, HIGH);
	digitalWrite(MotorAip2, LOW);
	digitalWrite(MotorBip1, LOW);
	digitalWrite(MotorBip2, LOW);
}

void right()
{
	
	
	digitalWrite(MotorAip1, LOW);
	digitalWrite(MotorAip2, LOW);
	digitalWrite(MotorBip1, LOW);
	digitalWrite(MotorBip2, HIGH);
}

void reverse()
{
	digitalWrite(MotorAip1, LOW);
	digitalWrite(MotorAip2, HIGH);
	digitalWrite(MotorBip1, HIGH);
	digitalWrite(MotorBip2, LOW);
}

void stop()
{
	digitalWrite(MotorAip1, LOW);
	digitalWrite(MotorAip2, LOW);
	digitalWrite(MotorBip1, LOW);
	digitalWrite(MotorBip2, LOW);
	voltage = Voltage();
	current = Current();
	Obstacle_distance = Ultrasonic();
	display_data();
	
}

float Current()
{
	unsigned int x=0;
	float AcsValue=0.0,Samples=0.0,AvgAcs=0.0,AcsValueF=0.0;
	for (int x = 0; x < 150; x++)
	{
		AcsValue = analogRead(Current_sensor);
		Samples = Samples + AcsValue;
		delay (3);
	}
	AvgAcs=Samples/150.0;
	AcsValueF = (-2.5 + (AvgAcs * (5.0 / 1024.0)) )/0.185;
	delay(50);
	return AcsValueF;
}

float Voltage()
{
	float adc_voltage = 0.0;
	float in_voltage = 0.0;
	float R1 = 30000.0;
	float R2 = 7500.0;
	float ref_voltage = 5.0;
	int adc_value = 0;
	
	adc_value = analogRead(Voltage_sensor);
	adc_voltage  = (adc_value * ref_voltage) / 1024.0;
	in_voltage = (adc_voltage / (R2/(R1+R2)))-3.23 ;
	return in_voltage;
}

int Ultrasonic()
{
	long Duration;
	int Distance;
	digitalWrite(Ultra_Trigger, LOW);
	delayMicroseconds(2);
	digitalWrite(Ultra_Trigger, HIGH);
	delayMicroseconds(10);
	digitalWrite(Ultra_Trigger, LOW);
	Duration = pulseIn(Ultra_Echo, HIGH);
	Distance = Duration * 0.034 / 2;
	return Distance;
}

float Encoder_right()
{
	unsigned long start_time = 0;
	unsigned long end_time = 0;
	int steps=0;
	float steps_old=0;
	float temp=0;
	float rpm=0;
	start_time=millis();
	end_time=start_time+100;
	while(millis()<end_time)
	{
		if(digitalRead(right_Encoder))
		{
			steps=steps+1;
			while(digitalRead(right_Encoder));
		}
	}
	temp=steps-steps_old;
	steps_old=steps;
	rpm=(temp/20)*10*60;
	return rpm;
}

float Encoder_left()
{
	unsigned long start_time = 0;
	unsigned long end_time = 0;
	int steps=0;
	float steps_old=0;
	float temp=0;
	float rpm=0;
	start_time=millis();
	end_time=start_time+100;
	while(millis()<end_time)
	{
		if(digitalRead(left_Encoder))
		{
			steps=steps+1;
			while(digitalRead(left_Encoder));
		}
	}
	temp=steps-steps_old;
	steps_old=steps;
	rpm=(temp/20)*10*60;
	return rpm;
}
int average_rpm(int rpm1, int rpm2)
{
	return (rpm1+rpm2)/2;
}

void SD_card()
{
	// make a string for assembling the data to log:
	String dataString = "";

	// read three sensors and append to the string:
	for (int analogPin = 0; analogPin < 3; analogPin++) {
		int sensor = analogRead(analogPin);
		dataString += String(sensor);
		if (analogPin < 2) {
			dataString += ",";
		}
	}

	// open the file. note that only one file can be open at a time,
	// so you have to close this one before opening another.
	File dataFile = SD.open("datalog.txt", FILE_WRITE);

	// if the file is available, write to it:
	if (dataFile) {
		dataFile.println(dataString);
		dataFile.close();
		// print to the serial port too:
		Serial.println(dataString);
	}
	// if the file isn't open, pop up an error:
	else {
		Serial.println("error opening datalog.txt");
	}
}
void forward_indication()
{
	display.clearDisplay();
	display.setCursor(0, 10);
	display.writeLine(64,5,54,20,SH110X_WHITE);
	display.writeLine(64,5,74,20,SH110X_WHITE);
	display.writeLine(64,10,54,25,SH110X_WHITE);
	display.writeLine(64,10,74,25,SH110X_WHITE);
	display.writeLine(64,15,54,30,SH110X_WHITE);
	display.writeLine(64,15,74,30,SH110X_WHITE);
	display.display();
}
void left_indication()
{
	display.clearDisplay();
	display.setCursor(0, 10);
	display.writeLine(64,5,49,15,SH110X_WHITE);
	display.writeLine(64,25,49,15,SH110X_WHITE);
	display.writeLine(59,5,44,15,SH110X_WHITE);
	display.writeLine(59,25,44,15,SH110X_WHITE);
	display.writeLine(54,5,39,15,SH110X_WHITE);
	display.writeLine(54,25,39,15,SH110X_WHITE);
	display.display();
}
void right_indication()
{
	display.clearDisplay();
	display.setCursor(0, 10);
	display.writeLine(64,5,79,15,SH110X_WHITE);
	display.writeLine(64,25,79,15,SH110X_WHITE);
	display.writeLine(69,5,84,15,SH110X_WHITE);
	display.writeLine(69,25,84,15,SH110X_WHITE);
	display.writeLine(74,5,89,15,SH110X_WHITE);
	display.writeLine(74,25,89,15,SH110X_WHITE);
	display.display();
}
void display_RPM(int value)
{
	display.setTextSize(2.2);
	display.setTextColor(SH110X_WHITE);
	display.setCursor(35, 40);
	display.print(value);
	display.setTextSize(1);
	display.setCursor(100, 40);
	display.print("rpm");
	display.display();
}

void display_data()
{
	display.clearDisplay();
	display.setCursor(10, 20);
	display.setTextSize(0.5);
	display.setTextColor(SH110X_WHITE);
	display.print("Voltage: ");
	display.print(voltage, 2);
	display.print(" V");
	display.setCursor(10, 30);
	display.print("Current: ");
	display.print(current, 2);
	display.print(" mA");
	display.setCursor(10, 40);
	display.print("Obstacle: ");
	display.print(Obstacle_distance, 10);
	display.print(" cm");
	display.display();
}


// Importing Libraries        
#include <Wire.h> //For I2C

#include <Servo.h> //For Servo 

#include "pitches.h" //Header file with pitch definitions

//Initalizing Variables

Servo myServo;

const int motor_pin = 3; 

int temp_address = 72;

int f;   // Variable to store Fahrenheit temperature

int pressCount = 0; //buton pressed amount

`const int BUTTON=2;

int Servopos =0; //Servo position

boolean lastButton = LOW;    // Variable containing the previous 

boolean currentButton = LOW; // Variable containing the current 

boolean ledOn = false;       // The present state of the LED (on/off)

const int SPEAKER=7; //Speaker Pin

const int WLED=4;        // White LED Anode on pin 4 

const int LIGHT=0;       // Light Sensor on Analog Pin 0

const int MIN_LIGHT=200; // Minimum Expected light value

const int MAX_LIGHT=900; // Maximm Expected Light value

int val = 0;             // Variable to hold the analog reading



// Reading Tempature output user-fucntion

void tempRead()

{

 analogWrite(motor_pin,0); //stopping motor
 
 delay(300); //dealy 0.3s

 
 
 Wire.beginTransmission(temp_address); //gathering temp
 
  Wire.write(0);
  
  Wire.endTransmission();


  Wire.requestFrom(temp_address, 1);

  int c = Wire.read();        // Read Celsius

  f = round(c * 9.0 / 5.0 +
  32.0);  // Convert to Fahrenheit

delay(50);

}

//Button debounce user-function

boolean debounce(boolean last)

{

 boolean current = digitalRead(BUTTON);   // Read the button state
 
 if (last != current)                     // if it's different…
 
 {
 
  delay(5);                               //Wait 5ms 
  
  current = digitalRead(BUTTON);          //Read it again
 
 }
 
 return current;                          //Return the current value

}

//Buttin being pressed function

void BPC()

{

currentButton = debounce(lastButton);                            //Read debounced state

 if (lastButton == LOW && currentButton == HIGH) //if it was pressed…

 {
  pressCount++;
 
  ledOn = !ledOn;                                                 //Toggle the LED value
 
 }
 
 lastButton = currentButton;                                      //Reset button value

}

//Reading button while DC motor on user-function

void waitAndReadButton(unsigned long duration) {

  unsigned long startTime = millis();
  
  while (millis() - startTime < duration) //While loop that runs only for durration of fan on
  
  {
  
    BPC(); // Keep checking the button during the wait
 
  }

}

//Photoresistor and LED turn on user-function

void nightLight()

{

val = analogRead(LIGHT);                      // Read the light sensor

 val = map(val, MIN_LIGHT, MAX_LIGHT, 255, 0); // Map the light reading
 
 val = constrain(val, 0, 255);                 // Constrain light value
 
 if (val >= 100) // If dark
 
 {
 
  digitalWrite(WLED,HIGH); //turn on LED
  
  tone(SPEAKER,NOTE_A4,250); //Playing noise on speaker

 }
 
 else 

{

  digitalWrite(WLED,LOW); //turning LED off

}

}

void setup()

{

  Wire.begin(); //Startign Wire for I2C
 
  Serial.begin(9600);
  
  pinMode(motor_pin, OUTPUT); //Motor pin as output
  
   pinMode (BUTTON, INPUT);    // Set button as input 
   
    myServo.attach(10);
    
    pinMode(WLED, OUTPUT);  // Set White LED pin as output



}

void loop()

{
 
  nightLight(); //Getting light value

tempRead();; //getting temp value

Serial.println(f);

if (f >=75) //if temp threshold is met

{

BPC(); //read button
 

if (Servopos >= 91) //Move serovo back to start

{

  Servopos = 0;

}

if (Servopos <= 90) //Oscillating the Servo

{

    myServo.write(Servopos);   // Move to the next degree
    
    Servopos = Servopos + 15; //Add the other degree

}
 
if (pressCount ==0) //Defult fan speed

{

    analogWrite(motor_pin,127); //turn fan on at 50%
    
    BPC(); //read button
    
    waitAndReadButton(5000); //Delay 5s

}

if (pressCount ==1) //Next PWM level

{

    analogWrite(motor_pin,191); //turn fan on at 75%
    
    BPC();//read button
    
    waitAndReadButton(5000);//Delay 5s

}

if (pressCount ==2) //Next PWM level

{

    analogWrite(motor_pin,254); //turn fan on at 100%
    
    BPC();//read button
    
    waitAndReadButton(5000);//Delay 5s

}

if (pressCount == 3) //Resetting button count to go back to defult speed

{

pressCount = 0;

}
}
}`

# bbacudo
# Please subscribe my Channel on Youtube and like my videos
# My channel's name is "BER NA" or search my name "Bernadette Bacudo"


#include <SoftwareSerial.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Servo.h>


#define sw1_pin 11
#define sw2_pin 12

Servo gwapa;
Servo servo;

SoftwareSerial GSM900 (7,8);
LiquidCrystal_I2C lcd (0x3F , 2, 1, 0, 4, 5, 6, 7, 3, POSITIVE);

const int BUZZER = 3;
const int RED = 6;   
const int GREEN = 5; // led indicator connected to digital pin
const int knockSensor = A0; // the piezo is connected to an analog pin
const int thresholdHIGH = 100;  // threshold value to decide when the detected knock is hard (HIGH)
const int thresholdLOW = 80;  // threshold value to decide when the detected knock is gentle (LOW)
const int secretKnockLength = 4; //How many knocks are in your secret knock
const int secretKnock[secretKnockLength] = {1, 1, 1, 1};
 
int secretCounter = 0; //this tracks the correct knocks and allows you to move through the sequence
int sensorReading = 0; // variable to store the value read from the sensor pin

volatile boolean sw1 = false;
volatile boolean sw2 = false;

uint8_t sw1ButtonState = 0;
uint8_t sw2ButtonState = 0;
uint8_t lastsw1ButtonState = 0;
uint8_t lastsw2ButtonState = 0;


void setup() {
  Serial.begin(9600);
  gwapa.attach(9);
  gwapa.write(0);
  servo.attach(10);
  lcd.begin(16,2);
  lcd.print("     LOCKED");
  pinMode(sw1_pin, INPUT_PULLUP);
  pinMode(sw2_pin, INPUT_PULLUP);
  pinMode(RED, OUTPUT);
  pinMode(GREEN,OUTPUT);
  pinMode(BUZZER, OUTPUT);

  GSM900.begin (19200);
  delay(20000); // give time to your GSM shield log on to network


}


 void but()
 {
 checkIfSw1ButtonIsPressed();
 checkIfSw2ButtonIsPressed();

     if( sw1){
       lcd.clear();
       lcd.print("      OPEN");
       sw1 = false;
       servo.write(0);
       delay(15);
    
       
     }
     else if( sw2){
      lcd.clear();
      lcd.print("     LOCKED");
      sw2 = false;
      servo.write(90);
      delay(15);
      }
    // waits for the servo to get there
 }

void checkIfSw1ButtonIsPressed()
    {
      sw1ButtonState   = digitalRead(sw1_pin);
        if (sw1ButtonState != lastsw1ButtonState)
          {
          if ( sw1ButtonState == 0)
            {
             sw1=true;
            }
              delay(50);
          }
        lastsw1ButtonState = sw1ButtonState;
     }

void checkIfSw2ButtonIsPressed()
     {
       sw2ButtonState   = digitalRead(sw2_pin);
        if (sw2ButtonState != lastsw2ButtonState)
         {
          if ( sw2ButtonState == 0)
            {
             sw2=true;
            }
          delay(50);
         }
      lastsw2ButtonState = sw2ButtonState;
     }



void sendSMS(){ //AT comand to set sim900 to sms mode
 GSM900.print("AT+CMGF=1\r");
 delay(100);

 GSM900.println("AT+CMGS = \"+639456288414\"");
 delay(100);

 GSM900.println("MASTER, AN ATTEMPT HAS BEEN MADE");
 delay(100);

 GSM900.println((char)26);  //end AT command with a ^Z, ASCII code 26
 delay(100);

 GSM900.println();
 delay(5000); //give time to send sms 
}

 
void loop() {

 but();
 
  sensorReading = analogRead(knockSensor);// read the piezo sensor and store the value in the variable sensorReading:
  
  if (sensorReading >= thresholdHIGH) { // First determine is knock if Hard (HIGH) or Gentle (LOW) //Hard knock (HIGH) is detected //Check to see if a Hard Knock matches the Secret Knock in the correct sequence.
    if (secretKnock[secretCounter] == 1) {
 
      secretCounter++;//The Knock was correct, iterate the counter.
      lcd.clear();
      lcd.print("    CORRECT");
      
      tone(BUZZER, 200);
      delay(150);
   
      noTone(BUZZER);
      digitalWrite(GREEN, HIGH);
      delay(250);
      digitalWrite(GREEN, LOW);
      lcd.clear();
 
      } 
      
    else {  //The Knock was incorrect, reset the counter

      
      secretCounter = 0;
      sendSMS(); //send the sms
      
      lcd.clear();
      lcd.print("     ALERT!");    
      tone(BUZZER, 493.883);
      delay(300);
      noTone(BUZZER);
      delay(300);
      lcd.clear();
      digitalWrite(RED , HIGH);
      delay(100);
      digitalWrite(RED , LOW);
      delay(100);

      lcd.print("     ALERT!");      
      tone(BUZZER, 493.883);
      delay(300);
      noTone(BUZZER);
      delay(300);
      lcd.clear();
      digitalWrite(RED , HIGH);
      delay(100);
      digitalWrite(RED , LOW);
      delay(100);

 
      lcd.print("     ALERT!");
      tone(BUZZER, 493.883);
      delay(300);
      noTone(BUZZER);
      delay(300);
      lcd.clear();
      digitalWrite(RED , HIGH);
      delay(100);
      digitalWrite(RED , LOW);
      delay(100);
      
      lcd.print("     ALERT!");
      tone(BUZZER, 493.883);
      delay(300);
      noTone(BUZZER);
      delay(300);
      digitalWrite(RED , HIGH);
      delay(100);
      digitalWrite(RED , LOW);
      delay(100); 
      lcd.clear();
 
      lcd.print("     LOCKED");
    }//close if
    delay(100);     //Allow some time to pass before sampling again to ensure a clear signal.  //Gentle knock (LOW) is detected                
  }
  
 else if (sensorReading >= thresholdLOW) {        //Check to see if a Gentle Knock matches the Secret Knock in the correct sequence.
      if (secretKnock[secretCounter] == 0) {
      
       secretCounter++;     //The Knock was correct, iterate the counter.
     
       lcd.clear();
       lcd.print("     CORRECT");
       tone(BUZZER, 200);
       delay(150);
       noTone(BUZZER);
       digitalWrite(GREEN, HIGH);
       delay(250);
       digitalWrite(GREEN, LOW);
       delay(500);
       lcd.clear();
     
     } 
     else {     //The Knock was incorrect, reset the counter.      
      secretCounter = 0;
      sendSMS(); //send the sms
      lcd.clear();
      lcd.print("     ALERT!");
      tone(BUZZER, 493.883);
      delay(300);
      noTone(BUZZER);
      delay(300);
 
      lcd.clear();
      digitalWrite(RED , HIGH);
      delay(100);
      digitalWrite(RED , LOW);
      delay(100);
     
      lcd.print("     ALERT!");
      tone(BUZZER, 493.883);
      delay(300);
      noTone(BUZZER);
      delay(300);

      lcd.clear();
      digitalWrite(RED , HIGH);
      delay(100);
      digitalWrite(RED , LOW);
      delay(100);

  
      lcd.print("     ALERT!");
      tone(BUZZER, 493.883);
      delay(300);
      noTone(BUZZER);
      delay(300);

      lcd.clear();
      digitalWrite(RED , HIGH);
      delay(100);
      digitalWrite(RED , LOW);
      delay(100);

  
      lcd.print("     ALERT!"); 
      tone(BUZZER, 493.883);
      delay(300);
      noTone(BUZZER);
      delay(300);


      digitalWrite(RED , HIGH);
      delay(100);
      digitalWrite(RED , LOW);
      delay(100);
      lcd.clear();
 
      lcd.print("     LOCKED");
      }//close if
 
      delay(100);
     }//close if else
 
 if (secretCounter == (secretKnockLength) ) {   
 
     servo.write(90);
             
     lcd.clear();
     lcd.print(" WELCOME MASTER");
     delay(10000);
     servo.write(0);
     lcd.clear();
     lcd.print("     LOCKED");

     secretCounter = 0;
       
     }//close success check
}//close loop



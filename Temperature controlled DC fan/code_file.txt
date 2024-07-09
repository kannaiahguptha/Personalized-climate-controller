// Necessary libraries included
#include <IRremote.hpp>
#include "LiquidCrystal.h"
#include <DHT.h>


// DHT Sensor defined
#define DHTPIN 7  
#define DHTTYPE DHT22 // DHT11

DHT dht(DHTPIN, DHTTYPE);

// LCD code defined
LiquidCrystal lcd(8, 12, 6, 5, 4, 3);
#define pwm 9

// IR defined 
#define IR_RECEIVE_PIN 2


//unsigned long receivedValue = 0;
String receivedValue = "";
String lastReceivedValue= ""; // To store the last received IR command

//Debouncing 
unsigned long lastReceiveTime = 0 ;
const unsigned long debounceDelay = 500 ; 



String necCodeButton1 = "47991122"; // for 1 and 2 keys to control User1 Profile
String necCodeButton2 = "cf6a91da"; // for 1 and 2 keys to control User1 Profile
String necCodeButton3 = "bfcb8920"; // for myRec key to control User2 Profile
String necCodeButton4 = "479d09d8"; // for myRec key to control User2 Profile
String necCodeButton5 = "a38004d4"; // for iTV key to control General Profile
String necCodeButton6 = "2b51858c"; // for iTV key to control General Profile
String necCodeButton7 = "d16a9500"; // for menu key to control Manual Profile
String necCodeButton8 = "49991448"; // for menu key to control Manual Profile 


void setup() {
  Serial.begin(9600);

  lcd.begin(16, 2);
  IrReceiver.begin(IR_RECEIVE_PIN, RC5); // Start the reciever with RC5 protocol 

  // DHT setup
  Serial.println("DHT11 Test");
  dht.begin() ;

}

void loop() {

  float temperatureC = dht.readTemperature();
  float humidity = dht.readHumidity();



  //int reading = analogRead(sensorPin);
  //float voltage = reading * 4.68 / 1024.0;
  //float temperatureC = (voltage - 0.5) * 100;

  lcd.setCursor(0, 0);
  lcd.print("temp: ");
  lcd.print(temperatureC); // Printing temperature on LCD
  lcd.print(" C");
  lcd.setCursor(0, 1);

  //debugging DHT11

  if ( isnan(temperatureC)  || isnan(humidity)){
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  Serial.print("Temperature: ");
  Serial.print(temperatureC);
  Serial.print("°C\t");
  Serial.print("Humidity:");
  Serial.print(humidity);
  Serial.print("%");



  // Check for changes in received IR code
  if (IrReceiver.decode()) {
    unsigned long currentTime = millis() ;
    if (currentTime - lastReceiveTime > debounceDelay){
       // print the received raw data
       Serial.print("Raw Data : ");
       Serial.println(IrReceiver.decodedIRData.decodedRawData, HEX);

receivedValue = String(IrReceiver.decodedIRData.decodedRawData, HEX);
 lastReceivedValue = receivedValue;

   // Print the received value 
    Serial.print("Received Value :");
    Serial.println(receivedValue);

    // Print teh decoded IR result in one line 
    IrReceiver.printIRResultShort(&Serial);

    IrReceiver.printIRSendUsage(&Serial);

    lastReceiveTime = currentTime; 

    }
    

    // Enable receiving of the next value
    IrReceiver.resume();
  }

  // Execute the corresponding user profile based on the last received IR command
  if (lastReceivedValue.equals(necCodeButton1) || lastReceivedValue.equals(necCodeButton2)  ) {
    userProfile1(temperatureC);
  } else if (lastReceivedValue.equals(necCodeButton3) || lastReceivedValue.equals(necCodeButton4)) {
    userProfile2(temperatureC);
  } else if (lastReceivedValue.equals(necCodeButton5) || lastReceivedValue.equals(necCodeButton6)) {
    generalProfile(temperatureC);
  } else if (lastReceivedValue.equals(necCodeButton7) || lastReceivedValue.equals(necCodeButton8)) {
     manualProfile(temperatureC);
  } 
  else {
    // If no specific profile is selected, use the manual profile (Potentiometer)
    manualProfile(temperatureC);
  }
}

void userProfile1(float temperatureC) {
  // USER1 Profile
  lcd.print("U1");
  if (temperatureC < 25) {
    analogWrite(pwm, 0);
    lcd.print("Fan OFF            ");
    delay(1000);
  } else if (temperatureC < 35) {
    analogWrite(pwm, 30);
    lcd.print("Fan Speed: 20%   ");
    delay(1000);
  } else if (temperatureC < 40) {
    analogWrite(pwm, 100);
    lcd.print("Fan Speed: 60%   ");
    delay(1000);
  } else if (temperatureC > 40) {
    analogWrite(pwm, 255);
    lcd.print("Fan Speed: 100%   ");
  }
}

void userProfile2(float temperatureC) {
  // USER2 Profile
  lcd.print("U2");
  if (temperatureC < 30) {
    analogWrite(pwm, 0);
    lcd.print("Fan OFF            ");
    delay(1000);
  } else if (temperatureC < 40) {
    analogWrite(pwm, 30);
    lcd.print("Fan Speed: 20%   ");
    delay(1000);
  } else if (temperatureC < 50) {
    analogWrite(pwm, 100);
    lcd.print("Fan Speed: 60%   ");
    delay(1000);
  } else if (temperatureC > 50) {
    analogWrite(pwm, 255);
    lcd.print("Fan Speed: 100%   ");
  }
}

// general profile :

void generalProfile(float temperatureC) {
  lcd.print("G");
  if (temperatureC < 25) {
    analogWrite(pwm, 0);
    lcd.print("Fan OFF            ");
    delay(1000);
  } else if (temperatureC < 30) {
    analogWrite(pwm, 30);
    lcd.print("Fan Speed: 20%   ");
    delay(1000);
  } else if (temperatureC < 40) {
    analogWrite(pwm, 100);
    lcd.print("Fan Speed: 60%   ");
    delay(1000);
  } else if (temperatureC < 50) {
    analogWrite(pwm, 100);
    lcd.print("Fan Speed: 80%   ");
    delay(1000);
  } else if (temperatureC > 50) {
    analogWrite(pwm, 255);
    lcd.print("Fan Speed: 100%   ");
    delay(1000);
  }
}

void manualProfile(float temperatureC) {
  // manual Profile
  lcd.print("M");
  int fanSpeed = map(analogRead(A1), 0, 1023, 0, 255); // Read potentiometer value and map it to fan speed range
  analogWrite(pwm, fanSpeed);
  if (fanSpeed == 0) {
    lcd.print("Fan Speed: 0% ");
    lcd.print("   ");
    delay(1000);
  } else if (fanSpeed < 50) {
    lcd.print("Fan Speed: 20% ");
    lcd.print("   ");
    delay(1000);
  } else if (fanSpeed < 100) {
    lcd.print("Fan Speed: 40% ");
    lcd.print("   ");
    delay(1000);
  } else if (fanSpeed < 150) {
    lcd.print("Fan Speed: 60% ");
    lcd.print("   ");
    delay(1000);
  } else if (fanSpeed < 200) {
    lcd.print("Fan Speed: 80% ");
    lcd.print("   ");
    delay(1000);
  } else {
    lcd.print("Fan Speed: 100% ");
    lcd.print("   ");
    delay(1000);
  }
}
Approach: Capture a 4-digit ID via keypad, generate a random 6-digit OTP, transmit it over Bluetooth (HC-05), and require the correct OTP within a non-blocking 30-second window before actuating a servo to "unlock."
cpp
#include <Keypad.h>
#include <LiquidCrystal.h>
#include <Servo.h>


LiquidCrystal lcd(12, 11, 5, 4, 3, 2);
Servo doorServo;


const byte ROWS = 4, COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {9, 8, 7, 6};
byte colPins[COLS] = {5, 4, 3, 2}; // adjust to avoid LCD pin conflicts in real wiring
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);


String enteredID = "";
String enteredOTP = "";
String generatedOTP = "";
bool waitingForOTP = false;
unsigned long otpStartTime = 0;
const unsigned long OTP_TIMEOUT = 30000;


void setup() {
  Serial.begin(115200); // HC-05 wired to hardware Serial or SoftwareSerial
  lcd.begin(16, 2);
  doorServo.attach(13);
  doorServo.write(0); // locked position
  randomSeed(analogRead(A0));
  lcd.print("Enter 4-digit ID");
}


String generateOTP() {
  int otp = random(100000, 999999);
  return String(otp);
}


void resetState() {
  delay(1500);
  enteredID = "";
  enteredOTP = "";
  waitingForOTP = false;
  lcd.clear();
  lcd.print("Enter 4-digit ID");
}


void loop() {
  char key = keypad.getKey();


  if (!waitingForOTP) {
    if (key) {
      if (key == '#') {
        if (enteredID.length() == 4) {
          generatedOTP = generateOTP();
          Serial.println("OTP:" + generatedOTP); // sent over Bluetooth to phone
          waitingForOTP = true;
          otpStartTime = millis();
          enteredOTP = "";
          lcd.clear();
          lcd.print("Enter OTP:");
        } else {
          lcd.clear();
          lcd.print("Invalid ID len");
          enteredID = "";
        }
      } else if (key >= '0' && key <= '9') {
        enteredID += key;
        lcd.setCursor(0, 1);
        lcd.print(String(enteredID.length()) + " digits*");
      }
    }
  } else {
    unsigned long elapsed = millis() - otpStartTime;
    if (elapsed >= OTP_TIMEOUT) {
      lcd.clear();
      lcd.print("OTP Expired");
      Serial.println("LOG: ID=" + enteredID + " OTP_TIMEOUT");
      resetState();
      return;
    }


    if (key) {
      if (key == '#') {
        if (enteredOTP == generatedOTP) {
          lcd.clear();
          lcd.print("Access Granted");
          doorServo.write(90); // unlock
          Serial.println("LOG: ID=" + enteredID + " ACCESS_GRANTED");
          delay(3000);
          doorServo.write(0);  // re-lock
        } else {
          lcd.clear();
          lcd.print("Access Denied");
          Serial.println("LOG: ID=" + enteredID + " ACCESS_DENIED");
        }
        resetState();
      } else if (key >= '0' && key <= '9') {
        enteredOTP += key;
        lcd.setCursor(0, 1);
        String masked = "";
        for (unsigned int i = 0; i < enteredOTP.length(); i++) masked += "*";
        lcd.print(masked);
      }
    }
  }
}

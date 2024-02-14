#include <Servo.h>
#include <SoftwareSerial.h>
#include <NewPing.h>

Servo myservo;
Servo servo;
SoftwareSerial sim800L(2, 3); //SIM800L Tx & Rx is connected to Arduino #2 & #3

#define Left 4          // left sensor
#define Right 5         // right sensor
#define Forward 6       // front sensor
#define GAS_SENSOR 7    // Gas sensor
#define LM1 8           // left motor
#define LM2 9           // left motor
#define RM1 10          // right motor
#define RM2 11          // right motor
#define pump 12         // water pump
#define TRIGGER_PIN A2
#define ECHO_PIN A3

const String PHONE = "+91**********"; // use your number with country code

NewPing sonar(TRIGGER_PIN, ECHO_PIN, 50);

int pos = 0;
boolean fire = false;

void setup() {
  Serial.begin(9600);
  sim800L.begin(9600);
  pinMode(Left, INPUT);
  pinMode(Right, INPUT);
  pinMode(Forward, INPUT);
  pinMode(GAS_SENSOR, INPUT);
  pinMode(LM1, OUTPUT);
  pinMode(LM2, OUTPUT);
  pinMode(RM1, OUTPUT);
  pinMode(RM2, OUTPUT);
  pinMode(pump, OUTPUT);
  myservo.attach(13);
  myservo.write(90);
  servo.attach(7);
  servo.write(0);
}

void loop() {
  myservo.write(90); // Sweep_Servo();
  
  if (digitalRead(Left) == 1 && digitalRead(Right) == 1 && digitalRead(Forward) == 1) {
    delay(500);
    digitalWrite(LM1, HIGH);
    digitalWrite(LM2, HIGH);
    digitalWrite(RM1, HIGH);
    digitalWrite(RM2, HIGH);
  }
  else if (digitalRead(Forward) == 0) {
    digitalWrite(LM1, HIGH);
    digitalWrite(LM2, LOW);
    digitalWrite(RM1, HIGH);
    digitalWrite(RM2, LOW);
    fire = true;
  }
  else if (digitalRead(Left) == 0) {
    digitalWrite(LM1, HIGH);
    digitalWrite(LM2, LOW);
    digitalWrite(RM1, HIGH);
    digitalWrite(RM2, HIGH);
  }
  else if (digitalRead(Right) == 0) {
    digitalWrite(LM1, HIGH);
    digitalWrite(LM2, HIGH);
    digitalWrite(RM1, HIGH);
    digitalWrite(RM2, LOW);
  }
  delay(400); // Change this value to change the distance

  if (digitalRead(GAS_SENSOR) == 0) {
    Serial.println("Gas is Detected.");
    send_sms();
  }

  while (fire == true) {
    put_off_fire();
    Serial.println("Fire Detected.");
    make_call();
  }
  
  int sensorReading = analogRead(A1);
  int range = map(sensorReading, 0, 1024, 0, 3);

  if (digitalRead(Left) == 1 && digitalRead(Right) == 1 && digitalRead(Forward) == 1) {
    stop();
  }
  else if (digitalRead(Forward) == 0) {
    switch (range) {
      case 0:
        Serial.println("** Close Fire **");
        fire = true;
        break;
      case 1:
        Serial.println("** Distant Fire **");
        objectAvoid();
        moveForward();
        sendSMS();
        break;
    }
  }
  else if (digitalRead(Left) == 0) {
    objectAvoid();
    moveLeft();
    sendSMS();
  }
  else if (digitalRead(Right) == 0) {
    objectAvoid();
    moveRight();
    sendSMS();
  }
  delay(300);
}

void put_off_fire() {
  digitalWrite(LM1, HIGH);
  digitalWrite(LM2, HIGH);
  digitalWrite(RM1, HIGH);
  digitalWrite(RM2, HIGH);
  digitalWrite(pump, HIGH);
  delay(500);

  for (pos = 50; pos <= 110; pos += 1) {
    myservo.write(pos);
    delay(10);
  }
  for (pos = 110; pos >= 50; pos -= 1) {
    myservo.write(pos);
    delay(10);
  }
  digitalWrite(pump, LOW);
  myservo.write(90);
  fire = false;
}

void make_call() {
  Serial.println("calling....");
  sim800L.println("ATD" + PHONE + ";");
  delay(20000); // 20 sec delay
  sim800L.println("ATH");
  delay(1000); // 1 sec delay
}

void send_sms() {
  Serial.println("sending sms....");
  delay(50);
  sim800L.print("AT+CMGF=1\r");
  delay(1000);
  sim800L.print("AT+CMGS=\"" + PHONE + "\"\r");
  delay(1000);
  sim800L.print("Gas Detected");
  delay(100);
  sim800L.write(0x1A);
  delay(5000);
}

void objectAvoid() {
  int distance = sonar.ping_cm();
  if (distance <= 15) {
    stop();
    lookLeft();
    lookRight();
    delay(100);
    if (rightDistance <= leftDistance) {
      turnLeft();
    } else {
      turnRight();
    }
    delay(100);
  }
  else {
    moveForward();
  }
}

void stop() {
  digitalWrite(LM1, LOW);
  digitalWrite(LM2, LOW);
  digitalWrite(RM1, LOW);
  digitalWrite(RM2, LOW);
}

void moveForward() {
  digitalWrite(LM1, HIGH);
  digitalWrite(LM2, LOW);
  digitalWrite(RM1, HIGH);
  digitalWrite(RM2, LOW);
}

void moveLeft() {
  digitalWrite(LM1, HIGH);
  digitalWrite(LM2, LOW);
  digitalWrite(RM1, LOW);
  digitalWrite(RM2, LOW);
}

void moveRight() {
  digitalWrite(LM1, LOW);
  digitalWrite(LM2, LOW);
  digitalWrite(RM1, HIGH);
  digitalWrite(RM2, LOW);
}

void lookLeft() {
  servo.write(180);
  delay(500);
  leftDistance = sonar.ping_cm();
}

void lookRight() {
  servo.write(0);
  delay(500);
  rightDistance = sonar.ping_cm();
}

void turnLeft() {
  servo.write(180);
  digitalWrite(LM1, LOW);
  digitalWrite(LM2, LOW);
  digitalWrite(RM1, HIGH);
  digitalWrite(RM2, LOW);
}

void turnRight() {
  servo.write(0);
  digitalWrite(LM1, HIGH);
  digitalWrite(LM2, LOW);
  digitalWrite(RM1, LOW);
  digitalWrite(RM2, LOW);
}

void sendSMS() {
  Serial.println("Sending SMS....");
  delay(50);
  sim800L.print("AT+CMGF=1\r");
  delay(1000);
  sim800L.print

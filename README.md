# smart-parking-system
#include <Servo.h>
#include <Wire.h>

Servo myservo;
Servo servo_new;
Servo servo_another;
#define ir_enter 2
#define ir_back 4
#define ir_sensor_3 7
#define ir_sensor_4 8
#define ir_sensor_5 13
#define ir_sensor_6 10

int S1 = 0, S2 = 0, sensor_3_State = 0, sensor_4_State = 0, sensor_5_State = 0, sensor_6_State = 0;
int flag1 = 0, flag2 = 0;
int slot = 6;

void setup() {
  Serial.begin(9600);  // Initialize Serial communication

  pinMode(ir_enter, INPUT);
  pinMode(ir_back, INPUT);
  pinMode(ir_sensor_3, INPUT);
  pinMode(ir_sensor_4, INPUT);
  pinMode(ir_sensor_5, INPUT);
  pinMode(ir_sensor_6, INPUT);

  myservo.attach(3);
  servo_new.attach(11);
  servo_another.attach(12);
  myservo.write(90);
  servo_new.write(90);
  servo_another.write(90);
}

void loop() {
  Read_Sensor();

  if (digitalRead(ir_enter) == 0 && flag1 == 0) {
    if (slot > 0) {
      flag1 = 1;
      if (flag2 == 0 && sensor_6_State == HIGH) {
        myservo.write(180);
        slot = slot - 1;
        sendSensorData();  // Send sensor data over Serial
      }
    } else {
      Serial.println("Sorry, Parking Full");
      delay(1500);
    }
  }

  if (digitalRead(ir_back) == 0 && flag2 == 0) {
    flag2 = 1;
    if (flag1 == 0 && sensor_4_State == HIGH) {
      myservo.write(180);
      slot = slot + 1;
      sendSensorData();  // Send sensor data over Serial
    }
  }

  if (sensor_3_State == LOW && sensor_4_State == HIGH && flag1 == 0) {
    Serial.println("IR Sensor 3 detected. Opening the gate for Parking Slot 1.");
    servo_new.write(180);
    sendSensorData();  // Send sensor data over Serial
  } else if (sensor_4_State == LOW && flag1 == 0) {
    Serial.println("Parking Slot 1 occupied. Cannot open the gate for Parking Slot 1.");
    servo_new.write(90);
    sendSensorData();  // Send sensor data over Serial
  }

  if (sensor_5_State == LOW && sensor_6_State == HIGH && flag2 == 0) {
    Serial.println("IR Sensor 5 detected. Opening the gate for Parking Slot 2.");
    servo_another.write(180);
    sendSensorData();  // Send sensor data over Serial
  } else if (sensor_6_State == LOW && flag2 == 0) {
    Serial.println("Parking Slot 2 occupied. Cannot open the gate for Parking Slot 2.");
    servo_another.write(90);
    sendSensorData();  // Send sensor data over Serial
  }

  if (flag1 == 1 && flag2 == 1) {
    delay(1000);
    myservo.write(90);
    flag1 = 0;
    flag2 = 0;
  }

  delay(1);
}

void Read_Sensor() {
  sensor_3_State = digitalRead(ir_sensor_3);
  sensor_4_State = digitalRead(ir_sensor_4);
  sensor_5_State = digitalRead(ir_sensor_5);
  sensor_6_State = digitalRead(ir_sensor_6);
}

void sendSensorData() {
  // Send sensor states over Serial
  Serial.print("S3:");
  Serial.print(sensor_3_State);
  Serial.print(",S4:");
  Serial.print(sensor_4_State);
  Serial.print(",S5:");
  Serial.print(sensor_5_State);
  Serial.print(",S6:");
  Serial.println(sensor_6_State);
}

// libraries
#include <Arduino.h>
#include <Wire.h>
#include <SPI.h>
#include <MFRC522.h>

// ========== global variables ==========
#define RFID_SS 5
#define RFID_RST 22
String authorizedUIDs[] = {"02 43 4F 05"};     // authorized UIDs (add more UIDs as needed)
int numAuthorized = 1; // number of authorized UIDs
MFRC522 rfid(RFID_SS, RFID_RST);
#define GREEN_LED 25
#define RED_LED 26
unsigned long indicatorStart = 0;
unsigned long indicatorTime = 0;
bool indicatorON = false;
String lastUID = "";
unsigned long lastScanTime = 0;

// functions

String getUID(){    // returns the UID of the scanned card as a string
  String uid = "";

  for (int i = 0; i < rfid.uid.size; i++) {
    if (rfid.uid.uidByte[i] < 0x10) {
      uid = uid + "0";
    }
    uid = uid + String(rfid.uid.uidByte[i], HEX);
    if (i < rfid.uid.size - 1) {
      uid = uid + "";
    }
  }
  uid.toUpperCase();
  return uid;
}

bool isAuthorized(String uid) {
  for (int i = 0; i < numAuthorized; i++){
    if (uid == authorizedUIDs[i]) {
      return true;
    }
  }
  return false;
}

void statIndicatorTimer(unsigned long duration) {
  indicatorStart = millis();
  indicatorTime = duration;
  indicatorON = true;
}

void allIndicatorsOFF() {
  digitalWrite(GREEN_LED, LOW);
  digitalWrite(RED_LED, LOW);
}

void onFreeRoom() {
  Serial.println("ROOM IS FREE, YOU MAY ENTER");
  allIndicatorsOFF();
  digitalWrite(GREEN_LED, LOW);
  statIndicatorTimer(2000);
}

void onOccupiedRoom(){
  Serial.println("ROOM IS OCCUPIED, DO NOT ENTER");
  allIndicatorsOFF();
  digitalWrite(GREEN_LED, HIGH);
  statIndicatorTimer(1000);
}

void onUnknownCard() {
  Serial.println("ACCESS DENIED, UNKNOWN CARD");
  allIndicatorsOFF();
  digitalWrite(RED_LED, HIGH);
  statIndicatorTimer(1000);
}

void OnTimeExpired() {
  allIndicatorsOFF();
  indicatorON = false;
  Serial.println("READY");
}

void onCardScanned(String uid) {    // called when a card is scanned
  if (uid == lastUID && millis() - lastScanTime <2000) {
    Serial.println("Repeat ignored");
    return;
  }

  lastUID = uid;
  lastScanTime = millis();

  Serial.print("Card UID: ");
  Serial.println(uid);

  if (isAuthorized(uid) == true) {    // check if the card is authorized
    onFreeRoom();
  } else {
    onUnknownCard();
  }
}

void setup() {
    Serial.begin(115200);
    Serial.println("Program started");

  //=========== RFID code ===========
    SPI.begin();
    rfid.PCD_Init();
    delay(10);
    rfid.PCD_DumpVersionToSerial();     // prints the firmware version
    Serial.println("Tap your student library card");

  //=========== LED code ===========
    pinMode(GREEN_LED, OUTPUT);
    pinMode(RED_LED, OUTPUT);
}

void loop() {
  // ========== RFID code ===========
  if (rfid.PICC_IsNewCardPresent() == true && rfid.PICC_ReadCardSerial() == true){
    String uid = getUID();
    onCardScanned(uid);
    rfid.PICC_HaltA();
    rfid.PCD_StopCrypto1();
  } 

  // ========== timer code ===========
  if (indicatorON == true && millis() - indicatorStart >= indicatorTime) {
    OnTimeExpired();
  }
}

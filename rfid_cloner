
//copied from http://colligomentis.com/2012/05/16/hid-reader-arduino-rfid-card-catcher/
//#include <SoftwareSerial.h>
#include <SdFat.h>

// since the LCD does not send data back to the Arduino, we should only define the txPin
//#define txPin 4 // White wire from Serial LCD screen
//const int LCDdelay = 10;  // conservative, 2 actually works

// SD card variables
const uint8_t chipSelect = 10; // CS from SD to Pin 10 on Arduino
SdFat sd; // file system object for SD card
SdFile file; // file object
char dataFile[] = "cards.txt"; // file to save card ids to

unsigned long bitHolder1 = 0;
unsigned long bitHolder2 = 0;
unsigned int bitCount = 0;
unsigned long cardChunk1 = 0;
unsigned long cardChunk2 = 0;
int parity = 0;
long previousMillis = 0; // will store last time LCD was updated
long interval = 3000; // interval at which to reset display (milliseconds)

// Begin Code for DATA0 and DATA1 Interrupts to get the binary data from the card
// For whatever reason, when wired to the micro DATA1 and DATA0 need to be reversed
void DATA1(void) {  
    bitCount++;
    if(bitCount < 23) {
      bitHolder1 = bitHolder1 << 1;
    }
    else {
      bitHolder2 = bitHolder2 << 1;
    }
}

void DATA0(void) {
   bitCount++;
   if(bitCount < 23) {
      bitHolder1 = bitHolder1 << 1;
      bitHolder1 |= 1;
   }
   else {
     bitHolder2 = bitHolder2 << 1;
     bitHolder2 |= 1;
   }
}

// End code for DATA0 and DATA1

// Begin Code for LCD

//SoftwareSerial LCD(0, txPin);
//
//void clearLCD(){
//  LCD.write(0xFE);   //command flag
//  LCD.write(0x01);   //clear command.
//  delay(LCDdelay);
//}

// End code for LCD

// Begin code for SD card

void writeSD() {
  
    // open the file. note that only one file can be open at a time,
    // so you have to close this one before opening another.
    ofstream file(dataFile, ios::out | ios::app);
  
    // if the file is available, write to it:
    if (file) {
      file << bitCount;
      file << " bit card: ";
      file << hex << cardChunk1;
      file << hex << cardChunk2;
      file << ", BIN: ";
      for (int i = 19; i >= 0; i--) {
        file << bitRead(cardChunk1, i);
      }
      for (int i = 23; i >= 0; i--) {
        file << bitRead(cardChunk2, i);
      }
      file << endl;
      
      // print to the serial port too
      //Serial.println("Wrote data to SD card");
    }
//    else {
//      clearLCD();
//      LCD.print("Error writing to file");
//    }
}

// End code for SD card

void setup() {
  
  //Serial.begin(57600);
  
  clearinterrupts();
  
  attachInterrupt(0, DATA0, RISING);  // DATA0 (green) on HID reader, Pin 2 on Arduino
  attachInterrupt(1, DATA1, RISING);  // DATA1 (white) on HID reader, Pin 3 on Arduino
  delay(10);

  // Initialize LCD screen
//  pinMode(txPin, OUTPUT);
//  LCD.begin(9600);
  
  // make sure that the default chip select pin 10 is set to
  // output, even if you don't use it:
  // Setup output pin to SD card
  pinMode(10, OUTPUT);
  pinMode(chipSelect, OUTPUT);
  
  // Initialize SD card
  while(!sd.begin(chipSelect, SPI_HALF_SPEED)) {
    Serial.println("No SD Card!");  
  }
  // Commented out with no LCD
//  if(!sd.begin(chipSelect, SPI_HALF_SPEED)) {
//    clearLCD();
//    LCD.print("Problem with SD card");
//  }
//  else {
//    clearLCD();
//    LCD.print("SD card initialized.");
//  }

  // Turn off onboard LED to indicate ready state
  digitalWrite(17, LOW);
    
  //Serial.println("READER_0001");
  
  //Reset and get ready for a card
  bitCount = 0; bitHolder1 = 0; bitHolder2 = 0;
}


void loop() {
    if (millis() - previousMillis > interval) {
      bitCount = 0; bitHolder1 = 0; bitHolder2 = 0; //in case something went wrong, clear the buffers
      previousMillis = millis(); // remember the last time we blinked the LED
//      clearLCD();
//      LCD.print("Present Card or Enter Pin");
      
    }
    
    // Check if bits received are a valid length to be a card
    if (bitCount >= 26) {
        delay(100);
        
        // Call function to get our two chunks of card values
        getCardValues();
        
        //Debug stuff
//        Serial.print("bitHolders: ");
//        Serial.print(bitHolder1, BIN);
//        Serial.print(", ");
//        Serial.println(bitHolder2, BIN);
//        Serial.print("cardChunk1: ");
//        Serial.println(cardChunk1, BIN);
//        Serial.print("cardChunk2: ");
//        Serial.println(cardChunk2, BIN);
//        Serial.print("Card Value (BIN): ");
//        Serial.print(cardChunk1, BIN);
//        Serial.println(cardChunk2, BIN);
//        Serial.print("Card Value (HEX): ");
//        Serial.print(cardChunk1, HEX);
//        Serial.println(cardChunk2, HEX);
        
        // Print the card values to the LCD screen
//        clearLCD();
//        LCD.print(bitCount);
//        LCD.print(" bit Card:    ");
//        LCD.print(cardChunk1, HEX);
//        LCD.print(cardChunk2, HEX);
        
        // Write the card values to the SD card
        writeSD();
        
        // Flash onboard LED to indicate card read
        digitalWrite(17, HIGH);
        delay(100);
        digitalWrite(17, LOW);
        
        //Reset and get ready for another card
        bitCount = 0; bitHolder1 = 0; bitHolder2 = 0;
        previousMillis = millis();
        delay(100); // Extend this if you want to see the card value on the LCD screen longer
    }
}

// Function to clear interrupts and prepare them for use
void clearinterrupts () {
  // the interrupt in the Atmel processor messes up the first negative pulse as the inputs are already high,
  // so this gives a pulse to each reader input line to get the interrupts working properly.
  // Then clear out the reader variables.
  // The readers are open collector sitting normally at a one so this is OK
  for(int i = 2; i < 4; i++){
    pinMode(i, OUTPUT);
    digitalWrite(i, HIGH); // enable internal pull up causing a one
    digitalWrite(i, LOW); // disable internal pull up causing zero and thus an interrupt
    pinMode(i, INPUT);
    digitalWrite(i, HIGH); // enable internal pull up
  }
  delay(10);
}

// Function to append the card value (bitHolder1 and bitHolder2) to the necessary array then tranlate that to
// the two chunks for the card value that will be output
void getCardValues() {
  switch (bitCount) {
    case 26:
        // Example of full card value
        // |>   preamble   <| |>   Actual card value   <|
        // 000000100000000001 11 111000100000100100111000
        // |> write to chunk1 <| |>  write to chunk2   <|
        
       for(int i = 19; i >= 0; i--) {
          if(i == 13 || i == 2){
            bitWrite(cardChunk1, i, 1); // Write preamble 1's to the 13th and 2nd bits
          }
          else if(i > 2) {
            bitWrite(cardChunk1, i, 0); // Write preamble 0's to all other bits above 1
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 20)); // Write remaining bits to cardChunk1 from bitHolder1
          }
          if(i < 20) {
            bitWrite(cardChunk2, i + 4, bitRead(bitHolder1, i)); // Write the remaining bits of bitHolder1 to cardChunk2
          }
          if(i < 4) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i)); // Write the remaining bit of cardChunk2 with bitHolder2 bits
          }
        }
        break;

    case 27:
       for(int i = 19; i >= 0; i--) {
          if(i == 13 || i == 3){
            bitWrite(cardChunk1, i, 1);
          }
          else if(i > 3) {
            bitWrite(cardChunk1, i, 0);
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 19));
          }
          if(i < 19) {
            bitWrite(cardChunk2, i + 5, bitRead(bitHolder1, i));
          }
          if(i < 5) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i));
          }
        }
        break;

    case 28:
       for(int i = 19; i >= 0; i--) {
          if(i == 13 || i == 4){
            bitWrite(cardChunk1, i, 1);
          }
          else if(i > 4) {
            bitWrite(cardChunk1, i, 0);
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 18));
          }
          if(i < 18) {
            bitWrite(cardChunk2, i + 6, bitRead(bitHolder1, i));
          }
          if(i < 6) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i));
          }
        }
        break;

    case 29:
       for(int i = 19; i >= 0; i--) {
          if(i == 13 || i == 5){
            bitWrite(cardChunk1, i, 1);
          }
          else if(i > 5) {
            bitWrite(cardChunk1, i, 0);
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 17));
          }
          if(i < 17) {
            bitWrite(cardChunk2, i + 7, bitRead(bitHolder1, i));
          }
          if(i < 7) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i));
          }
        }
        break;

    case 30:
       for(int i = 19; i >= 0; i--) {
          if(i == 13 || i == 6){
            bitWrite(cardChunk1, i, 1);
          }
          else if(i > 6) {
            bitWrite(cardChunk1, i, 0);
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 16));
          }
          if(i < 16) {
            bitWrite(cardChunk2, i + 8, bitRead(bitHolder1, i));
          }
          if(i < 8) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i));
          }
        }
        break;

    case 31:
       for(int i = 19; i >= 0; i--) {
          if(i == 13 || i == 7){
            bitWrite(cardChunk1, i, 1);
          }
          else if(i > 7) {
            bitWrite(cardChunk1, i, 0);
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 15));
          }
          if(i < 15) {
            bitWrite(cardChunk2, i + 9, bitRead(bitHolder1, i));
          }
          if(i < 9) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i));
          }
        }
        break;

    case 32:
       for(int i = 19; i >= 0; i--) {
          if(i == 13 || i == 8){
            bitWrite(cardChunk1, i, 1);
          }
          else if(i > 8) {
            bitWrite(cardChunk1, i, 0);
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 14));
          }
          if(i < 14) {
            bitWrite(cardChunk2, i + 10, bitRead(bitHolder1, i));
          }
          if(i < 10) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i));
          }
        }
        break;

    case 33:
       for(int i = 19; i >= 0; i--) {
          if(i == 13 || i == 9){
            bitWrite(cardChunk1, i, 1);
          }
          else if(i > 9) {
            bitWrite(cardChunk1, i, 0);
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 13));
          }
          if(i < 13) {
            bitWrite(cardChunk2, i + 11, bitRead(bitHolder1, i));
          }
          if(i < 11) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i));
          }
        }
        break;

    case 34:
       for(int i = 19; i >= 0; i--) {
          if(i == 13 || i == 10){
            bitWrite(cardChunk1, i, 1);
          }
          else if(i > 10) {
            bitWrite(cardChunk1, i, 0);
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 12));
          }
          if(i < 12) {
            bitWrite(cardChunk2, i + 12, bitRead(bitHolder1, i));
          }
          if(i < 12) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i));
          }
        }
        break;

    case 35:        
       for(int i = 19; i >= 0; i--) {
          if(i == 13 || i == 11){
            bitWrite(cardChunk1, i, 1);
          }
          else if(i > 11) {
            bitWrite(cardChunk1, i, 0);
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 11));
          }
          if(i < 11) {
            bitWrite(cardChunk2, i + 13, bitRead(bitHolder1, i));
          }
          if(i < 13) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i));
          }
        }
        break;

    case 36:
       for(int i = 19; i >= 0; i--) {
          if(i == 13 || i == 12){
            bitWrite(cardChunk1, i, 1);
          }
          else if(i > 12) {
            bitWrite(cardChunk1, i, 0);
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 10));
          }
          if(i < 10) {
            bitWrite(cardChunk2, i + 14, bitRead(bitHolder1, i));
          }
          if(i < 14) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i));
          }
        }
        break;

    case 37:
       for(int i = 19; i >= 0; i--) {
          if(i == 13){
            bitWrite(cardChunk1, i, 0);
          }
          else {
            bitWrite(cardChunk1, i, bitRead(bitHolder1, i + 9));
          }
          if(i < 9) {
            bitWrite(cardChunk2, i + 15, bitRead(bitHolder1, i));
          }
          if(i < 15) {
            bitWrite(cardChunk2, i, bitRead(bitHolder2, i));
          }
        }
        break;
  }
  return;
}

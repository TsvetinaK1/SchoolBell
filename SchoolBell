// Date and time functions using a DS1307 RTC connected via I2C and Wire lib
#include "RTClib.h"
#include <Wire.h>
#include <SD.h>

// Pin assignments
int chipSelect = 2;  // Chip select pin for SD card
File file;

// Buzzer pin
int buzzer = 7;

// Variables for:
String time = "";         // time
int row = 0;              // file row index
bool waitNewDay = false;  // if waiting for the next day
bool waitAlarm = false;   // if it is waiting for the alarm
int hours;                // hours for next alarm
int minutes;              // minutes for next alarm

DateTime day;

RTC_DS1307 rtc;

void setup() {

  Serial.begin(9600);  // Start serial communication

  //rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));

  // Initialize the RTC module
  if (!rtc.begin()) {
    Serial.println("RTC module not found!");
  }

  pinMode(buzzer, OUTPUT);      // Set buzzer pin as output
  pinMode(chipSelect, OUTPUT);  // Set SD card chip select pin as output

  // Attempt to initialize the SD card
  if (!SD.begin(chipSelect)) {
    Serial.println("SD card not found!");
    delay(100);
    setup();  // Retry setup if SD card initialization fails
  }
}

void loop() {



  // If RTC is not running, set it to the compile time
  if (!rtc.isrunning()) {
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  }

  DateTime now = rtc.now();  // Get current time from RTC

  // If not waiting for a new day
  if (!waitNewDay) {

    // Gets the day
    day = now.day();

        Serial.print(now.hour(), DEC);
    Serial.print(':');
    Serial.print(now.minute(), DEC);
    Serial.print(':');
    Serial.print(now.second(), DEC);
    Serial.println();

    // Check if it's a weekend (Saturday or Sunday)
    if (now.dayOfTheWeek() == 6 || now.dayOfTheWeek() == 7) {

      Serial.println("No alarm today!");

      if (now.dayOfTheWeek() == 6) {

        // Delay one day to save resources (if it's Saturday)
        delay(86400000);
      }
    }

    if (!waitAlarm) {
      getTime(now);  // Get alarm time from file

      hours = time.substring(0, 2).toInt();  // Extract hours from time string
      minutes = time.substring(3).toInt();   // Extract minutes from time string
    }

    DateTime future(now.year(), now.month(), now.day(), hours, minutes, 0);  // Create DateTime object for next alarm

    // Check if current time is greater than next alarm
    if (now > future) {

      // If it is greater, get to the next row of the file
      row++;
      waitAlarm = false;
      Serial.println(time);
      Serial.println("Next alarm!");
      return;
    }

    TimeSpan result = future - now;  // Calculate time difference between current time and future alarm time

    // Trigger alarm based on time difference
    if (result.seconds() == 0 && result.hours() == 0 && result.minutes() == 0) {
      // Alarm goes off at exact time
      Serial.println("The alarm goes off!");
      tone(buzzer, 660);
      delay(700);
      tone(buzzer, 550);
      delay(700);
      tone(buzzer, 440);
      delay(700);
      noTone(buzzer);

      // The alarm sounds for 3 seconds
      delay(3000);


      // Gets the next alarm
      row++;
      waitAlarm = false;

      Serial.println("New row!");
    }
    // Some checkpoints
    else if (result.seconds() == 0 && result.minutes() == 0 && result.hours() == 1) {

      // Alarm goes off after 1 hour
      Serial.println("The alarm goes off after 1 hour!");
    } else if (result.seconds() == 0 && result.minutes() == 30 && result.hours() == 0) {

      // Alarm goes off after 30 minutes
      Serial.println("The alarm goes off after 30 minutes!");
    } else if (result.seconds() == 0 && result.minutes() == 15 && result.hours() == 0) {

      // Alarm goes off after 15 minutes
      Serial.println("The alarm goes off after 15 minutes!");
    }
  } else if (day != now.day()) {
    waitNewDay = false;
  }
}

// Function to get alarm times from the file
void getTime(DateTime now) {
  file = SD.open("file.txt", FILE_READ);  // Open file on SD card for reading

  if (file) {

    // Skip the already read lines in the file
    for (int i = 0; i < row; i++) {
      while (file.read() != '\n') {}
    }

    // Read and print the third line (alarm time)
    time = file.readStringUntil('\n');

    if (time == "") {

      // Reset row index and flag to wait for a new day
      row = 0;
      waitNewDay = true;

      Serial.println("Waiting new day");
    }

    file.close();      // Close the file
    waitAlarm = true;  // Set flag to wait for the alarm
  } else {

    Serial.println("Could not open the file (reading).");
    setup();  // Retry setup if file opening fails
  }
}

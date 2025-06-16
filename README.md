# Digital-Stopwatch-stm32
A digital stopwatch using STM32 Blue Pill with LCD and LED
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Initialize LCD with I2C address 0x27 and size 16x2
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Define button pins
#define START_STOP_PIN PA2
#define RESET_PIN PA5

// Stopwatch variables
unsigned long startTime = 0;
unsigned long elapsedTime = 0;
bool isRunning = false;
bool lastStartStopState = HIGH;
bool lastResetState = HIGH;

void setup() {
  lcd.init();
  lcd.backlight();
  lcd.clear();

  pinMode(START_STOP_PIN, INPUT_PULLUP);
  pinMode(RESET_PIN, INPUT_PULLUP);

  lcd.setCursor(0, 0);
  lcd.print("Smart Stopwatch");
  lcd.setCursor(0, 1);
  lcd.print("Press Start");

  delay(2000);
  lcd.clear();
  displayTime(0);
  lcd.setCursor(0, 0);
  lcd.print("Ready:");
}

void loop() {
  // Read current button states
  bool currentStartStop = digitalRead(START_STOP_PIN);
  bool currentReset = digitalRead(RESET_PIN);

  // Start/Stop toggle on button press
  if (currentStartStop == LOW && lastStartStopState == HIGH) {
    delay(50); // debounce
    if (digitalRead(START_STOP_PIN) == LOW) {
      toggleStopwatch();
    }
  }
  lastStartStopState = currentStartStop;

  // Reset button press
  if (currentReset == LOW && lastResetState == HIGH) {
    delay(50); // debounce
    if (digitalRead(RESET_PIN) == LOW) {
      resetStopwatch();
    }
  }
  lastResetState = currentReset;

  // If running, update elapsed time
  if (isRunning) {
    elapsedTime = millis() - startTime;
  }

  displayTime(elapsedTime);
  delay(100);
}

void toggleStopwatch() {
  if (isRunning) {
    isRunning = false;
    elapsedTime = millis() - startTime;
    lcd.setCursor(0, 0);
    lcd.print("Stopped:       ");
  } else {
    startTime = millis() - elapsedTime;  // resume from previous time
    isRunning = true;
    lcd.setCursor(0, 0);
    lcd.print("Running:       ");
  }
}

void resetStopwatch() {
  if (!isRunning) {
    elapsedTime = 0;
    displayTime(0);
    lcd.setCursor(0, 0);
    lcd.print("Reset:         ");
    delay(1000);
    lcd.setCursor(0, 0);
    lcd.print("Ready:         ");
  }
}

void displayTime(unsigned long timeMillis) {
  unsigned long ms = timeMillis % 1000;
  unsigned long totalSeconds = timeMillis / 1000;
  unsigned long s = totalSeconds % 60;
  unsigned long totalMinutes = totalSeconds / 60;
  unsigned long m = totalMinutes % 60;
  unsigned long h = totalMinutes / 60;

  char timeString[21];
  snprintf(timeString, sizeof(timeString), "%02lu:%02lu:%02lu.%03lu", h, m, s, ms);

  lcd.setCursor(0, 1);
  lcd.print("                "); // Clear previous line
  lcd.setCursor(0, 1);
  lcd.print(timeString);
}
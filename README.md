# Capacitors
// ESP32 Capacitor Discharge Timer - Enhanced
const int capPin = 32;     // Analog pin connected to capacitor
const int buttonPin = 25;  // Digital pin connected to the button
const int dischargeThreshold = 1500; // ADC value to consider capacitor discharged (e.g., ~1.2V)
const float vRef = 3.3;    ESP32's analog reference voltage
bool timing = false;
unsigned long startTime;
unsigned long dischargeTime;

// Function to calculate voltage from ADC value
float readADC_Voltage(int adcValue) {
  return (adcValue * vRef) / 4095.0;
}

void setup() {
  Serial.begin(115200);
  pinMode(buttonPin, INPUT_PULLUP); // Use ESP32's internal pull-up resistor
  analogReadResolution(12); // Set ADC to 12-bit resolution (0-4095)
  Serial.println("ESP32 Capacitor Discharge Timer - Ready");
  Serial.println("Press and release the button to start timing...");
}

void loop() {
  int capValue = analogRead(capPin);
  float capVoltage = readADC_Voltage(capValue);

  // Check if button is pressed (LOW because of INPUT_PULLUP)
  if (digitalRead(buttonPin) == LOW) {
    delay(50); // Simple debounce
    if (digitalRead(buttonPin) == LOW && !timing) { // Check if button is still pressed and we're not already timing
      Serial.println("Button pressed. Capacitor charging...");
      timing = true;
      // Wait for button release to start discharge timing
      while (digitalRead(buttonPin) == LOW) {
        delay(10);
      }
      Serial.println("Button released. Starting timer.");
      startTime = micros(); // Start timer with microsecond resolution
    }
  }

  // If we are in the timing state and the capacitor voltage falls below the threshold
  if (timing && capValue < dischargeThreshold) {
    dischargeTime = micros() - startTime; // Calculate elapsed time
    timing = false; // Stop timing

    // Print the results
    Serial.print("Discharge time: ");
    Serial.print(dischargeTime / 1000.0); // Convert microseconds to milliseconds
    Serial.print(" ms | Final Voltage: ");
    Serial.print(capVoltage, 2);
    Serial.println(" V");
    Serial.println("Press button again for a new measurement.");
  }

  // A small delay for stability
  delay(10);
}

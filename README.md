# ZACwire™ Library to read TSic sensors
[![Only 32 Kb](https://badge-size.herokuapp.com/lebuni/ZACwire-Library/master/ZACwire.h)](https://github.com/lebuni/ZACwire-Library/blob/master/ZACwire.h) 
[![GitHub issues](https://img.shields.io/github/issues/lebuni/ZACwire-Library.svg)](https://github.com/lebuni/ZACwire-Library/issues/) 
[![GitHub license](https://img.shields.io/github/license/lebuni/ZACwire-Library.svg)](https://github.com/lebuni/ZACwire-Library/blob/master/LICENSE)


Arduino Library to read the ZACwire protocol on ESP8266 and ESP32, wich is used by TSic temperature sensors 206, 306 and 506 on their signal pin.

`ZACwire<int pin> obj(int Sensor)` tells the library which input pin of the controller (eg. 2) and type of sensor (eg. 306) it should use

`.begin()` returns true if a signal is detected on the specific pin and starts the reading via ISRs. It should be started at least 120ms before the first .getTemp().

`.getTemp()` returns the temperature in °C and gets usually updated every 100ms. In case of a bad signal, it returns 222

`.end()` stops the reading for time sensititive tasks, which shouldn't be interrupted.


## Benefits compared to former TSic libraries
- saves a lot of controller time, because no delay() is used and calculations are done by bit manipulation
- low memory consumption
- misreading rate lower than 0.01%
- reading an unlimited number of TSic simultaneously
- higher accuracy (0.1°C offset corrected)
- simple use






## Example
```c++

#include <ZACwire.h>

ZACwire<2> Sensor(306);		// set pin "2" to receive signal from the TSic "306"

void setup() {
  Serial.begin(500000);
  
  if (Sensor.begin() == true) {     //check if a sensor is connected to the pin
    Serial.println("Signal found on pin 2");
  }
  delay(120);
}

void loop() {
  float Input = Sensor.getTemp();     //get the Temperature in °C
  
  if (Input == 222) {
    Serial.println("Reading failed");
  }
  
  else {
    Serial.print("Temp: ");
    Serial.println(Input);
  }
  delay(100);
}
```






## Fine-Tuning
Depending on the sensor build quality, the voltage and on the temperature range the sensor is operating in, there might be some fine-tuning necessary to get less failed readings.

```c++
ZACwire<int pin> obj(int Sensor, byte defaultBitWindow, byte offset, bool core)
```

`byte defaultBitWindow` is the expected BitWindow in µs. According to the datasheet it should be around 125µs, but to my experience the code starts better with 130µs.
Change this, if the **first few readings** of the sensor fail (t = 222°C).

`byte offset` is an offset of the measured BitWindow, which needs to be added because your µC doesn't count the micros() properly inside ISRs. The more ISRs you use, the higher you should make that offset. Just try it out! The default offset is at 10µs.
Change this, if you get **failed readings at any random time**.

`bool core` can only be used on a dual core ESP32. You can decide on which core the ISR should run, default is CPU0.

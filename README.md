
[![Arduino CI](https://github.com/RobTillaart/X9C10X/workflows/Arduino%20CI/badge.svg)](https://github.com/marketplace/actions/arduino_ci)
[![Arduino-lint](https://github.com/RobTillaart/X9C10X/actions/workflows/arduino-lint.yml/badge.svg)](https://github.com/RobTillaart/X9C10X/actions/workflows/arduino-lint.yml)
[![JSON check](https://github.com/RobTillaart/X9C10X/actions/workflows/jsoncheck.yml/badge.svg)](https://github.com/RobTillaart/X9C10X/actions/workflows/jsoncheck.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/RobTillaart/X9C10X/blob/master/LICENSE)
[![GitHub release](https://img.shields.io/github/release/RobTillaart/X9C10X.svg?maxAge=3600)](https://github.com/RobTillaart/X9C10X/releases)


# X9C10X

Arduino Library for X9C10X series digital potentiometer.


## Description

This **experimental** library provides a X9C10X base class and 
four derived classes for specific digital potentiometer.

| type   | resistance | tested  |  notes       |
|:------:|:----------:|:-------:|:-------------|
| X9C10X |    generic |    Y    |  base class  |
| X9C102 |    1 KΩ    |    N    |  10 \* 10^2  |
| X9C103 |   10 KΩ    |    Y    |  10 \* 10^3  |
| X9C104 |  100 KΩ    |    N    |  10 \* 10^4  |
| X9C503 |   50 KΩ    |    N    |  50 \* 10^3  |


_Note: Ω Ohm sign = ALT-234_

The library keeps track of the position of the potentiometer,
but the user should set it with **setPosition(pos, true);**
Otherwise the lib and device will probably not be in sync.  


#### split the lib?

It is possible that the library will split in the future in two versions.
This needs a lot of thoughts and hands on.

- a version that does not keep track of the internal position.
This could become a very basic library without any feedback about position.
Possibly this would become a base class?
- a version (much like current one) that does keep track of the position.
There are some conceptual / behaviour problems with keeping track of the
position especially after restart.


### PINOUT


```cpp
//  PINOUT  X9C10X TOP VIEW  (see datasheet)
//
//       +--------+
//  INC  | o    o |  VCC
//  U/D  | o    o |  CS
//  RH   | o    o |  RL
//  GND  | o    o |  Rwiper
//       +--------+
//
//  INC   pulses
//  U/D   UP = 1 DOWN = 0
//  VCC   +5V
//
```

It is advised to use pull up resistors - e.g. 10 KΩ - on the CS, UD and INC line. 
This will help the lines to start in a defined state and will
improve the signal quality. 
The pulses esp. INC can be quite short.


## Interface


## X9C10X base class

Use **\#include "X9C10X.h"**

- **X9C10X(uint32_t Ohm = 10000)** Constructor, default initializes the resistance to 10000 Ω. 
To calibrate one can fill in any other value e.g. 9950 Ω.
- **void begin(uint8_t pulsePin, uint8_t directionPin, uint8_t selectPin)** 
sets the pins used by the device.

Note: **begin()** changed in 0.2.0 as the implicit parameter position
was removed for the explicit function call to **setPosition()**.
If **setPosition()** is not called, the device uses the last stored
value as position. Unfortunately this cannot be read from the device.
This will result in a mismatch between the internal position and the 
external one.

Note: **begin()** has a hard coded 500uS delay so the device can wake up.

Note: multiple devices can be controlled, just by giving them an unique selectPin.
This behaviour is similar to the SPI select pin.

- **void setPosition(uint8_t position, bool forced = false)** sets the wiper 
to a position between 0 and 99. 
The movement is relative to the current (cached) position.
If forced is set to true, the wiper will be moved to the closest "end" position 
and from there moved to the requested position. 
The internal position is replaced by the new position. 
- **uint8_t getPosition()** returns the current (internal) position.
- **bool incr()** moves one position up (if possible). 
Returns true if moved and false if already at end position
according to internal position math.
- **bool decr()** moves one position down (if possible).
Returns true if moved and false if already at end position
according to internal position math.
- **uint32_t getOhm()** returns the position expressed in Ohm.
The returned value does depend on the value passed in the constructor.
- **uint32_t getMaxOhm()** returns the maximum value ( =  parameter from constructor). 
- **uint32_t Ohm2Position(uint32_t value, bool invert = false)**
Calculates (with rounding) the position nearest to the requested value. 
If **invert == true** it uses the other wiper end as reference.


#### Store 

Warning: use with care (not tested).

- **uint8_t store()** stores the current position in the NVRAM of the device, 
and returns the current position so it can later be used as position parameter for **setPosition()**. 

Note: this **store()** function blocks for 20 milliseconds.


#### Calibration

One can measure the resistance between RH and RL and use this value to set 
in the constructor. Although the value will not differ much from the datasheet
one can get more precise values from the **getOhm()** function this way.


## Derived classes 

There are 4 derived classes, each with a other (appropriate) default value for the resistance. 

- **X9C102(uint32_t Ohm = 1000)** idem.
- **X9C103(uint32_t Ohm = 10000)** idem.
- **X9C104(uint32_t Ohm = 100000)** idem.
- **X9C503(uint32_t Ohm = 50000)** idem.

These classes have the same interface as the base class.
The only difference is that the type is set.


#### Performance

The table below is tested on a (relative slow) Arduino UNO 16 MHz with IDE 1.18.19.
Other processors might give similar or faster times. See performance example.

X9C10X_LIB_VERSION: 0.1.2

 | function            | time in us | notes                   |
 |:--------------------|:----------:|:------------------------|
 | getPosition         |     4      | cached value            |
 | setPosition(33)     |   304      | from 0 to 33            |
 | setPosition(66)     |   524      | from 0 to 66            |
 | setPosition(99)     |   780      | from 0 to 99            |
 | setPosition(33)     |   288      | from 0 to 33            |
 | setPosition(66)     |   276      | from 33 to 66           |
 | setPosition(99)     |   276      | from 66 to 99           |
 | setPosition(33)     |    12      | from 33 to 33 = no move |
 | setPosition(66)     |     4      | from 66 to 66 = no move |
 | setPosition(99)     |     4      | from 99 to 99 = no move |
 | getMaxOhm           |     4      | cached value            |
 | getOhm()            |    48      |                         |
 | incr()              |    28.8    | average 10 calls        |
 | decr()              |    29.2    | average 10 calls        |

Time per step is 780 / 99 = ~8 us per step on an UNO.


## Operation

See examples.


#### Voltage divider 

The digital potentiometer (esp 10 KΩ and up) can be used as a voltage divider.
Connect RL to GND and RH to +5V and you can do 5V in 100 steps of ~0.05V
A voltage of **3V3** would be **setPosition(66)**. 

Note: check datasheet for the range of the max voltage allowed.


## Future

- update documentation
- test different platforms
- improve the hardcoded 500us delay in **begin()**
- add error codes ?
- add examples
  - voltage divider
- investigate and test **store()**
- in the constructor rename **Ohm** parameter to value? 
  - The potentiometer can be used as a voltage divider (see above)
    so a better parameter name could be the anonymous **value**.
  - **getOhm()** ==> **getValue()**
  - **getMaxOhm()** ==> **getMaxValue()**
  - think milliVolt, ohm, lux, speed, etc. 
    User can do this too with **getPosition()**



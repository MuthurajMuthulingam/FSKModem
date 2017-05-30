FSKModem
=============

The FSKModem framework allows sending and receiving data from any iOS or OS X device via the head phone jack. It uses [frequency shift keying (FSK)](http://en.wikipedia.org/wiki/Frequency-shift_keying) to modulate a sine curve carrier signal to transmit bits. On top of that it uses a serial protocol to transmit single bytes and a simple packet protocol to cluster bytes. 

## Overview

1. [System requirements](README.md#system-requirements)
2. [Project setup](README.md#project-setup)
3. [Usage](README.md#usage)
4. [Talking to Arduino](README.md#talking-to-arduino)
5. [Known Issues](README.md#known-issues)
6. [Credits / Acknowledgements](README.md#credits--acknowledgements)
7. [License](README.md#license)

## System requirements

iOS 8.0+ or Mac OS X 10.7+

## Project setup

If you want to use FSKModem you need to add the following frameworks to your _Link Binary with Library_ build phase of your project:

* AudioToolbox.framework
* AVFoundation.framework

## Usage

```objc
JMFSKModemConfiguration* configuration = [JMModemConfiguration highSpeedConfiguration];
JMFSKModem* modem = [[JMFSKModem alloc]initWithConfiguration:configuration];

[modem connect];
```

### Sending data

```objc
NSString* textToSend = @"Hello World";
NSData* dataToSend = [textToSend dataUsingEncoding:NSASCIIStringEncoding];

[modem sendData:dataToSend];
```

### Receiving data

Register a delegate on your `JMFSKModem` instance and implement the `JMFSKModemDelegate` protocol to be notified whenever data arrives.

#### Setting the delegate object

```objc
modem.delegate = myModemDelegate;
```

#### Delegate implementation

```objc
-(void)modem:(JMFSKModem *)modem didReceiveData:(NSData *)data
{
	NSString* receivedText = [[NSString alloc]initWithData:data encoding:NSASCIIStringEncoding];
	NSLog(@"%@", receivedText);
}
```
## Talking to Arduino

The FSKModem allows you to talk to Arduino microcontrollers by using a simple circuit. 

### Requirements

* 4-pole audio cable with 3.5mm male connectors
* Audio Jack Circuit / [Breakout](http://www.switch-science.com/catalog/600/)
* [SoftModem Arduino library](https://code.google.com/p/arms22/downloads/detail?name=SoftModem-005.zip)

_Note_: Switch Science offers a breakout board that saves you the hassle of building the circuit yourself. You can obtain one from [Tinkersoup](https://www.tinkersoup.de/a-569/).

### Sample sketch

```c++
#include <SoftModem.h>

SoftModem modem;

static const byte START_BYTE = 0xFF;
static const byte ESCAPE_BYTE = 0x33;
static const byte END_BYTE = 0x77;

static const unsigned int BAUD_RATE = 57600;

void setup()
{
  Serial.begin(BAUD_RATE);
  delay(1000);
  modem.begin();
}

void decodeByte()
{
  static boolean escaped = false;
  
  while(modem.available())
  {
    byte c = modem.read();
    
    if(escaped)
    {
      Serial.print((char)c);
      escaped = false;
      
      continue;
    }
    
    if(c == ESCAPE_BYTE)
    {
      escaped = true;
      
      continue;
    }
    
    if(c == START_BYTE)
    {
       continue;
    }
    
    if(c == END_BYTE)
    {
      Serial.print('\n');
        break;
    }
    
    Serial.print((char)c);
  }
}

void encodeByte()
{
  if(Serial.available())
  {
  modem.write(START_BYTE);
    while(Serial.available())
    {
      byte c = Serial.read();
      if(c == START_BYTE || c == END_BYTE || c == ESCAPE_BYTE)
      {
        modem.write(ESCAPE_BYTE);
      }
      modem.write(c);
    }
    modem.write(END_BYTE);
  }
}

void loop()
{
  decodeByte();
  encodeByte();
}
```
## Known Issues

iOS determines if plugged-in headphones also offer microphone capabilities. Unfortunately, iOS does not detect the Switch Science breakout board to include a microphone. If you run your app in the iOS simulator on Mac OS X everything works fine as Mac OS X detects the breakout as headphones with microphone. Consequently, you can send data from your iOS devices to Arduino but not the other way around. I assume that this issue is due to a difference in resistance of the microphone pole of the breakout board and the Apple Earpods. A custom circuit might resolve this issue.

## Credits / Acknowledgements

This project uses code from arm22's [SoftModemTerminal](https://code.google.com/p/arms22/wiki/SoftModemBreakoutBoard
) application.

The guys from Perceptive Development provide a great read on their usage of [FSK in their Tin Can iOS App](http://labs.perceptdev.com/how-to-talk-to-tin-can/).

## License

The MIT License (MIT)

Copyright (c) 2014 Jens Meder

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

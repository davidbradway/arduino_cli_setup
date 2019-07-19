# Arduino-Cli notes

To follow these notes https://github.com/arduino/arduino-cli, here were my steps.

I first got an Adafruit FLORA working on my laptop via the Arduino GUI:
- https://learn.adafruit.com/getting-started-with-flora?view=all
- https://learn.adafruit.com/adafruit-arduino-ide-setup/arduino-1-dot-6-x-ide

## CLI setup

Then I downloaded the arduino-cli for Windows and followed along with the first link above and this one: https://learn.sparkfun.com/tutorials/efficient-arduino-programming-with-arduino-cli-and-visual-studio-code/introduction-to-the-arduino-cli

I renamed the executable, moved it to `C:\Program Files (x86)\arduino-cli`, and added that path to the `PATH` User Environment Variable.

## Adding support for 3rd-party cores (Adafruit, ESP8266, Digispark, etc)

I couldn't find the `arduino-cli.yaml` to add the URLs to the Board Manager until I found: https://github.com/arduino/arduino-cli/issues/5

Then from CMD, I did: `arduino-cli.exe config init` and this showed the path (or maybe generated the file?): `Config file PATH: C:\Users\dpb6\AppData\Local\Arduino15\arduino-cli.yaml`

Then in that `yaml` file, I added the last 3 lines below to add Adafruit and ESP8266 boards:
```
proxy_type: auto
sketchbook_path: C:\Users\dpb6\Documents\Arduino
arduino_data: C:\Users\dpb6\AppData\Local\Arduino15
board_manager:
  additional_urls:
    - https://adafruit.github.io/arduino-board-index/package_adafruit_index.json
    - http://arduino.esp8266.com/stable/package_esp8266com_index.json
```

Then, back in CMD, I did: `arduino-cli core update-index` to download the JSON files from the new URLs:
```
Updating index: package_index.json downloaded
Updating index: package_index.json downloaded
Updating index: package_adafruit_index.json downloaded
Updating index: package_esp8266com_index.json downloaded
```

## Connecting a board

Then I connected the FLORA microcontroller to a USB port and did: `arduino-cli board list` which returned:
```
Port  Type              Board Name FQBN
COM10 Serial Port (USB) Unknown
```
Since the Board Name and FQBN are unknown, I needed to install support for the FLORA.

I searched for the type of board in the board manager lists: `arduino-cli core search flora`, which returned:
```
Searching for platforms matching 'flora'
ID            Version Name
TeeOnArdu:avr 1.0.3   Adafruit TeeOnArdu
adafruit:avr  1.4.13  Adafruit AVR Boards
```

I installed the board I wanted: `arduino-cli core install adafruit:avr`

I then listed the cores that were attached: `arduino-cli core list`, which showed:
```
ID                  Installed Latest Name
adafruit:avr@1.4.13 1.4.13    1.4.13 Adafruit AVR Boards
```

Then, I Listed All `arduino-cli board listall flora` to get the Fully Qualified Board Name (FQBN):
```
Board Name     FQBN
Adafruit Flora adafruit:avr:flora8
```

And I listed the attached boards again `arduino-cli board list` to see that the Board Name and FQBN are now associated with the FLORA plugged in via USB:
```
Port  Type              Board Name     FQBN
COM10 Serial Port (USB) Adafruit Flora adafruit:avr:flora8
```

## Compiling

I was getting errors on "compile", until I found this: https://github.com/arduino/arduino-cli/issues/177#issuecomment-484291281 and did the following:
`arduino-cli core install arduino:avr`

After which I could compile the code: `arduino-cli compile --fqbn adafruit:avr:flora8 C:/Users/dpb6/Downloads/repos/myFirstSketch` which returned:
```
Build options changed, rebuilding all
Sketch uses 4130 bytes (14%) of program storage space. Maximum is 28672 bytes.
Global variables use 149 bytes of dynamic memory.
```

## Uploading

And finally, I uploaded the binaries to the board: `arduino-cli upload -p COM10 -v --fqbn adafruit:avr:flora8 C:/Users/dpb6/Downloads/repos/myFirstSketch`:
```
avrdude: Version 6.3-20171130
         Copyright (c) 2000-2005 Brian Dean, http://www.bdmicro.com/
         Copyright (c) 2007-2014 Joerg Wunsch

         System wide configuration file is "C:\Users\dpb6\AppData\Local\Arduino15\packages\arduino\tools\avrdude\6.3.0-arduino14/etc/avrdude.conf"

         Using Port                    : COM11
         Using Programmer              : avr109
         Overriding Baud Rate          : 57600
         AVR Part                      : ATmega32U4
         Chip Erase delay              : 9000 us
         PAGEL                         : PD7
         BS2                           : PA0
         RESET disposition             : dedicated
         RETRY pulse                   : SCK
         serial program mode           : yes
         parallel program mode         : yes
         Timeout                       : 200
         StabDelay                     : 100
         CmdexeDelay                   : 25
         SyncLoops                     : 32
         ByteDelay                     : 0
         PollIndex                     : 3
         PollValue                     : 0x53
         Memory Detail                 :

                                  Block Poll               Page                       Polled
           Memory Type Mode Delay Size  Indx Paged  Size   Size #Pages MinW  MaxW   ReadBack
           ----------- ---- ----- ----- ---- ------ ------ ---- ------ ----- ----- ---------
           eeprom        65    20     4    0 no       1024    4      0  9000  9000 0x00 0x00
           flash         65     6   128    0 yes     32768  128    256  4500  4500 0x00 0x00
           lfuse          0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           hfuse          0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           efuse          0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           lock           0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           calibration    0     0     0    0 no          1    0      0     0     0 0x00 0x00
           signature      0     0     0    0 no          3    0      0     0     0 0x00 0x00

         Programmer Type : butterfly
         Description     : Atmel AppNote AVR109 Boot Loader

Connecting to programmer: .
Found programmer: Id = "CATERIN"; type = S
    Software Version = 1.0; No Hardware Version given.
Programmer supports auto addr increment.
Programmer supports buffered memory access with buffersize=128 bytes.

Programmer supports the following devices:
    Device code: 0x44

avrdude: devcode selected: 0x44
avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.01s

avrdude: Device signature = 0x1e9587 (probably m32u4)
avrdude: safemode: lfuse reads as FF
avrdude: safemode: hfuse reads as D0
avrdude: safemode: efuse reads as CB
avrdude: reading input file "C:/Users/dpb6/Downloads/repos/myFirstSketch/myFirstSketch.adafruit.avr.flora8.hex"
avrdude: writing flash (4130 bytes):

Writing | ################################################## | 100% 0.49s

avrdude: 4130 bytes of flash written
avrdude: verifying flash memory against C:/Users/dpb6/Downloads/repos/myFirstSketch/myFirstSketch.adafruit.avr.flora8.hex:
avrdude: load data flash data from input file C:/Users/dpb6/Downloads/repos/myFirstSketch/myFirstSketch.adafruit.avr.flora8.hex:
avrdude: input file C:/Users/dpb6/Downloads/repos/myFirstSketch/myFirstSketch.adafruit.avr.flora8.hex contains 4130 bytes
avrdude: reading on-chip flash data:

Reading | ################################################## | 100% 0.15s

avrdude: verifying ...
avrdude: 4130 bytes of flash verified

avrdude: safemode: lfuse reads as FF
avrdude: safemode: hfuse reads as D0
avrdude: safemode: efuse reads as CB
avrdude: safemode: Fuses OK (E:CB, H:D0, L:FF)

avrdude done.  Thank you.
```

This ends the README!

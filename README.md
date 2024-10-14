# PaintaudioMidiCaptain
Micropython to control the Paint Audio Midi Captain.   
This is a fascinating pedal with ten switches with three neopixel indicators per switch, a ST7789 TFT display, rotary encoder, uart midi, wireless midi and two expression pedals.    
It uses the Raspberry Pi RP2040 microntroller chip.   

The wireless midi is 2.4GHz but not using the BLE protocol. It uses a BK2461 chip (which runs a similar radio interface as the nRF24L01).   

## Information sources

Based on discoveries in these repos:    

```
https://github.com/nicola-lunghi/hiper-midicaptain
https://github.com/gstrotmann/MidiCaptain4Kemper

```


## Pedal layout    

```
Switch layout

1   2   3   4   UP
A   B   C   D   DOWN
````


## GPIO usage    

| GPIO     | Description                                             |
|----------|---------------------------------------------------------|
| GPIO 0   | Rotary encoder                                          | 
| GPIO 1   | Switch 1                                                | 
| GPIO 2   | ?                                                       | 
| GPIO 3   | ?                                                       | 
| GPIO 4   | LC12S Chip Select (low)                                 | 
| GPIO 5   | LC12S Set Data / Command (data is high, command is low) | 
| GPIO 6   |                                                         | 
| GPIO 7   | Neopixel                                                | 
| GPIO 8   |                                                         | 
| GPIO 9   | Switch A                                                | 
| GPIO 10  | Switch B                                                | 
| GPIO 11  | Switch C                                                | 
| GPIO 12  | TFT DC                                                  | 
| GPIO 13  | TFT CS                                                  | 
| GPIO 14  | TFT SPI CLK                                             | 
| GPIO 15  | TFT SPI MOSI                                            | 
| GPIO 16  | UART TX  (Midi and LCS12S)                              | 
| GPIO 17  | UART RX  (Midi and LCS12S)                              | 
| GPIO 18  | Switch D                                                | 
| GPIO 19  | Switch Down                                             | 
| GPIO 20  | Switch Up                                               | 
| GPIO 21  |                                                         | 
| GPIO 22  |                                                         | 
| GPIO 23  | Switch 4                                                | 
| GPIO 24  | Switch 3                                                | 
| GPIO 25  | Switch 2                                                | 
| GPIO 26  |                                                         | 
| GPIO 27  | Expression 1                                            | 
| GPIO 28  | Expression 2                                            | 
| GPIO 29  |                                                         | 


## Wireless configuration and use    

Code to set and use the wireless connection - to send a sample midi message.
It seems that both the midi uart and wireless midi use GPIO 16 and 17, a single uart in the RP2040.   The wireless midi uses baud rate 38,400 and midi uart uses 31,250, the midi standard.   Not sure how that works it needs further investigation.    
The initialisation of the BK2461 board is at 9,600 baud and it seems to forget the configuration between reboots.   
The configuration data has a different format the LC12S board, which seems to be the closest commercially available BK2461 board.  Same start bytes, length and general format, but different in the address use.   

```
from  board import GP4, GP5, GP16, GP17
import busio
from digitalio import DigitalInOut, Direction, Pull
import time
    
uart = busio.UART(tx=GP16, rx=GP17, baudrate=9600, timeout = 0.1)

chip_sel = DigitalInOut(GP4)
chip_sel.direction = Direction.OUTPUT
set_data = DigitalInOut(GP5)
set_data.direction = Direction.OUTPUT
set_data.value = False
chip_sel.value = False
time.sleep(0.1)

config   = [0xAA, 0x5A, 0x08, 0x44, 0x11, 0x33, 0x00, 0x06, 0x00, 0x06, 0x00, 0x64, 0x00, 0x00, 0x00, 0x12, 0x00, 0x16]

uart.write(bytes(config))
time.sleep(0.3)
print("".join(" x%02x" % i for i in config))
recv = uart.read(18)
print("".join(" x%02x" % i for i in recv))

set_data.value = True

# change baudrate
uart.baudrate = 38400

# simple MIDI message - Eb note on
msg = [0x80, 0x33, 0x7f] 

while True:
    uart.write(bytes(msg))
    time.sleep(1)
```


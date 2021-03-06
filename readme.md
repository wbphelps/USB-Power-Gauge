USB Power Gauge updates by Eugene Skopal
===============
Version 2.2 -- 02-Dec-2013
--------------------------
Observations / Changes
----------------------
I noticed that I was just about out of memory for the updated sketch (99%) was in use.  I updated the sketch to conserve a little more memory by using only Print() and not Println().  The build shows 98% in use.

I had downloaded the "trinket" support files from ADAfruit to work on the USB Power Gauge.  It occurred to me that the ATtiny85 has 8192 bytes of flash, but the sketch reported only 5310 bytes available.  This is specified in the ..\Arduino\hardware\attiny\boards.txt file that is part of the trinket support files supplied by Adafruit.  It limits the trinket sketch to 5310 bytes of memory so that the trinket bootloader is not overwritten.  Since we do not use a bootloader, there is no reason to conserve that space, so I updated the boards.txt file to include an �Adafruit USB Power Gauge� board that has 8192 bytes of memory.  You need to copy that file over the existing boards.txt, and change the sketch to use the "Adafruit USB Power Gauge" board to get access to the extra memory.  The build now shows 63% in use.

Version 2.1 -- 01-Dec-2013
--------------------------
Observations
------------
When testing the USB Power Gauge I noticed that when the voltage was low, the green LED went out as it was designed to do.  I found this confusing because with no load, it appeared that there was no power, not that the voltage was too low.

Also, when looking for the appropriate place to post a notice about this new firmware, I noticed that others had asked for the power gauge to support 2A (10w) power supplies (for Apple IPAD).

Code Changes
------------
  1) Compute the calibration value for 2.56v reference
  
  2) Read currents up to 2.56 Amps
  
  3) Flash Green LED at 4 hz if voltage is less than 4.5 volts
  
  4) Blink Green LED at .5 hz if watts > 5.125, display watts as 6,7,8,9,10
  
  5) Blink Flashing Green LED if low voltage and high watts
      (Flashes 4 times in one second, then off for one second)

Version 2.0 -- 30-Nov-2013
--------------------------
Original Code Observations
--------------------------
When using the USB Power Gauge I found the green power led was flickering as well as the first blue led.
I was concerned that my USB hub was not providing a solid 5v supply, so I hooked up a USB-TTL serial cable and checked the voltage.  I found that the voltage was over 5v and that it was being misreported.  This meant there was some other reason for the green LED flickering.

The first blue LED flickered randomly even though there was no load connected.

Code Review Observations
------------------------
I reviewed the code available here on GitHub and discovered the reason for the Green LED flickering was the SoftwareSerial implementation.  Even though the original code cleverly used interrupts to control the Charlie-Plexing (http://en.wikipedia.org/wiki/Charlieplexing) of the LED's SoftwareSerial turned off the interrupt system while it was outputting the individual characters causing a noticeable delay in the LED display.  The original code tried to time the output of characters to minimize the flickering, but it was not really successful.

The incorrect 5V value was due to the calibration value being incorrect for the USB-Power-Gauge I had received.

Finally, the blue LED flickering was due to two issues.  First, the output of the current monitor is very noisy, with random, but infrequent, readings of several ma of current even with no load connected.  Secondly, the original code uses a clever "gamma" table to try and vary the brightness of the LEDs for current between 1 and 1000 ma.  Unfortunately, the implementation resulted in the LED turning on even when the value was only 1ma.

Code Changes
------------
I tracked down code that implemented a UART using a TIMER (http://www.siwawi.arubi.uni-kl.de/avr_projects).  I renamed it TimerSerial and re-worked it to run on the ATtiny85 in the Arduino environment.  This fixed the flickering Green LED.

I added code to automatically update the calibration value twice if it is missing.  This allowed me to erase the EEPROM using AVRDUDE and a USBTinyISP.  When the USB-Power-Gauge reboots after erasing the EEPROM, it computes a calibration value based on the inaccurate 5v supply of the programmer.  I then simply unplug the USB-Power-Gauge and plug it into a Desktop computer USB port that presents a very accurate 5v supply.  The calibration value is recomputed and is a good deal more accurate.

I rewrote the main loop to average the current value for one second.  This removed the random noise from the current sensor.  The code now displays both the average current and the peak current each second.

I corrected the "gamma" display routines to leave the LED completely off for very small values of current.

Finally, I added a feature TimerSerial that allows the program to sense the value of the TX pin.  When it isn't sending a status update it watches for the TX pin to be pulled (by a 1K resistor to Ground).  If it's pulled low for 3 seconds, once it is released, a new calibration value is written.


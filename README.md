# Thermostat...expanded for Arduinos with more flash available
This adds features that the first thermostat sketch could not fit into a number of 32K flash Arduinos that did not make 
their full flash capacity available for sketches. Specifically, don't expect the auto thermostat mode option to be 
available in Leonardos, Esplora nor Yún, for examples, so that those boards can have calibrated KY-013 with DHT capability 
with this version.  At some point in future enhancements, I foresee a time when the sketch will require the board to have 
at least 64K flash size.

You'll need to download four files: the .ino and the three .h files.  

First, a recap:

# Arduino Home Thermostat and Automation Hub
I use this sketch in the Arduino UNO, Mega2560, nano, etc. as my thermostat, and you can easily modify
it for humidistat as well.  I have compiled it for all boards I could select in the IDE and ensured it would compile for 
the boards having at least 32K flash.

Currently DHT11, DHT22, and KY-013 sensors only are supported.  Obviously, the DHT sensors can connect to Digital and 
dual-mode pins while the KY-013 can connect to Analog and dual-mode pins. Note that the default accuracy of a KY-013 analog sensor depends on the DC voltage supplying the "-" pin (these sensors are labeled opposite what the sketch formula is!)
For the best accuracy, adjust the per-sensor pin calibration value OR supply these sensors with an adjustable and stable supply of about 5 volts +/- a volt.  Adjust the voltage for accurate readings.

This sketch also makes Arduino digital pins to be readable and settable from an optional serial-connected host computer.  
The host computer can control and read both the digital voltage levels and DHT data of any pin.  For maximum capability, 
the host computer can optionally run [a] capturing daemon[s] to receive and process the logging output from the Arduino.  
Temperature changes logging, an option if main logging is also on, is on by default.  The host computer can utilize 
temperature changes data to track and analyze or to incorporate into a ceiling fan control algorithm, just to name a couple
uses.  See the auxiliary host processing scripting that I use in Ubuntu in the file above titled similarly.  In Ubuntu 
remember to grant membership to the dialout group for the host Ubuntu computer to establish serial communications to the 
Arduino.

Being connected via USB to my Ubuntu headless firewall, I have remote control of it from over the Internet.  It will send 
alerts to the host for furnace failure to heat. I use a bash script on the host to email me in event of furnace failure.  
EEPROM is utilized so the thermostat settings are persistent across power cycling.  I even update the compiled sketch into 
the board remotely!

See the screen shot of the help screen that it can display.  Sensors must be either DHT11, DHT22, or KY-013 in this version.  It expects two sensors - a primary and a backup (secondary) sensor for failsafe operation.  Because sensors are
read without utilizing interrupts, any digital pin other than the serial communications can be used for DHT sensors!  A
little-documented feature is that the pins driving sensors can be forced to only allow DHT devices by adding 128 to the
EEPROM location storing the digital pin number for that sensor.  This is a safety feature: in case the DHT sensor fails,
the 128 added to the pin number stored in EEPROM prevents the pin from going into analog mode and interpreting any voltage
appearing on the pin as a signal from an analog (KY-013) temperature sensor.

This sketch is also a wrapper to allow the host computer to read and control the Arduino digital pins with or without taking
advantage of the thermostat functionality!  As examples:

-  I have my host computer calculate when my porchlights are to be turned on and off through a pin by lookup file of sunset
   and sunrise times

-  I have my coffee maker outlet connected to another pin and host-controlled on a schedule 

-  The host computer can read temperatures and humidities from more DHT11/22 devices on other pins.  Outdoor temperature
    and humidity can thus be acquired by the host computer

-  Ceiling fan control

-  Etc., etc.!  See the help screen for details.

This sketch occupies all or nearly all available flash (program) memory in boards having 32K of flash.  Therefore, you'll 
not be able to add features in most 32K flash Arduinos.  Actual compiled sketch size varies in the range of 27K-28K, so 
boards that allow the entirety of the 32K flash for the user's sketch will have some flash space available for you to add 
features.

This sketch compiled with IDE version 1.8.6 on 03/14/18.  Older versions of IDE and future library memory footprint 
expansions can neccessitate tweaking.

TTGO XI/WeMo XI NOTE:  This board trades flash to simulate EEPROM at a cost of 2 to 1.  This sketch assumes your compiler 
EEPROM command line settings are set for 1K EEPROM.  That is the minimum block size of EEPROM the board allows.  Any more 
EEPROM by the compiler and this sketch will not fit.  Fortunately for me, my default compiler settings were correct to this
requirement, otherwise I wouldn't know how to change them back to what they are.

IMPORTANT SAFETY NOTE - Proper electrical isolation by relay or opto-isolation must be observed whenever connecting to 
building electrical or furnace controls or other electrical equipment.  For the furnace controls, I use ralays driven by 
open-collector NPN transistors with a 12vdc power supply.  Don't forget the diode across the relay coil.  For 120vac
switching, I use opto-coupled dc-drive SSRs, again, driven by NPN transistors and 12vdc.

TODO - Future enhancements are reserved for boards having greater than 32K flash size: I2C port expansion, duct damper 
operation for multi-thermostat-single-furnace environments.

WANTED:  ASSISTANCE CONDENSING THE SIZE OF THIS SKETCH BY USE OF ASM CODE.  ANY TAKERS?  POST AN ISSUE, PLEASE.

# KY-013 per-sensor calibration is NOW included as of version .0.9.

An array of calibration offsets, one per Analog pin, begins at the EEPROM address stored in EEPROM addresses 12 and 13.
Both 12 & 13 had to be used to store an EEPROM address because the size of EEPROM is greater than 256.  MSB is in 13
(little endian, though I protest the de facto nomenclature as being backwards).  The factory defaults this value for entire
array to one that worked for uno when the KY-013 was powered by 5vdc: -31, for all Analog pins.  Since the -31 is displayed
as an unsigned char, it shows as 225 when viewing persistent memory.  When manually changing a calibration offset for a
sensor, you'll need to enter the value as an unsigned char inasmuch as values above 127 to 255 are actually negative
offsets: 255 = -1 and 200 = -56.

So you ask, "What does the -31 correlate to?"  Right now as this sentence is being typed, 20% of that number is a simple
add to the computed temperature reading.  The other 80% of that value is scaled by how close/distant the reading is from 
near mid-scale (565) - the closer to that near mid-scale point, the more of that last 80% of the value is added to the raw 
analog read value from the pin, prior to the calculations that produce a temperature from the raw value.  The source code 
is written with those two elements of the ratio nearby each other in an editor for easy adjustment if you'd rather have 
more of the offset being absolute or be based on the analog value received by the pin.  I don't expect this to be the final 
calibration algorithm I employ.

Again, just to clarify, that value is set to a default value when the sketch configures EEPROM at first-run, and you, the
user/sketch-editor are expected to fine-tune the value.  There is a separate value for each analog pin in an array whose
beginning bounds is the address contained in EEPROM ("persistent memory") location 12-13. The reason for the indirect 
addressing?  Because that array size is dependent on how many Analog pins are on the board, thus causing any following 
items stored to be unpredictably located, and I'd rather have that stable EEPROM location of 14 to store an upcoming EEPROM
feature (probably user-defined pin names).

Note that to convert an address stored spanning two locations, you consider the second location to be showing the number of 
times 256 can fit in the final number:  if location 13 has a 3, for example, that becomes 768 added to whatever value is in loation 12.


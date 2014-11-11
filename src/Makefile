# Name: Makefile
# Author: <insert your name here>
# Copyright: <insert your copyright message here>
# License: <insert your license reference here>

# This is a prototype Makefile. Modify it according to your needs.
# You should at least check the settings for
# DEVICE ....... The AVR device you compile for
# CLOCK ........ Target AVR clock rate in Hertz
# OBJECTS ...... The object files created from your source files. This list is
#                usually the same as the list of source files with suffix ".o".
# AVRDUDE_PROGRAMMER ... Options to avrdude which define the hardware you use for
#                uploading to the AVR and the interface where this hardware
#                is connected. We recommend that you leave it undefined and
#                add settings like this to your ~/.avrduderc file:
#                   default_AVRDUDE_PROGRAMMER = "stk500v2"
#                   default_serial = "avrdoper"
# FUSES ........ Parameters for avrdude to flash the fuses appropriately.

#DEVICE
DEVICE		= attiny85
CLOCK		= 16500000

#PROGRAMMER
PROGRAMMER	= stk500v1
SERIAL_DEVICE = /dev/cu.usbserial-A5029U9R
BAUDRATE	= 19200
FUSES		= -Uefuse:w:0xff:m -Uhfuse:w:0xdf:m -Ulfuse:w:0xe2:m

#AVR
AVRDIR		= /Applications/Development/Arduino.app/Contents/Resources/Java/hardware/tools/avr
AVRBIN		= $(AVRDIR)/bin
AVRCONFIG	= $(AVRDIR)/etc/avrdude.conf
OBJECTS		= main

# ATMega8 fuse bits used above (fuse bits for other devices are different!):
# Example for 8 MHz internal oscillator
# Fuse high byte:
# 0xd9 = 1 1 0 1   1 0 0 1 <-- BOOTRST (boot reset vector at 0x0000)
#        ^ ^ ^ ^   ^ ^ ^------ BOOTSZ0
#        | | | |   | +-------- BOOTSZ1
#        | | | |   +---------- EESAVE (set to 0 to preserve EEPROM over chip erase)
#        | | | +-------------- CKOPT (clock option, depends on oscillator type)
#        | | +---------------- SPIEN (if set to 1, serial programming is disabled)
#        | +------------------ WDTON (if set to 0, watchdog is always on)
#        +-------------------- RSTDISBL (if set to 0, RESET pin is disabled)
# Fuse low byte:
# 0x24 = 0 0 1 0   0 1 0 0
#        ^ ^ \ /   \--+--/
#        | |  |       +------- CKSEL 3..0 (8M internal RC)
#        | |  +--------------- SUT 1..0 (slowly rising power)
#        | +------------------ BODEN (if 0, brown-out detector is enabled)
#        +-------------------- BODLEVEL (if 0: 4V, if 1: 2.7V)
#
# For computing fuse byte values for other devices and options see
# the fuse bit calculator at http://www.engbedded.com/fusecalc/


# Tune the lines below only if you know what you are doing:
AVRDUDE_PROGRAMMER = -c$(PROGRAMMER) -P$(SERIAL_DEVICE) -b$(BAUDRATE)
AVRDUDE = $(AVRBIN)/avrdude $(AVRDUDE_PROGRAMMER) -p $(DEVICE) -C $(AVRCONFIG)
COMPILE = $(AVRBIN)/avr-gcc -Wall -Os -DF_CPU=$(CLOCK) -mmcu=$(DEVICE)

# symbolic targets:
all:	$(OBJECTS).hex

.c.o:
	$(COMPILE) -c $< -o $@

.S.o:
	$(COMPILE) -x assembler-with-cpp -c $< -o $@
# "-x assembler-with-cpp" should not be necessary since this is the default
# file type for the .S (with capital S) extension. However, upper case
# characters are not always preserved on Windows. To ensure WinAVR
# compatibility define the file type manually.

.c.s:
	$(COMPILE) -S $< -o $@

flash:	all
	$(AVRDUDE) -U flash:w:$(OBJECTS).hex:i

fuse:
	$(AVRDUDE) $(FUSES)

# Xcode uses the Makefile targets "", "clean" and "install"
install: flash fuse

# if you use a bootloader, change the command below appropriately:
load: all
	bootloadHID $(OBJECTS).hex

clean:
	rm -f $(OBJECTS).hex $(OBJECTS).elf $(OBJECTS).o

# file targets:
$(OBJECTS).elf: $(OBJECTS).o
	$(COMPILE) -o $(OBJECTS).elf $(OBJECTS).o

$(OBJECTS).hex: $(OBJECTS).elf
	rm -f $(OBJECTS).hex
	$(AVRBIN)/avr-objcopy -j .text -j .data -O ihex $(OBJECTS).elf $(OBJECTS).hex
	$(AVRBIN)/avr-size --format=avr --mcu=$(DEVICE) $(OBJECTS).elf
# If you have an EEPROM section, you must also create a hex file for the
# EEPROM and add it to the "flash" target.

# Targets for code debugging and analysis:
disasm:	$(OBJECTS).elf
	avr-objdump -d $(OBJECTS).elf

cpp:
	$(COMPILE) -E $(OBJECTS).c

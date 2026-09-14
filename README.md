# SDCore
So I went and made an entire SD card because I thought my PicoCalc didn't have enough juice.

I am not responsible for your usage or misusage of this project.

This SD core is based on the RP2350A by raspberry pi and features:
- Hardware-level control of the routing of the SD pins
- Three state indicating LEDs (power, program, and select)
- 16MB of flash
- A micro-SD card slot
- Additional GPIO output (see below)
- An access point for programming when setting pin 6 to 3.3V

# Images
None yet, we'll see when the order arrives.

# Hardware pin control
I added a tri-state demultiplexer ([NMUX27518EPWJ](https://assets.nexperia.com/documents/data-sheet/NMUX27518E.pdf)) to switch between 2 modes, direct transfer mode and sniffing mode (default is direct transfer mode). In direct transfer mode, data0-data3, CLK and CMD are all mapped to the RP2350A and set up for communication via **SPI1**. While sniffing mode also maps the data pins to the MCU, but on **SPI0**, it also directs them towards the micro-SD card slot allowing the MCU to sniff/spy on the communication between the host device and the micro-SD card.

> [!IMPORTANT]
> When using direct transfer mode SPI1 is used and in sniffing mode we use SPI0. The pinouts are inherently different. Please look at the [pinout](#pinout) section for details.

The different SPI controllers are utilized in order to communicate with the host and the micro-SD card is held separately and/or simultaneously.
# How to program the RP2350A
There are 2 main scenarios, either you have the programmer board, or you don't. In the case that you do, connect the USB-C port to your PC or laptop, plug the SD core into the programmer board while holding the button on the board, once you see the USB mass storage on your laptop (which you have presumably plugged into the programmer board already) you can release the button.

If you do not own a programmer board, (though it is highly recommended that you do) you can order one with the given gerber, BOM and P&P files which can be found in the `SDCore_progBoard/prodution` folder which also holds a few key values for ordering with PCB Assembly on [JLCPCB](jlcpcb.com).

> [!NOTE]
> The `.zip` file is the compressed gerber files, `bom.csv` is the BOM and `positions.csv` is the P&P file.

If ordering this PCB is not an option, please continue in the subsection below.

## Making a programmer board at home
You'll need:
- A reliable 3.3V power source
- a switch,
- a breakout board of any USB end (male or female, type-A or type-C, whichever you can use to plug into your PC) with a D- and a D+ pin. If there are multiple of each, short them such that you only have **one** D- and **one** D+,
- a breakout board of a **standard SD card** with ALL of these pins present: 3 (VSS/GND), 4 (VDD/3V3), 6 (the magic pin), 8 (DATA_1) and 9 (DATA_2) - other pins are optional.

> [!WARNING]
> If any of these SD pins are missing from the breakout, you will not be able to program your SDCore.

If you don't have an SD breakout, look have a look at [this](https://www.instructables.com/Cheap-DIY-SD-card-breadboard-socket/) article on how to make one with all the needed pins.

Now you connect it as follows:
1. GND to pin 3 (may also be labeled as VSS or GND)
2. 3.3V to pin 4 (may also be labeled as VDD, VCC or 3V3)
3. USB D- (data negative/minus) to pin 8 (may also be labeled as DATA1 or DAT1)
4. USB D+ (data negative/minus) to pin 9 (may also be labeled as DATA2 or DAT2)
5. Connect one end of the switch to 3.3V and the other to pin 6 (may also be labeled as VSS or GND)

You may leave all the other pins unconnected or tie them to GND.

Normally, pin 6 is an alternate GND pin, however the SD core uses it to signify the BOOTSEL function and boot into the USB mass storage bootloader. 
> [!WARNING]
> Using this setup for a regular SD card will lead to shorts.

> [!CAUTION]
> This device does not support booting via UART, USB is currently the only supported boot form.

You can now plug this setup into your PC, enable the 3.3V source and connect the SD core while holding the switch. Once you see a USB mass storage device you may release the switch.

# Pinout
By default, the SD pins are configured to go to the MCU directly over the following pins:
```
Using SPI1
GPIO 8 - SPI1 MISO
GPIO 9 - SPI1 CSn
GPIO 10 - SPI1 CLK
GPIO 11 - SPI1 MOSI

GPIO 0 - DATA1 (pin 8)
GPIO 1 - DATA2 (pin 9)
```
in sniffing mode, they connect as follows:
```
Using SPI0
GPIO 4 - SPI0 MISO
GPIO 5 - SPI0 CSn
GPIO 6 - SPI0 CLK
GPIO 7 - SPI0 MOSI

GPIO 24 - DATA1 (pin 8)
GPIO 25 - DATA2 (pin 9)
```

The selection signal is controlled by GPIO 3 and is pulled down to GND with 10kΩ. Driving this signal high will put the demultiplexer into the sniffing mode pinout, whereas the direct mode's pinout becomes disconnected.

The connections/pins 3V3, GND, SWCLK, SWDIO, RUN, GPIO 2, GPIO 12-23 and GPIO 26-29 are exposed via a board-to-board connector on the SDCore. A breakout cable has not yet been added to this repo.

> [!CAUTION]
> The USB signals of the RP2350A are always connected to the DATA 1 and 2 pins.
> It is the user's responsibility to ensure the MCU does not read USB when it is not supposed to, this also means disabling stdio over USB ***and*** stdio over UART when not using the programmer board.

# Credits
All mine baby.

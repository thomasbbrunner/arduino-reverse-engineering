# Reverse Engineering an Arduino Application

Recently, I received a used Arduino Uno Rev3 SMD board as a gift. I knew that the previous owner had used the board for prototyping, but didn't know exactly for what. I thought that it would be interesting to try and extract the binary from the Arduino and to disassemble it.

Moreover, as it was my first time using an AVR microcontroller, it would give me an introduction to the development tools of AVR microcontrollers.

## Requirements

`avrdude`: program to manipulate the memory of AVR microcontrollers. It can be downloaded from Linux package managers. 

For Ubuntu: `sudo apt install avrdude`

`avr-objdump`: program to display the memory contents in a more human-readable way. It can also be downloaded from Linux package managers. 

For Ubuntu: `sudo apt install binutils-avr`

## Section 0: Connecting the Arduino to the PC

First connect the Arduino to the computer using the USB cable. 

## Section 1: Extracting the program

The first step is to extract the program from the microcontroller's flash memory. Use `avrdude` to read the memory's contents:

Extract raw binary:

`avrdude -p m328p -c arduino -D -P<port> -b 115200 -v -U flash:r:flashdump.bin:r`

Extract hex:

`avrdude -p m328p -c arduino -D -P<port> -b 115200 -v -U flash:r:flashdump.hex:i`

*Remember to replace `<port>` by the actual port that your Arduino is connected to (Google can help you here).*

## Section 2: Investigating the program

### Looking for strings

An interesting thing to try is to see which strings can be found in the binary. This could give us some hints on what the program does. To extract strings run:

`strings flashdump.bin`

For the case of the mysterious program in my new Arduino, there were strings like `/arduino_vacuum`, `arduino setting suction ON`, `std_msgs/Bool` and `std_msgs/Time`. This indicates that this Arduino was connected to some sort of pump, and that it was using a ROS (Robot Operating System) interface. Pretty neat!

### Taking a look at the binary

Now, we also want to see the contents of the binary for ourselves, maybe we find something interesting there. As we are dealing here with a binary file, we can't simply open it in the editor. Instead, we must use a program called `xxd`, which will create a hex dump of the binary:

`xxd flashdump.bin`

At first glance the output may look like a complicated mess. But if we examine it more carefully, we can find, for example, the strings we found in the previous part:

```
...
(Address) (Data)                                   (Data as chars)
00002e90: 7365 7200 2f74 6f6f 6c5f 7661 6375 756d  ser./tool_vacuum
00002ea0: 000f 000e 0005 0004 0000 00b5 0780 0001  ................
00002eb0: 7374 645f 6d73 6773 2f54 696d 6500 6364  std_msgs/Time.cd
...
```

You'll also see that the program is split into two parts. There is some "stuff" at the top, some other "stuff" at the bottom, and the middle is filled with 00 or FF. Well, what you are seeing there are the application and bootloader sections of the flash memory. The top of the flash (low addresses) is were your application is stored. The bottom of the flash (high addresses) is where the bootloader is stored. And the middle is just unused memory.

### Disassembling the binary

To get the assembly code of your program, all you need to do is run `objdump` on the `.hex` file:

`avr-objdump -Dx -m avr5 flashdump.hex`

And that's it! We should have now the assembly code of the binary file!

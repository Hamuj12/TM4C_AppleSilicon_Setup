# Setting up Keil development on Apple Silicon Macs
This guide will show you how to run Keil MicroVision5 in a Windows VM as well as being able to flash/debug the TM4C123GXL.

## Setting up Keil MicroVision5

### Prerequisites
This guide was developed using Parallels, but should work with any VM software. Parallels is endorsed by Microsoft and comes configured with Windows 11 on ARM. It is not a free software, a student license is $49/year and a pro student license is $59/year.

You can download Parallels from [here](https://www.parallels.com/products/desktop/download/).

### Installing Keil
To install Keil MicroVision5, you can follow the steps exactly as outlined in Dr. Valvano's guide [here](https://users.ece.utexas.edu/~valvano/EE445L/downloads.htm#Keil). Stop at the "How to install LaunchPad drivers for the TM4C123" section.

After installing Keil, you'll need to install a few more things:

1. The TivaWare library. You can download it from [here](http://www.ti.com/tool/sw-tm4c).
2. ValvanoWare_v5. You can download it from [here](http://users.ece.utexas.edu/~valvano/arm/ValvanoWareTM4C123v5.zip).
3. EE319KWare. You can download it from [here](http://users.ece.utexas.edu/%7Evalvano/Volume1/EE319K_Install.exe).

### Setting up Keil and testing build
Now that you have Keil and the libraries installed, you can open Keil and try to build a project. To test, I used the `PeridiocTimer1AInts_4C123` project from the ValvanoWare_v5 folder. 

1. Open Keil and click on `Project` -> `Open Project...` and navigate to the `PeridiocTimer1AInts_4C123` folder. Click on `PeridiocTimer1AInts_4C123.uvprojx` and click `Open`.
2. Now we need to setup some options for the target. Click on the `Options for Target 'PeridiocTimer1AInts_4C123'` button. 
![Options for Target](./assets/OptionsForTarget.png)
3. In the `Target` tab, under the code generation section, make sure that the ARM compiler is set to "Use default compiler version 6".
![Target Tab](./assets/TargetTab.png)
4. In the `Output` tab, make sure that the `Create HEX File` box is checked. Also make sure that the name of the output file is `Timer1A.elf`. In the future make sure your output file name ends in `.elf` so that we can use it for debugging.
![Output Tab](./assets/OutputTab.png)
5. In the `C/C++` tab, under the `Languge/Code Generation` section, make sure that Language C is set to "c99".
![C/C++ Tab](./assets/CCTab.png)
6. In the `User` tab, add a User Command to the `After Build/Rebuild` section. The command should be `fromelf.exe --bin Objects/Timer1A.elf --output Objects/Timer1A.bin`. This will convert the .axf file to a .bin file that we can flash to the board. We are using the `Timer1A.elf` file because that is the name of the file that is generated when we build the project, so if you are using a different project, you will need to change the input and output file names.
![User Tab](./assets/UserTab.png)
7. Click `OK` to save the settings.

Now we can build the project. Click on the `Build` button. If everything is setup correctly, you should see something like this in the output window:
![Build Output](./assets/BuildOutput.png)

Check the `PeridiocTimer1AInts_4C123/Objects` folder and you should see a `Timer1A.bin` file. This is the file that we will flash to the board.

## Setting up flashing/debugging using OpenOCD

### Prerequisites
To flash and debug the TM4C123GXL, we will be using OpenOCD. You can install OpenOCD using MacPorts or Homebrew. I used MacPorts, but I will include the Homebrew commands as well.

#### MacPorts
1. Install MacPorts from [here](https://www.macports.org/install.php).
2. Install OpenOCD using `sudo port install openocd`.
3. To see what files were installed, you can use `port contents openocd`.

#### Homebrew
1. Install Homebrew from [here](https://brew.sh/).
2. Install OpenOCD using `brew install openocd`.
3. To see what files were installed, you can use `brew list openocd`.

### Flashing using OpenOCD
You may need to install some more packages for OpenOCD to work. I had to install `libusb` and `pkg-config`, but you may need to install more. You can install them using MacPorts or Homebrew.

Now connect the TM4C123GXL to your Mac using a USB cable. Open a terminal and run `ioreg -p IOUSB`. You should see something like this showing that the board is connected:
![ioreg](./assets/ioreg.png)

Now we can flash the board. Open a new terminal and navigate to the `PeridiocTimer1AInts_4C123/Objects` folder. Run `openocd -f /opt/local/share/openocd/scripts/board/ti_ek-tm4c123gxl.cfg`. You should see something like this:
![OpenOCD Start](./assets/OpenOCDStart.png)

This is a gdb server that can be used for debugging.

Open another terminal and navigate to the `PeridiocTimer1AInts_4C123/Objects` folder. Run `telnet localhost 4444`. You should see something like this:
![OpenOCD Telnet](./assets/OpenOCDTelnet.png)

This is a telnet server that we can use to pass OpenOCD commands for flashing.

Run `program Timer1A.bin`. You should see something like this:
![OpenOCD Program](./assets/OpenOCDProgram.png)

Now hit the reset button on the board and you should see the LED cycle through the colors. You've successfully flashed the board!

### Debugging using OpenOCD

#### Prerequisites

Install the ARM GNU Toolchain from [here](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads). I downloaded the `arm-gnu-toolchain-13.2.rel1-darwin-arm64-arm-none-eabi.pkg` for my Mac.

Once you've completed the setup, navigate to the binaries folder using your terminal. For me, the location was `/Applications/ArmGNUToolchain/12.3.rel1/arm-none-eabi/bin`. You should see a bunch of files like this in the explorer:

![GNU Toolchain Binaries](./assets/Binaries.png)

#### Debugging

In your terminal run `./arm-none-eabi-gdb <path to your .elf file>`. You should see something like this output:

![GNU Toolchain GDB](./assets/GDB.png)

Then run `target extended-remote localhost:3333` to connect to the OpenOCD server.

Then run `monitor reset halt` to reset the board and halt the processor.

Then run `load` to load the program onto the board.

From here you can set breakpoints using `b <line number>` run `continue` to run the program, `step` to step through the program, `next` to step over the program, and `print <name>` to print the value of a variable.

For more debugging commands, you can check out [this guide](https://cs.baylor.edu/~donahoo/tools/gdb/tutorial.html) which has a lot of useful commands. to learn gdb.

One disclaimer is that debugging on the TM4C with gdb can be kind of flaky at times. Especially when using timers and interrupts, using gdb to step through can cause the program to behave differently than it would if it was running normally. I've found that using debugging techniques like printing variables/messages over UART or to a physical LCD connected too the TM4C is much more stable than debugging. 

That isn't to say that you shouldn't use gdb! It is a very powerful tool, but just be aware that it can cause some weird behavior.

I haven't yet found a good solution for a GUI to debug the TM4C, like Keil does, but please reach out if you know of one!
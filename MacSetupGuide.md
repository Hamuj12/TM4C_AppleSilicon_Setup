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
2. Now we need to set up some 
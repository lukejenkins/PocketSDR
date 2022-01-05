# **PocketSDR - An Open-Source GNSS SDR, ver. 0.4**

## **Overview**

PocketSDR is an open-source GNSS (Global Navigation Satellite System) receiver
based on SDR (software defined radio) technology. It consists of a RF front-end
device, some utilities for the device and GNSS-SDR APs (application programs)
written in Python. It supports almost all signals for GPS, GLONASS, Galileo,
QZSS, BeiDou and SBAS.

The RF front-end device consists of 2 CH Maxim MAX2771 GNSS RF front-end IC
(LNA, mixer, filter, ADC, frequency synthesizer) and Cypress EZ-USB FX2LP USB
2.0 controller to connect to host PCs. The front-end CH1 is dedicated for GNSS
L1 band (1525 - 1610 MHz) and CH2 is for GNSS L2/L5/L6 band (1160 - 1290 MHz).
The frequency of the reference oscillator (TCXO) is 24.000 MHz and ADC sampling
frequency can be configured up to 24 MHz.

PocketSDR contains some utility programs for the RF front-end device to
configure the device, capture and dump the digitized IF (inter-frequency) data.
These supports Windows, Linux and other environments.

PocketSDR also provides GNSS-SDR APs to show the PSD (power spectrum density)
of captured IF data, search the GNSS signals, track these signals and decode
navigation data in them. The supported GNSS signals are as follows. These APs
are written in Python by very compact way. They are easily modified by users
to add user's unique algorithms. 

GPS: L1C/A, L1CP, L1CD, L2CM, L5I, L5Q, GLONASS: L1C/A, L2C/A, Galileo: E1B,
E1C, E5aI, E5aQ, E5bI, E5bQ, E6B, E6C, QZSS: L1C/A, L1C/B, L1CP, L1CD, L1S,
L2CM, L5I, L5Q, L5SI, L5SQ, L6D, L6E, BeiDou: B1I, B1CP, B1CD, B2I, B2aD, B2aP,
B2bI, B3I, SBAS: L1C/A, L5I, L5Q

<img src="image/pocket_sdr_image.jpg" width=80%>

--------------------------------------------------------------------------------

## **Package Structure**
```
PocketSDR --+-- bin     PocketSDR utility binary programs for Windows
            +-- src     PocketSDR utility source programs
            +-- python  PocketSDR utility Python scripts
            +-- lib     External shared library for Python scripts
            +-- conf    Configuration files for device settings
            +-- util    Windows driver installation utility (ref [3])
            +-- doc     Documents (ref {1], [2])
            +-- FW      Firmware source programs and images
            |   +-- cypress  Cypress libraries for EZ-USB firmware development
            |                (ref [4])
            +-- HW      PocketSDR CAD data and parts list for hardware
            |           (*.brd and *.sch are for Eagle, *.f3d is for Fusion 360)
            +-- image   Image files for documents
            +-- sample  Sample digital IF data captured by PocketSDR
```

--------------------------------------------------------------------------------

## **Installation for Windows**

* Extract PocketSDR.zip to an appropriate directory <install_dir>.

* Attach PocketSDR to PC via USB cable.

* Install USB driver (WinUSB) for PocketSDR.
    * Execute zadig-2.6.exe in <install_dir>\PocketSDR\util.
    * Execute menu Options - List All Devices and select "EZ-USB" (USBID 04B4
      1004). 
    * Select WinUSB (v6.1.xxxx.xxxxx) and Push "Replace Driver" or  "Reinstall
      Driver".
    
    <br>
* Add the PocketSDR binary programs path (<install_dir>\PocketSDR\bin) to 
  the command search path (Path) of Windows environment variables.

* Add the PocketSDR Python scripts path (<install_dir>\PocketSDR\python) to 
  the command search path (Path) of Windows environment variables.

* To rebuild the binary programs, you need MinGW64 and libusb-1.0 library. 
  Refer MSYS2 (https://www.msys2.org/) for details.

--------------------------------------------------------------------------------

## **Installation for Linux**

* Extract PocketSDR.zip to an appropriate directory <install_dir>.
```
    $ unzip PocketSDR.zip
```
* Install libusb-1.0 developtment package. For Ubuntu:
```
    $ sudo apt install libusb-1.0-0-dev
```
* Move to the source program directory, edit makefile and build utilities.
```
    $ cd <install_dir>/src
    $ vi makefile
    ...
    #LIBUSB = -L/mingw64/lib -llibusb-1.0
    LIBUSB = -lusb-1.0
    ...
    $ make
    $ make install
```
* Add the PocketSDR binary programs path (<install_dir>/PocketSDR/bin) to 
  the command search path.

* Usually you need to have root permission to access USB devices. So you add
sudo to execute pocket_conf, pocket_dump like:
```
   $ sudo pocket_conf ../conf/pocket_L1L6_12MHz.conf
   $ sudo pocket_dump -t 10 ch1.bin ch2.bin
```

--------------------------------------------------------------------------------

## **Utility Programs for RF frontend**

PocketSDR contains the following utility programs.

- **pocket_conf**: SDR device configurator
- **pocket_scan**: Scan and list USB Devices
- **pocket_dump**: Capture and dump digital IF data of SDR device

For details, refer comment lines in src/pocket_conf.c, src/pocket_scan.c, 
src/pocket_dump.c.

--------------------------------------------------------------------------------

## **GNSS-SDR APs (Application Programs)**

PocketSDR contains the following application programs for GNSS-SDR.

- **pocket_psd.py**: Plot PSD and histgrams of digital IF data
- **pocket_acq.py**: GNSS signal acquisition in digital IF data
- **pocket_trk.py**: GNSS signal tracking and navigation data decoding in digital IF data

For details, refer comment lines in python/pocket_psd.py, python/pocket_acq.py
and python/pocket_trk.py. You need Python 3, Numpy, Scipy and matplotlib to
execute Python scripts. pocket_trk.py uses external shared libraries of
LIBFEC [5] and RTKLIB [6] in lib/ directory.  These were built for Windows
(64bit) and Linux for x86_64 CPU.

--------------------------------------------------------------------------------

## **Execution Examples of Utility Programs and GNSS-SDR APs**

```
    $ pocket_conf
    ...
    $ pocket_conf conf/pocket_L1L6_12MHz.conf
    Pocket SDR device settings are changed.
    
    $ pocket_dump -t 5 ch1.bin ch2.bin
      TIME(s)    T   CH1(Bytes)   T   CH2(Bytes)   RATE(Ks/s)
          5.0    I     59768832  IQ    119537664      11922.8
    
    $ pocket_psd.py ch1.bin -f 12 -h
    $ pocket_acq.py ch1.bin -f 12 -fi 3 -sig L1CA -prn 1-32,193-199
    PRN   1: SIG= L1CA, COFF=  0.12817 ms, DOP=  3500 Hz, C/N0= 37.4 dB-Hz
    PRN   2: SIG= L1CA, COFF=  0.85242 ms, DOP= -3500 Hz, C/N0= 36.8 dB-Hz
    PRN   3: SIG= L1CA, COFF=  0.39400 ms, DOP= -2000 Hz, C/N0= 37.0 dB-Hz
    PRN   4: SIG= L1CA, COFF=  0.96692 ms, DOP=  2000 Hz, C/N0= 44.7 dB-Hz
    ...
    $ pocket_acq.py ch1.bin -f 12 -fi 3 -sig L1CA -prn 26

    $ pocket_acq.py ch1.bin -f 12 -fi 3 -sig L1CA -prn 26 -3d

    $ pocket_acq.py ch2.bin -f 12 -sig L6D -prn 194 -p
    
    $ pocket_trk.py L1_24M.bin -prn 1-32 -f 24 -fi 6
	  TIME(s)   SIG  PRN  STATE   LOCK(s)  C/N0 (dB-Hz)        COFF(ms)   DOP(Hz)   ADR(cyc)  SYNC #NAV #ERR
	    1.550  L1CA    1   LOCK     1.539  42.5 ||||||||      0.8017699    3218.8     4949.3  ---     0    0
	    0.011  L1CA    2   IDLE     0.000   0.0               0.0000000       0.0        0.0  ---     0    0
	    0.011  L1CA    3   IDLE     0.000   0.0               0.0000000       0.0        0.0  ---     0    0
	    0.011  L1CA    4   IDLE     0.000   0.0               0.0000000       0.0        0.0  ---     0    0
	    0.011  L1CA    5   IDLE     0.000   0.0               0.0000000       0.0        0.0  ---     0    0
	    0.011  L1CA    6   IDLE     0.000   0.0               0.0000000       0.0        0.0  ---     0    0
	    1.550  L1CA    7   LOCK     1.539  45.7 ||||||||||    0.6995059     582.8      901.5  ---     0    0
	    1.550  L1CA    8   LOCK     1.539  47.6 |||||||||||   0.0733506    -777.2    -1196.8  ---     0    0
	    0.011  L1CA    9   IDLE     0.000   0.0               0.0000000       0.0        0.0  ---     0    0
	    1.550  L1CA   10   LOCK     1.539  38.6 |||||         0.8658548      -6.2       -9.4  ---     0    0
	...
    $ pocket_trk.py L1_24M.bin -prn 194 -sig L1CA -f 24 -fi 6 -log trk.log  -p
    ...
    $ pocket_trk.py L5_24M.bin -prn 13 -sig E5AI -f 24 -log trk.log -p -ts 0.2
    ...
``` 

<img src="image/image001.jpg" width=49%>
<img src="image/image002.jpg" width=49%>
<img src="image/image003.jpg" width=49%>
<img src="image/image004.jpg" width=49%>
<img src="image/image005.jpg" width=49%>
<img src="image/image006.jpg" width=49%>
<img src="image/image007.jpg" width=49%>

--------------------------------------------------------------------------------

## **Rebuild F/W and Write F/W Image to PocketSDR**

* Install Cypress EZ-USB FX2LP Development Kit (ref [4]) to a Windows PC. As
default, it is installed to C:\Cypress and C:\Keil.

* Execute Keil uVision2 (C:\Keil\UV2\uv2.exe).

* Execute Menu Project - Open Project, select <install_dir>\PocketSDR\FW\pocket_fw.Uv2>
and open the project.

* Execute Menu Project - Rebuild all target files and you can get a F/W image
as <install_dir>\PocketSDR\FW\pocket_fw.iic.

* Attach PocketSDR via USB cable to the PC.

* Open Windows Device Manager, select "EZ-USB" as "Universal Serial Bus Device",
select right-button menu Update Driver, select "Browse your computer for driver software"
and input the CyUSB driver path (C:\Cypress\USB\CY3684_EZ-USB_FX2LP_DVK\1.1\Drivers).
After the driver installation , you find "Cypress FX2LP Sample Device" as 
"Universal Serial Bus Controller" in Windows Device Manager.

* Execute USB Control Center (C:\Cypress\USB\CY3684_EZ-USB_FX2LP_DVK\1.1\Windows Applications\
c_sharp\controlcenter\bin\Release\CyControl.exe).

* Select Cypress FX2LP Sample Device, execute menu Program - FX2 - 64KB EEPROM,
select the F/W image <install_dir>\PocketSDR\FW\pocket_fw.iic and open it.

* If you see "Programming succeeded." in status bar, the F/W is properly written
to PocketSDR.

* To use utility programs for PocketSDR, you need to reinstall WinUSB driver for
PocketSDR. Refer "Installation for Windows" above.

--------------------------------------------------------------------------------

## **References**

[1] Maxim integrated, MAX2771 Multiband Universal GNSS Receiver, July 2018

[2] Cypress, EZ-USB FX2LP USB Microcontroller High-Speed USB Peripheral 
  Controller, Rev. AB, December 6, 2018

[3] Zadig USB driver installation made easy (https://zadig.akeo.ie/)

[4] Cypress, CY3684 EZ-USB FX2LP Development Kit
    (https://www.cypress.com/documentation/development-kitsboards/cy3684-ez-usb-fx2lp-development-kit)

[5] https://github.com/quiet/libfec

[6] https://github.com/tomojitakasu/RTKLIB

--------------------------------------------------------------------------------

## **History**

- 2021-10-20  0.1  1st draft version
- 2021-10-25  0.2  Add Rebuild F/W and Write F/W Image to PocketSDR
- 2021-12-01  0.3  Add and modify Python scripts
- 2021-12-25  0.4  Add and modify Python scripts


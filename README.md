# dcesdg
2020 PSU Capstone for DC Energy Storage Design Guide

Requires Excel and PLX-DAQ. The latter can be found here: https://www.parallax.com/downloads/plx-daq. Download and 
install to obtain the requisite Excel template. Macros must be enabled in Excel.

This progrom was written for use with an Arduino Uno R3 and to be used in conjunction with the cell cycler apparatus 
created for Viking Motorsports. Please refer to the program flow diagram found in the DCESDG. The program uses nested 
finite state machines to automatical progress through a set number of charge and discharge cycles. I will detail the 
input parameters below. Data will sent to excel using a usb connection.

line 12: const int n_or_f = 1;                     // number of cycles, enter 0 for test until fail
    Enter the desired number of cycles to be tested or enter 0 for run to fail.
    
line 13: const float voltUpperLim =      4.2;      // charge to this vaoltage
    Enter the upper voltage limit to charge to. Take care to not set above what the charger will charge to.

line 14: const float voltLowerLim =      3.2;      // discharge to this voltage
    Enter the lower voltage limit that the cell will be discharged until. Use discretion and consider the lower limit of 
    the cell being tested.

line 15: const int waitAfterCharge =     300;      // seconds of wait time after charge cycle
    Enter in seconds a desired wait period between charge complete and discharge begin.

line 16: const int waitAfterDischarge =  300;      // seconds of wait time after discharge cycle
    Enter in seconds a desired wait period between discharge complete and charge begin.
    
line 17: const int thermalUpperLim =     80;       // test fail limit in deg C
    Enter the maximum allow thermaal setpoint in degrees celsius. Exceeding this limit will hard stall the program. 
    A hardware reset will be required to resume testing.

line 18: const int thermalLowerLim =     25;       // set additional wait criterea in deg C, set to 9999 for no thermal wait
    Enter a desired thermal limit. Following a successful charge or discharge, the apparatus will wait for the cell to cool 
    to the setpoint before continuing the cycle. 

line 19: const float captureRate =       0.2;      // data collection rate in Hz
    Enter the desired data capture rate in Hz.

line 20: const int failMode =            0;        // 0 = hard stop on fail, 1 = cycle reset on fail
    Fail mode override. Enter 1 to override the hard fail mode. Fail will instead restart the cycle.

line 21: const float resistorVal =       1;        // enter resistor value in Ohms for energy calculation
    Currently unused.

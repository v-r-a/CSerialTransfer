A few learnings regarding configuring the FTDI USB driver to achieve real-time communication performance:

For real-time communication performance through USB 2.0 ports, the following settings have been tested. The following settings are available only for FT232R (e.g., USB to RS485 converter or USB to UART converter) based USB-to-serial bridges. Other USB-to-serial ICs, such as CH340, do not have extensive settings in the driver software.

READING INCOMING DATA:
The best real-time reading performance can be achieved if the incoming packet size is below 64 bytes (less than 62 bytes to be precise). The event character setting helps in achieving the best reading performance. Thankfully SerialTransfer Library has a predefined packet start and stop byte. We use it to set the event character as follows:

Set the USB0 port eventchar to 129+256=385 (10000001 marks the end of the packet) to flush immediately on packet receipt. Read [here](https://superuser.com/questions/411616/how-to-enable-and-set-event-characters-for-ftdi-drivers).

Set it by running the following commands in the Linux terminal. Replace /ttyUSB0 by name of the USB port in use. (see connected devices by: ls /dev/ttyUSB* command):

`sudo -i`

`echo 385 > /sys/bus/usb-serial/devices/ttyUSB0/event_char`

Low latency may not be always better due to the way FTDI USB driver software is written. Either leave it to the default of 16ms or slightly lower, down to 4 ms, say.
Setting the USB0 port latency_timer to 10ms (change the port name appropriately):
`echo 10 | sudo tee /sys/bus/usb-serial/devices/ttyUSB0/latency_timer`

WRITING DATA:
A USB frame occurs every 1ms. We may not be able to write any faster than that, i.e., we can send a data packet every 1ms. Set your packet size and baud rate so that packet transmission is over well within 1ms if you want to send commands at 1kHz. Further, the OS scheduler can delay the task, and the packet may not be sent in the next USB frame. You may try increasing the process priority etc, at your own risk.

BAUD RATE:
I have tested the communication at 1M baud rate. It works fine with Arduino Uno/ Nano/ Mega.


Find all the details of FTDI USB driver in [this](https://github.com/v-r-a/CSerialTransfer/blob/master/AN232B-04_DataLatencyFlow.pdf) pdf
# CSerialTransfer
Linux Serial Transfer Library adapted from PowerBrokers SerialTransfer as mirror library.

[https://github.com/PowerBroker2/SerialTransfer](https://github.com/PowerBroker2/SerialTransfer)

The Syntax is nearly identical to the arduino version, with a few exceptions:

  The initial constructor is
  
    typedef void (*functionPtr)()
    SerialTransfer::SerialTransfer(const char *device, int baudRate, functionPtr _callbacks = NULL, uint8_t _callbacksLen = 0) 
    
    With device usually being some version of /dev/ttyeACM0 on linux and accepted
    baudrates being : 9600, 19200, 38400, 115200 and 1000000.
    
example:
  SerialTransfer::SerialTransfer myTransfer("/dev/ttyACM0", 19200, callbacks, callbacksLen);
  
  Also consideration needed when transfering objects as packing differs from 8bit to 32bit, 
  
  struct myobj{
    uint8_t num;
    int num;}
   The above sent from an arduino is 5 bytes, while sent from a 32 bit machine is 8 bytes.
   

  

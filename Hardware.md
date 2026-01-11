## Board description.
Structurally, the host communication scheme on the K1 Creality board and the microcontrollers can be described like this:

![](/images/pcb_overview.jpg "MCU communication")

From the microprocessor to the peripherals of interest there are three ports:
- ttyS7 (the microcontroller on the main board is connected to it, USART1 pins PA2 and PA3)
- ttyS1 (an RS232 transceiver SP3222 is connected to it and then to the print head board)
- ttyS9 (an RS232 transceiver SP3222 is connected to it and then to the load-cell/force-sensor board)
- secondary 5V power enable signal

On the picture, green and red mark the logic voltage levels — 3.3V and 12V respectively. The USB-UART converter and the single-board computer UART outputs are designed for 3.3V so we cannot connect the cable from the print head directly; instead we have to solder to the SP3222E. Our task is to disable all three (or two, if you don't plan to experiment with load cells in the future) ports, turn off the Wi‑Fi module power (so it doesn't interfere or saturate the network), and connect the freed ports to a USB-UART converter.
In addition, it is highly desirable for stability to disable the EN signal from the processor.

## 1. Connecting the main microcontroller

For convenience of disconnecting the microcontroller from the host, the manufacturer provided zero-ohm resistors on the board after series 20Ω resistors connected to the pins. Thus, by desoldering two zero-ohm resistors and soldering to the pads we can connect to the ports through the current-limiting resistors.

![](/images/mcu_resistors.jpg "MCU resistors")

Red marks the resistors that need to be removed. Green marks the connection points for the USB-UART converter.

![](/images/mcu_connection.jpg "MCU connection")

## 2. Connecting the print head and the load-cell board

As with ttyS7, the manufacturer provided zero-ohm resistors for the ttyS1 and ttyS9 ports.

![](/images/sp3222_resistors.jpg "SP3222E resistors")

You need to desolder the resistors marked in red. Then solder wires from the converter to the SP3222E IC. We will ...

![](/images/SP3222E_datasheet.jpg "SP3222E datasheet")

T1IN and R1OUT — print head port; T2IN and R2OUT — load-cell board port.
We connect TX of UART2 to T1IN, RX of UART2 to R1IN and TX of UART3 to T1IN, RX of UART3 to R1IN

![](/images/SP3222E_connection.jpg "SP3222E connection")

## 3. Disabling the EN signal

To be able to power-cycle the microcontrollers, the board provides a transistor Q5 that connects the 5V1 circuit and ...
To retain the ability to connect an SWD programmer to the microcontroller you need to change the schem...

![](/images/EN_mod.jpg "ENABLE mod")

If connecting a programmer to the microcontroller is not required, a single jumper between ...

![](/images/Q6_short.jpg "Q6_short")

## 4. Disabling the WiFi module

To disable the module, desolder the choke in the power supply line.

![](/images/WiFi_module.jpg "WiFi")

## 5. Bluepill pinout

The pinout is given in the [source](https://github.com/r2axz/bluepill-serial-monster); here is the part we need:

| Signal |   Direction   |     UART1     |     UART2     |     UART3     |
|:-------|:-------------:|:--------------|:--------------|:--------------|
|   RX   |      IN       |      PA10     |      PA3      |      PB11     |
|   TX   |      OUT      |      PA9      |      PA2      |      PB10     |

Connect all ports. I connected UART1 to the main microcontroller, UART2 to the load-cell board, and UART3 to the print head board.

I connected GND to the conveniently located AMS1117 ground.

![](/images/Bluepill_GND.jpg "GND connection")

## 6. Powering the host

I powered the host via a DC-DC converter. I soldered a cable with a USB Type-C connector to the converter for connecting ...

![](/images/DC-DC.jpg "DC-DC")

## 7. Reusing the stock WiFi antenna

I reused the stock antenna by simply cutting off the cable and connecting it to the single-board computer connector. In the end ...

![](/images/WiFi_antenna.jpg "WiFi antenna")

That completes the hardware part; you can proceed to the [software](Software.md)

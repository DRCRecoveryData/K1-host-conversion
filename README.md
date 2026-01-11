# K1-host-conversion

## Connecting a single-board computer (Orange Pi or any other) to the K1.

Pros - a clean Klipper setup, the ability to use a host with extended functionality (more memory, presence of a CAN bus, [...])

Cons - the stock display will not work, the load cells (strain gauges) will not work. You need to plan for them in advance [...]

Required parts:

1. Single-board computer. I used an Orange Pi Zero 3 1GB RAM

2. USB-TTL x2 converter (optional).  
I made this conversion expecting to connect three microcontrollers to the host — one for the mainboard [...]  
![](/images/bluepill.jpeg "Bluepill")  
You can also buy a ready-made converter on an FTDI chip, for example this one: https://aliexpress.ru/item/1005006850550816.html [...]

3. DC-DC converter to power the single-board computer (if necessary). I use an Orange Pi with the required supply voltage [...]  
![](/images/DC_DC_example.jpg "DC DC")

## [Hardware changes, connecting the host.](/Hardware.md)

## [Software changes, firmware.](/Software.md)

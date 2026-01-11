# Software changes, firmware.

## Prerequisites:

Install an operating system on the host single-board computer and, using [KIAUH](https://github.com/dw-0/kiauh) (or any other method convenient for you), install the required services. Example:

```
sudo apt-get update && sudo apt-get install git -y
```  

```
cd ~ && git clone https://github.com/dw-0/kiauh.git
```   

```
./kiauh/kiauh.sh
```

Then in the install menu choose the services you need.

At this stage you should NOT write the MCU connection config, otherwise the required ports will be occupied and you will have to enter [...]


## 0. Installing the bluepill-serial-monster firmware



## 1. Bootloader flashing

To replace the bootloader we need the script `mcu_utils.py` from the [cryoz](https://github.com/cryoz/k1_mcu_flasher) repository and, instead of the original [...] (the original bootloader is inconvenient because the handshake must be done within 15 seconds, etc.)

Clone the repositories we need:

```
cd ~ && git clone https://github.com/cryoz/k1_mcu_flasher
```  

Find the ports to which our microcontrollers are connected:

```
dmesg | grep tty
```

You will get output similar to:

```
[    0.000600] printk: console [tty1] enabled
[    1.716300] printk: console [ttyS0] disabled
[    1.716377] 5000000.serial: ttyS0 at MMIO 0x5000000 (irq = 285, base_baud = 1500000) is a 16550A
[    1.716712] printk: console [ttyS0] enabled
[   12.810369] systemd[1]: Created slice Slice /system/serial-getty.
[   13.964922] systemd[1]: Found device /dev/ttyS0.
[   14.283800] cdc_acm 4-1:1.0: ttyACM0: USB ACM device
[   14.285835] cdc_acm 4-1:1.2: ttyACM1: USB ACM device
[   14.287994] cdc_acm 4-1:1.4: ttyACM2: USB ACM device
[   14.627703] mtty_probe init device addr: 0x00000000eab98f5e
[   20.149292] mtty_open device success!
```

In my case UART1-3 of the converter were detected by the system as `ttyACM0`–`ttyACM2`.  

To replace the inconvenient original bootloader (it is inconvenient because the handshake must be completed within 15 seconds, [...] ), download the firmware file to the single-board computer: [/binaries/deployer.bin](/binaries/deployer.bin)

To flash katapult we use the `mcu_utils.py` script.

Go to the folder with the script:

```
cd ~/k1_mcu_flasher
```

Reboot the motherboard (I simply removed the fuse from the motherboard for a few seconds and put it back [...])

```
python3 mcu_util.py -c -i /dev/ttyACM2 -g -v
```

Where `/dev/ttyACM2` is the port to which the microcontroller for flashing is connected. In my case `ttyACM2` is the port of the microcontroller for flashing the bootloader.

If everything is successful, you will get console output:

```
send handshake
rcv data b'75'
handshake confirmed
send version request
rcv data b'6e6f7a305f3132305f4733302d6e6f7a305f3030335f303030e8'
version received! b'noz0_120_G30-noz0_003_000'
FW Version: noz0_120_G30-noz0_003_000
```

After that, run the script to flash `deployer.bin`:

```
python3 mcu_util.py -c -i /dev/ttyACM2 -v -u -f ~/deployer.bin
```

If everything is successful you will get:

```
send handshake
send sectorsize request
rcv data b'02fd'
sector size received! 2
send update request
[flash_update] [1] rcv data b'758a'
[flash_update] [1] update request confirmed!
[flash_update] [2] rcv data b'758a'
[flash_update] [2] FW size confirmed!
[flash_update] rcv data b'758a'
[flash_update] chunk flashed
[flash_update] rcv data b'758a'
[flash_update] chunk flashed
[flash_update] rcv data b'20df'
[flash_update] [3] flash completed
Firmware updated successfully
send app_start request
rcv data b'758a'
app started!
App started
```

Check that katapult was installed:

```
python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -s
```

If successful the output will be:

```
Connecting to Serial Device /dev/ttyACM2, baud 230400
Attempting to connect to bootloader
Katapult Connected
Software Version: v0.0.1-91-fdl3
Protocol Version: 1.1.0
Block Size: 64 bytes
Application Start: 0x8002000
MCU type: stm32f103xe
Status Request Complete
```

Repeat the flashing process for the main microcontroller — again reboot the motherboard and run:

```
python3 mcu_util.py -c -i /dev/ttyACM0 -g -v
```

Then:

```
python3 mcu_util.py -c -i /dev/ttyACM0 -v -u -f ~/deployer.bin
```

Now the bootloaders are flashed and you can proceed to building and flashing Klipper binaries. 

## 3. Flashing Klipper

### 3.1 Toolhead board

Go to the klipper folder and run `make menuconfig` (or in KIAUH choose "Advanced" - "Build")

```
cd ~/klipper
```  

```
make menuconfig
```

Select options as shown in the screenshot:

![](/images/nozzle_mcu_menuconfig.jpg "toolhead menuconfig")

Exit and save the settings, then build:

```
make
```

Now flash the resulting binary:

```
python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -s
```

Then:

```
python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -f ~/klipper/out/klipper.bin
```  

The output should be:

```
Connecting to Serial Device /dev/ttyACM2, baud 230400
Detected Klipper binary version v0.12.0-458-gd886c176, MCU: stm32f103xe
Attempting to connect to bootloader
Katapult Connected
Software Version: v0.0.1-91-fdl3
Protocol Version: 1.1.0
Block Size: 64 bytes
Application Start: 0x8002000
MCU type: stm32f103xe
Flashing '/home/orangepi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 34 pages
Verifying (block count = 531)...

[##################################################]

Verification Complete: SHA = 9A0A75F1987338EE8D4E1A5736E793B1E63E8914
Programming Complete
```

### 3.2 Main microcontroller

Repeat the same steps as in the previous item, but choose the appropriate port (for me it is `ttyACM0`) and configure the build for the main MCU [...]

![](/images/main_mcu_menuconfig.jpg "mcu menuconfig")


## 4. Basic minimal config

For verification you can use my minimal config. It assumes you have CRTouch installed, the [Kli[...] module, and so on.

[Config](/config/)

``` 
``` 

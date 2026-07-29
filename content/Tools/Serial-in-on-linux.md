Using GTKterm app

Find out what port your serial cable is plugged into:
```bash
dthomas@lgfedora:~$ dmesg | grep tty
[    0.119494] printk: console [tty0] enabled
[    1.591268] serial8250: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
[    7.707751] systemd[1]: Created slice system-getty.slice - Slice /system/getty.
[    7.708786] systemd[1]: Reached target getty.target - Login Prompts.
[  424.829380] usb 3-6: FTDI USB Serial Device converter now attached to ttyUSB0
[  498.255356] ftdi_sio ttyUSB0: FTDI USB Serial Device converter now disconnected from ttyUSB0
[  499.952411] usb 3-4: FTDI USB Serial Device converter now attached to ttyUSB0
[ 1020.298236] ftdi_sio ttyUSB0: FTDI USB Serial Device converter now disconnected from ttyUSB0
[ 1022.774544] usb 3-6: FTDI USB Serial Device converter now attached to ttyUSB0
[ 1115.380773] ftdi_sio ttyUSB0: FTDI USB Serial Device converter now disconnected from ttyUSB0
[ 1117.430371] usb 3-4: FTDI USB Serial Device converter now attached to ttyUSB0

```

See last line that says "now attached"
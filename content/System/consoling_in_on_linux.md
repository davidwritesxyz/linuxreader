---
title: Consoling in on Linux
date: 2023-11-06T06:20:36-07:00
draft: true
---
Consoling in on Linux
Plug console cable in

find out what your serial line name is:

` sudo dmesg | grep -i FTDI`

Open putty > change to serial > change the tty line name

Ex: /dev/ttyUSB0

Make sure your serial settings are correct

Press open > when terminal appears press enter

Install screen
`sudo dnf -y install screen`
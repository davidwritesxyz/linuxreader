---
title: 'Consoling in to MX80 from linux'
description: 'How to console into an MX80 router from linux'
---

Plug console cable in

find out what your serial line name is:

```bash
$ dmesg | grep -i FTDI
```

Open putty > change to serial > change the tty line name

Make sure your serial settings are correct

[https://www.juniper.net/documentation/us/en/hardware/mx5-mx10-mx40-mx80/topics/task/management-devices-mx5-mx10-mx40-mx80-connecting.html](https://www.juniper.net/documentation/us/en/hardware/mx5-mx10-mx40-mx80/topics/task/management-devices-mx5-mx10-mx40-mx80-connecting.html)

Press open > when terminal appears press enter

Juniper Password recovery

ttps://www.juniper.net/documentation/en_US/junos/topics/task/configuration/authentication-root-password-recovering-mx80.html

[https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/rescue-and-recovery-config-file.html#load-commit-configuration](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/rescue-and-recovery-config-file.html#load-commit-configuration)

accidentally deleted the wrong line in Juniper.conf file ? failing over to juniper.conf

[https://www.juniper.net/documentation/en_US/junos/topics/concept/junos-configuration-files.html](https://www.juniper.net/documentation/en_US/junos/topics/concept/junos-configuration-files.html)

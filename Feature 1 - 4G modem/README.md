
# Feature 1: 4G Modem

To be clear, I am not a Linux user or Linux admin. This was a learning experience with a lot of trial and error. Thanks to ChatGPT and a lot of Googling, I got it working.

![Connections](4Gdiagram.png)

## Installation

What you need:

1. A SIM card without a PIN code
2. A modem (I went with this one: PIS-2004 - Quectel EG25-G [Link](https://www.elfadistrelec.no/en/quectel-eg25-mini-pcie-lte-module-with-antennas-nebra-pis-2004/p/30221005))
3. Two antennas

## Setup

First, verify that PiKVM finds the modem we installed.

Make sure you are in write mode:

```sh
[root@pikvm ~]# rw
```

```sh
[root@pikvm ~]# lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 003: ID 2c7c:0125 Quectel Wireless Solutions Co., Ltd. EC25 LTE modem
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub

[root@pikvm ~]# lspci -nn
00:00.0 PCI bridge [0604]: Broadcom Inc. and subsidiaries BCM2711 PCIe Bridge [14e4:2711] (rev 20)
01:00.0 USB controller [0c03]: VIA Technologies, Inc. VL805/806 xHCI USB 3.0 Controller [1106:3483] (rev 01)
```

Arch Linux needs to use ModemManager, but to use ModemManager you need NetworkManager.

Install both NetworkManager and ModemManager on the PiKVM:

```sh
sudo pacman -S networkmanager
sudo pacman -S modemmanager
```

After installation, verify that all dependencies are installed:

```sh
sudo systemctl list-dependencies ModemManager
sudo systemctl list-dependencies NetworkManager
```

Then start the services:

```sh
sudo systemctl start NetworkManager
sudo systemctl enable NetworkManager
sudo systemctl status NetworkManager

sudo systemctl start ModemManager
sudo systemctl enable ModemManager
sudo systemctl status ModemManager
```

Verify that the modem is operating in modem mode and that the modem is up and running:

```sh
[root@pikvm ~]# mmcli -L
    /org/freedesktop/ModemManager1/Modem/0 [QUALCOMM INCORPORATED] QU
```

Create a script to check the connection and log the output:

```sh
echo "#!/bin/bash" > /usr/local/bin/check_connection.sh
echo "date" >> /usr/local/bin/check_connection.sh
echo "nmcli d" >> /usr/local/bin/check_connection.sh
echo "mmcli -m 0" >> /usr/local/bin/check_connection.sh
chmod +x /usr/local/bin/check_connection.sh
```

Install cron and start it:

```sh
sudo pacman -S cronie
sudo systemctl enable cronie
sudo systemctl start cronie
```

To make a job run, use the following command:

```sh
sudo systemctl status cronie
echo "*/5 * * * * /usr/local/bin/check_connection.sh >> /var/log/check_connection.log 2>&1" | sudo crontab -
```

Add the following line:

```sh
*/5 * * * * /usr/local/bin/check_connection.sh >> /var/log/check_connection.log 2>&1
```

Verify by checking the status and the log file:

```sh
[root@pikvm ~]# sudo systemctl status cronie
* cronie.service - Command Scheduler
     Loaded: loaded (/usr/lib/systemd/system/cronie.service; enabled; preset: disabled)
     Active: active (running) since Sat 2024-06-08 12:55:13 UTC; 6min ago
   Main PID: 9634 (crond)
      Tasks: 2 (limit: 4018)
        CPU: 201ms
     CGroup: /system.slice/cronie.service
             |- 9634 /usr/sbin/crond -n
             `-10383 /usr/sbin/anacron -s

Jun 08 13:00:00 pikvm CROND[10231]: (root) CMD (/usr/local/bin/check_connection.sh >> /var/log/check_connection.log 2>&1)
Jun 08 13:00:04 pikvm CROND[10225]: (root) CMDEND (/usr/local/bin/check_connection.sh >> /var/log/check_connection.log 2>&1)
Jun 08 13:00:04 pikvm CROND[10225]: pam_unix(crond:session): session closed for user root
Jun 08 13:01:00 pikvm CROND[10377]: (root) CMD (run-parts /etc/cron.hourly)
Jun 08 13:01:00 pikvm anacron[10383]: Anacron started on 2024-06-08
Jun 08 13:01:00 pikvm CROND[10376]: (root) CMDEND (run-parts /etc/cron.hourly)
Jun 08 13:01:00 pikvm anacron[10383]: Will run job `cron.daily' in 32 min.
Jun 08 13:01:00 pikvm anacron[10383]: Will run job `cron.weekly' in 52 min.
Jun 08 13:01:00 pikvm anacron[10383]: Will run job `cron.monthly' in 72 min.
Jun 08 13:01:00 pikvm anacron[10383]: Jobs will be executed sequentially

[root@pikvm ~]# cat /var/log/check_connection.log
Interface: eth0 is up
Interface: wwan0 is up
eth0 can reach 1.1.1.1
Checking that default route is using eth0 as gateway
Default route is set correctly

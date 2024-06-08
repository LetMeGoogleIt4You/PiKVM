# Feature 1:  4G modem

To be clear I am not a linux user or linux admin, this was leaning full event with a lot of trail and error. 
Thanks two chatgpt and alot of googling I got it working. 



## Installstion

What you need:

1. one simcard without a pin code
2. one modem
3. two antanns 

## Setup

First verify that PiKVM find the modem we installed. 

Make sure you are in write mode 

```
[root@pikvm ~]# rw
```

```
[root@pikvm ~]# lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 003: ID 2c7c:0125 Quectel Wireless Solutions Co., Ltd. EC25 LTE modem
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub


[root@pikvm ~]# lspci -nn
00:00.0 PCI bridge [0604]: Broadcom Inc. and subsidiaries BCM2711 PCIe Bridge [14e4:2711] (rev 20)
01:00.0 USB controller [0c03]: VIA Technologies, Inc. VL805/806 xHCI USB 3.0 Controller [1106:3483] (rev 01)
```


Archlinux need to use modemmanager but for using modemmanager you need networkmanager. 

Intsall both networkmanager and modemmanager on the PiKVM

```
sudo pacman -S networkmanager  
sudo pacman -S modemmanager
```


After installation verify that all all dependencies is installed

```
sudo systemctl list-dependencies ModemManager
sudo systemctl list-dependencies NetworkManager
```

Then start the servcies:


```
sudo systemctl start NetworkManager
sudo systemctl enable NetworkManager
sudo systemctl status NetworkManager


sudo systemctl start ModemManager
sudo systemctl enable ModemManager
sudo systemctl status ModemManager
```


Verify that the modem is operating in modem mode and the modem is up and running

```
[root@pikvm ~]# mmcli -L
    /org/freedesktop/ModemManager1/Modem/0 [QUALCOMM INCORPORATED] QUECTEL Mobile Broadband Module


[root@pikvm ~]# mmcli -m 0
  -----------------------------------
  General  |                    path: /org/freedesktop/ModemManager1/Modem/0
           |               device id: <snip>
  -----------------------------------
  Hardware |            manufacturer: QUALCOMM INCORPORATED
           |                   model: QUECTEL Mobile Broadband Module
           |       firmware revision: EG25GGBR07A08M2G
           |          carrier config: ROW_Generic_3GPP
           | carrier config revision: 05010820
           |            h/w revision: 10000
           |               supported: gsm-umts, lte
           |                 current: gsm-umts, lte
           |            equipment id: 860195053124688
  -----------------------------------
  System   |                  device: /sys/devices/platform/scb/fd500000.pcie/pci0000:00/0000:00:00.0/0000:01:00.0/usb1/1-1/1-1.3
           |                 physdev: /sys/devices/platform/scb/fd500000.pcie/pci0000:00/0000:00:00.0/0000:01:00.0/usb1/1-1/1-1.3
           |                 drivers: option, qmi_wwan
           |                  plugin: quectel
           |            primary port: cdc-wdm0
           |                   ports: cdc-wdm0 (qmi), ttyUSB0 (ignored), ttyUSB1 (gps), 
           |                          ttyUSB2 (at), ttyUSB3 (at), wwan0 (net)
  -----------------------------------
  Status   |                    lock: sim-pin2
           |          unlock retries: sim-pin (3), sim-puk (10), sim-pin2 (3), sim-puk2 (10)
           |                   state: registered
           |             power state: on
           |             access tech: lte
           |          signal quality: 89% (recent)
  -----------------------------------
  Modes    |               supported: allowed: 2g; preferred: none
           |                          allowed: 3g; preferred: none
           |                          allowed: 4g; preferred: none
           |                          allowed: 2g, 3g; preferred: 3g
           |                          allowed: 2g, 3g; preferred: 2g
           |                          allowed: 2g, 4g; preferred: 4g
           |                          allowed: 2g, 4g; preferred: 2g
           |                          allowed: 3g, 4g; preferred: 4g
           |                          allowed: 3g, 4g; preferred: 3g
           |                          allowed: 2g, 3g, 4g; preferred: 4g
           |                          allowed: 2g, 3g, 4g; preferred: 3g
           |                          allowed: 2g, 3g, 4g; preferred: 2g
           |                 current: allowed: 2g, 3g, 4g; preferred: 4g
  -----------------------------------
  Bands    |               supported: egsm, dcs, pcs, g850, utran-1, utran-4, utran-6, utran-5, 
           |                          utran-8, utran-2, eutran-1, eutran-2, eutran-3, eutran-4, eutran-5, 
           |                          eutran-7, eutran-8, eutran-12, eutran-13, eutran-18, eutran-19, 
           |                          eutran-20, eutran-25, eutran-26, eutran-28, eutran-38, eutran-39, 
           |                          eutran-40, eutran-41, utran-19
           |                 current: egsm, dcs, pcs, g850, utran-1, utran-4, utran-6, utran-5, 
           |                          utran-8, utran-2, eutran-1, eutran-2, eutran-3, eutran-4, eutran-5, 
           |                          eutran-7, eutran-8, eutran-12, eutran-13, eutran-18, eutran-19, 
           |                          eutran-20, eutran-25, eutran-26, eutran-28, eutran-38, eutran-39, 
           |                          eutran-40, eutran-41, utran-19
  -----------------------------------
  IP       |               supported: ipv4, ipv6, ipv4v6
  -----------------------------------
  3GPP     |                    imei: <snip>
           |           enabled locks: fixed-dialing
           |             operator id: <snip>
           |           operator name: <snip>
           |            registration: <snip>
           |    packet service state: attached
  -----------------------------------
  3GPP EPS |    ue mode of operation: csps-2
           |     initial bearer path: /org/freedesktop/ModemManager1/Bearer/0
           |  initial bearer ip type: ipv4v6
  -----------------------------------
  SIM      |        primary sim path: /org/freedesktop/ModemManager1/SIM/0

```

Next step is to add the networking info to the modem. 

Since i using telenor my APN is I will be using smart.telenor
For name i will be usning 4GConnection
My ifname (aka primary port) is cdc-wdm0


```
[root@pikvm ~]# sudo nmcli connection add type gsm ifname cdc-wdm0 con-name 4GConnection apn telenor.smart
[root@pikvm ~]# sudo nmcli connection up 4GConnection
```

Verify with the followings commands


```
[root@pikvm ~]# nmcli connection show
NAME                UUID                                  TYPE      DEVICE   
4GConnection        d3a4ac43-2f9e-4db2-9a60-975ba2bfaa10  gsm       cdc-wdm0 
<snip>


[root@pikvm ~]# ip ad
<snip>
4: wwan0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/none 
    inet <snip>  brd <snip> scope global noprefixroute wwan0
       valid_lft forever preferred_lft forever
    inet6 <snip> scope global noprefixroute 
       valid_lft forever preferred_lft forever
```

Here is documentation for archlinux and another good links if you want to read more about 4G modems

https://wiki.archlinux.org/title/Mobile_broadband_modem

https://www.jeffgeerling.com/blog/2022/using-4g-lte-wireless-modems-on-raspberry-pi


## Feilover script

Now that we have a our ethernet and 4G network up and running I made a failover script.

the script will check both interfaces, ping 1.1.1.1 if interface is up and priorise ethernet connection using lower metric. 
I very new to bash so most of the code come from chatgpt. there my be a better way to solve this.  
I set this script run once every 5th minuts using cronie. 

Make the script


```
sudo nano /usr/local/bin/check_connection.sh

```


Add the following code to the script

```
#!/bin/bash

# IP to ping
PING_IP=1.1.1.1

# Interface names
ETH_IF="eth0"
WWAN_IF="wwan0"

# Get the interface states using `ip link`
ETH_STATE=$(ip link show $ETH_IF | grep -oP '(?<=state )\w+')
WWAN_STATE=$(ip link show $WWAN_IF | grep -oP '(?<=state )\w+')

# Get the current gateways and metrics
ETH_ROUTE=$(ip route | grep -m 1 "default" | grep $ETH_IF)
WWAN_ROUTE=$(ip route | grep -m 1 "default" | grep $WWAN_IF)
ETH_GW=$(echo $ETH_ROUTE | awk '{print $3}')
WWAN_GW=$(echo $WWAN_ROUTE | awk '{print $3}')
ETH_METRIC=$(echo $ETH_ROUTE | awk '{print $NF}')
WWAN_METRIC=$(echo $WWAN_ROUTE | awk '{print $NF}')

# Set default metrics if they are empty or invalid
ETH_METRIC=${ETH_METRIC:-10}
WWAN_METRIC=${WWAN_METRIC:-700}

# Function to ping and return status
ping_check() {
  local iface=$1
  ping -I $iface -c 3 $PING_IP > /dev/null 2>&1
  return $?
}

#If both interface is up and working
if [ "$ETH_STATE" == "UP" ] && ([ "$WWAN_STATE" == "UP" ] || [ "$WWAN_STATE" == "UNKNOWN" ]); then
  echo "Interface: eth0 is up"
  echo "Interface: wwan0 is up"
  
  # Ping from eth0
  ping_check $ETH_IF
  ETH_PING=$?
  
  # Ping from wwan0
  ping_check $WWAN_IF
  WWAN_PING=$?
  
  # If eth0 can reach the internet
  if [ $ETH_PING -eq 0 ]; then
    echo "eth0 can reach 1.1.1.1"
    echo "Checking that default route is using eth0 as gateway"
    if [ "$ETH_METRIC" -lt "$WWAN_METRIC" ]; then
      echo "Default route is set correctly"
    else
      echo "Updating default route"
      ip route del default
      ip route add default via $ETH_GW dev $ETH_IF metric 10
      ip route add default via $WWAN_GW dev $WWAN_IF metric 700
    fi
  # If eth0 can't reach the internet but wwan0 can
  elif [ $WWAN_PING -eq 0 ]; then
    echo "eth0 can't reach 1.1.1.1 but wwan0 can"
    echo "Checking that default route is using wwan0 as gateway"
    if [ "$WWAN_METRIC" -lt "$ETH_METRIC" ]; then
      echo "Default route is set correctly"
    else
      echo "Updating default route"
      ip route del default
      ip route add default via $WWAN_GW dev $WWAN_IF metric 1
      ip route add default via $ETH_GW dev $ETH_IF metric 700
    fi
  else
    echo "Can't reach 1.1.1.1 from eth0 or wwan0, contact your administrator"
  fi

# If only is wwan0 is up working
elif [ "$ETH_STATE" == "DOWN" ] && ([ "$WWAN_STATE" == "UP" ] || [ "$WWAN_STATE" == "UNKNOWN" ]); then
  echo "Interface: eth0 is down"
  echo "Interface: wwan0 is up"
  
  # Ping from wwan0
  ping_check $WWAN_IF
  WWAN_PING=$?
  
  if [ $WWAN_PING -eq 0 ]; then
    echo "Interface wwan0 can reach 1.1.1.1"
    echo "Checking that default route is using wwan0 as gateway"
    if [ "$WWAN_METRIC" -lt "$ETH_METRIC" ]; then
      echo "Default route is set correctly"
    else
      echo "Updating default route"
      ip route del default
      ip route add default via $WWAN_GW dev $WWAN_IF metric 1
    fi
  else
    echo "Interface wwan0 can't reach 1.1.1.1, contact your administrator"
  fi
else
  echo "Both interfaces are down or in an unknown state"
fi
```

make the script executebel


```
sudo chmod +x /usr/local/bin/check_connection.sh
```


Run the script manully


```
sudo /usr/local/bin/check_connection.sh
```

Now that the script is working,  we set it  up to run every 5th minnuts. 
To do that we to install cronie and start it

```
sudo pacman -S cronie


sudo systemctl enable cronie
sudo systemctl start cronie
```

to make a job run the following command

sudo systemctl status cronie

```
echo "*/5 * * * * /usr/local/bin/check_connection.sh >> /var/log/check_connection.log 2>&1" | sudo crontab -

```

Add the following line 

```
*/5 * * * * /usr/local/bin/check_connection.sh >> /var/log/check_connection.log 2>&1
```

Verify by checking the status and the log file

```
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
```

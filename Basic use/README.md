# PiKVM v4 Plus Physical Ports

Let's start by looking at the physical ports of the PiKVM.

## Front
At the front, we have the following ports:

![Front](Front.png)

1. **USB**: Used to connect to the device and acts as a mouse and keyboard.
2. **PiKVM HDMI**: This is the HDMI output of the PiKVM.
3. **HDMI Out**: Connected to the monitor so that PiKVM acts as a man-in-the-middle on the video stream.
4. **ATX**: Used to control the power of the computer/server. For this to work, you need to use an ATX card and install it between the power/reset button and the power connections on the motherboard.
5. **HDMI Port**: Connects to the computer/server HDMI out so the PiKVM gets the video stream from the device.

## Back Side

![Back](Back.png)

1. **12V Power**
2. **USB to the PiKVM**
3. **RJ45 or USB C**
4. **5V USB C Power**
5. **SD-card for operating system**
6. **Ethernet port for internet access**

## More Detailed Overview
Here is a more detailed port view for the device:

![Ports](PiKVMPorts.jpg)

## Extra

This ATX card can be connected between the power buttons and motherboard for remote control of the power/reset buttons. It comes with two mounts, one low and one high profile, so it fits inside the motherboard.

![ATX](ATXcard.jpg)

![Mounts](Mounts.jpg)

There is also a special VGA to HDMI adapter. From my reading, there may be problems if you use another adapter, so be aware.

![HDMI](HDMIadapter.jpg)

# Getting Started with PiKVM

Let's connect the PiKVM to power, internet, and computer/server.

![Connections](PiKVMConnection.png)

## Step 1: Accessing the Device

The bare minimum you need is to connect the PiKVM to power and ethernet.
Remember not to turn off the device until it's fully booted for the first time.

You can access the PiKVM via HTTPS or SSH.

There are multiple ways to find the IP address of the PiKVM:
1. You can look at the display on the PiKVM.
2. You can look at the MAC address table on the switch and find the MAC address in the ARP table of the router.
3. You can also scan the network with a network scanner to find the device.

Use the following default credentials to access the PiKVM:

- Web login: 
  - **Username**: admin
  - **Password**: admin

- SSH login: 
  - **Username**: root
  - **Password**: root

## Step 2: Housekeeping

lets so some basic keeping for the device. 

For enabling write mode

To enable write-mode, run command rw (under root).
```
[root@pikvm ~]# rw
```


To disable it, run command ro.
```
[root@pikvm ~]# ro
```


If you receive the message "Device is busy", perform reboot




### Step 2.1 Change root password

When loggined to ssh and in wr mode use the following command


```
[root@pikvm ~]# passwd root
```


### Step 2.2  Change web portal password


When loggined to ssh and in wr mode use the following command

```
[root@pikvm ~]# kvmd-htpasswd set admin
```

If you require additional user for the Web UI access, use the following:

```
[root@pikvm ~]# kvmd-htpasswd set <user> # Set a new user with password or change of an existing one
[root@pikvm ~]# kvmd-htpasswd del <user> # Remove/delete a user
```


### 2.3 update the device


To update, run following commands under the root user:

```
[root@pikvm ~]# pikvm-update
```

If you encounter an error like:

```
[root@pikvm ~]# pikvm-update
bash: pikvm-update: command not found
```

It's most likely you have an old OS release. You can update the OS as follows:

```
[root@pikvm ~]# rw
[root@pikvm ~]# pacman -Syy
[root@pikvm ~]# pacman -S pikvm-os-updater
[root@pikvm ~]# pikvm-update
```

Next time you will be able to use the usual method with pikvm-update.

### 2.4 Verify that PiKVM is working as inteded 


Connecet the PiKVM to a computer/server and check that everthing is working normaly. 

see this link for more built in featuers



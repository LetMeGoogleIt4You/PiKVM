
# Feature 2 - Console Server

Since I have many devices (Cisco networking devices) using console connections, I want the PiKVM to also act as a console server.

![Connections](ConsoleDiagram.png)

You will need a Serial Adapter. I recommend the Gearmo 4 Port USB to Serial RS232 Adapter. [Link](https://www.amazon.com/dp/B004ETDC8K?ref_=pe_386300_442618370_TE_sc_as_ri_0)

It has one USB connection to four serial connections.

![Connections](ConsoleCabel.jpg)

SSH to the PiKVM and enable read-write mode:

```sh
rw
```

Install ser2net and start it:

```sh
[root@pikvm ~]# sudo pacman -S ser2net
[root@pikvm ~]# sudo systemctl start ser2net
[root@pikvm ~]# sudo systemctl enable ser2net
```

Connect your USB to Serial Adapter:

```sh
[root@pikvm ~]# dmesg | grep ttyUSB
[   14.402293] usb 1-1.3: GSM modem (1-port) converter now attached to ttyUSB0
[   14.416729] usb 1-1.3: GSM modem (1-port) converter now attached to ttyUSB1
[   14.431156] usb 1-1.3: GSM modem (1-port) converter now attached to ttyUSB2
[   14.445514] usb 1-1.3: GSM modem (1-port) converter now attached to ttyUSB3
[14064.346437] usb 1-1.1: FTDI USB Serial Device converter now attached to ttyUSB4
[14064.366557] usb 1-1.1: FTDI USB Serial Device converter now attached to ttyUSB5
[14064.386687] usb 1-1.1: FTDI USB Serial Device converter now attached to ttyUSB6
[14064.406874] usb 1-1.1: FTDI USB Serial Device converter now attached to ttyUSB7
```

My USB Serial Device port numbers are ttyUSB4-7.

Now update the ser2net.yaml file:

```sh
[root@pikvm ~]# sudo nano /etc/ser2net/ser2net.yaml
```

For Cisco devices, add the following config to ser2net.yaml:

```yaml
connection: &con01
    accepter: telnet,tcp,2001
    enable: on
    options:
      kickolduser: true
      telnet-brk-on-sync: true
    connector: serialdev,
              /dev/ttyUSB4,
              9600n81,local

connection: &con02
    accepter: telnet,tcp,2002
    enable: on
    options:
      kickolduser: true
      telnet-brk-on-sync: true
    connector: serialdev,
              /dev/ttyUSB5,
              9600n81,local

connection: &con03
    accepter: telnet,tcp,2003
    enable: on
    options:
      kickolduser: true
      telnet-brk-on-sync: true
    connector: serialdev,
              /dev/ttyUSB6,
              9600n81,local

connection: &con04
    accepter: telnet,tcp,2004
    enable: on
    options:
      kickolduser: true
      telnet-brk-on-sync: true
    connector: serialdev,
              /dev/ttyUSB7,
              9600n81,local
```

Restart the ser2net service:

```sh
sudo systemctl restart ser2net
```

Verify the status of the ser2net service:

```sh
[root@pikvm ~]# sudo systemctl status ser2net
```

Connect to the device using telnet and the port number:

```sh
[root@pikvm ~]# telnet localhost 2000
Trying ::1...
Connected to localhost.
Escape character is '^]'.

User Access Verification
Username: admin
Password:
CiscoDevice#
```

To exit the telnet session, use "ctrl" + "]" and type "quit".

Be warned: If you are using a non-English keyboard, you may have a problem with the "ctrl" + "]" combination. For me, I had to use "ctrl" + "¨" (two keys right from the "P" key).

Reference:
https://superuser.com/questions/486496/how-do-i-exit-telnet

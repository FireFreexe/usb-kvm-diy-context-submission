must have
- micro sd card
- microsd card reader (mine is a usb one that looks like a flash drive)
- rasberry pi 5

of course the steps will be slightly different if you are using a different pi model, but the earlier models have much more documentation on setting them up to use gadget mode

Download the official rasberry pi imager and run it

choose your device (Pi 5), OS (PI OS 64bit), and storage which will be your sd reader

for ease of settup, edit the OS configuration to whatever suits your. I changed the hostname to something shorter, set up my username and password, wifi, timezone, and SSH.

then just confirm your way through the menue and have it write to the microSD card

now boot up your pi, find its ip lease in your router, and connect to it via ssh

update everything as is normal with a fresh install

```
sudo apt update
sudo apt upgrade -y
```

then update the firmware to allow the gadget support

```
sudo rpi-update
```

after is updates, reboot

```
sudo reboot
```

now to edit some files

Add modules-load=dwc2 to the end of /boot/firmware/cmdline.txt file
Add libcomposite to the end of the /etc/modules file
Add dtoverlay=dwc2 to /boot/firmware/config.txt
comment out otg_mode=1 from /boot/firmware/config.txt

now make the file /usr/local/sbin/usb-gadget.sh and put this in it:

```
#!/bin/bash

cd /sys/kernel/config/usb_gadget/
mkdir -p display-pi
cd display-pi
echo 0x1d6b > idVendor # Linux Foundation
echo 0x0104 > idProduct # Multifunction Composite Gadget
echo 0x0103 > bcdDevice # v1.0.3
echo 0x0320 > bcdUSB # USB2
echo 2 > bDeviceClass
mkdir -p strings/0x409
echo "fedcba9876543213" > strings/0x409/serialnumber
echo "Ben Hardill" > strings/0x409/manufacturer
echo "Display-Pi USB Device" > strings/0x409/product
mkdir -p configs/c.1/strings/0x409
echo "CDC" > configs/c.1/strings/0x409/configuration
echo 250 > configs/c.1/MaxPower
echo 0x80 > configs/c.1/bmAttributes

#ECM
mkdir -p functions/ecm.usb0
HOST="00:dc:c8:f7:75:15" # "HostPC"
SELF="00:dd:dc:eb:6d:a1" # "BadUSB"
echo $HOST > functions/ecm.usb0/host_addr
echo $SELF > functions/ecm.usb0/dev_addr
ln -s functions/ecm.usb0 configs/c.1/

#RNDIS
mkdir -p configs/c.2
echo 0x80 > configs/c.2/bmAttributes
echo 0x250 > configs/c.2/MaxPower
mkdir -p configs/c.2/strings/0x409
echo "RNDIS" > configs/c.2/strings/0x409/configuration

echo "1" > os_desc/use
echo "0xcd" > os_desc/b_vendor_code
echo "MSFT100" > os_desc/qw_sign

mkdir -p functions/rndis.usb0
HOST_R="00:dc:c8:f7:75:16"
SELF_R="00:dd:dc:eb:6d:a2"
echo $HOST_R > functions/rndis.usb0/dev_addr
echo $SELF_R > functions/rndis.usb0/host_addr
echo "RNDIS" >   functions/rndis.usb0/os_desc/interface.rndis/compatible_id
echo "5162001" > functions/rndis.usb0/os_desc/interface.rndis/sub_compatible_id

ln -s functions/rndis.usb0 configs/c.2
ln -s configs/c.2 os_desc

udevadm settle -t 5 || :
ls /sys/class/udc > UDC

sleep 5

nmcli connection up bridge-br0
nmcli connection up bridge-slave-usb0
nmcli connection up bridge-slave-usb1
sleep 5
service dnsmasq restart
```

then make it executable:

```
sudo chmod +x /usr/local/sbin/usb-gadget.sh
```

now make the file /lib/systemd/system/usbgadget.service and put this in it:

```
[Unit]
Description=My USB gadget
After=network-online.target
Wants=network-online.target
#After=systemd-modules-load.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/local/sbin/usb-gadget.sh

[Install]
WantedBy=sysinit.target
```

then enable it

```
sudo systemctl enable usbgadget.service
```

now we make a bridge to connect over. note the IP address, we will use it later to connect through the usb-c cable

```
sudo nmcli con add type bridge ifname br0
sudo nmcli con add type bridge-slave ifname usb0 master br0
sudo nmcli con add type bridge-slave ifname usb1 master br0
sudo nmcli connection modify bridge-br0 ipv4.method manual ipv4.addresses 10.55.0.1/24
```

install dns masq

create the file /etc/dnsmasq.d/br0 and put this in it:

```
dhcp-authoritative
dhcp-rapid-commit
no-ping
interface=br0
dhcp-range=10.55.0.2,10.55.0.6,255.255.255.248,1h
dhcp-option=3
leasefile-ro
```

this should handle the putting it in gadget mode. lets turn on vnc before we move forward.

```
sudo raspi-config
```

interface options:
VNC
enable
finish
reboot

everything should be ready and up and running. before i go ahead and use the vnc with the ipad, i will install the mini kvm software though command lines

i just followed their walkthrough on the github

sudo wget https://github.com/TechxArtisanStudio/Openterface_QT/releases/download/v0.2.0/openterfaceQT.linux.arm64.deb

sudo apt install -y libqt6core6 libqt6dbus6 libqt6gui6 libqt6network6 libqt6multimedia6 libqt6multimediawidgets6 libqt6serialport6 libqt6svg6 libusb-1.0-0-dev

sudo usermod -a -G dialout $USER

echo 'KERNEL== "hidraw*", SUBSYSTEM=="hidraw", MODE="0666"' | sudo tee /etc/udev/rules.d/51-openterface.rules

sudo udevadm control --reload-rules
sudo udevadm trigger

sudo dpkg -i openterfaceQT.deb

openterfaceQT


must have
- micro sd card
- microsd card reader (mine is a usb one that looks like a flash drive, but any will do)
- rasberry pi 5

Of course the steps will be slightly different if you are using a different pi model, but the earlier models have much more documentation on setting them up to use gadget mode.

Download the official rasberry pi imager and run it.

Choose your device (Pi 5), OS (PI OS 64bit), and storage (which will be your sd reader).

For ease of settup, edit the OS configuration to whatever suits you. I changed the hostname to something shorter, set up my username and password, wifi, timezone, and SSH. Internet will only be needed for initial setup and installing the mini-kvm software.

Then just confirm your way through the menue and have it write to the microSD card.

Now boot up your pi, find its ip lease in your router, and connect to it via ssh.

Update everything as is normal with a fresh install.

```
sudo apt update
sudo apt upgrade -y
```

Then update the firmware to allow the gadget support.

```
sudo rpi-update
```

After is updates, reboot.

```
sudo reboot
```

Now to edit some files:

Add `modules-load=dwc2` to the end of `/boot/firmware/cmdline.txt` file
Add `libcomposite` to the end of the `/etc/modules` file
Add `dtoverlay=dwc2` to `/boot/firmware/config.txt`
Comment out `otg_mode=1` from `/boot/firmware/config.txt`

Now make the file `/usr/local/sbin/usb-gadget.sh` and put this in it:

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

Then make it executable:

```
sudo chmod +x /usr/local/sbin/usb-gadget.sh
```

Now make the file `/lib/systemd/system/usbgadget.service` and put this in it:

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

Then enable itL

```
sudo systemctl enable usbgadget.service
```

Now we make a bridge to connect over. Note the IP address, we will use it later to connect through the usb-c cable.

```
sudo nmcli con add type bridge ifname br0
sudo nmcli con add type bridge-slave ifname usb0 master br0
sudo nmcli con add type bridge-slave ifname usb1 master br0
sudo nmcli connection modify bridge-br0 ipv4.method manual ipv4.addresses 10.55.0.1/24
```

Then install dns masq:

```
sudo apt-get install dnsmasq
```

Now create the file `/etc/dnsmasq.d/br0` and put this in it:

```
dhcp-authoritative
dhcp-rapid-commit
no-ping
interface=br0
dhcp-range=10.55.0.2,10.55.0.6,255.255.255.248,1h
dhcp-option=3
leasefile-ro
```

This should handle putting the Pi in gadget mode. Lets turn on VNC before we move forward.

```
sudo raspi-config
```

Click through the interface options as followed:
VNC
enable
finish
reboot

Now, Everything should be ready and up and running. Before I go ahead and use the VNC with the IPad, I will build the mini-kvm software from source. I only did this because at the time of writing, their arm linux download did not work. As long as you get the software working on your Pi this tutorial should work, since i do not edit the software at all. In fact, I just followed their walkthrough on the github, which I will copy here:

```
# Build environment preparation   
sudo apt-get update -y
sudo apt-get install -y \
    build-essential \
    qmake6 \
    qt6-base-dev \
    qt6-multimedia-dev \
    qt6-serialport-dev \
    qt6-svg-dev \
    libusb-1.0-0-dev \
    qt6-tools-dev
```

```
# Setup the dialout permission for Serial port
sudo usermod -a -G dialout $USER
# On some distros (e.g. Arch Linux) this might be called uucp
sudo usermod -a -G uucp $USER

# Setup the hidraw permission
echo 'KERNEL== "hidraw*", SUBSYSTEM=="hidraw", MODE="0666"' | sudo tee /etc/udev/rules.d/51-openterface.rules 
sudo udevadm control --reload-rules
sudo udevadm trigger
```

```
# Get the source
git clone https://github.com/TechxArtisanStudio/Openterface_QT.git
cd Openterface_QT

# Generate language files (The lrelease path may vary depending on your system)
/usr/lib/qt6/lrelease openterfaceQT.pro

# Build the project
mkdir build
cd build
qmake6 ..
make -j$(nproc)
```

```
# Run
./openterfaceQT
```

```
# If you can't control the mouse and keyboard (with high probability that did not correctly recognize the serial port)

# solution
sudo apt remove brltty
# after run this plug out the openterface and pulg in again
ls /dev/ttyUSB*
# if you can list the usb the serial port correctly recognized
# Then we need give the permissions to user for control serial port you can do this:
sudo ./openterfaceQT
# or (dialout/uucp)
sudo usermod -a -G dialout <your_username>
sudo reboot
# back to the build floder
./openterfaceQT
```

Now, everything should be up and running, and you should be able to open up the software. Now lets connect using the IPad.

Plug in the usb-c cable into your ipad and into your Pi's power input. The pi should turn on and start running everything it needs to start up. Keep in mind that the IPad supplies little power, so the startup will take a bit and might occasionally crash due to low voltage. If the Pi's light turns red, that means it crashed, so just unplug it and replug it in, and it should eventually work.

Now, install a VNC app onto the IPad, I use RealVNC. You will try to connect to 10.55.0.1, whcih is the IP I mentioned earlier, so if you changed it then, change it now. If the connection times out, and the Pi's light is still green, the Pi is probably still setting up, so give it a few more minutes. Eventually it should let you connect, and you are done!

This connection is entirely internal through the usb-c cable, so no outside internet is required, and since you are using the battery from the IPad, you will not need a new source of power as long as the IPad stays charged. The IPad is now the power source, keyboard, mouse, and monitor for the Pi, and since the Pi has mini-kvm on it, can be the keyboard mouse and monitor for any target computer you can plug in to.

From here, just follow normal mini-kvm documentation to get finished. Plug in the Pi to the mini-kvm with usb-a cables, and plug in the hdmi and usb cables from the mini-kvm to the target conmputer, and you can now control the target directly from the IPad.

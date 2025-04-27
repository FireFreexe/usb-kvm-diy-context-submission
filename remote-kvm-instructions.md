This part of the instructions are a lot simpler than the other. This is sort of just a bonus add-on that I'm throwing in, so enjoy!

# Remote bios-level KVM using a Pi

So, if you have a Pi, you can use it in tandom with the Openterface mini-kvm to remotely get bios-level access to your machine. What I did for my solution was set up the Pi, turn on VNC, and download the mini-kvm software, which is all I need to go to get into my machine.

These instructions will start as if you have not done anything yet, but you may be able to skip some steps if you have already followed my [ipad kvm instructions](https://github.com/FireFreexe/usb-kvm-diy-context-submission/blob/main/ipad-kvm-instructions.md).

# Get your Pi running

Download the official rasberry pi imager and run it.

Choose your device (Pi 5), OS (PI OS 64bit), and storage (which will be your sd reader).

For ease of settup, edit the OS configuration to whatever suits you. I changed the hostname to something shorter, set up my username and password, wifi, timezone, and SSH. Internet will only be needed for initial setup and installing the mini-kvm software.

Then just confirm your way through the menue and have it write to the microSD card.

Now boot up your pi, find its ip lease in your router, and connect to it via ssh.

# Initialize the Pi and turn on VNC

Update everything as is normal with a fresh install.

```bash
sudo apt update
sudo apt upgrade -y
```

Now run this to get to the menu to turn on VNC.

```bash
sudo raspi-config
```

Click through the interface options as followed:
VNC
enable
finish
reboot

Now, Everything should be ready. Now lets install the mini-kvm software.

# Install OpenterfaceQT

I will be building the mini-kvm software from source. I only did this because at the time of writing, their arm linux download did not work. As long as you get the software working on your Pi this tutorial should work, since i do not edit the software at all. In fact, I just followed their walkthrough on the github, which I will copy here:

```bash
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

```bash
# Setup the dialout permission for Serial port
sudo usermod -a -G dialout $USER
# On some distros (e.g. Arch Linux) this might be called uucp
sudo usermod -a -G uucp $USER

# Setup the hidraw permission
echo 'KERNEL== "hidraw*", SUBSYSTEM=="hidraw", MODE="0666"' | sudo tee /etc/udev/rules.d/51-openterface.rules 
sudo udevadm control --reload-rules
sudo udevadm trigger
```

```bash
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

Now that the software is downloaded, go ahead and hook up the mini-kvm. From here, just follow normal mini-kvm documentation to get finished. Plug in the Pi to the mini-kvm with usb-a cables, and plug in the hdmi and usb cables from the mini-kvm to the target conmputer, and you can now control the target fromt the Pi.

Now, everything should be ready to go, and you should be able to open up the software.

```bash
# Run
./openterfaceQT
```

# Connecting to the Pi remotely

Now that vnc is enabled and OpenterfaceQT is installed, we can vnc into the Pi from our main computer. Download a vnc viewer (I use RealVNC) and connect to the Pi using its IP address. You should now be able to control the Pi, and by extension, the Target.

Additionally, you could choose to now install an app such as TeamViewer onto the Pi, so that you would be able to connect to the Pi from anywhere with an internet connection. At my home, I have a VPN set up that allows me to remote into my home network, so when im away from home, I can just use the VPN to get into my network then use RealVNC to connect to the Pi, allowing me to gain bios-level access to my target.

How you choose to use this is up to your, but this walkthrough was just to show the basic setup.

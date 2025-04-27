As of time of submission, the [Openterface mini-KVM](https://openterface.com/) has loads of usage, but sadly is not able to be used natively with an IOS device, such as an IPad. This was devastating news for me, since my University supplied everyone with IPads, so I never purchased a laptop, and have been instead finding ways to use an IPad for everything that my Computer Science major requires. I have programmed, sshed, and everything else you can imagine from this device, so now I just needed to find a way to use it as a bios-level KVM.

For my submission, I put together a way to use an IPad, a Raspberry Pi, the mini-kvm, 1 usb-c cable, and the cables necessary for the mini-kvm (in my case, two usb-a to usb-c cables and an hdmi cable) to work as a full standalone KVM that works with no internet connection and no power source other than the battery in the IPad. As a bonus, I also included a way to use the Raspberry Pi to either use VNC to remote into the target computer from anywhere on the network, or use an application such as TeamViewer to remote into the target computer from anywhere with internet access.

# Standalone KVM

So first I will talk about how Set up a standalone KVM using the Pi, IPad, and mini-kvm, and I go over the specifics in [ipad-kvm-instructions.md](https://github.com/FireFreexe/usb-kvm-diy-context-submission/blob/main/ipad-kvm-instructions.md).

What I did was first put the Pi in gadget mode so that the Pi could be powered by the IPad. This lets me plug in the Pi to the IPad through their respective power ports, and the Pi can power on. Next, to connect to the Pi, I set up an internal bridge that makes the Pi's usb-c power port also sort of act as a network port. With everything set up, and VNC turned on, I can now use the usb-c cable connecting the Pi and IPad as both a power cable and a network cable, allowing me to use a VNC viewer on the IPad to connect and fully control the Pi.

Here are some pictures:

This one shows how after setup, you can install and run the mini-kvm software and hook it up to use the IPad as a kvm for the target computer.

![PXL_20250427_191916410 MP](https://github.com/user-attachments/assets/d519fd1d-6dee-46bc-9d37-894440c69886)

This photo shows closer how the Pi is only connected to the IPad and the mini-kvm.

![PXL_20250427_191939854 MP](https://github.com/user-attachments/assets/c1255b33-c2b0-43c9-a58d-9632192afe84)

Overall, since you are using an internal network to connect over VNC, this solution works without wifi or any other internet, and since the IPad is the power source, you do not need to find a wall outlet or anything.

This puts me in the wonderful spot of being able to use my IPad as a bios level kvm for my headless servers, which is much easier than anything I as doing vefore the mini-kvm came out. But the mini-kvm also allows for some other usage when paired with a Pi:

# Remote KVM

if you have one server than constantly needs to be controlled, then you can just choose to let the Pi be there permanently. Here, I connected my Pi to the wifi and found its IP address, so now I can leave the mini-kvm hooked up and control the server directly from my main computer, which was never possible for me before since my server and my computer are in different rooms.

It works wonderfully!

![PXL_20250427_193624842 MP](https://github.com/user-attachments/assets/7656cfd6-7aed-4fef-b127-ca6dbae319e0)

Additionally, if you either have a vpn into your home network, or install a program like TeamViewer onto the Pi, you could remote into your server from anywhere with internet. I personally have a vpn to my home, so a lot of my servers I can ssh to while im on campus, but now I have the ability to get remote access to the bios of my main server! This server has enough issues that when it reboots, I usually need to change bios settings, and now I am able to play witht he bios setting when im not even at home, which was never a possibility before the mini-kvm.

# Conclusion

When I learned about the mini-kvm, I had to have one. It simply lets me do more with what I have. While the mini-kvm software doesnt have native support for IOS yet, I found a way to use what I have to make it work, and hopefully this helps at least a few others out there that are in a similar situation as me. Thank you for reading this far, this is my first ever documented project, and my first time using github for my personal projects. I hope you have a wonderful day! :3

# TurtleGS

LattePanda Mu based cyberdeck intended to act as a rocketry ground station.

# 2026-06-03: Battery Charging

**Total time spent: 6 hours**

Hi. So. 
I started with the BQ2589 and the HUSB238A that I was working with last entry. The BQ is battery charging, the HUSB is usb PD negotiation. 
First thing I immediately noticed is that the schematic symbol for the BQ is weird.  

![image.png](https://cdn.hackclub.com/019e8f72-7e2f-7b6a-be89-f0ad1f3ee219/image.png

Like, this sucks. HoweverI decided to keep trying.

I then noticed the datasheets suck too. 

![image.png](https://cdn.hackclub.com/019e8f73-427f-7524-a8e1-a61f65709dcb/image.png)

The HUSB for instance, which I was trying to wire first, had a single page datasheet with zero info. No resistor values, no explanations, nothing. It was useless. 

So, I decided to switch chips. I immediately turned to Ti products as I know the documentation is great. I stared at their catalogue for a while, trying to find a chip combo that could handle high speed usb PD charging and had 1s lipo support. 

Eventually, I settled on the TPS25751D for USB-PD negotiations and the BQ25798 for battery charging. This combo can handle 20v charging at 3.25a, which is the highest battery charger I have.

This is a cool set of chips. They talk to eachother over serial and can be programmed with different charging profiles and whatnot via a simple web app and cheap programming dongle. Pretty cool!

While looking for application circuit examples, it turns out Ti has an entire battery charging board design that is public using this exact chipset. I just had to modify it to be set for 1s instead of adjustable 2-4s. Which is easy! 

![image.png](https://cdn.hackclub.com/019e8f76-f715-7b05-8150-69ecac638e78/image.png)

Ti is really just awesome like that. Along with the two aforementioned chips I learned I would need a 5v boost converter, a 3v3 LDO, and an EEPROM chip for storing the charging profile. And some programming/debug headers. 

The chips actually all talk to eachother over serial! Isn't that awesome. The MCU can interject itself into this path and do on the fly programming and charging control, but I don't want to add that layer of complexity, so I will be keeping it standalone. Program it once and forget. 

By following the awesome Ti datasheet, I was able to get this circuit: 

![image.png](https://cdn.hackclub.com/019e8f78-eb94-74e2-9bb5-421aa12f03ee/image.png)

One USBPD IC, One battery charging IC, one 5v buck, one 3v3 LDO, an eeprom, and some headers. This battery charging setup gets an entire page of schematic! It's kinda crazy compared to how small circuits that run on low speed charging can be. I mean, I literally went from my last project fitting entire boards into one page to now having a battery charging page. Crazy. 

One cool thing about this setup is that it has a dedicated thermistor for monitoring battery temps. I am really looking forward to having this as this device will get pretty warm sitting in the sun at launches. Hopefully, this prevents the battery from kerploding.

This took a long time but having the Ti reference was great and made the whole thing way more straightforward. Most of the time was just interpreting what the circuit was doing so I could recreate it! And figuring out the changes needed for 1s of course. 

I have two debug headers that are 2x5 pin for programming the chips and one 1x4 header for debugging serial stuff, since that's the crux of the system and how the two chips talk. Otherwise, they will literally start a battery fire. 
Not too shabby!

# 2026-06-02: The Beginning

**Total time spent: 2 hours**

Okay.
This has been floating around my head for a while but basically I was thinking. I don't have a laptop. I cannot afford a laptop. But I do have hackclub.
What's better than a laptop? A cyberdeck!
But a general purpose one is kinda lame. Good thing I do rocketry. 

So here we are. A rocketry ground station cyberdeck. 
I will preface by saying the scope creep will be strong with this one.


Initial feature list:

Custom CM5 Carrier Board Based.
LORAWAN enabled for controlling electronics and monitoring rocket. Both dedicated 915mhz lorawan transmitting as well as general RTL-SDR monitoring. 
Ethernet
USBC Charging and IO
Triple USBA IO
XT 30 EMatch continuity checker
Dual HDMI
Built in touch screen
Custom Keyboard
Custom Track Ball
Custom Scroll Wheel
Some buttons!


I think that is everything. Quite a lot of features - this is gonna be a long one, with a lot of pcb design. 

I already started the preliminary PCB design. Mainly, component selection. I will likely need to have a multi page schematic to fit everything cleanly, but that is ok. 

![image.png](https://cdn.hackclub.com/019e89b1-fba4-7c51-ae5f-3fabd7482350/image.png)

For now I have this big schematic covered in components. It has the IO ports, m.2 nvme port, usb stuff, battery charging stuff, and lorawan stuff. Oh and the CM5. I need to figure out my schematic hierarchy. Maybe: 
MCU/Storage/RF
Everything IO/Ports
Inputs (keyboard, scroll wheel, track ball)/Sensors.

Maybe? I am not sure. I will workshop it.
I am actually pretty excited about this despite how nightmarish it is going to be. Let's do this thing! 



# TurtleGS

LattePanda Mu based cyberdeck intended to act as a rocketry ground station.


# 2026-06-16
**Total time spent: 4.5 hours**

Alright so. I decided to tackle the usbc 3.2 ports. I want to have both ports able of charging the battery with decent speed (20v) and high data speed.

First things first I had to find a new usbc port with the required rx/tx lines for high speed. From there, I wanted to do the TVS filtering. I started following this diagram from TI:

<img width="1331" height="767" alt="image" src="https://github.com/user-attachments/assets/93a4e796-a86e-4e17-adfb-74121b060cee" />

<img width="1106" height="622" alt="image" src="https://github.com/user-attachments/assets/ee3a1ae6-60a0-4cd3-9816-6483370a3094" />

<img width="1578" height="595" alt="image" src="https://github.com/user-attachments/assets/364bf096-4117-46ce-bf8a-bcce5e0d00e4" />

This sucks. Like, really bad. It will be impossible to route, takes so much space, and the parasitics are probbaly mind boggling. So, I decided to look for chips that do multi input filtering. This way it's nice and compact but still gives me all the tvs protection. 

<img width="1328" height="953" alt="image" src="https://github.com/user-attachments/assets/21e6e12f-e921-4e2b-b54f-916e86ba5c95" />

After a LOT of digging I ended up at this. One IC specifically for handling Vbus, since that is high voltage, and then 3 of another IC for rx/tx/d+/d-. These diodes act as protection from surges, static, water shorting, etc. so my circuit doesn't fry. That took about 2 hours.

Now for what took the rest of the time. Trying to figure out multi port charging. 
Tl:Dr It is extremely difficult to pull off as a hobbyist. Why? I don't know. But I will just have one charging port.

I started with just trying to have two usb pd negotiators and one charging IC. However. USB PD requires that the negotiator chip talk with the charger IC for safety purposes. For some reason, nobody has bothered to implement it in a way that allows for more than one negotiator per charger. With all the Ti products for example, none of them offer i2c addressing, so they will always end up conflicting with eachother and could even pose a fire risk. 
All the chargers that can handle it require talking to your CPU itself. The problem with that is... that is really, really difficult to configure. It's not available in BIOS clearly and it isn't in software. It's like cpu side firmware stuff that you need specialized tools to work with. This is why laptop manufacturers always partner with CPU manufacturers (and charger manufacturers) - they get those tools. 
At one point I found a dual port charger IC from TI and got super excited!

But. It was not 2 in 1 out. It was 2 in 2 out. It solved none of my issues as you still had conflicting logic of the two power paths. Bleh.

So, after all that effort, talking with friends, even making a reddit post, I decided it would just be best if I only had charging on one port and data on both. That's fine and makes my job simpler, I just have to remember which one is charging when I go to plug the device in. 
Anywho, I didn't end up getting to routing the signals to the usb 3.2 logic controller, which is what handles all the signal processing to pass onto the lattepanda and whatnot. I will do that next entry! I also want to work on HDMI next entry. That too will need TVS diode protection but I should be able to use the same chips I think! 
I spent so long researching and reading datasheets. I have 60 chrome tabs open right now. Bleh.

# 2026-06-15: Big News!
**Total time spent: 3 hours**

Hi. So.
This project has been sponsored by LattePanda/DFRobot! I will be receiving a LattePanda Mu to base this project around.

That means some things have to change! My MCU of course, my power plan a little, my format, my USB plan, etc. A lot of changes.

First change - get the lattepanda in and start labeling pins! I got the symbol for this fella from an open source carrier board found here: https://oshwlab.com/yoge2013/mini-carrier-demo-for-lattepanda-mu
This is a very well done mini carrier and will be good inspiration for mine. 

<img width="1592" height="1126" alt="image" src="https://github.com/user-attachments/assets/14d71498-c3af-44d7-8d75-5130911b6733" />

Because of how big this is and how many pins there are, it gets two sections in its schematic. One is for power and one is for literally everything else. Power is on the right, else on the left.

I am not sure if I will be using all of these pins but I will sure use a lot of them, so they are getting labels just in case.

Now, for what changes.
First of all my power architecture changes a little as I need to boost 1s lipo voltages up to 12v for the Mu. This is simple enough and can be taken care of by a single boost converter. This circuit was figured out by Ti's webench power supply configurator, a glorious tool that made this so easy. I inputted what I had and what I needed and it spat out a bunch of options - I picked the simplest and most efficient one. 

<img width="972" height="586" alt="image" src="https://github.com/user-attachments/assets/38c93c61-525b-492b-bbaf-db97d4c397b0" />

I also realized my 5v setup was a buck, not a boost, so I need to give it the 12v instead of raw system battery voltage. Easy fix. 
So the power path goes Battery -> Regulator/Charger -> 12v -> 5v -> 3v3
I think that should work. I don't know if I need 1v8 yet... we are gonna find out. 

Anywho, I also stole the m.2 storage setup from the carrier board. "Stole". It needs a little modification to fit my stuff but I will do that later. 

<img width="1758" height="802" alt="image" src="https://github.com/user-attachments/assets/bca3db0b-235f-43eb-b344-88e5f480fba5" />

One of them is actually for a wifi card while the other is actually storage. The Mu does not have built in wifi like the CM5 does. Probably doesn't fit, this thing is sooo dense.

Anywho. Some more changes will need to be made. 
For starters, I don't think I need the USB Hub IC's anymore? Since the Mu has 8x USB 2.0 + 4x Usb 3.2. But I think some of them have to be multiplexed pins from pcie/sata? Basically meaning dual function pins and I can only pick one function, via bios. Since I need pcie for both storage and wifi, I might get cut short on usb ports. But that might not be an issue! I need to look into it more and do some planning.

What else... The lorawan stuff probably cannot be over SPI anymore? I don't actually know how that works with windows... Or I just give in and run linux. I dunno. Gotta think about it.

Continuity checker will probably be made hardware only? Or it just becomes yet another usb device. Maybe I will want a usb hub IC just so I have more IO available. 

RTL-SDR is usb as always. Screen... I have two options. I can either deal with routing hdmi or try to just use the ribbon on the mu. The issue with that is there can be mismatches in pinouts on the ribbon cables - that can't happen with hdmi. So I think I will probably just expose HDMI. One external for external screens and one internal for the main screen, I think. 

That's all for this entry. I am gonna continue reading lattepanda documentation and watch some videos on carrier board design (omg... those exist!!!).

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



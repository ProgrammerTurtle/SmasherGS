# TurtleGS

LattePanda Mu based cyberdeck intended to act as a rocketry ground station.

# 2026-06-26 
**Total time spent: 2 hours**

Keyboard! I finally decided to do some pcb design for this project and put the whole keyboard together. 
Did I remember to take progress screenshots? 
Nope. So. 

<img width="1924" height="776" alt="image" src="https://github.com/user-attachments/assets/048ff681-b6e3-4922-8434-98cc54d3dd87" />

This is the board. I need to add some mounting still but I cannot figure out what fits without messing with cad a bit, as this pcb doesn't have a representation of the switches.
Here was my process.
1. Organize. I started by manually laying out the whole key grid. Easyeda has tools to do this, but they cannot handle multiple components that group together (like switches and diodes). So, I did it all by hand. The keys are on a 17.5x18.5mm grid. This should put 1mm between each key. I then put all the brains on the left of the board for no reason other than I felt like it. In hindsight I maybe should have put them on the top, but I don't think I want to redo it   .
2. Wire the grid. This was tedious as I had to go and do the traces for each diode and key, but I got it done. 
3. Wire the brains. This is for 2 layer, so I have no gnd or 3v3 planes, something I have grown very used to. 
4. Wire the grid to the brains. This took the most time as I had to keep redoing traces when I figured out I blocked something. 
That's it ! A super quick ortholinear 50% keyboard with USBC for my deck. It may be detachable, hence the USBC. I haven't decided yet but I will soon.

That's the easy part of this project done. Next is a bunch of hard routing. Ugh. 

# 2026-06-25
**Total time spent: 2.5 hours**                

Little fixes, little annoyances. 
Ok so. I had to make a bunch of small fixes in prepping for pcb time and ugh. For instance, changing every instance of a + or - to _P or _N. Why? Because I decided I like that more and want it to be consistent across my whole schematic. What else... I ran DRC a few times to catch issues. Everything is wired up now. 
Running DRC also showed me that some components were missing footprints, so I had to resolve that. For the most part that was an easy fix. But, I found out my usba 3.0 port was completely messed up and was somehow half of a 2 port schematic, with no footprint. So I had to fully replace that component. Fun fact - majority of LCSC usba 3.0 ports have no footprint or schematic. Problematic. I found one though! 

<img width="999" height="674" alt="image" src="https://github.com/user-attachments/assets/df2c5bf1-51c4-4e12-8597-722099f5193c" />

Tadaaa. 

Oh I also realized I forgot fans and a power switch setup. Fans were easy, I just stole the circuit from the lattepanda reference design because I know that works. 

<img width="1601" height="1120" alt="image" src="https://github.com/user-attachments/assets/e1cf83e4-e834-4d52-bedc-251b1e5a6af5" />

Fan 1 is the cooler attached to the lattepanda while fan 2 will be some sort of additional inlet or something. I haven't decided yet but I will figure it out. It's nice to have either way. >

<img width="1603" height="1125" alt="image" src="https://github.com/user-attachments/assets/22c2530e-3419-4ebe-bf09-d389e2498f94" />

Power switch is simple but actually made me realize I had no way to cut power to anything but the mu, or everything. All or nothing basically. And that is not great for battery life, especially when my battery charging/monitoring circuit NEEDS to maintain power or else I could get a lipo fire (and I can't charge my battery, and I can't turn on stuff, etc). So, I split my 5v into two sections. 5v for the battery side and 5v for everything else. Then I put a load switch between the two and wired it to the power button. 
90% of the power draw that isn't the mu is on 5v. 3V3 draws practically nothing and it's for the best that I maintain 3v3 power, as the Mu needs that for bios stuff. 12v is also needed to stay on as the Mu doesn't fully lose power, it goes into a shut-down standby mode waiting to be turned back on. This protects your data and whatnot. In desktop PCs, the PSU never actually turns off unless you actively turn it off! The power button just kills everything else. Same philosophy here. 

Now I think I really am ready to start the pcb. I do want to maybe take a look in cad first to get an idea of what to do, but I could also just take a reserved design approach and work around the pcb. My two main design goals would thus be access to I/O and keeping it thin. Those are pretty simple if you just keep it in mind. I do need to make sure there is space for internal components and the screen and whatnot. 

PCB Soon! 

# 2026-06-24
**Total time spent: 3 hours**

GPS and LORA!! These are the last two major things of the project. I am forgoing the continuity checker now because all my flight computers do continuity checking and, more importantly, I don't feel comfortable applying current to explosive charges using anything I designed. 

First was GPS. I spent a while researching bias tee circuits so I could properly supply 5v to my active antenna. Bias tees are circuits that inject DC power into coaxial, aka AC, lines without disrupting the signal. Black magic in my opinion. Witchcraft. Regardless, after all my research, a friend pointed out that ublox has an integration manual that gives you the exact circuit needed. Because of course I would find that after my research. 

<img width="1690" height="621" alt="image" src="https://github.com/user-attachments/assets/fda857d8-0c65-4be6-bf3f-270c5a0c7f8e" />

It's actually quite simple in the end. I wired power, ground, uart for data, and the antenna line. That's it. I just have to remember that the antenna line is impedance matched to 50 ohm, which means I need to specify board stackup and all that jazz. I already need to do some of that stuff for other high speed sensitive signals like usb 3.2, ethernet, and hdmi, so nothing new for this project. This should be fully functiona GPS for my deck because linux is cool and supports uart well. This gps module is ten bucks... sheesh.

Anywho, next, and somehow the last part of the entire schematic, was LORAWAN. I was initially using an SX1262 module from SEEED Studio but unfortunately none of those feature UART, only SPI. Technically speaking, the Mu has SPI, but it is in practice only able to be used for audio. So, I had to find something with UART. 
I ended up finding the RAK3172-T. This is a cool module that uses a fancy MCU from ST that has lorawan integrated. Actually, it is the exact same mcu I am using for a camera controller board. That camera controller project is the reason I am putting lorawan on this deck in the first place (there are other reasons too now, but that was the main driving one). So, funnily, we come full circle. 
Because this is a full MCU it does need firmware updates, so I have to include an STLink header and reset/boot buttons, but thats fine.   

<img width="1111" height="918" alt="image" src="https://github.com/user-attachments/assets/3d721387-aefa-4342-94c9-428740008a3b" />

Besides that I hook up uart and power and we are good to go. 

<img width="1603" height="1119" alt="image" src="https://github.com/user-attachments/assets/bedc687d-4331-4835-b0e6-c1a58ca6dff7" />

That leaves us with our last schematic page completed.

Kinda.
I determined that, unfortunately, I cannot do this as one big PCB like I had originally intended. That's chill, I just have to break off the keyboard, which I can do. So, the keyboard is now its own board. That means adding a usbc port to the keyboard for internal wiring. 

<img width="1606" height="1136" alt="image" src="https://github.com/user-attachments/assets/d4850766-bc53-4da7-89b2-eb862da8d042" />

Easy peasy, in the bottom left. It's just usb2.0, so nothing fancy. 

Oh, I also need to add a 3v3 regulator. Huh. I just realized that literally while typing this. Let me grab the one from my deck. 

<img width="1628" height="1128" alt="image" src="https://github.com/user-attachments/assets/0e32dff6-72c0-4509-9bf9-394dd5fc62a2" />

Problem solved. This takes 5v from VBUS and converts it down to 3v3 for everything else. Simple fix.
The cool thing about having my keyboard be a separate board is I could make the keyboard detachable. Will I do that? Not sure. It would be pretty neat though. 

This also meant adding another internal USBA port for the keyboard wire. While I was at it, since I have extra usb lines, I added another external USBA port aswell. 

<img width="1612" height="1132" alt="image" src="https://github.com/user-attachments/assets/7553a945-de71-4d34-8947-9b78ee76b3a4" />

I organized the schematic a little while I was at it to make things look nice. I now have 3 internal USBA, 2 external, and 2 external USBC. That's 7 ports total, 4 of which are accessible externally. The 3 internal are reserved for an rtl-sdr, the keyboard, and powering the screen. 

I actually went through and organized all the unorganized sheets! It looks pretty nice now. 

<img width="1603" height="1117" alt="image" src="https://github.com/user-attachments/assets/8a437655-4ee1-4d6b-a487-d04af6c55b54" />

<img width="1599" height="1126" alt="image" src="https://github.com/user-attachments/assets/5e4ed394-df83-4205-bbef-77e39afe6a24" />

There's USBC Ports 1 and 2. 

<img width="1597" height="1136" alt="image" src="https://github.com/user-attachments/assets/1a0912e2-a3a0-4c58-8741-22f0e1210a29" />

There's Ethernet.

<img width="1591" height="1124" alt="image" src="https://github.com/user-attachments/assets/5a8a9a70-8487-495c-88c8-73d0c2c9cd7c" />

There's storage and wifi. 

<img width="1606" height="1128" alt="image" src="https://github.com/user-attachments/assets/b01c4195-66a3-43df-bb03-93578ebdcb59" />

And last, but not least, power finalized. I cannot really organize hdmi much because of the way I intertwined all the stuff, but they don't look bad. Well I can split off the ESD diodes like this:

<img width="1606" height="1125" alt="image" src="https://github.com/user-attachments/assets/a806f878-6d41-45e2-80c2-b4f234b4397e" />

Yeah sure, why not. Both pages for hdmi are identical.

That's it. All I have left before I can start the PCB is a bit of spatial planning and double check EVERYTHING. Quadruple checking. You know the drill.

Alright. This is looking good!


WAAAAIIITTT I forgot something.

The LattePanda Mu arrived! This was sponsored by DFRobot - I am so incredibly thankful for them. It's so small. Look at it!

<img width="2160" height="2880" alt="image" src="https://github.com/user-attachments/assets/bcdf4dde-04f8-475f-a4e5-614efcf7778b" />

Man, this project is gonna be awesome. Ok. End of entry. 

# 2026-06-23
**Total time spent: 3 hours**

Today was ethernet and some little touches here and there. Connecting stuff to the lattepanda mu, changing some usba ESD diodes, planning, etc.

First, ethernet. I do not understand ethernet. I tried hard to understand and use a TI IC for it and make my own circuit. I couldn't figure it out. So, I made my own version of a circuit I found on an existing lattepanda mu carrier board - the lattepanda MOKA. https://github.com/piecol/lattepanda_MOKA
It's a pretty cool project and has good examples of some high speed routing and schematic design. 

<img width="1612" height="1095" alt="image" src="https://github.com/user-attachments/assets/ed430b3c-1e1a-4336-9627-8b627742c850" />

This is what the MOKA has for ethernet. 

<img width="1839" height="1300" alt="image" src="https://github.com/user-attachments/assets/320a6500-c857-4126-8538-659a1f9a16f1" />

And this is what I have. Similar, but I made a fair amount of changes. Notably: 
Different ESD diodes, I like the TI ones.
Different AC coupling capacitors, the MOKA used the wrong ones? I guess it wasn't an issue but they are supposed to be 220nf, not 100nf. Straight from lattepanda/intel, so I know it's right. 
DIfferent resistors to match datasheets better. 
Different crystal.
Different pin names of course.

I think that's it? And I made everything bigger and easier to see. At their core, the circuits are similar, but I made a fair amount of changes. Call it a *Reference*. 
Oh, I also hooked up 3v3 VCC to the ethernet port. For some reason the moka just has a capacitor straight to ground?? Which is a silly mistake and probably a non issue but could affect the performance of your magnetics (if they even get power at all without that... datasheet isn't very helpful.)

So that's ethernet! This will be super nice. I am using an IC that supports ethernet evaluation and fancy debug stuff so I should be able to use this for debugging if I wanted to. Good cyberdeck ability. 

Next.

<img width="1516" height="723" alt="image" src="https://github.com/user-attachments/assets/fd7221a2-bbfa-4692-abc1-c0280e9a5e86" />

I changed my USBA to use the TI ESD diodes from my other pages since they are much nicer than the weird cheapo chinese one that I had before. 

Next again!

Some little touches here and there on wiring. 

<img width="804" height="558" alt="image" src="https://github.com/user-attachments/assets/9dfc2de4-0873-48fc-8c8f-b382b07564c6" />

Look! USB has the keyboard wired in! That's just an example of one little touch. Basically all pin mangement though. Double checking, triple checking, quadruple checking. I will never stop checking. 

Anywho, I also mentioned planning. I am switching to Linux. Planning on windows initially was a little silly but I wanted to be able to use fusion360. I could maybe use proxmox with windows? Depends what ram version I get of the Mu. We will see!
This also means I shouldn't need to run GPS and LORAWAN as USB devices, but rather straight to the mcu over i2c/spi/uart (haven't decided). Technicallyyyy windows supports that but you need ring 0 level drivers which are incredibly annoying to deal with because of security shtuff. So, I just won't deal with it! Linux to the rescue. This is probably the right choice for a lot of reasons I have yet to determine. 

That's all for today. Next is gonna be finally tacking GPS and Lorawan I think? Or maybe I put it off more by doing trackball stuff. Though I may just make the trackball a USB module so that I can move it.... We will see. 
It's funny, I am already past 24 hours of work on this... and I haven't started any routing or cad. None. Zero. 
Yippee. 

# 2026-06-22
**Total time spent: 4 hours**

Howdy again! Second entry for today. I tackled HDMI and... the keyboard! 
First, hdmi. I have two hdmi ports. One on the inside, one the outside. I need a port, protection diodes, and some power stuff/other misc. requirements. 

<img width="1601" height="1122" alt="image" src="https://github.com/user-attachments/assets/3ae4bd7c-80bd-4703-9d90-9cd73eaffdec" />

I started with just getting everything wired up. I followed a reference design from lattepanda to figure out which pin goes where. You can see a bunch of resistors on the data lines - those are an intel requirement for hdmi. They connect to a mosfet that is controlled by the state of the PC (shutdown, sleep, etc). They help with compatibility I think? Idk, it's just a necessity. 
Bottom left is the ESD diodes. They're the same diodes from my usb shenanigans - they are advertised to work for both. 

<img width="1607" height="1123" alt="image" src="https://github.com/user-attachments/assets/44bf09a1-6f79-4b44-8edd-d960862c9542" />

Next was I2C. The HDMI side needs 5v while the lattepanda side needs 3v3. So, I use a logic converter or whatever it's called. This just converts cleanly between two reference voltages, 3v3 and 5v. 

<img width="759" height="405" alt="image" src="https://github.com/user-attachments/assets/01a52ad3-d3a7-4b99-aef3-ace99a47a798" />

When I scrolled down further in the datasheet I saw I needed decoupling and pullup resistors. So, there's that. Oops.

Oh I also forgot to cover the bottom right. That's hot plug detecting. Basically, it lets the intel graphics processor know when the cable has been plugged in or unplugged while the device is on. That way it can reconfigure or whatever. 
5v power for this also has a diode on it so that it can NEVER EVER BACKFEED. EVER. That can destroy stuff!

Anywho, I have two ports, so all of that gets duplicated for round 2. 

<img width="1502" height="919" alt="image" src="https://github.com/user-attachments/assets/94107b9f-c523-4ae5-923f-a256ae1e7dd0" />

The pinout follows the same scheme as the first one despite the different naming. It's still the same thing, just different names. 

<img width="661" height="666" alt="image" src="https://github.com/user-attachments/assets/2c829278-5a55-4aec-9a8a-87c9a15f8d98" />

You can see they follow the same order and everything. 

That's actually all for hdmi for now. Routing will be a different story since they're high speed differential pairs and thus need really fancy treatment... We will get there eventually. I hope.

Next.
Keyboard! I started by making my matrix. 

<img width="1328" height="584" alt="image" src="https://github.com/user-attachments/assets/e68b9dd0-eccc-4828-ae6b-f0138dc0330c" />


I am doing a 60 key 50% choc keyboard, mimmicking this bad boy:

<img width="1328" height="584" alt="image" src="https://github.com/user-attachments/assets/97153bc9-2d90-47bc-98e2-f0495f9a6d81" />

Called the jj50. It doesn't use choc switches but thats ok. It will be hot swappable as I don't like hard soldering keyswitches. Sockets are also dirt cheap. 
Next was mcu. It's going to be a hardwired usb device, but it still needs an mcu to pretend to be that usb device even though its on the same pcb as the rest of the cyberdeck. 
I started with an rp2040. Then asked my brother if he has used one. And he said to use something else. So after a bit of browsing I chose an stm32f072cbt6 for simplicity and ease. Man, was it simple. 

<img width="1850" height="1295" alt="image" src="https://github.com/user-attachments/assets/9f3be723-2bdc-42d1-9357-5cffa4f4f6c1" />

Like, that's it y'all. ESD diodes, reset and boot buttons, and the mcu, which just gets given power, two usb lines, and the keyboard matrix. Nothing else. It was surprisingly simple and I spent so long checking the datasheet to make sure I wasn't doing something wildly wrong. I am not! (To my knowledge).

In just a couple hours we have fully finished the HDMI and keyboard schematic. Unlike the USB I don't have to tune EQ based off my pcb traces, so I can just finish those bad boys right now.
In a days work, we have done USBC, HDMI, and keyboard. Yippee! 

<img width="220" height="364" alt="image" src="https://github.com/user-attachments/assets/cfba1d80-a976-4f5f-a95a-3a8f6536e001" />

Oh, I hit 10 pages on my schematic btw. This is a BIG project. 

## What's left?
We have:
Ethernet
LORAWAN/Meshtastic
Accelerometer and GPS (may remove)
Trackball
Knob
Continuity Checker (may remove, tho it is cool)
And making sure everything is connected to the Mu properly

That's all for schematic! Basically, a bunch of the inputs and some sensors. I may remove the accelerometer and GPS because I am really not sure they will be very useful compared to how annoying they will be. We will see, I need to think about it!

Ta ta, again. I will not be journaling again today. I am sleepy. 



# 2026-06-22
**Total time spent: 2 hours**

Howdy.
I figured out the USB stuff! Turns out, I can just wire any two GPIO pins from the TPS25751 to the flip and enable pins of my usbc mux and then I can assign those roles through programming. Ezpz.
I enabled adaptive EQ, so for the port side of the mux I don't need to config any EQ. I think I will need to config EQ for the host side, but I cannot do that until I know my trace lengths! So it is officially a "later me problem".

<img width="1608" height="1123" alt="image" src="https://github.com/user-attachments/assets/f3fffc14-5cc1-47f0-b955-e208486004af" />

Tada. Usbc port with charging. 
The other usbc port was pretty easy. I used a controller that integrates both power control and the mux into one IC, making my life easier. 

<img width="1612" height="1125" alt="image" src="https://github.com/user-attachments/assets/ca5e44c7-8069-4a38-9b12-18624180d3e4" />

This side doesn't have a charging circuit since as I covered previously, that is surprisingly difficult to pull off. But what it does have is a vbus power switch to hook vbus to my 5v source for downstream mode. That way it can power peripherals like mice and keyboards. My other port should be able to handle that automatically through the tps25751 so I don't need a vbus switch. That's the only difference besides some pin naming between the two schematics! Both have, from left to right, the port (and PD negotiator if present), the TVS diodes, and then the usbc controller/mux/redriver/they're all the same. 

I chose the tps25221 for my vbus controller because it is active both low and high, while I need active low, and because TI has great datasheets. Most of the chips I was finding are only active high, so its nice that this one is both, as my usbc controller outputs active low through the ID pin. This means the pin gets pulled low when the usbc port is connected. 

Anywho, I can move on from usbc for now. Actually USB in its entirety since I already have my USBA 2.0 ports wired too. I am not sure if they are connected to the Mu yet, they might not be, but that is ok since I need to figure some other stuff out first before I go determining pin assigments for certain. For example...

HDMI! That is what I will be tackling in my next entry. I will have two HDMI 2.1 ports, one internal for the built in screen and one external for external screens. As I have mentioned before, I could try to use ribbon for the built in screen, but I don't want to worry about pinout issues. So I will just suffer a little. By default, the lattepanda mu exposes 2 out of the 3 max HDMI ports, which is perfectly what I need. No bios changes needed yet, to my knowledge!

Ta ta for now. Surely hdmi won't be a huge headache, right?

Right?


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



# TurtleGS

LattePanda Mu based cyberdeck intended to act as a rocketry ground station.


# 2026-07-01
**Total time spent: 3 hours**
I routed the second hdmi port and the ethernet port!
HDMI was super straight forward, just a copy of the other one rotated 90 degrees. Did take me a second to remember what I did but I got it done. 

<img width="801" height="826" alt="image" src="https://github.com/user-attachments/assets/61010f73-cc89-4482-9694-33c7e86416c6" />

Next was ethernet. I had no idea what I was doing for this but here we go.
I started connecting the diff pairs of the connector to my ethernet controller. This converts the ethernet junk to PCIE for my Mu to deal with. 

<img width="1562" height="781" alt="image" src="https://github.com/user-attachments/assets/e799f028-d107-4893-ac41-4316513f189c" />

All length matched, impedance matched to 100 ohms, spaced out properly, ESD filtering, etc. All sorts of nonsense.
It was at this point, when I went to wire the passives for my ethernet controller that I realized - hey, this needs 1v0. I don't have 1v0 in my power circuit. 

Time to speedrun a power circuit. Ready... Set... Go!!

I started by pulling up TI WeBench to get a power circuit schematic. I inputed 1v and 500ma and it outputted a couple options. I picked the first one, the cheapest. 

<img width="1252" height="822" alt="image" src="https://github.com/user-attachments/assets/3847c8e9-14a9-4990-910c-9b651fb10323" />

It gave me this. I used it to make: 

<img width="1093" height="861" alt="image" src="https://github.com/user-attachments/assets/9fe3b4c4-1265-4231-aed7-7a252315e32c" />

This. Identical circuit, just some pins moved on the symbol. I threw this onto the Ethernet page because my Power page is full, and because this is gonna be near the ethernet connector as I don't trust the 1v to survive a path across the board. Just in case. 

<img width="1845" height="1292" alt="image" src="https://github.com/user-attachments/assets/aa7cb031-2f59-40db-969e-bf721d9c9d2c" />

After that, I synced the schematic to my pcb, and threw together the circuit on board. 

<img width="1198" height="717" alt="image" src="https://github.com/user-attachments/assets/ffaa36a1-5241-4ef2-9626-91ccb9541e70" />

Stupid simple, so it's stupid reliable. Boom. Power circuit done. 

With that fixed I could go back to adding all the passives and power hookups for my ethernet controller. 

<img width="1246" height="709" alt="image" src="https://github.com/user-attachments/assets/6078627a-e22d-444e-a18a-bc553c60cf28" />

Bam. Done. 
This thing has so many power hookups, it's unreal. 4 different pins get 1v0, 3 different pins get 3v3. That's 7 pins. Wtf. 
It's ok though, I got it done. I did have to route decoupling internally for a couple pins just because they got placed next to high speed pairs for some reason (stupid placement realtek. stupid placement.).
I think if I do this again I will use a TI IC for ethernet because the realtek documentation is mid and 1v0 is stupid. Give me 3v3 and 5v like a sane person.

Anywho, with that done, all the ports on the board are wired. Yup. All of them. HUGE progress. But - what do I have left? 

m.2 slots
CMOS battery
fans
gps
lorawan

final power routing (5v and 3v3)
final signal routing (hook it all up to the mu)

That's it! Only 7 things on my checklist. M.2 slots are more diff pairs because of course they are, but after that it's smooth sailing stupid logic until the final signal routing. And even then, that is partially stupid logic too (high speed = smart logic, low speed = stupid logic). 
I guess I'll tackle M.2 next? TBD. 

# 2026-06-30
**Total time spent: 2.5 hours**

I routed 1 hdmi port and 4 usba 2.0 ports! I am making solid progress!
First was hdmi. This required defining a new differential pair rule for 100 ohms instead of 90 ohms (usb spec). Easy enough.

<img width="907" height="618" alt="image" src="https://github.com/user-attachments/assets/70229b4f-0785-4f53-ab12-59d885f71b66" />

That's the rule. Anywho.
HDMI has a pretty interesting circuit on it actually. When the device goes to sleep or shuts off, it pulls all the hdmi lines to gnd with a mosfet. I always thought the way computers did sleep for hdmi was something way fancier but no. It literally just shunts the signal to gnd. Pretty neat in my opinion!

<img width="962" height="328" alt="image" src="https://github.com/user-attachments/assets/ecb87611-e90d-4d63-82bd-bf26cd816c17" />

That's the big circuit with resistors on the left. All into a mosfet that triggers with sleep or shutdown. The sls_s0 operates that way - only active (low) when the device is in use. 

Recreating this was a bit of a hassle honestly and really made me think. But I got it figured out in the end. 

<img width="1191" height="674" alt="image" src="https://github.com/user-attachments/assets/a524c529-5a47-4daf-bdaa-2b772a2c75b9" />

From right to left it goes connector, ESD diodes, resistors, and then ac filtering capacitors. AC filtering capacitors ugh. Doesn't that sound like it should filter out AC? But actually, they work by blocking DC and only letting AC through. I think that should be named DC filtering but idk I ain't no electrician. 
Rant aside, those resistors will get connected to a common line in the signal layer (layer 3) and then to a mosfet. 

<img width="1212" height="708" alt="image" src="https://github.com/user-attachments/assets/daf706bc-9390-4498-ba51-446db710e36d" />

Next was an I2C level shift. The HDMI side of the I2C lines operate on 5 volt while my device side operates on 3 volt. So, I use a little IC to handle the level shift and make sure my data stays clean. 
Did you know there is I2C in HDMI? I sure didn't, but it is fascinating. Basically, the I2C lines are how your device figures things out like screen size, resolution, refresh rate, color profiles, etc. Basically, it is how the device and screen "talk" about what each is capable of and what they need from eachother. Over I2C! I've personally seen 30 foot hdmi cables. You're telling me I2C lines can be 30 feet long without issue?? Wow. Fascinating stuff.

Second rant aside, the next circuit was my hot plug detection and 5v power. This is the last bit.

<img width="976" height="504" alt="image" src="https://github.com/user-attachments/assets/2b29a7bd-598c-4b9d-b889-dbf202567496" />

These just get lumped together because the pins are together. 5V is on the very left with the rectangle thing, that's a diode. Everything to the right of that is hot plug detect. It basically tells my device that a screen was unplugged/plugged in while my device was on so that it knows to reconfigure everything. Pretty neat. 
That's a full hdmi port! It looks like this: 

<img width="1015" height="676" alt="image" src="https://github.com/user-attachments/assets/ff07d4c7-5c54-4603-9c38-a7a4ea8ba69a" />

Next was the USB ports. One is external and the other three are internal. 

<img width="1566" height="610" alt="image" src="https://github.com/user-attachments/assets/5c2a6933-3fc1-4816-aa7a-8864532d8cdb" />

<img width="693" height="357" alt="image" src="https://github.com/user-attachments/assets/95aacc51-eafd-4fe1-a3cc-9178709a2341" />

They are all basically carbon copies save for two of the internal ones sharing an ESD diode module just because they can and I want to cut down on components. They all get ESD filtering on the data pair, plus a fuse and decoupling capacitor on the 5V power line. That way an overcurrent blows the fuse and not my whole board. 

That's it for this entry! I am on a roll today and honestly may not sleep for a while to just keep going. Or I will. I don't know, it's late.
Next is the second HDMI port and the Ethernet port. Not sure which will come first. Probably HDMI since I know how to do it. 


# 2026-06-30
**Total time spent: 3.5 hours**

The second usbc port!
I have decided that differential pairs are my greatest enemy. Truly, this is miserable 

<img width="1182" height="540" alt="image" src="https://github.com/user-attachments/assets/343133f5-5038-4b27-86ec-4c2fa12579c1" />

I worked from the "bottom" up, routing traces through their passives in impedance and length matched pairs.

<img width="1682" height="768" alt="image" src="https://github.com/user-attachments/assets/8d1696b9-6036-445a-a984-be48ad1ea352" />

This worked surprisingly well and I ran into minimal issues. Something cool I was able to do was go back and use open channels on existing ESD diodes rather than adding more. Specifically, that one at the top. It had one pair from the first usbc port, and now has a pair from the second port. Since they both had to be routed around the plug legs like that, they end up in the same spot. 

<img width="1109" height="612" alt="image" src="https://github.com/user-attachments/assets/c63a7480-c7e5-4686-b980-b2a9197ee267" />

RX lines get just resistors while TX lines get capacitors and resistors. That's what the smaller passives are on two of the lines - capacitors. 

Once I had those 4 lines routed, I get to route the less crucial lines. There's still a diff pair in these ones, but since it's usb 2.0, it is way less important where that bad boy goes. It's lower speed. Higher speed = more fragile = more important. 

<img width="1398" height="787" alt="image" src="https://github.com/user-attachments/assets/fe5e06f1-13ba-401d-b757-3c3087af2c71" />

For these routing I had to add the other chips necessary for the port. One for protecting vbus shorts and one for connecting vbus to 5 volt, since this port needs to be a power source, not just a sink. That way it can power other devices. 

<img width="1361" height="691" alt="image" src="https://github.com/user-attachments/assets/8f883e10-9428-4305-bc62-51efce2d4d69" />

This is what those vbus lines look like. They will be connected to 5v ultimately, but I have yet to route 5v. Soon. 

With a couple more traces, I had this USBC port complete. 

<img width="1569" height="798" alt="image" src="https://github.com/user-attachments/assets/25127dcc-110a-4b35-9717-87c8b7cdac29" />

This one is a mess. And it was super tedious as always. But after this I get to route just some nice chill usb 2.0 as a break before I route HDMI. Hdmi has waay more passives on the diff pairs for some reason. HDMI and Ethernet also mean I get to deal with writing new differential pair rules as these have different impedances from USBC. 

Anywho, here's both usbc ports together. 

<img width="818" height="828" alt="image" src="https://github.com/user-attachments/assets/bfc39229-ceda-4ad8-a4ce-73c19459c1bb" />

It's really getting busy in here! Can't wait to add more.


On the logistics side of things, I got approved for super hardware builder for Outpost aswell as got my flight stipends approved. That means I am going to Open Sauce/Outpost! I need to finish this project by July 7 to truly "qualify", though if I already have flights...
I'm not worried, I'll get it done in time. 


# 2026-06-29 
**Total time spent: 4.5 hours**

Power circuitry and some usbc 3.2.

So. High speed routing is probably what I hate most in the world right now. It's so annoying, it's all diff pairs, you gotta be so careful. Bleh. Impedance and length matching. Bleh. 
Regardless, I started routing the very first usbc port just because I have to work around it's traces. 

<img width="1281" height="727" alt="image" src="https://github.com/user-attachments/assets/c1968b9f-6393-4038-a358-5aaa0e6867a9" />

This took an hour. This. It's 8 trace routes. Although because of how easyeda works it's split into like 16 different configured differential pairs. Each one needs the rules assigned to tell it spacing and width. Then I had to actually route them. TX lines are supposed to get their passives closer to the USBC port while RX lines get it closer to the device, but both kinda got it in the middle because of how the spacing works out. Oh, and they all get piped through ESD diodes. Then I had to length match them within like, 0.1mm. Blehh.

<img width="1675" height="508" alt="image" src="https://github.com/user-attachments/assets/eb2fc6a0-594a-4394-91e7-38bd7f743c8f" />

<img width="1701" height="504" alt="image" src="https://github.com/user-attachments/assets/4e5e2774-1a13-49ef-958f-28afea20870e" />

After that I worked more on power circuitry. I needed to route I2C lines within the power circuitry so that the ICs could talk to eachother, as well as add more passives that I had forgotten. Mostly decoupling. I also shifted the VBUS line down so I would have room to wire 5v and 3v3 to all the ICs crammed in there. Before that they were all kinda blocked on my power plane, which defeats the point of a power plane. 

More on usbc. I had to change nets (pins) a bunch in my schematic to get this to work because I needed them to line up with the physical pin and component locations so I wasn't crossing paths or anything.
While doing this, I noticed something. My USBA 3.2 port was doubled up on pins, sharing with my first USBC port. That's not right...
Turns out, I had made a big oopsie. By default the lattepanda mu only exposes 2 usb3.2 paths. Exposing more would require messing up my hdmi or ethernet. And that doesn't work. So, I actually had to delete my usba 3.2 port! That's fine, I have two usbc and a usba still, the usba is just 2.0. But, be honest, who actually needs usba 3.2 when you have usbc? Worst case I plug in a hub. 

<img width="1348" height="927" alt="image" src="https://github.com/user-attachments/assets/d43be22e-e8b3-4915-b257-b1979b4fefb7" />

<img width="1195" height="1160" alt="image" src="https://github.com/user-attachments/assets/18d97ca6-caa7-4659-87bb-46447e3d19ca" />

This gives me a decent bit more space and actually makes my life easier. Yay.

Anywho, I think the power circuitry and first usbc port are actually done? Ish. I need to route 3v3 and 5v still within the power circuit, and I need to connect the USBC port to the Lattepanda Mu. But those will come later as there are a lot of other components those work around. 

I spent way too long figuring out how to do high speed traces. These usbc traces are all matched to 90 ohms, all length matched within their pairs, all spaced out, connected to ESD diodes, their passives, aaaaah. There is so much to keep track of. 

<img width="1302" height="774" alt="image" src="https://github.com/user-attachments/assets/2f43c5f4-c1d8-4a84-a1ef-2a97876a1e9d" />

This is the net rule menu for the usbc traces. There are 12 more not pictured, roughly. Each one gets assigned the "usbc" rule I wrote, which just defines spacing and width. I believe spacing and width are like 0.254mm and 0.155mm? Something like that, I know the width I said is right. Spacing is less important as it's just "make pairs close, make not pairs far". 

I think. This is all just my interpretation and it could be wrong. We won't really find out for sure until I try to use the board!

Regardless, I made some serious progress and am getting excited for this to be done! I think I can use this to qualify for OutPost! Hopefully. I really want to go to open sauce again. 

Next I think is the second USBC port and starting the last of the power routing. Yay. 


# 2026-06-29
**Total time spent: 4 hours**

Power circuitry!

I wasn't sure where to start and I certainly didn't want to start with any highspeed routing. So, we are doing the power circuit first.
I started with simply just... trying to get everything connected and compact. This would prove harder than I anticipated. 

<img width="905" height="839" alt="image" src="https://github.com/user-attachments/assets/8c054dba-9393-4c1f-8f06-c51fafbf33eb" />

It's a big mess honestly. I was working without ratlines so that I can actually see what I am doing, and that means I forgot some components. So it actually just got worse for a bit. 

<img width="1305" height="976" alt="image" src="https://github.com/user-attachments/assets/64fd7153-ef35-4882-ad66-eb31b8e40073" />

The big thick trace is 12v main power. This is up to like, 35 watts, so it needs to be super big. Hence the 3mm wide trace. We have roughly:
Bottom is 5v. Top middle is 12v. Top right is battery charger. 
You can kinda see the ICs buried in there. Those are the focal points. 

The very first thing I did to make this better was move 5v. I realized there's no reason it needs to be so smushed in there. 

<img width="1414" height="757" alt="image" src="https://github.com/user-attachments/assets/caa290a6-c086-4608-852b-9462ce8c8f81" />

So, it gets scooted over to the left. Much better. Zooming in on 5v...

<img width="939" height="682" alt="image" src="https://github.com/user-attachments/assets/d164d8e1-309e-47af-8f1e-4d4da62b6e5b" />

<img width="848" height="630" alt="image" src="https://github.com/user-attachments/assets/5f676b04-87dc-42e5-829b-66435a67768d" />

I did some rearranging to make it less wonky and added the components I had forgotten - mainly decoupling caps. I am really good at forgetting decoupling caps. Since 5v is the main power for the system besides 12v for the mu, it gets a big big decoupling cap. That's the big thing on the left. 
There are actually three ICs in here. One is the 5v buck converter, one is the 5v load switch for the system, and one is the 3v3 LDO. I completely forgot 3v3 is in there! Not that it's easy to tell. 

<img width="1377" height="752" alt="image" src="https://github.com/user-attachments/assets/16fdb059-b176-4373-98e3-217ee2ea2c1b" />

This is the current state of the power circuitry. We have from left to right: 5v and 3v3, 12v, battery charger, and charger EEPROM. Plus the battery and thermistor connectors. 

<img width="1318" height="781" alt="image" src="https://github.com/user-attachments/assets/693a08c3-53e2-4a53-879c-cce8a1e31a09" />

Here is a view of the internal power layer. I used a mix of wide traces and copper pours for this so that I can guarantee my traces are wide enough. I also used multiple vias for connections in a lot of spots to give more "wire" for the power to go over. It's way less of a mess now than when we started, and it was annoying to get to this point. I am still not sure if I am super happy with it, but I think it is good enough for now. I still need to wire up the i2c control, but that requires the USBC PD controller be wired. And I haven't gotten that far yet. I think that will be next? Though I may need to route some high speed traces first so I know what I am working around. Not totally sure, I'll probably ask one of my smarter PCB friends what to do. I am gonna get this done, whether through perserverance or straight up spite. 


# 2026-06-27
**Total Time Spent: 1.5 hour**

Beginning of the PCB. The big one. And planning. 
Okay. So. 

I imported all the pcb stuff and dear lord is there a lot of components. 

<img width="1710" height="1019" alt="image" src="https://github.com/user-attachments/assets/c3e9c6c3-2d2c-482f-b995-fbfdf562cd30" />

So. What's the first step? Sort it! 

I sorted everything by schematic page and then within the pages by section/component. That way I know what capacitors and resistors belong to what just by glancing at my design. 
During this sorting, I felt that my battery stuff was looking off. There was an excess of components and some weird duplicates. Turns out, I had accidentally copied a couple things that Ti included in their reference circuit that exist only for internal Ti debug. AKA, they are useless to me. So I went and removed those. Extra programming headers mainly, with some resistors paired with em. 

<img width="1104" height="765" alt="image" src="https://github.com/user-attachments/assets/c1e2e1e1-84e7-4788-814f-f909b9a3ef51" />

It's kinda goofy how big of a spot the battery connector gets but its ok. 

Anywho, with that out of the way, I now had a sorted design.   

<img width="1708" height="1025" alt="image" src="https://github.com/user-attachments/assets/92bfc69d-fdb5-4cbd-9cb3-af12c30d0f01" />

You can see the clusters of components and whatnot.
Looking at it now, I realize that one of the usbc ports should be by the power circuity (bottom right) since it is part of that.

<img width="1622" height="988" alt="image" src="https://github.com/user-attachments/assets/86bd63a3-14c1-48d7-aead-7d5baceb3857" />

Not that you can read the words, but you can see stuff moved. 

<img width="1558" height="955" alt="image" src="https://github.com/user-attachments/assets/8693eedf-dad1-4528-b79e-60f10e368b36" />

Here it is with ratlines by the way. Completely unreadable.

Anywho, game plan. I can't quite figure out sizing but I can kinda let the pcb decide that. This is gonna be a bit of a thick boi but that's alright. I think the mu will be on the back of the deck, not the front, as I can put a vent back there. And then the screen can go on the other side. I have a few requirements though:
Aim for a max width of 230mm. I have a lot more space in Y than X (x is the 230mm) to expand if needed, because of the screen. 
External ports on the right if possible.
Antennas up top if possible. 
Internal ports on the bottom if possible. 
Mu somewhere in the middle! 

That's my plan for now. Obviously, I can change where things end up to fit my format better. But I think that plan should work out. I will need some usb male to female cables to make routing everything work I think. Mostly for the internal rtl-sdr. 

PCB Time! 

# 2026-06-27
**Total Time Spent: 1 hour**

CAD Break! I decided to toss the keyboard pcb in cad so I could get a full mockup with the switches and just get a general vibe for the scale. 

<img width="1652" height="839" alt="image" src="https://github.com/user-attachments/assets/88c4ea31-dfd0-4190-b2a5-31c6dd6e2a56" />

<img width="2066" height="1079" alt="image" src="https://github.com/user-attachments/assets/68baef09-cb6d-45f2-a32c-642b0c0c145b" />

It turned out really good! I just did it with a simple rectangular pattern, the hardest part was lining up everything initially. The keycaps are an open source design I found called MOTE. https://www.printables.com/model/864126-mote-choc-low-profile-flat-keycaps
I really like the flat look. On the real thing I will probably get legended keycaps but I don't think I will cad that. I mean maybe I could if I feel like placing a ton of decals (I don't). Maybe I'll get bored.

I actually had to swap out the boot and reset switches 3 seperate times while I was trying to find something low profile enough to fit under the top plate. This will have a solid plate over it after all and I can't have that hitting the buttons. The original ones were way too tall. The second revision was better but still too tall. So for the third I got the literal lowest profile I could find. These bad boys are half a millimeter tall. 

<img width="1438" height="890" alt="image" src="https://github.com/user-attachments/assets/2a3108c4-5d08-4888-b9c4-baac3c82f38a" />

They should work though! Was only mildly tedious to export the 3d file, make changes, export again, make changes, export again, make changes, and export for the last time. Just typing that was annoying. But I did it for the love of the game. 

I really need to stop procrastinating on the mu board. But, I might do some general cad of the whole deck first. Not sure. TBD.

# 2026-06-26
**Total Time Spent: 2.5 hours**

Keyboard 2 - Electric Boogaloo. 

So. I fiugred out that my footprint for the keyswitch sockets was wrong as it did not contain the necessary mounting holes for the switches. This meant I had to redo the keyboard.

I took this opportunity to also move the brains up top. This makes the keyboard way less wide, allowing me to make the whole deck less wide. Hooray.

I also took progress photos this time.

<img width="1833" height="645" alt="image" src="https://github.com/user-attachments/assets/c2c9c415-44c5-4bf0-a790-4a0437efd790" />

I started by organizing the key grid. Because this is an ortholinear keyboard, it really is just a basic grid. Each diode got put in a little slot at the top, where there is a cutout in the key switch. This is important to avoid collisions of the keys with the diodes.

Next I wired up the rows and columns. This is always the most time consuming part. 

<img width="1608" height="719" alt="image" src="https://github.com/user-attachments/assets/bfceb8d8-c9e3-4405-9ef1-91e6f27f0f74" />

<img width="1633" height="685" alt="image" src="https://github.com/user-attachments/assets/20c52229-088b-4eb4-8b76-2e254c12487e" />

Once I had the grid done, I decided to do something I don't normally do. I adjusted the pin locations to match the physical layout, allowing me to make a MUCH better looking circuit. It went through a few iterations. 

<img width="1929" height="266" alt="image" src="https://github.com/user-attachments/assets/5302e8da-065f-4b0f-82ec-1035fcd32998" />

<img width="669" height="229" alt="image" src="https://github.com/user-attachments/assets/29c18173-9a2d-4e0f-b94a-7a0bc6ba0ced" />

Here are two of them. The second looks way better. 
The columns all connect to the sides while the rows attach to roughly the bottom right. This makes the whole circuit very well organized and creates "wings" of sort out of wiring for the columns. I love it. 

Next was the brains! I challenged myself to make this as compact as posSIble - good practice for when I have to do the Mu board. 

<img width="1658" height="423" alt="image" src="https://github.com/user-attachments/assets/a3bd8acb-e432-468f-980d-ef93f416c326" />

It looks pretty great! Left to right it goes USB, 3v3 LDO, MCU, and then debug switches. 

<img width="1272" height="689" alt="image" src="https://github.com/user-attachments/assets/f894cff7-8162-4303-9863-6d131e54e375" />

Here is a close up of the usb section. It is so compact!

This board is 2 layer too. If I can get this compact on 2 layer, imagine what I can do with 4 or 6 (aka the mu board).

I really didn't want to redo this board but I am very glad I did. It is way narrower, at 228.5mm wide instead of 260mm, allowing me to keep this deck small. Specifically, small enough to fit structural parts on my 3D printers without splitting them up. Because it's way cooler to not have ugly seams/splits!

I am getting really excited for this project. The keyboard board is so good. 


# 2026-06-26 
**Total time spent: 2.5 hours**

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
**Total time spent: 3.5 hours**                

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
**Total time spent: 4.5 hours**

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
**Total time spent: 4 hours**

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
**Total time spent: 4.5 hours**

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
**Total time spent: 3 hours**

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
**Total time spent: 4 hours**

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



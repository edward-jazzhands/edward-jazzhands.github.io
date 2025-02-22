---
layout: post
title: Building a home NAS array with salvaged parts
cover-img: /assets/img/1-building-a-home-nas/cover.jpg
thumbnail-img: /assets/1-building-a-home-nas/thumb.jpg
share-img: /assets/img/1-building-a-home-nas/cover.jpg
tags:  [TrueNAS, DIY, HomeLab, Refurbishing]
comments: true
---

At my place of employment, I was lucky to acquire an old and monstrous computer case that was being thrown in the trash.

The computer itself had no value - it was some ancient server from 20 years ago. However, it immediately drew my eyes because of the case. I've been into refurbishing computers for basically my entire life, so I have a good sense of what parts can be valuable. There was something quite peculiar about this case - It had a 4-drive HDD bay built into it.

![Please excuse how the NAS array is already removed. I took this photo after finishing](/assets/img/1-building-a-home-nas/0_case_front.jpg) ![Here you can see the bay with the front of the case removed](/assets/img/1-building-a-home-nas/1_case_front.jpg)

I did not yet know what it was or if I could use it, but I knew that I had to have it. After getting it home and taking the case apart, I realized that I could slide out the entire HDD bay:

![Sliding out the bay](/assets/img/1-building-a-home-nas/3_slide_out.jpg) ![Bay removed from case](/assets/img/1-building-a-home-nas/4_removed.jpg)

Now we're getting somewhere. However, I still didn't know what I was dealing with. So I started taking photos of the PCB and microchips, hoping that I could get some info on the company and what type of bay I had on my hands.

![Inside of the bay](/assets/img/1-building-a-home-nas/7_inside.jpg) ![Megawin chip closeup](/assets/img/1-building-a-home-nas/chip_closeup.jpg)

I eventually learned, through a bit of research that Chenbro is a respected manufacturer of server equipment, and Megawin makes normal SATA controllers. This is good news - it's essentially just verifying that all I need is a way to connect it to my computer, and it should work perfectly. And hopefully I should be able to set up some RAID, as well.

But how does it plug in? That's the key piece of information that took me a bit. The back has an odd connector, and it came with a funny cable:

![Back of the bay](/assets/img/1-building-a-home-nas/8_back.jpg) ![Hi-Res stock photo of the cable](/assets/img/1-building-a-home-nas/SAS-Cable-4-Blue.jpg)

After a bunch of googling, I realized that what I was looking at is called "SAS", or Serial-Attached SCSI. Without boring you with details, just know these are the industry standard for connecting a hard drive bay to a server, generally through some kind of PCI-e card. There's several types of SAS - the one on this bay is called SFF-8087. Luckily however, I have the cable that came with the bay, since it was inside the computer, and it has four SATA connectors on the other end. Simple enough - So all I need is four SATA ports, and the bay is usable! Granted, the cable will need to run from the HDD bay to inside the computer, but I can work with that. (There might be a way to avoid this with adapters, but I don't know for sure, and I didn't want to get sidetracked.)

Now, the SAS to SATA cable is not that long, so I want these 4 SATA ports to be close to the back of my PC. This means plugging them directly into the motherboard is not an option (And besides, there's not enough SATA ports on the motherboard). This means I have to make an expense to make this work: A PCI-e SATA expansion card. Now I'd never dabbled in RAID before so I was genuinely not sure if it's better to buy a RAID controller or if I should do software RAID. After a few hours of digging into this subject, I learned that RAID controller cards are recently being phased out in favor of software RAID except for extreme use cases like datacenters. Nowadays, the CPU in your home PC is so fast that any kind of consumer grade RAID controller card just can't compete with it. It's better to just let the CPU handle it, especially if you are going to be using a Linux-based NAS OS that comes with a really good software RAID implementation.

This is good news for the budget-conscious person. This means that you actually want to avoid purchasing a card with a built-in RAID controller. That suits us since they're far more expensive. A nice passive card goes for about 30 bucks on Amazon. I purchased this one here: [Amazon link](https://www.amazon.ca/dp/B08KTFRFZW?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_3) (Not an affiliate link). There's numerous no-name brands but many seem to use the same ASMedia chipset, which is well known and widely supported. So that's reassuring.

Next I need a computer. I have a couple extra old SFF PCs in my house, so my ultimate goal through all of this was to set up one of those fancy "Home Lab" servers that I'd heard so much stuff about. People also sometimes use mini PCs or Raspberry Pis for this. In my case, a skinny office tower with an i7-4770 and 16GB of ram that's been sitting in my closet shall finally be put to good use. I figured that if I am going to have an external NAS array, then I should do it properly and have a dedicated NAS PC. Then I could store all my data on a RAID array like a real professional. 

I ended up going with TrueNAS, and I do not regret that decision. The details of that will be for another article (Which I will insert here when I write it). The important information is that I installed TrueNAS on the PC, I installed the SATA expansion card, and it detected and worked automatically with no issues. TrueNAS seemed to come with the right drivers for the ASMedia chipset. I then plugged the NAS Bay into the card Alright, we're off to the races now.[^1] (Please ignore the bending of the bracket in the following photo. It's terrible, but I mistakenly purchased a card that didn't come with a low-profile bracket... don't make that mistake.)

![This is only somewhat janky](/assets/img/1-building-a-home-nas/12_sata_card.jpg)

[^1]: (It must be noted - when I tested this card in my windows 10 PC, it did not work perfectly. It could only connect 2 HDDs at a time. I attempted to download drivers but that didn't seem to solve the issue. However, when used in my other computer running TrueNAS, it immediately worked perfectly without any additional drivers. So take that as you will. I didn't test the card extensively in other computers, and perhaps Linux support is just better. The Amazon comments seem to suggest a few others had a similar problem.)

### Construction time

Time to throw some wood on this bad boy:

![Wood added successfully](/assets/img/1-building-a-home-nas/9_add_wood.jpg)

Of course it's gonna need a power supply. I've got just the thing - a PSU that I ripped out of a busted AiO PC. This has been sitting in my closet a while. I knew it would come in handy some day!

![This PSU has finally found its purpose](/assets/img/1-building-a-home-nas/10_power.jpg)

Since I'm using a dedicated PSU for the bay, I need a way to turn it on. Computer PSUs can be turned on manually by shorting 2 wires (green and black, 4th and 5th from bottom right with clip facing you). This can be achieved easily with a paperclip. Usually this is only done for testing power power supplies, but I don't need to turn this on and off. Once it's on, it's meant to stay on 24/7. I will eventually be purchasing some wires and buttons to do this better. But for now, this paperclip is jammed in nice and tight. (There's no current running through it, by the way, it's just a control signal. So it's not dangerous even though it looks janky.)

![I swear this is perfectly safe](/assets/img/1-building-a-home-nas/11_power_clip.jpg)

I also knew that I needed some cooling. When it was inside the old server case, the NAS bay had its own fan. With the wood panels on the side, the box is gonna be trapping all the heat inside. But it is designed for air to pass over the drives, and there's vents on the front panel. So I needed to improvise a bit. Turns out, it's nothing that a couple well-placed zip-ties cannot solve (Note the SAS cable has to go up under the fan to plug in.) I tested it with the fan turned off later on and it made an enormous difference - Hard drive temps of 30C with the fan, and 60C without it. That small bit of air passing over the drives makes a huge difference. Especially with the wooden panels, the bay traps all the heat inside, so the fan is extra important here.

![Zip-tying the fan onto the back](/assets/img/1-building-a-home-nas/13_fan.jpg)

### Up and Running

Time to get some hard drives hooked up. I happened to have a bunch of HDDs in my spare parts closet (among many other things... all from refurbishing old computers). I think it doesn't look terrible once its running with the LEDs lit up. I'll eventually do something about that power cable but it's fine for now:

![The lights are what sell it](/assets/img/1-building-a-home-nas/cover.jpg)

For the record, HDDs can be had on used stuff marketplaces for very cheap, often 10 or 15 bucks each, and I have acquired several that way as well. They work fine. Another way to get drives is bulk / lot sales on Ebay. However you choose to acquire them, just know that hard drives, like most machines, will usually keep running if they've been running for the last 10 years or so. The odds are not very high that it's suddenly gonna crap out on you after you buy it. If you're getting it for very cheap, then it's not a big deal anyway. That's what the RAID array solves! If one of them dies, you have data redundancy. Just pop the dead drive out of the bay, stick in a new drive of equal size, and TrueNAS will automatically handle the rest.

At long last, I get to boot up and try out TrueNAS. Here in my setup you can see I've got two 1TB drives in a RAID 1 setup (mirroring), and the other two drives just store my Steam games so they're not using RAID:

![TrueNAS up and running!](/assets/img/1-building-a-home-nas/TrueNAS.jpg)

And that's it! If you're wondering about the whole Steam games being on the NAS array - that will be the topic of another article. (I will link that here when I finish it). For now, I hope you enjoyed reading this and maybe learned something from it.








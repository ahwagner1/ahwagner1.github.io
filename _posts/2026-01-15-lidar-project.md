---
layout: default
title: "UART"
---

# Lidar Project
The goal for this project is to use a lidar sensor to build a classic military style radar display.
You know the one, the classic quadrant radar sweeping back and forth with dots where items are.

Something like this:

![Radar Screen](/assets/images/lidar-inspiration.png)

#### Topics:
- [Background and Motivaiton](#background-and-motivation)
- [Plans](#plans)
- [Topics Covered](#what-i-need-to-learn)
- [Build](#build)

## Background and Motivation
Now looking at the commits and post history here, it might seem a bit odd that I am building this project right now.
The most complex thing in embedded I've built so far is bare-metal UART with interrupts. 
So isn't this a bit advanced for my current skill?

Probably...

But at the same time, I'm a firm beliver that building things is the best way to learn. 

Up to this point, I've just been going down a checklist list of things I should learn.
- Blinking an LED with for-loop based timer? Check
- Use GPIO to control LED state? Check
- UART one way Tx? Check
- UART two way Tx/Rx with interrupts? Check
- ... and so on

And there's nothing wrong with this. It's been a good way to get introduced to bare-metal embedded programming.
But I just don't feel like I'm making the progress I want to.

I like setting lofty goals and figuring out as I go along. 
I feel that I learn much better when thrown to the deep-end and forced to figure things out.

That's why I'm building this project. 
You'll see below that the plans I have come up with are advanced for my current skill level. 
But they are achievable. I don't think each of the topics I'll need to learn are too hard that I'll quit.

## Plans
This project revolves around using a lidar sensor, mounted on some sort of swiveling mechanism to scan a 90 degree quadrant.
Then this data will be displated to a screen. 

This should result in a "radar" display that updates live when objects are planec and removed from the scan area.

So how do I plan on accomplishing this. Lets dive in.

### Lidar sensor
This is the heart of the project. For this project I am going to be using a [TF-Luna](https://en.benewake.com/TFLuna/index.html) Lidar sensor. 

This sensor stood out for two main reasons:
1. Price
- It's ~$25US on Amazon and AliExpress right now so I could buy a couple without breaking the bank
2. It supports UART *and* I2C
- For the current plan, UART will work nicely. 
In the future, I sort of want to chain multiple of these sensors together and I2C will be nice for that

### Mount
For this, I plan to use a cheap servo motor I have laying around (somewhere). 
According to the datasheet, it supports up to 180 degrees of sweep which is plenty for my needs.

I'll probably 3d print some small tray that holds the TF-Luna sensor and attaches to the servo.

### MCU
This project will use one of my STM32F401RE Nucleo boards as the brains.
It's the only MCU I have and it should be plenty capable based on my rough napkin math.

### Display
This one was probably the hardest choice.
Most small hobbiest screens for embedded programming are tiny. 
To make the radar smooth and semi-professional looking, I wanted something with some decent resolution.

Looking on Mouser and Digikey, LCD screens were too expensive.

I then debated using the 32x32 or 64x32 LED matrix displays from Adafruit.

But then it hit me, why not VGA?

VGA is extremely will documented. On paper, the pinout and protocol are not hard to understand. 
Old school VGA monitors are also stupid cheap and easy to come by.

To me, this seems like a no-brainer.
I can get a $10 VGA monitor from Facebook or Goodwill and have access to higher resolutions than 
I could with the LED displays.

My only concern is if the MCU can handle VGA, UART, and the servo control all at the same time.

### Connecting it all together
So how do I plan on making this work?

I mentioned before that I was worried if the STM32 could handle VGA on top of the UART and Servo control.

If I was doing standard UART polling, I think it might be a problem. 

Like if the work flow looked like:
Interrupt signals UART byte -> Read byte -> Store in memory -> Repeat

I could see a continous scan from the sensor eating up a good chunk of compute.

So how to resolve this?

I think the answer is DMA. According to the reference manual, 
I can hook up the UART stream from the lidar sensor directly to memory without needing any CPU work.

This should leave the CPU plenty of time for working on rendering the VGA output as well as controlling the servo.

### What I need to learn
I mentioned earlier that this is probably a bit advanced for where I'm at right now.

Well here are all the new topics I'll need to learn from scratch:
- UART 
    - I know UART comm, but I'll need to learn how to write a reliable driver for the sensor comm protocol
- DMA
    - I need to learn how to stream data to memory safely and make sure the CPU can access the correct data
- Timers
    - I'll need timers for servo control, and VGA timings
- PWM
    - I'll need to learn PWM to control the server motor
- VGA
    - I need to learn the VGA protocol for rendering my radar
- I2C
    - This is more of a stretch goal for adding more sensors to the project when done with one sensor

Quite a lot to get through as a newbie. 
But I already have done a quick browse through the reference manual and datasheet and I think this is entirely doable.
None of these topics seem too complex to handle with my current knowledge.

I think this is the beauty of building projects to learn things. 

If I had just been going down the checklist, doing small exercises, learning all of these topics would have taken forever.

Also, I'm a believer that applying topics to real problems is a great way to learn. 
It really forces you to understand how the entire problem works.
By building this project, I'm willing to bet that my knowledge of each of the topics above will be much stronger than if I had just learned these by doing exercises going down the checklist.

# Build
.. still building ..
## Jingle bells, jing... Eeh... Na, let's test LoRa.

2022-12-28

*"Those who restrain desire do so because theirs is weak enough to be restrained; and the restrainer or reason usurps its place & governs the unwilling."*

*William Blake*

![lora](/static/images/lora/l0.png)

The best thing about the Christmas holidays is the time you get to work on your weekend projects which normally are pushed to the back of the queue because of time constraints, despite my (absurdly) bohemian lifestyle. Either way, about a year ago I got an idea: a low-powered, single board computer that would be completely off the grid and can be turned on, controlled and turned off remotely, without any modern creature comforts: cellular network, internet connection, potentially even off the power grid(through a power bank or a solar panel). The powering on and powering off is relatively straightforward: microcontroller and a relay: I have half a dozen of those laying around. However, as it turns out, triggering such actions remotely without the standard creature comforts is not a trivial task. Even more so when you take into account the fact that I want to be able to have some form of shell. I initially tried a simple POC using a 433MhZ FM transmitters/receivers. While that did work, this created more problems than it solved: painfully slow and providing considerably less range than a regular ESP32 WiFi. LoRa had been on my radar for several years but I had never really bothered with it all that much: while the idea is somewhat different, it was often portrayed as a decentralization all-in-one solution for IoT. I have worked on projects/in companies pushing for decentralization and I'll be honest, I can't help but roll my eyes when I hear the word "decentralization": works on paper but not in practice. However the more I looked into it, the more it seemed like a good candidate even though it comes with a range of limitations:

* The size of the payloads is microscopical - 256 bytes per packet.
* There is no multiplexing.
* If you go a step further with LoRaWAN, you have to take the [fair usage policy](https://www.thethingsnetwork.org/docs/lorawan/duty-cycle/) into account.
* Given the small payload and that a microcontroller will be used, an advanced encryption like AES is possible but I would not be able to cram all the other features I want into the limited memory.

However the fact that a single chip can in theory give a range of 10KM, albeit in "ideal conditions" sounded like science fiction, considering how little power LoRa requires. And while I did think about building a custom PCB to my needs, I figured that it would be way too time consuming and also a bit outside my comfort zone that is software. Luckily I stumbled upon [LoRa32 V2.1_1.6](https://www.lilygo.cc/en-bg/products/lora3). Even though I'm still in the process of making a pcb to connect it more gracefully to a raspberry pi zero, the module offers a lot more than I realistically need: on top of the ESP32 and a LoRa module, it comes with a display and a micro SD card reader. My main concern was the lack of multiplexing and how busy were those frequencies. I listened in on them in several large cities, Berlin being the largest one, and to my surprise, there was a complete radio silence.

![l2](/static/images/lora/l2.png)

This is not a selected period in time, it looks like that pretty much all the time so the lack of multiplexing would not be an issue for the time being.

With that out of the way, I decided to split that effort into two groups: your regular urban environment and the (somewhat) "ideal conditions". Disclaimer: I have a slightly unfair advantage compared to most people: while I live in a large city, my flat is located at one of the highest areas and on the last floor. And if that's not enough, a door from my bedroom takes me straight to the roof of the building. So if anyone wants to give that same thing a go, your mileage may vary.

### Test prep

I wanted to keep this as simple as possible. In a nutshell I have two Lilygo modules: one which I'd carry around with me and another listening from the roof. Whenever I send a message from the one I'm carrying around, the other would reply back, granted the signal gets through.


### The urban environment test.

While both the tests I did were completely improvised, this one was completely half-assed by all standards. Because I had no meaningful way to monitor what is being received, I figured the best thing I could do is to put my laptop on the roof and SSH into it while I'm walking around the neighborhood with my phone and the other module and pinging messages.

![urban](/static/images/lora/l1.jpg)

I didn't take exact measurements initially since it seems there was a clear pattern: Up to 500 meters, there were no losses whatsoever, regardless of obstructions: buildings, trees, cars, nothing seemed to make a difference. Beyond that I started seeing some deterioration: large buildings started to have an effect, although not always, I suspect the signal was bouncing off enough to get picked up again. At 1 kilometer, is where things turned ugly: anything beyond that point was lost forever. Which was kind of disappointing, considering the 10 kilometer "ideal conditions" claim. Although I've spent a good chunk of my life in this area, I looked at the map and found an interesting spot: Almost two kilometers away but with a direct line of sight and so I went for it. To my surprise, messages started getting through. Not only that but there were no losses and all messages I sent, I could see on the SSH terminal and I could see them coming back to me as well from the module on the roof.

This called for a much more elaborate test: one I had to think through a bit more.

### The ideal scenario test.

While I was aware that having a direct line of sight was important, I feel like I somewhat underestimated the importance a bit. The first idea I had was to add another module to the equation, put it up on a drone and create a mesh: a module on the roof, another 2-300 meters up in the air and one in my hand: the one I have with me sends a signal, the one on the drone acts as a proxy and relays messages back and forth. But after giving some more thought, I steered away from that idea: adds complexity which is not a great policy for experiments/POC. Also I scrapped the replying part in the firmware since it only adds overhead and I can access the serial monitor over SSH. That said, I wanted to take my laptop with me. Problem: I can't provide power to a desktop computer on the roof so I had to figure out an alternative. That of course is how the idea came to life: a raspberry pi zero and for the purpose of the experiment: a power bank. But that brought a whole set of other problems and even more half-assed solutions:

1. The raspberry pi zero could connect to my WiFi but it was painfully slow and unreliable. Solution? Old smartphone with a data sim card.
2. Problem number two: no static IP to use to SSH into the raspberry pi. Solution? Ngrok.
3. Raspberry pi zero has a 32 bit an ARM cpu. Solution? Compile ngrok for ARM.
4. Headles raspberry pi, hard to see the host and port provided by Ngrok. Solution? Boot script that waits for a WiFi connection, starts Ngrok and then uses the Ngrok http API to determine the host and port and send me a text message.

I kinda wanna mention something: Signal is by far the best messaging app. Not for it's UI or special features, but for it's security. However implementing anything with it is a nightmare and a hack. I hate Telegram with a passion, I truly do but credit where it's due: creating a bot takes seconds.

![signal is better](/static/images/lora/l7.png)

Got a bit side-tracked here. Anyway, once the prep work was done, it was back to the roof. A POC is not a POC if you did not use duct tape. And I'm happy to say that duct tape came into play.

![duct tape](/static/images/lora/l2.jpg)
![duct tape](/static/images/lora/l8.jpg)

Several climbs to the roof later and I was ready to go on a small roadtrip - toss the laptop and some hardware(and some tea) in the back of the car and it was go time. After an hour of traffic jams and having to plow through some snow in the mountain to get around some cars that had spun and blocked the roads, I got to my destination. Mind you there were a few things I did not consider. There weren't all that many trees in the way. Sure, there were branches but I figured that their effect would be negligible. However nothing was going through so I sat down and did the math, taking the height of my building + the additional 2.5 meters up the lightning rod where I had put the Lilygo module. Also taking my current location + the elevation, and the all-beloved a^2 + b^2 = c^2 and it turned out I was a bit out of range for the "ideal conditions": just over 11.5 kilometers. I sent out a few more messages but nothing came through. Then as I was about to start packing and head home, I remembered what the purpose of the building right behind me was. It was a long shot, but I plugged in the SDR to check what was going on. Well... There was some interference.

![ffff1](/static/images/lora/l6.jpg)
![fffff](/static/images/lora/l3.png)

Alright, fair, so let's move the Lilygo module a bit just to make sure. And with some additional help from the duct tape:

![up](/static/images/lora/l9.jpg)

![up](/static/images/lora/l4.png)

Yes, the message is malformed, but it was a partial success. Again, as I was willing to call it a partial success, pack up and head back home, I realized there was an important part I had completely overlooked: there was an additional balcony behind the TV tower which in should in principle provide a direct line of sight. Since I was already there, I figured I have nothing to lose and give it a go. Worst case scenario, my theory is incorrect.

![ds](/static/images/lora/l7.jpg)

Well to my surprise, my theory was not incorrect. As a matter of fact, this was the definitive winner: All messages I sent from there made it through with hardly any delays.

![l6](/static/images/lora/l6.png)

The bottom line is, the Lilygo module and LoRa in particular really do cover the requirements I have for my evil hack device... Did I say that out loud? Oh well... I won't tell if you don't.

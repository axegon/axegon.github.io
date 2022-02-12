## The  ultimate macro keyboard part 1

2022-02-12

*"It is certain that the inanimate objects by which you are surrounded have a direct action on the brain."*

*Jules Verne*

![fox](/static/images/fox.jpg)

To be clear, I am not a gamer by any stretch of the imagination, which may be seen as somewhat ironic, when you take into consideration the fact that I worked for a gaming company for almost 9 years. Yet there is something extremely cool about macro keyboards, even if you are not a gamer, but a developer, designer or anything else. And I've had plenty of thoughts around them and I also spent quite a lot of time on Amazon hunting for one. But none of them offered exactly what I wanted. On one hand most of them came with mechanical buttons, which I have always hated with a passion. I really can't see anything appealing in a typewriter from the Soviet Union - half a kilometre travel distance at every keystroke or the absurdly annoying sound. Since the pandemic started, I've really stepped up my home setup game and for the first time I have a desktop computer since the mid 2000's. Well make that two as a matter of fact but that's a different story. So let's take a step back: I am 98% Linux user(2% representing Windows since I failed to get Fusion 360 running smoothly on Linux, even with 2x 14-core Xeons, 64GB of ram and a Quadro M5000. Nowhere nearly as smooth as a quad-core i7 Intel nuc(albeit the Enthusiast kit, which punches way above it's weight). But with that in mind, I heavily rely on many Linux utilities and getting a third party macro keyboard to do what I want it to do would have been yet another pain in the ass. For instance, gpaste is an integral part of my life, and even though the gnome extension is very helpful, it still adds additional work when you want to switch to something you copied even 20 seconds earlier. After pondering a bit, I came up with a list of features I wanted:

* Customizable macros - record your keyboard inputs over a period of time, and then repeat them at the flick of a button. Pretty helpful feature if you want to re-format a file without resorting to AWK for instance - add a new line, replace some value, add a comma or any other trivial action.
* I already mentioned the clipboard history.
* Media control: although my keyboard has designated buttons for volume, skipping a song, pausing and all that jazz, I use the same keyboard for both my desktop computers, while most of the media only runs on my workstation - that removes the necessity to switch back and forth.
* Web navigation: similar to the point above, a bit more trivial considering my mouse does have additional buttons dedicated to it, but doesn't hurt to add it.
* Notifications: when you have 4 monitors, it's pretty easy to miss a notification in the corner on one of the screens. A glowing LED in front of you however is a different story.

I figured that I have several ESP32's laying around and they seemed like a great candidate: plenty of memory, connectivity options, good chunk of GPIO's as well. And this is where I made my first mistake - I didn't bother to read the datasheet. I spent countless hours digging through the libraries in order to figure out why I wasn't able to compile anything because of the bluetooth library. The vast majority of ESP32's come with a Bluetooth module(which I in fact intended to use).

![esp32s2](/static/images/esp32s2.jpg)

The ESP32-S2 however does not. Facepalm numero uno. Going over the other boards I have, I stumbled upon something even better: The ESP32 from Heltec, which on top of having a Bluetooth, has LoRa(which I won't use) and also a tiny 0.96 inch OLED display, which in terns meant that I wouldn't need to do any additional work in order to hook up an I2C display to the project. Happy days! Another problem: though it also comes with plenty of GPIO's, the headers were not soldered. Even more annoyingly, my soldering iron is one I got for ~5 euros around 10 years ago and the ribbon cable was less than a millimetre away from the headers.

![esp32heltec](/static/images/esp32heltec.jpg)

But despite all that I'm happy to say that this was far from my worst soldering job, even upon close inspection. A fact further proved after a test with a multimeter.

![soldered-headers](/static/images/sheaders.png)

And here things start getting really tricky: the question of security: Opening up such a device freely isn't the wisest of ideas, so requiring a PIN is a pretty good idea. So obviously line up some buttons on a breadboard with a 10k ohm resistors. 12 in total for the time being: 10 digits, backspace and enter. 

![keyboard-breadboard](/static/images/keyboardbreadboard.jpg)

I can immediately tell what you are thinking, and I completely agree: 2 shift registers and I'd free up most of the GPIO which I currently use and I could add another 6 buttons. That of course would imply that I have 2 shift registers laying around, which unfortunetaly I don't. So this will be pushed back to the second part of the project. For the time being, having those working as it is, is good enough. Also my jumper cable collection is somewhat limited so colours jump a bit here and there.

![keyboard-pt1](/static/images/keyboardpt1.gif)

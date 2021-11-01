## The state of embedded Rust for your average enthusiast.

2021-11-01

*"To fall in love is to create a religion that has a fallible god"*

*Jorge Luis Borges*

![Rusty rust](/static/images/rust_embedded_head.jpeg)


I rarely say this about anything(or anyone for that matter) but I love Rust. And to be perfectly honest my entire history with Rust has been a classic case of how love manifests in my life: It starts off with "What manner of horrible things has led to the conception of this monstrosity?", to "Hey, this isn't entirely horrible!", to "I can't live without this!". That has been my precise experience with Rust. Now to get the obvious out of the way: I am not an embedded developer. Frankly the closest thing I've come to work on in my professional career as a developer was to write a Linux driver for a fiscal printer at an old job. And as bad as this might sound, the lack of documentation and support made things considerably worse and ultimately felt closer to brute forcing than actual development(in addition to the countless weeks of swearing, sweat and tears).

In any case, embedded development as an enthusiast is an incredibly fun hobby, especially if you are a physics nerd, which I most certainly am. I have tried multiple off-the-shelf options and despite it's many limitations, the Arduino ecosystem still holds the crown. I have tried Circuitpython and TinyGo(disclosure: I hate both of them). Having said that, using Arduino as a base line for comparison with Rust might be a bit unfair but I really can't think of a better alternative. And here's the thing: The Arduino world is an oversimplified(and to a fair degree butchered) C++ blob. So when I first realized that Rust can be used for embedded development, I was excited like never before. To me, Rust is the young, attractive, smart and lovable sibling of C++ with none of the drawbacks... Well you might get a slap in the face while you are peacefully sleeping but most of us are willing to live with that. So as excited as I was, I dived heads first into the deep unknown of embedded Rust. And what a ride this has been... Sadly not in a good way.

You see, the great thing about Arduino is how easy it is to get started - you pick up an Arduino Uno from anywhere, download the IDE, plug in the USB and... That's it, you're good to go. In similar fashion, I picked up an STM32, [STM32F303VCT6](https://www.st.com/en/microcontrollers-microprocessors/stm32f303vc.html) to be exact, since most of the documentation and examples were written around STM32. In all honesty I had never tried STM32 before and I was really excited about it. On paper it's a huge step up from an Arduino - more memory, an ARM cortex-m processor, absurd amount of gpio, built in accelerometer, gyroscope, magnetometer and an array of LED's baked onto the board for just 14 bucks. Sure, your average ESP32 for comparison offers WiFi and Bluetooth in addition to even better specs(minus the sensors and LED's), but the STM32 certainly beats the Arduino. So what is wrong about all this? Well, as I said, getting started with an Arduino takes less than a minute. Embedded Rust with the most popular option for development took me an entire day to setup. Some of this can be attributed to the fact that I don't rock the most popular Linux distro out there, but even if I were, that wouldn't have made a world of difference. Even if you closely follow the guides from the [Embedded Rust book](https://docs.rust-embedded.org/book/), you need to nail it perfectly: toolchain, gdb, openocd, the correct memory region information for the specific board you are using, the specific rust libraries that support your board, the device permissions and so on: Everything must be \_\_ABSOLUTELY\_\_ perfect. And more often than not, if you won't get it right and the outcome won't be as evident as you might expect it to be. Everything compiles and just as you thought that all that's left is to flash your device, boom:

```
Loading section .vector_table, size 0x400 lma 0x0
Loading section .text, size 0x1134 lma 0x400
Load failed
```

Wait, what?!?!?!? In this particular instance it turned out that I had my memory region information set up wrong and I had to dive into the boards specifications in order to find the correct one. And sure enough, this was far from the only issue I encountered and all that just to get the simple blinking "Hello world" example. Now make no mistake, the reason why the Arduinos are so easy to work with is the fact that all of this is abstracted away from you by the IDE which does it for you, granted that you've selected the correct board and port. And to my mind, this is the one thing holding embedded Rust back - the insanely steep entry, which is made worse by the fact that you are pretty much at starting point as soon as you pick up a different board, even if the differences are practically invisible. This all came up as a discussion on Hacker News some time ago and I openly expressed my frustration. A reply I got that really caught my attention was this:

"I’m not paid to tinker with a setup, I’m paid to make embedded products that solve customer problems."

So I recon that it's not only my frustration as an enthusiast embedded developer, it is people who do embedded development for a living as well. I want Rust to succeed in the embedded world, I really do. You have a clever(albeit slow) compiler, you have a picture-perfect built-in linter and even without the standard library, you have dependency management. It has all the ingredients to be the biggest leap in embedded development and the only thing that is truly missing is the one thing that put Arduino on everyone's radar: an crazy dumb way to get everyone interested on board. You can have absolutely no previous experience with development as a whole and I bet that you would be able to learn how to read data from sensors, manipulate an i2c interface display(which, fun fact, was for a personal project I was developing, but that's a story for another time but tl;dr - best you don't invite troll developers to your wedding) or work with a servo motor in a weekend with minimal help from anyone. On the other hand, I am willing to bet a large portion of a crowd of truly experienced developers would give up after being stuck with a setup for half a day and industrial quantities of coffee. And while I don't intend to turn my back on embedded rust, I certainly hope this won't end up being yet another toxic relationship and ultimately "I wish I had never met you". Only time will tell...

![iBlinky Blink](/static/images/rust_blinks.gif)

### Interesting links

* [Embedded Rust book](https://docs.rust-embedded.org/book/)
* [Awesome embedded Rust](https://github.com/rust-embedded/awesome-embedded-rust)
* [Not Yet Awesome Embedded Rust](https://github.com/rust-embedded/not-yet-awesome-embedded-rust)
* [An Overview of the Embedded Rust Ecosystem](youtube.com/watch?v=vLYit_HHPaY)
* [Insights from Writing an Embedded IoT App (Almost) Entirely in Rust](https://embeddedartistry.com/blog/2020/09/21/insights-from-writing-an-embedded-iot-app-almost-entirely-in-rust/)
* [Discord server with a dedicated embedded channel with lots of knowledgeable individuals](https://discord.gg/rust-lang-community)

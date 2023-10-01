## To photograph a lightning: Toiling tirelessly.

2023-10-01

*"At the next stop I am off and ahead in the darkness"*

*Jack London*

![Pew pew](/static/images/tape_ftw.jpeg)

Every now and then, I find myself bored in the evenings, just scrolling mindlessly through HackerNews and Reddit. Last week, I had this plan to take a few days off work and leave the city for a day or two. Needless to say, something had to happen to ruin those plans. Specifically, the car’s AC decided to break down. So, my mindless scrolling led me to the weather forecast, and it showed thunderstorms were on their way. I’ve taken photos of lightning before, and it’s pretty straightforward. But it is annoying and tedious. So why spend 15 minutes doing something manually, when you can waste hours automating it? Admittedly, since I got my first Pixel phone back in 2018, I haven’t bothered to improve my DSLR skills. I’ve had no use for my DSLR since, and it’s just been collecting dust. Mind you, as great as Pixel cameras are, there are specific cases where they simply can’t compete, although this is largely a software rather than a hardware issue. But, developing apps isn’t my thing: I dislike the JVM, Java, and even though Kotlin is somewhat pleasant, it’s still reliant on the horrid JVM. Flutter is good for simple apps, but whenever something computationally intensive is needed, you end up using some lower-level language and the FFI, which is also annoying. So, hacking the DSLR seemed simpler.

Capturing lightning requires some prep and manual work. I’m not a photographer, but I think capturing lightning at night is easier: you use long exposure and tweak the ISO depending on the lighting conditions—street lights, cars, reflections, and such. Once you have it figured out, open the shutter, grab a coffee, and wait. When lightning strikes, close the shutter. Then repeat and hopefully, you nail it.

The DSLR I have gathering dust is the Nikon D3400. This leaves me with two options. The first one is using libgphoto2 and a single board computer. While this offers more options and flexibility, managing an operating system and peripherals seems like overkill. This brings me to option two: microcontrollers. First things first, we need a way to interface with the camera. The Nikon D3400 has an infrared sensor, which is what most (if not all) off-the-shelf remotes use to interface with it.

![Nikon D3400 IR sensor](/static/images/ir_camera_sensor.png)


Which means the first question that needs answering is how  does the signal from a remote control look like? Bad news is I don't have a IR remote or an oscilloscope(and in all fairness I really wish I had the latter). Luckily people have already done that a long time ago. [The earliest mention](https://bayesianadventures.wordpress.com/2013/08/09/nikon-ml-l3-ir-remote-hack/) I could find dates back to 2013. At first glance, it is pretty simple and simply a question of timing:

![IR camera signal](/static/images/ir_signal.png)
    
I won't go into details here, the code is really straightforward.

![IR breadboard_test](/static/images/ir_remote_beadboard.png)

Naturally, there’s a hitch: the protocol is one-way, meaning the microcontroller won’t receive any feedback. So, if things don’t go as planned, this could turn into endless brute-forcing, just hoping something works. This is something I’m pretty familiar with, having written a driver for a fiscal printer at my first job with what could only be described as terrible documentation (and that’s putting it mildly). Anyway, it just takes a few lines of code to test if the shutter triggers, and then flashing the code to the ESP32.

```
#include <Arduino.h>

#define IRpin 6

void toggle(int cycles, int final_delay = 0) {
  for(int i = 0; i < cycles; i++) {
    digitalWrite(IRpin, HIGH);
    delayMicroseconds(7);
    digitalWrite(IRpin, LOW);
    delayMicroseconds(7);
  }
  delayMicroseconds(final_delay);
}

void sendSignal(void) {
    toggle(76, 27810);
    toggle(16, 1540);
    toggle(16, 3545);
    toggle(16);
}

void setup() {
  Serial.begin(115200);
  pinMode(IRpin, OUTPUT);
}

void loop() {
  delay(6000);
  Serial.println("Sending signal...");
  sendSignal();
  delay(600);
}
```

If something works first try, chances are there is a catastrophic bug somewhere hidden which will bite your head off long long after you have any recollection of what you did:

![IR breadboard_test](/static/images/ir_first_test.gif)

But with all those things set, the only piece left is to get the microcontroller to detect lightning. This brings me to my favorite principle, [KISS](https://en.wikipedia.org/wiki/KISS_principle): keep it simple with a photoresistor. Photoresistors are little wonders: basically, they’re passive electrical components that change resistance depending on the light they receive. And because they’re passive, there are no performance penalties—you get the info you need on the spot. The straightforward approach would be to constantly read the resistance, compare it to the last reading, and see if there’s a change. But this can have its own set of problems, especially in a city setting, like if you’re up high in a building (like me). If a nearby neighbor flips on a light, it can change the resistance, and there goes your shot. So, once again, I find myself solving a hardware problem with software.

This is where the unpredictable elements of nature come in. There’s no exact way to determine how long a lightning flash will last as it depends on many factors. However, it’s generally accepted that the average duration of a lightning flash is around 30 microseconds. So, a little cleverness is needed here. So, to start, the photoresistor:

You can grab a standard photoresistor at any local electronics shop for a few cents, and pair it with a 2k ohm resistor. In this case, we’re reading the values from pin 8. And after tweaking the original firmware a bit:

```https://mod.reddit.com/mail/all/1c590q 
#include <Arduino.h>

#define PHOTORESISTOR_PIN 8

void setup() {
  Serial.begin(115200);
}

void loop() {
  int resistance_val = analogRead(PHOTORESISTOR_PIN);
  Serial.print("Resistance: ");
  Serial.println(resistance_val);
  delay(500);
}
```

![Photoresistor test](/static/images/photoresistor_test.gif)

![Photoresistor readouts](/static/images/photoresistor_readout.png)

You can easily see when the lights were turned on and off, with the remaining light emitting from my monitor, varying due to a video playing in one corner. This is where a little cunning is necessary: Light from a passing car or a neighbor flicking on their lights will show instantly. Commercial lightning detectors typically leave this problem for the user to solve by providing a sensitivity setting. Some achieve this through a potentiometer, others by applying a factor in the software. However, our goal here diverges; often, there are secondary lightning strikes quickly following the first one.

So, the approach here is to, once the microcontroller is powered on, use the first 100 readings as a baseline to get a rough feel for the environment. It’s not flawless, but it’s a decent makeshift solution when you’ve only got about an hour to spare. From there, any value increase that is 42% above the baseline can likely be considered lightning. We then wait for 4 readings close to the baseline, serving as a clear signal that the event has concluded, and the shutter can be closed. If you are wondering why 42, please get out. Finally, to conclude the code:


```
#include <Arduino.h>

#define IRpin 6
#define PHOTORESISTOR_PIN A0

class CameraRemoteController {
  int readoutCount;
  float baseline;
  int normalizedCount;
  int aboveBaselineCount;
public:
  CameraRemoteController() : readoutCount(0), baseline(0), normalizedCount(0), aboveBaselineCount(0) {}

  void toggle(int cycles, int final_delay = 0) {
    for(int i = 0; i < cycles; i++) {
      digitalWrite(IRpin, HIGH);
      delayMicroseconds(7);
      digitalWrite(IRpin, LOW);
      delayMicroseconds(7);
    }
    delayMicroseconds(final_delay);
  }

  void sendSignal(void) {
    toggle(76, 27810);
    toggle(16, 1540);
    toggle(16, 3545);
    toggle(16);
  }

  void processReadout(int readout) {
    if (readoutCount < 10) {
      baseline += readout;
      readoutCount++;
      if (readoutCount == 10) {
        baseline /= 10.0;
        Serial.print("Baseline: ");
        Serial.println(baseline);
      }
      return;
    }
    
    Serial.print("Resistance: ");
    Serial.println(readout);

    if (readout > baseline * 1.42) {
      if (normalizedCount >= 4) {
        sendSignal();
        normalizedCount = 0;
      }
      aboveBaselineCount++;
    } else {
      if (aboveBaselineCount > 0) normalizedCount++;
    }
  }
};

CameraRemoteController controller;

void setup() {
  Serial.begin(115200);
  pinMode(IRpin, OUTPUT);
}

void loop() {
  int resistance_val = analogRead(PHOTORESISTOR_PIN);
  controller.processReadout(resistance_val);
  delay(10);
}
```

And just seconds before the storm hits, electrical tape becomes the hero of the day (I wish I had duct tape for the meme, but oh well). Surprisingly, it worked way better than I had anticipated. Maybe I’ll design a 3D printed case for it someday... Oh and the actual result:

![real pew pew](/static/images/lightning.png)

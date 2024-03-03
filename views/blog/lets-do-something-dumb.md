## Let's do something dumb!

2024-03-04

*"A limited number of types, good and bad serve for all ages"*

*Romain Rolland*

![too lazy...](/static/images/eclipse.png)

For whatever reason, I felt like doing something stupid today. Over the past year or so, my stack of development boards has been shrinking a lot: home automation, weird devices, drones, UAV's or simple, one-off, half-assed, dirty hacks to get something going. I have ordered a few but they are still on their way so I have to deal with what I have laying around. Which left me with 3 options: An Arduino Uno, which I was never a fan of to begin with: I hate the IDE, the ATmega328P is very underpowered and despite the large amounts of documentation, I really don't enjoy working with them. In addition to this, the Uno is being reserved for a CNC project I've been postponing for god knows how long. The second option is a dev board bt Lilygo. The problem there is Lilygo themselves: while the products are great in terms of features, the documentation and examples are nothing short of crap. Which left me with option number ESP32 Wroom chip and a programming board. And I should mention that I don't have SD card modules laying around either so this is even more half-assed than I would have wanted.

![that's the one](/static/images/thats-it.png)

To give an idea of what I have in mind, I need to take a step back and talk about Brainfuck. For as long as I can remember, I always considered it a meme language. And as far as I'm concerned, it is. Looking at Brainfuck code, it does feel like something that's meant to make people's eyes bleed out but it is Turing-complete. It is a bit bland though: no functions, no variables, just walking up and down 30,000 bytes and doing simple operations. So in the spirit of doing somethig stupid, let's make a Brainfuck interpreter, that runs on an embedded chip. And I know...

![sh_chat](/static/images/chat.png)

The language itself is stupid simple with only 8 instructions: moving the pointer left and right, using `<` and `>` respectively, increment the value of the byte where the pointer is at with either `+` or `-`, produce output with `.`, loops using `[]` and reading input using `,`. There are minimal interpreters out there that are literally one-liners but that feels like cheating a bit. Given that this will run on ESP32 Wroom with the Arduino Framework, I settled on something which is a bit more readable than those one liners, even though it's still painful to read:


```
const int MEMORY_SIZE = 30000;

byte memory[MEMORY_SIZE] = {0};
byte* ptr = memory;

void brainfuckInterpreter(String code) {
  int codeLength = code.length();
  for (int i = 0; i < codeLength; i++) {
    switch (code[i]) {
      case '>':
        ++ptr;
        break;
      case '<':
        --ptr;
        break;
      case '+':
        ++(*ptr);
        break;
      case '-':
        --(*ptr);
        break;
      case '.':
        Serial.write(*ptr);
        break;
      case ',':
        if (Serial.available() > 0) {
          *ptr = Serial.read();
        }
        break;
      case '[':
        if (!*ptr) {
          int loop = 1;
          while (loop > 0) {
            i++;
            if (code[i] == '[') loop++;
            if (code[i] == ']') loop--;
          }
        }
        break;
      case ']':
        if (*ptr) {
          int loop = 1;
          while (loop > 0) {
            i--;
            if (code[i] == '[') loop--;
            if (code[i] == ']') loop++;
          }
        }
        break;
    }
  }
}
```

What makes this task a tiny bit easier is the fact that there are *some* creature comforts, such as `String`. As I mentioned, I don't have an SD card reader and the dev kit does not come up with one either. Either way, by the time I got to this point, I got a bit tired/lazy to make a shell for it so I'm leaving it into the firmware. That said, pull requests [are open](https://github.com/axegon/esp32.bf), so...



![well now...](/static/images/well_now.png)

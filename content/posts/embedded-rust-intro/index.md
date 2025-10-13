---
title: "An Introduction to Embedded Rust"
date: 2025-09-10T08:43:07+02:00
draft: false # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: ''
categories:
  - iot
  - rust
  - embedded
tags:
  - embedded
  - rust
---

I've been dabbling into programming embedded devices recently and it's been quite an interesting ride.
I went with Rust as the language of choice for a couple of reasons, mostly influenced by [Steve Klabnik](https://steveklabnik.com/writing) of RoR fame.

I found these books very helpful [for understanding the rust language](https://rust-book.cs.brown.edu/) as well as using it to [program embedded applications](https://www.theembeddedrustacean.com/c/ser-std).

This series documents my understanding of some embedded fundamentals, and could be viewed as an introduction to embedded rust programming using ESP32. (Although the referenced book uses the C3, I used S3, because that was the hardware I bought).

### Objective
In this series of post I'll be discussing the process of building a simplified version of the Whac-a-mole game.
The goal is to measure the player's response time using Timers. The game consists of:

- 🔵 A Blue LED, representing the mole, which radonmly lights up.
  
- A Buton - The player’s objective is to press it as quickly as possible when the blue LED lights up.
  
- 🔴 A Red LED to indicate failure feedback, when the player didn't response or response was too slow.

- 🟢 A Green LED that lights up if the player pressed the button quick enough, in response to the blue led turning on.

Afer every round, the game is restarted.

---

## Step 1 - Wriring things up

### Tools
**Hardware *(Simulated with Wokwi)* **
- ESP32-S3.
- Blue LED to display the prompt for the user to respond('whack').
- Green LED to indicate a success (quick enough response).
- Red LED to indicate a failure (slow/no response).
- Button - user presses this in response to the blue led turning on.


### Setup
Some preliminary steps need to be taken care of:
1. [Install rust and cargo](https://docs.espressif.com/projects/rust/book/installation/index.html)
2. [Install requirements for ESP32-S3 development](https://docs.espressif.com/projects/rust/book/installation/riscv-and-xtensa.html)
3. [Install express-if requirements for std applications](https://docs.espressif.com/projects/rust/book/installation/std-requirements.html)


#### Repo setup
Now that we have that out of the way, we can begin with the project - I'll it `whacky`.

First I need to setup the project repository using [Cargo](https://doc.rust-lang.org/cargo/getting-started/first-steps.html) - Rust's package manager.

1. I'll be using express-if's template for generating embedded rust projects.
 
```
cargo generate esp-rs/esp-idf-template cargo
```
Running this command pulls the template from [the esp-rs repository](https://github.com/esp-rs/esp-idf-template.git). The prompts that follow help to populate custom configuration values for the project.

```🤷 Project Name? › *whacky*```

```🤷 Which MCU to target? › *esp32s3*```

```🤷 Configure advanced template options? › *false*```

After which I get the feedback that the project has been created successfully:
```
🔧   Initializing a fresh Git repository
✨   Done! New project created
```

Now, when I list the files in my dir, I see all the initial files for my project, the result  `ls -R whacky` should look like:

```
build.rs            rust-toolchain.toml src
Cargo.toml          sdkconfig.defaults

whacky/src:
main.rs
```

_💡 typical next step would be to initialize this git repo and point it to a desired remote._

#### Setup **up** hardware simulattion using Wokwi
Now that the repository has been created, I need to set up the project for use with [Wokwi](https://docs.wokwi.com/), which I used for simulating the hardware components for *Whacky*.

1. Install the Wokwi extension for VSCode and get a [free dev license](https://wokwi.com/license).
2. Install [the Wokwi CLI](https://docs.wokwi.com/wokwi-ci/cli-installation) and run `wokwi init` to configure locations for the elf and firmware.
Mine looked something like this:
```
[wokwi]
version = 1
elf = "target/xtensa-esp32s3-espidf/debug/whacky"
firmware = "target/xtensa-esp32s3-espidf/debug/whacky"
```

#### Setting up the simulated hardware components:
Now that I have wokwi simulator set up, I need to define how [the various hardware I specified above](#tools) will communicate with each other.

I `right-click + open-with + text editor` to edit the `src/diagram.json` generated in the step above.
Below is the list of the Wokwi simulated hardware components:

#### The ESP32-S3 board
As I mentioned previously, I bought the ESP32S3 because I needed more capability, as a result I simulated using Wokwi's `board-esp32-s3-devkitc-1`.
The boards houses all the GPIO pins and timers used for this project. 

#### A [button](https://docs.wokwi.com/parts/wokwi-pushbutton)
This serves as the user input. The user pushes the button in response to the display led.

#### Three LEDs [(wokwi-led)](https://docs.wokwi.com/parts/wokwi-led)
I have 3 wokwi-leds in this setup as follows:

A blue LED, named `led-display`. This represents the mole, it gets switched on for a random duration between
500 and 1500 milliseconds.

A green (`led-success`) gets switched on for a few seconds if the user pushed the button before `led-display` got toggled off.

A red (`led-failure` LED that gets turned on for a few seconds if the user failed to push the button before `led-display` toggled off.

_🗣️ It's also worth noting that for each LED, I have a [resistor](https://docs.wokwi.com/parts/wokwi-resistor) that connects the LED's anode (positive pin) to the GPIO pin on the ESP board. The resistors here serve a current-limiting purpose, so as to protect the LEDs from damage due to too much current from the GPIO pin._

The final state of `src/diagram.json` looks like this:
{{< highlight json "linenos=inline">}}
{
  "version": 1,
  "author": "Anosike Osifo",
  "editor": "wokwi",
  "parts": [
    {
      "type": "board-esp32-s3-devkitc-1",
      "id": "esp",
      "top": 50,
      "left": 0,
      "rotate": 90,
      "attrs": { "builder": "rust-std-esp"}
    },
    {
      "type": "wokwi-pushbutton",
      "id": "button",
      "top": 185,
      "left": -225,
      "rotate": 0,
      "attrs": { "color": "brown", "flip": "1" }
    },
    {
      "type": "wokwi-led",
      "id": "led-display",
      "top": 120,
      "left": -220,
      "rotate": -90,
      "attrs": { "color": "blue" }
    },
    {
      "type": "wokwi-resistor",
      "id": "resistor-display",
      "top": 134.45,
      "left": -163.75,
      "rotate": 0,
      "attrs": { "value": "1000" }
    },
    {
      "type": "wokwi-led",
      "id": "led-success",
      "top": 0,
      "left": -55,
      "attrs": { "color": "green" }
    },
    {
      "type": "wokwi-resistor",
      "id": "resistor-success",
      "top": 75,
      "left": -42,
      "rotate": 90,
      "attrs": { "value": "1000" }
    },
    {
      "type": "wokwi-led",
      "id": "led-failure",
      "top": 0,
      "left": 100,
      "attrs": { "color": "red", "flip": "1" }
    },
    {
      "type": "wokwi-resistor",
      "id": "resistor-failure",
      "top": 75,
      "left": 65,
      "rotate": 90,
      "attrs": { "value": "1000" }
    }
  ],
  "connections": [
    [ "button:1.r", "esp:21", "green", [] ],
    [ "button:2.r", "esp:GND.4", "black", [] ],
    
    [ "led-display:A", "resistor-display:1", "blue" ],
    [ "resistor-display:2", "esp:14", "red" ],
    [ "led-display:C", "esp:GND.1", "black" ],

    [ "led-success:A", "resistor-success:1", "green" ],
    [ "led-success:C", "esp:GND.3", "black" ],
    [ "resistor-success:2", "esp:12", "red" ],

    [ "led-failure:A", "resistor-failure:1", "red" ],
    [ "led-failure:C", "esp:GND.2", "black" ],
    [ "resistor-failure:2", "esp:7", "red" ]
  ],
  "serialMonitor": { "display": "terminal" },
  "dependencies": {}
}
{{< /highlight >}}


When the completed diagram.json is opened in default mode (with the wokwi diagram editor), this is what it should look like:
{{< img src="images/wokwi-setup.png" alt="wokwi setup" caption="Wired-up board" class="max-height-img">}}

---

## Step 2 - Programming the hardware.

Now that the input and output components have been wired up onto the ESP32 board, it's time to configure them to run based on the game logic.

The code respository for this project can be found [here](https://github.com/osifo/rust-embedded/tree/main/ser-whack-a-mole)

In the next post in tihs series, I'll be discussing how I got the core logic of the project working:
- working with [Espressif ESP-IDF](https://idf.espressif.com/), a framework for devloping IoT projects on ESP devices.
- High-level of GPIO communication (pins, pin drivers, configuring as input or output, etc)
- Using Timers and Counters
- Inerrupts vs polling
- The Rust paradigms/concepts used while implementing the above.

I hope to share the next post in a few days...
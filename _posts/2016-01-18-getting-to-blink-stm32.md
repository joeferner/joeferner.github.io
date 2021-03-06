---
layout: post
title: "Getting to Blink on a STM32 Nucleo F103RB using STM32CubeMX, Make and GCC"
tags:
  - stm32
  - STM32CubeMX
---

## Prerequisites

* [Install STM32CubeMX](http://joeferner.github.io/2014/07/03/stm32cubemx-linux/)
* Install arm-none-eabi-gcc

## Makefile

Start by creating a directory and copy my [Makefile](https://raw.githubusercontent.com/joeferner/stm32-makefile/master/Makefile) into it.

## project.mk

Create a file called `project.mk` in the root of your new directory with the following content:

{% highlight makefile %}
USER_CFLAGS =
SRCS = src/main.c
SSRCS =
{% endhighlight %}

This file tells the Makefile where your source files are.

## STM32CubeMX

Create a new project in STM32CubeMX that matches your Nucleo board.

![New Project]({{ site.url }}public/2016-01-18-getting-to-blink-stm32/new-project.jpg)

Create a more friendly name for your LED pin. Right click `PA5 (LD2 [Green Led])`, select `Enter User Label`, and enter `MYLED`.

![Pin Name]({{ site.url }}public/2016-01-18-getting-to-blink-stm32/pin-name.jpg)

Save the STM32CubeMX project into your directory.

## main.c

Create a file called `src/main.c` with the following content:

{% highlight c %}
#include <pinout.h> // generated by Makefile to build/pinout.h

volatile GPIO_PinState lastState;

void sleep_ms(uint32_t ms);
void sleep_us(uint32_t us);

void setup() {
  lastState = GPIO_PIN_RESET;
}

void loop() {
  GPIO_PinState newState = lastState == GPIO_PIN_RESET ? GPIO_PIN_SET : GPIO_PIN_RESET;
  HAL_GPIO_WritePin(PIN_MYLED_PORT, PIN_MYLED_PIN, newState);
  lastState = newState;
  sleep_ms(500);
}

void sleep_ms(uint32_t ms) {
  volatile uint32_t i;
  for (i = ms; i != 0; i--) {
    sleep_us(1000);
  }
}

void sleep_us(uint32_t us) {
  volatile uint32_t i;
  for (i = ((SystemCoreClock / 8000000) * us); i != 0; i--) {}
}
{% endhighlight %}

Similar to an Arduino there is a `setup` function which gets called when the program starts. There is also a `loop` function which gets continuouly called until the power is turned off.

## Flash your Nucleo

After you plug your Nucleo board in, run `make write`. You should now see the green LED flash.

## Additional Help

To get additional help run `make help`.

You can find the example project and Makefile [here](https://github.com/joeferner/stm32-makefile).

---
layout: post
title: "Installing STM32CubeMX on Linux"
tags:
  - stm32
  - STM32CubeMX
---

STMicroelectronics has a nice little tool to plan out your pin mapping for their STM32 microcontrollers, called STM32CubeMX, but unfortunately they packaged it in to an .exe file. I first tried running the exe under wine but I was greeted with a message that it requires Java. I know a lot of these installers are really just executable zip files so I just ran unzip on it.

{% highlight bash %}
mkdir stm32cubemx
cd stm32cubemx
unzip ../SetupSTM32CubeMX-4.3.0.exe
{% endhighlight %}

Taking a look at "META-INF/MANIFEST.MF" you'll see it want's to run "com.izforge.izpack.installer.bootstrap.Installer" so I did that next.

{% highlight bash %}
java -cp . com.izforge.izpack.installer.bootstrap.Installer
{% endhighlight %}

Looking at the install directory we find another exe. Lets unzip again. This time in place (zip bomb, but that's ok).

{% highlight bash %}
cd <install location>
unzip STM32CubeMX.exe
{% endhighlight %}

Looking at "META-INF/MANIFEST.MF" we now see "com.st.microxplorer.maingui.IOConfigurator" so I ran that.

{% highlight bash %}
java -cp . com.st.microxplorer.maingui.IOConfigurator
{% endhighlight %}

Seems to work just fine.

![Screenshot]({{ site.url }}public/2014-07-03-stm32cubemx-linux/stm32cubemx-screenshot.jpg)

---
layout: post
title: "Running Intel's flash tool lite for Edison in non-Debian Linux"
tags:
  - linux
  - edison
  - docker
  - usb
  - gui
---

I used to run Ubuntu but I have since moved on to Arch Linix. Unfortunatly people still distribute
software as deb packages or rpms. I also like Docker and I don't like poluting my system. Intel
does have [manual instructions](https://software.intel.com/en-us/flashing-firmware-on-your-intel-edison-board-linux)
to flash the firmware but where's the fun in that.

I decided to take a different route. I went ahead and created
[a Docker image](https://github.com/joeferner/docker-flash-tool-lite) to run the GUI flash tool.
Or if you choose you can also run the command line flash tool in the Docker image as well.

With Docker I was able to act like a Debian system, I was able to forward the X11 window, and I was
able to map my local USB devices.

## Acting like a Debian system

This was the easiest part. Just add `FROM debian` to the top of the
[Dockerfile](https://github.com/joeferner/docker-flash-tool-lite/blob/master/Dockerfile) then
`RUN dpkg -i /phoneflashtoollite_5.2.4.0_linux_x86_64.deb`.

## Forwarding the X11 windows

This took a little more work but nothing really uncharted here and is documented elsewhere. The things
I needed to do for flash tool was add `ENV QT_GRAPHICSSYSTEM native` to the
[Dockerfile](https://github.com/joeferner/docker-flash-tool-lite/blob/master/Dockerfile). Then when running
docker you need a command similar to this:

{% highlight bash %}
docker run --rm -it \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -h flash-tool-lite \
  flash-tool-lite \
  "$@"
{% endhighlight %}

## Giving Docker access to USB

More command line options added to the docker run command. You need to give docker privileged access
so that it can mount host hardware devices using `--privileged=true`. Then you need to map the host's
USB devices into the docker container as a volume `-v /dev/bus/usb:/dev/bus/usb`.

{% highlight bash %}
docker run --rm -it \
  --privileged=true \
  -v /dev/bus/usb:/dev/bus/usb \
  -h flash-tool-lite \
  flash-tool-lite \
  "$@"
{% endhighlight %}

## Conclusion

That's about it. You can view all the code here [docker-flash-tool-lite](https://github.com/joeferner/docker-flash-tool-lite). Finally here are some screen shots.

![Flash in progress]({{ site.url }}public/2015-07-13-intel-flash-tool-lite-linux/flash-in-progress.png)

![Flash success]({{ site.url }}public/2015-07-13-intel-flash-tool-lite-linux/flash-success.png)

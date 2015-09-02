---
layout: post
title: "Dokku First Impressions"
tags:
  - dokku
  - docker
---

My son is into Skylanders and he likes to keep track of his collection. He was using the [Skylanders Android app](https://play.google.com/store/apps/details?id=com.activision.skylanders.collectionvault&hl=en)
but because of a bug it no longer allows him to add Skylanders. So I ended up building him a [replacement](https://github.com/joeferner/skylanders-inventory). 
I needed someplace to deploy it to, and having experience with [DigitalOcean](https://www.digitalocean.com/) I figured that would be a good place.
I also have experience with [Docker](https://www.docker.com/) so I thought that might be nice to use as well since I might want to run a couple other things
on that server. What I felt was missing from my tool belt was a good way to deploy new code and manage my Docker containers.

After doing some googling I ended up with [Dokku](http://progrium.viewdocs.io/dokku/). Dokku is like your own personal [Heroku](https://www.heroku.com/). 
Deployments are done using git pushes to your server and best of all it manages your Docker images and the [nginx](http://wiki.nginx.org/Main) configuration to route
traffic using sub-domains.

I did run into a couple of issues but these were mostly my fault, but tracking them down took some time so I wanted to document them here.

First, and this is really a Docker best practice, is to allow configuration of your application using environment variables. Dokku allows you to set these
variables using `dokku config:set <app> KEY=VALUE` and they get passed to your application. In my application, all I really needed was a data directory so
in my case I set `DATA_DIR` to `/data`. `PORT` is a special case and dokku will automatically set this environment variable when it starts up and it will
expect your web server to be running on this port.

Dokku made mapping the data directory easy as well by allowing you to pass additional arguments into your docker run using `dokku docker-options:add <app> <phase(s)> OPTION`.
The phase threw me off a little in that the `run` phase is not what you want. The `run` phase is used when you want to run something from the command line in your docker
container using `dokku run <app> <cmd>`. What you need is to add the options to both the `run` and `deploy` phases.

Another complication I had was that I was using [TypeScript](http://www.typescriptlang.org/) and [DefinitelyTyped](http://definitelytyped.org/) which required some steps
to run before running the application. I tried stuffing all the commands into the `Procfile` but it didn't like running multiple commands in there. I ended up just creating
a [`dokku.sh` script](https://github.com/joeferner/skylanders-inventory/blob/master/dokku.sh) in the root of my project to boot strap and run the application.
Then in the `Procfile` I just called that shell script.

Finally, make sure your application listens to all interfaces. My node.js application was only listening to localhost which caused nginx to return `502 Bad Gateway`.
After changing expressjs to `app.listen(process.env.PORT || 3000, '0.0.0.0', ...)` it all started up.


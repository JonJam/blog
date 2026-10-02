---
layout: post
title: "My DigitalOcean App Platform Hackathon entry: stock-checker - Part 3: Developing stock-checker app"
tag:
  - golang
  - docker
  - web-automation
  - twilio
excerpt: "How I built the stock-checker app in Go, including the packages, Docker setup and Twilio SMS integration used to monitor Xbox Series X stock."
---

![Part 3: Developing stock-checker app]({{ '/assets/images/2020-12-26-part-3-developing-stock-checker-app-4bnc-5ih9vfz7aw1afriviq0w.png' | relative_url }})

This post is part of a series detailing my journey with [`golang`](https://golang.org/) from learning the language to entering the [DigitalOcean App Platform Hackathon](https://dev.to/devteam/announcing-the-digitalocean-app-platform-hackathon-on-dev-2i1k).

The app I built can be found on [GitHub](https://github.com/JonJam/stock-checker).

Part 3 details:

- `golang` packages used to build stock-checker
- `docker` setup for both development and production
- Integrating with Twilio SMS API

# Overview

stock-checker is a `golang` app that spawns a task every hour to check whether any of the following UK retailers have an Xbox Series X console in stock:

- [Argos](http://argos.co.uk/)
- [Amazon](https://www.amazon.co.uk/)
- [Currys](https://www.currys.co.uk)
- [Game](https://game.co.uk)
- [John Lewis](https://www.johnlewis.com/)
- [Shopto](https://www.shopto.net/)
- [Smyths](https://www.smythstoys.com)

This is done using a web automation library that navigates each site in a headless chromium browser.

If the app determines stock is available, a text message is sent using [Twilio](https://www.twilio.com) to the configured mobile number.

# golang packages

A number of different packages were used to build this app which are detailed in this section.

These packages were discovered on [Awesome Go](https://awesome-go.com).

## Web Automation - go-rod/rod

[`rod`](https://github.com/go-rod/rod) is a high-level driver for the [DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/).

This package is used to navigate the various retailer's sites to find the Xbox Series X console listing. Once found, it checks whether the console is in or out of stock.

Notable features include:

- [Bypass bot detection](https://go-rod.github.io/#/emulation?id=bypass-bot-detection)
- [Custom browser launch](https://go-rod.github.io/#/custom-launch)
- [Concurrent execution using PagePool](https://go-rod.github.io/#/page-pool)

## Configuration - spf13/viper

[`Viper`](https://github.com/spf13/viper) is a configuration package that supports reading configuration files in various formats, as well as from environment variables.

This is being used to control:

- The scheduler interval
- Debug settings for `rod`
- Enables/disables sending notifications and the secrets used to communicate with Twilio.

I used an `envfile` to define the app's configuration. Originally, I tried using a `YAML` file but due to a couple of issues, I had to swap the file format.

To help others, here are the issues I ran into when using `YAML`:

- [GitHub Issue](https://github.com/spf13/viper/issues/1029): If you want to override a `YAML` subobject property with an environment variable, you have to name the variable using the following format: `PREFIX_SECURE.KEY`. [DigitalOcean App Platform environment variables](https://www.digitalocean.com/docs/app-platform/how-to/use-environment-variables/) do not support `.` in the name.
- [GitHub Issue](https://github.com/spf13/viper/issues/1012): If you want to override a `YAML` subobject property with an environment variable and unmarshal that object to a struct, this isn't currently possible. This is because `viper.UnmarshalKey` doesn't take account of environment variables.

## Scheduler - go-co-op/gocron

[`goCron`](https://github.com/go-co-op/gocron) is a scheduling package that lets you run functions periodically at a pre-determined interval.

This is being used to run a task once an hour that checks the various retailers and if applicable, sends a notification.

# docker support

To simplify running the app on [DigitalOcean](https://www.digitalocean.com/), I added [`docker`](https://www.docker.com/) support which this section covers.

## Dockerfile

This amazing [JetBrains blog post series](https://blog.jetbrains.com/go/2020/05/04/go-development-with-docker-containers/) greatly inspired the `Dockerfile` created for this project.

Using [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) it provides both a development image with debugging support and a production image.

```
# Builder
FROM golang:1.15.6-buster AS builder
COPY . /src
WORKDIR /src
RUN go get github.com/go-delve/delve/cmd/dlv
RUN go build -gcflags="all=-N -l" -o app-dev
RUN go build -o app

# Base runner
FROM debian:10.7 AS base-runner
RUN apt-get update && apt-get install -y \
    ca-certificates \
    chromium \
    && rm -rf /var/lib/apt/lists/*

# Dev runner
FROM base-runner AS dev-runner
WORKDIR /server
COPY config.env /server
COPY --from=builder /src/app-dev /server/app
COPY --from=builder /go/bin/dlv /server
EXPOSE 40000
CMD ["/server/dlv", "--listen=:40000", "--headless=true", "--api-version=2", "--accept-multiclient", "exec", "/server/app"]

# Prod runner
FROM base-runner AS prod-runner
WORKDIR /server
COPY config.env /server
COPY --from=builder /src/app /server
CMD ["./app"]
```

## docker-compose

To assist with development, this `docker-compose` file starts a development container with debugging enabled and enables a few `rod` debug options via environment variables.

```
version: "3.9"

services:
  app:
    build:
      context: ./..
      target: dev-runner
    security_opt:
      - seccomp:unconfined
    cap_add:
      - SYS_PTRACE
    container_name: stock-checker-$USER

    ports:
      - "40000:40000" # DEBUG

    environment:
      - SC_ROD_TRACE=true
      - SC_ROD_PAGEPOOLSIZE=1
```

## Remote debugging using VS Code

To remote debug the app within `docker`, we need to create a launch configuration by following this [guide](https://github.com/golang/vscode-go/blob/master/docs/debugging.md#remote-debugging).

The resulting config looks like this:

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch and debug in Docker",
            "type": "go",
            "request": "attach",
            "mode": "remote",
            "cwd": "${workspaceFolder}",
            "port": 40000,
            "host": "127.0.0.1",
            // This corresponds to the directory the app is built in
            "remotePath": "/src"
        },
    ]
}
```

# Integrating with Twilio

[Twilio](https://www.twilio.com) is being used to send a SMS when the app detects a Xbox Series X console is in stock.

Originally, I was going to use [`saintpete/twilio-go`](https://github.com/saintpete/twilio-go) which I discovered [here](https://www.twilio.com/docs/libraries/community-supported-libraries). However I ran into issues with this package due to incompatible package versions.

In the end, I went with a simpler approach and ended up following this [blog post](https://www.twilio.com/blog/2017/09/send-text-messages-golang.html) to integrate with their SMS API directly.

# Next

That's it for this post.

In the final part, I will detail my [DigitalOcean App Platform Hackathon](https://dev.to/devteam/announcing-the-digitalocean-app-platform-hackathon-on-dev-2i1k) submission as well as deploying to DigitalOcean.

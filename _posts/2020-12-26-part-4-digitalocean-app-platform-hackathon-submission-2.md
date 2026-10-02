---
layout: post
title: "My DigitalOcean App Platform Hackathon entry: stock-checker - Part 4: DigitalOcean App Platform Hackathon Submission"
tag:
  - golang
  - digitalocean
  - docker
  - hackathon
excerpt: "The final part of my Go stock-checker journey, covering the DigitalOcean App Platform Hackathon submission and deployment."
---

![Part 4: DigitalOcean App Platform Hackathon Submission](/assets/images/2020-12-26-part-4-digitalocean-app-platform-hackathon-submission-2-445-02tppn0au5sgjupvk0wp.png)

This post is part of a series detailing my journey with [`golang`](https://golang.org/) from learning the language to entering the [DigitalOcean App Platform Hackathon](https://dev.to/devteam/announcing-the-digitalocean-app-platform-hackathon-on-dev-2i1k).

The app I built can be found on [GitHub](https://github.com/JonJam/stock-checker).

Part 4 details the DigitalOcean App Platform Hackathon submission and how it is deployed.

## What I built

### Category Submission

Random Roulette

### App Link

N/A

### Screenshots

![Alt Text](/assets/images/2020-12-26-part-4-digitalocean-app-platform-hackathon-submission-2-445-zexeolf7l0hksklih0gm.png)

### Description

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

This is deployed to a [DigitalOcean App Platform Worker](https://www.digitalocean.com/docs/app-platform/how-to/manage-workers/) using a `Dockerfile`.

### Link to Source Code

[GitHub](https://github.com/JonJam/stock-checker)

### Permissive License

[MIT](https://github.com/JonJam/stock-checker/blob/main/LICENSE)

## Background

See [Part 1](https://dev.to/jonjam/my-golang-journey-part-1-background-and-learning-golang-23ni) of this series.

### How I built it

See [Part 3](https://dev.to/jonjam/part-3-developing-stock-checker-app-4bnc) of this series.

### Hosting and deployment

This `golang` app is deployed using `docker` to a [Worker component](https://www.digitalocean.com/docs/app-platform/how-to/manage-workers/) on [DigitalOcean's App Platform](https://www.digitalocean.com/docs/app-platform/).

The DO App is defined using an [Application Reference specification](https://www.digitalocean.com/docs/app-platform/references/app-specification-reference/) which is shown below:

```
name: stock-checker-app
region: fra
workers:
- name: bg-worker-stock-checker
  github:
    branch: main
    deploy_on_push: true
    repo: JonJam/stock-checker
  dockerfile_path: Dockerfile
  instance_count: 1
  instance_size_slug: basic-xs
  envs:
  - key: SC_TWILIO_ENABLED
    scope: RUN_AND_BUILD_TIME
    value: "true"
  - key: SC_TWILIO_ACCOUNTSID
    scope: RUN_AND_BUILD_TIME
    type: SECRET
    value: ""
  - key: SC_TWILIO_AUTHTOKEN
    scope: RUN_AND_BUILD_TIME
    type: SECRET
    value: ""
  - key: SC_TWILIO_NUMBERTO
    scope: RUN_AND_BUILD_TIME
    type: SECRET
    value: ""
  - key: SC_TWILIO_NUMBERFROM
    scope: RUN_AND_BUILD_TIME
    type: SECRET
    value: ""
```

This App was then deployed using [doctl](https://www.digitalocean.com/docs/apis-clis/doctl/).

Originally I was going to use a [Deploy to DigitalOcean button](https://www.digitalocean.com/docs/app-platform/how-to/add-deploy-do-button/), however worker components are not currently supported.

Prior to this project, I wasn't familiar with Digital Ocean so using the App Platform was new to me. The resources I found useful are listed below:

- [App Platform - Workers](https://www.digitalocean.com/docs/app-platform/how-to/manage-workers/)
- [App Platform - Environment variables](https://www.digitalocean.com/docs/app-platform/how-to/use-environment-variables/)
- [App Platform - App Specification Reference](https://www.digitalocean.com/docs/app-platform/references/app-specification-reference/)
- [App Platform - Tech Talk](https://www.digitalocean.com/community/tech_talks/defining-your-app-specification-on-digitalocean-app-platform)
- [doctl - Reference](https://www.digitalocean.com/docs/apis-clis/doctl/)

### Additional Resources / Info

-

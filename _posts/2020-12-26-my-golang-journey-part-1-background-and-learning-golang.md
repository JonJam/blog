---
layout: post
title: "My DigitalOcean App Platform Hackathon entry: stock-checker - Part 1: Background and learning golang"
tag:
  - golang
  - learning
  - digitalocean
  - hackathon
excerpt: "How I came up with the stock-checker project idea and the resources I used to learn Go before building it."
---

![Part 1: Background and learning golang]({{ '/assets/images/2020-12-26-my-golang-journey-part-1-background-and-learning-golang-23ni-sajjbu8rnbfpx92ejj6i.png' | relative_url }})

> The cover image is from [MariaLetta/free-gophers-pack](https://github.com/MariaLetta/free-gophers-pack)

This post is part of a series detailing my journey with [`golang`](https://golang.org/) from learning the language to entering the [DigitalOcean App Platform Hackathon](https://dev.to/devteam/announcing-the-digitalocean-app-platform-hackathon-on-dev-2i1k).

The app I built can be found on [GitHub](https://github.com/JonJam/stock-checker).

Part 1 details:

- How I came up with the project idea
- How I learnt golang

# Background

In August 2020, I wanted to start a new side project and learn something new.

At the top of my backlog was [`golang`](https://golang.org/).

In the following months, I completed online courses and read blogs, guides, language specification etc to get familiar with the language.

At the beginning of December, I thought it was about time I built something.

At the same time, my youngest brother who had missed getting a [Xbox Series X](https://www.xbox.com/en-GB/consoles/xbox-series-x) on the UK launch day decided that he wanted one. What transpired was every time a UK retailer potentially had a re-stock, my two brothers and I would be checking their website throughout the day to try buy one; only to fail every time.

Then I saw this [DEV post](https://dev.to/marisayou/how-to-get-a-playstation-5-when-it-s-always-out-of-stock-5d4i). This inspired me to build an app in `golang` that would check various UK retailers for a Xbox Series X and notify my brothers and I when available.

Lastly I saw that [DigitalOcean](https://www.digitalocean.com/) were hosting a [Hackathon](https://dev.to/devteam/announcing-the-digitalocean-app-platform-hackathon-on-dev-2i1k). This provided an opportunity to also get experience working with a cloud platform I hadn't used before and more importantly, gave me a deadline to work to.

> This series details the journey of building the stock-checker app:

- Part 1 - Learning `golang` (this one)
- Part 2 - Development environment
- Part 3 - Developing the stock-checker app
- Part 4 - Deploying to DigitalOcean

# Learning golang

How I tend to learn something new is by trying to expose myself to as much information as possible from a variety of sources, even if I can't remember it all.

This is so that when I start building something I do it the right way and also, it gives me mental "hints" to further lookup things during development.

With that, here are the resources I used to learn `golang`.

## Online courses

- [Learn How To Code: Google's Go (golang) Programming Language](https://www.udemy.com/share/1013gw/)
- [Master Go (Golang) Programming:The Complete Go Bootcamp 2021](https://www.udemy.com/share/102aeq/)
- [Go Bootcamp: Master Golang with 1000+ Exercises and Projects ](https://www.udemy.com/share/101YFa/)
- [Go: The Complete Developer's Guide (Golang)](https://www.udemy.com/share/101XnU/)
- [Web Development w/ Google’s Go (golang) Programming Language](https://www.udemy.com/share/101Xnq/)

## Documentation

### golang.org

Particular areas of [golang.org](https://golang.org/):

- [A Tour of Go](https://tour.golang.org/welcome/1)
- [How to write Go Code](https://golang.org/doc/code.html)
- [Language Specification](https://golang.org/ref/spec)
- [Effective Go](https://golang.org/doc/effective_go.html)

### Go Blog

The [Go Blog](https://blog.golang.org/) has many posts going into depth about aspects of the language.

### golang/go GitHub

The [GitHub Wiki](https://github.com/golang/go) contains many resources for every aspect of programming with `golang` from learning, tools and libraries.

### go.dev

[go.dev](https://go.dev/) details the different use cases of `golang` such as cloud and network services as well as companies using it for that purpose and libraries/frameworks to consider.

# Next

That's it for this post.

In part 2, I will talk about my development environment.

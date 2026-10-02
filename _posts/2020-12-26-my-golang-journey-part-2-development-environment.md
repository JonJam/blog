---
layout: post
title: "My DigitalOcean App Platform Hackathon entry: stock-checker - Part 2: Development environment"
tag:
  - golang
  - development-environment
  - visual-studio-code
  - golangci-lint
excerpt: "How I set up a Go development environment with Visual Studio Code, debugging, formatting, language support and golangci-lint."
---

![Part 2: Development environment](/assets/images/2020-12-26-my-golang-journey-part-2-development-environment-2edg-ic5s5vy52czom5643e5w.png)

> The cover image is from [MariaLetta/free-gophers-pack](https://github.com/MariaLetta/free-gophers-pack)

This post is part of a series detailing my journey with [`golang`](https://golang.org/) from learning the language to entering the [DigitalOcean App Platform Hackathon](https://dev.to/devteam/announcing-the-digitalocean-app-platform-hackathon-on-dev-2i1k).

The app I built can be found on [GitHub](https://github.com/JonJam/stock-checker).

Part 2 details the setup of my `golang` development environment.

# Setup `golang`

## Install `golang`

First step is to install `golang` which can be done by following this [page](https://golang.org/doc/install).

## Create workspace

> This step is not required when using [modules](https://blog.golang.org/using-go-modules). See the note on this [page](https://golang.org/doc/code.html).

Create a [workspace](https://golang.org/doc/gopath_code.html#Workspaces) which results in the directory structure shown below.

```
$HOME
    └── go
        ├── bin
        └── src
```

## Set environment variables

Next I added the following environment variables to my `.zshrc` file.

```
export GOPATH=/Users/jonjam/go
export GOBIN=/Users/jonjam/go/bin
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

More information about these variables can be found [here](https://golang.org/doc/install/source#environment).

# Setup Visual Studio Code

There are plenty of [choices](https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins) when it comes to IDEs. I went with [Visual Studio Code](https://code.visualstudio.com/), since I was already familiar with it.

This section details configuring it for `golang`.

## Install and activate Go extension

First to add `golang` support to VS Code, install the [Go extension](https://marketplace.visualstudio.com/items?itemName=golang.Go).

After installing the extension, there are some additional command line tools that also need to be installed to support the extension. This can be done by following this [page](https://github.com/golang/vscode-go#activate-the-go-extension).

## Configure formatting tool

The default formatting tool in VS Code is [`goreturns`](https://github.com/sqs/goreturns) which doesn't work with modules (see [here](https://github.com/golang/vscode-go/blob/master/docs/modules.md) for more information).

[`goimports`](https://pkg.go.dev/golang.org/x/tools/cmd/goimports) on the other hand does support modules.

Change the `go.formatTool` setting to `goimports`.

## Enable the Go language server

Besides formatting, some of the other helper tools do not support modules (see [here](https://github.com/golang/vscode-go/blob/master/docs/modules.md) for more information).

The [Go language server](https://github.com/golang/tools/tree/master/gopls) should be used; this can be enabled by following these [steps](https://github.com/golang/vscode-go/blob/master/docs/gopls.md#enable-the-language-server).

## Setup debugging

Debugging `golang` apps in VS Code is provided by [Delve](https://github.com/go-delve/delve).

Follow these [steps](https://github.com/golang/vscode-go/blob/master/docs/debugging.md) to install and configure Delve.

# Setup linter

Now that VS Code is setup, the final step is to choose a linter to assist with development.

I chose [`golangci-lint`](https://github.com/golangci/golangci-lint). It is a fast go linters runner that is capable of running multiple linters in parralel.

I discovered this on [golang wiki](https://github.com/golang/go/wiki/CodeTools).

## Install

`golangci-lint` can be installed locally by following these [steps](https://golangci-lint.run/usage/install/#local-installation).

## Configure linting tool

To integrate `golangci-lint` with VS Code, it is just a matter of changing a couple of [settings](https://golangci-lint.run/usage/integrations/).

## Create configuration file

The final step is to create a `.golangci.yml` for the project which defines what linters are enabled. This can also be used to enable/disable specific rules.

Below is the `.golangci.yml` file from stock-checker app:

```
linters:
  disable-all: true
  enable:
    # Default enabled linters
    - deadcode
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - structcheck
    - typecheck
    - varcheck

    # Added
    # goimports aligns with VS Code setup and includes gofmt
    - goimports
    - golint
```

# Next

That's it for this post.

In part 3, I will talk about developing the project.

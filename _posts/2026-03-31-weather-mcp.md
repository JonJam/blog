---
layout: post
title: "Agentic development in practice: Building a weather MCP server with Spring AI and Cursor"
tag:
  - spring-ai
  - mcp
  - cursor
  - java
excerpt: "A practical look at building a weather MCP server with Spring AI and using Cursor to support the development workflow."
---

![Agentic development in practice: Building a weather MCP server with Spring AI and Cursor](/assets/images/2026-03-31-weather-mcp-1idj-gzsitzbinekyfscgag12.png)

## Introduction

At the beginning of 2026, the UK was experiencing an [unusually wet start](https://www.metoffice.gov.uk/blog/2026/how-much-rain-have-we-had-in-february-and-winter) — something that seemed to dominate everyday conversation. That got me thinking:

Why shouldn’t AI agents be able to join in too?

So I set out to build a weather-powered [Model Context Protocol (MCP) server](https://modelcontextprotocol.io/docs/getting-started/intro) using [Spring AI](https://spring.io/projects/spring-ai), developed with [Cursor agents](https://cursor.com/home).

Along the way, I got to explore agentic development, learn new tooling, and solve a few unexpected challenges.

## What I built

I built an [Accuweather](https://www.accuweather.com/)-powered MCP server that enables AI agents to query:

- Current weather
- Hourly forecasts
- Daily forecasts

![A screenshot of the weather forecast for Sheffield, United Kingdom within Claude](/assets/images/2026-03-31-weather-mcp-1idj-7g31dhj768s2bqxcxk22.png)

The server is published on the [official MCP Registry](https://registry.modelcontextprotocol.io/?q=io.github.JonJam%2Faccuweather-mcp) and can be integrated into tools like Claude.

This allows LLMs to access structured, real-time weather data via tools—rather than relying on potentially outdated or hallucinated responses.

## How it works

The MCP server exposes weather functionality as tools that AI agents can call via the Model Context Protocol.

At a high level:

- Spring AI handles MCP integration and tool definitions
- The server communicates with the AccuWeather API
- MCP tools are exposed using annotations
- AI agents (e.g. Claude) invoke these tools via structured requests

This architecture allows the model to fetch live data when needed, making responses more accurate and reliable.

## How I got up to speed

### Cursor

Cursor provides excellent resources for getting started with agentic development that includes plenty of guidance, recommendations and best practises.

- [Docs](https://cursor.com/docs)
- [Courses](https://cursor.com/learn)
- [Workshops](https://cursor.com/workshops)

### Model Context Protocol

If you’re new to MCP, the [official documentation](https://modelcontextprotocol.io/docs/getting-started/intro) is a great starting point.

I’d also strongly recommend reading the specification itself — it covers important details like:

- Error handling
- Security considerations
- Implementation patterns

### Spring AI

The [Spring AI reference documentation](https://docs.spring.io/spring-ai/reference/index.html) provides solid guidance on MCP and broader Generative AI concepts such as chat clients and RAG.

One of the most useful resources was the [examples repository](https://github.com/spring-projects/spring-ai-examples), which demonstrates how everything fits together—particularly how to use [MCP annotations](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-annotations-overview.html) to define tools.

## Gotchas

These were the issues that slowed me down the most and are likely to help others building something similar.

### Cursor

My goal for this project was to lean into agentic development and minimise hand-written code.

Initially, I tried a one-shot approach: generating a full plan and letting the agent implement the feature. It worked—but produced far more code than I wanted, and used patterns I wouldn’t normally choose.

To improve this, I made a few changes:

- **Added guard rails**: Unit and integration tests, code formatting with [spotless](https://github.com/diffplug/spotless), code analysis using [checkstyle](https://checkstyle.org/)
- **Incremental rule creation**: Whenever the agent made a mistake (e.g. using Maven instead of Gradle, or missing [JSpecify](https://jspecify.dev/) nullability annotations), I added rules to [AGENTS.md](https://github.com/JonJam/accuweather-mcp/blob/main/AGENTS.md)
- **Used Plan mode and auto model selection**: This helped optimise usage and extend the limits of the Hobby plan.
- **Develop in increments**: Instead of building full features in one go, I broke them into layers: HTTP client, Gateway and MCP tool. This made it easier to review and course-correct.
- **Referenced existing code**: Once patterns were established, I used them as examples to guide the agent.

By the time I implemented the hourly forecast feature, I was able to generate a [plan](https://github.com/JonJam/accuweather-mcp/blob/main/.cursor/plans/hourly_forecast_feature_4e364946.plan.md) and delegate execution to a Cloud Agent—only needing to review the result.

![A screenshot of the output produced by a Cursor Cloud Agent when building the hourly forecast feature](/assets/images/2026-03-31-weather-mcp-1idj-jr7w9oh6i3odvfx1zouh.png)

### Spring AI

A few things weren’t immediately obvious from the docs:

- **Turn off console logging**: MCP servers using stdio must only output JSON-RPC messages. Any logging to stdout will [break the integration with an MCP client](https://modelcontextprotocol.io/docs/develop/build-server#logging-in-mcp-servers-3).

```yaml
logging:
  console:
    enabled: false
```

- **MCP Server annotations**: To use [MCP Server Annnotatons](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-annotations-server.html) which simplifies the creation of MCP server functionality like tools, you need to include the `org.springframework.ai:spring-ai-mcp-annotations` dependency and import them from `org.springaicommunity.mcp.annotation.*`.

- **Debugging with MCP Inspector**: If you want to use the [MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector) to debug your MCP server, use the following command:

```bash
npx @modelcontextprotocol/inspector -e 'JAVA_TOOL_OPTIONS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005' java -jar "build/libs/accuweather-mcp-local-snapshot.jar"
```

- **Testing with Claude**: If you want to test your MCP server using Claude, refer to the [MCP docs](https://modelcontextprotocol.io/docs/develop/build-server#testing-your-server-with-claude-for-desktop-3). Since I use [sdkman](https://sdkman.io/), the `command` value needed to be `~/.sdkman/candidates/java/current/bin/java`.

### Publishing to the MCP registry

Publishing to the [Official MCP registry](https://registry.modelcontextprotocol.io/) introduced a few additional challenges:

- **Docker multi-platform builds**: [semenatic-release docker plugins ](https://semantic-release.gitbook.io/semantic-release/extending/plugins-list#community-plugins) only support `amd64`. Since I was on an ARM-based Mac I needed `arm64` support and because I was already using [jib](https://github.com/GoogleContainerTools/jib) to containerize my application, I opted to use [@semantic-release/exec](https://github.com/semantic-release/exec) and jib directly to publish to Docker Hub.
- **Validating server.json**: The [mcp-publisher CLI](https://modelcontextprotocol.io/registry/quickstart#step-3-install-mcp-publisher) includes a `validate` command - use it early to verify your MCP server definition (`server.json`). I added this step to CI after encountering issues with description length during publishing.

## Closing thoughts

This project was a great way to explore agentic development in practice — moving beyond simple prompts to structured, tool-driven systems. As well as getting familiar with the Model Context Protocol and Spring AI.

And if nothing else, at least your AI will finally be able to complain about the weather too.

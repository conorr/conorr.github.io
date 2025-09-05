# What is Model Context Protocol?

If you've worked with AI coding assistants, you've probably heard of Model Context Protocol, or MCP. I have heard of it, but had never really taken the time to understand what it is.

So, what the heck is it? Let's take a deeper look.

## Basics

MCP is an open protocol that standardizes how AI models connect to external tools, data sources, and systems.

But let's back up for a second. What exactly is a protocol? 

Protocol is a fancy word for an agreed set of rules for how two programs talk to each other. The two programs could be on the same computer or on two different computers entirely.

For example, HTTP is a protocol that specifies how web browsers and servers request and exchange data (like web pages, images, or files) over the Internet.

So what kind of "computer talk" does MCP regulate? Again, it's basically how AI models talk to plugins such as tools and data sources.

As with many network protocols, MCP follows a client-and-server model. There are MCP clients and MCP servers.

MCP clients are usually AI assistants or LLMs: Claude, ChatGPT, Gemini, Copilot, et cetera.

MCP servers are programs that offer some kind of functionality that is not baked into the LLM. MCP servers need to be exposed over the Internet to be reachable by MCP clients.

Let's look at an example.

## MCP server example

Suppose you had a weather station at your house and you published real-time weather data on the Internet. Suppose you want ChatGPT to be able to use that data.

In order to do this, you would wrap your weather station API in an MCP server, and declare its capabilities like so:

```json
{
  "methods": {
    "get_current_weather": {
      "description": "Get the latest weather readings",
      "params": [],
      "returns": {
        "temperature_f": "number",
        "humidity_percent": "number",
        "wind_speed_mph": "number"
      }
    },
    "get_forecast": {
      "description": "Return forecast for the next 24 hours",
      "params": [],
      "returns": "array"
    }
  }
}
```

Next, after configuring ChatGPT to be aware of your MCP server, you could then ask it:

> What’s the weather like right now at my house?

ChatGPT would look up the available methods from your server and ask:

```json
{
  "method": "get_current_weather",
  "params": {}
}
```

Your MCP server would reply with:

```json
{
  "temperature_f": 72.4,
  "humidity_percent": 58,
  "wind_speed_mph": 6
}
```

ChatGPT would then take this structured data and return a conversational answer:

> Right now at your house it’s 72°F, with 58% humidity, and a light breeze around 6 miles per hour.

Cool. This works well and good with ChatGPT; but what if you want another LMM such as Claude to query your weather station?

This is the point of open standards. Model Context Protocol allows interopability between MCP servers and MCP clients.
Your MCP server can work wth Claude, Gemini, Copilot, and even the AI agent that your friend's cousin built, as along as your friend's cousin's AI agent speaks the protocol.

Even though MCP was developed by Anthropic, it's an open source, open standard protected by the [MIT License](https://en.wikipedia.org/wiki/MIT_License), so in theory it is non-proprietary.

## Conclusion

Now that I know that MCP is an open standard and that  it allows interoperabilty between proprietary LLMs, I think it's a pretty cool idea. There should be open standards for so many things! Text messaging is a big one.

Unfortunately, it's not always in the interest of big tech companies to develop or adhere to open standards. As evidenced by Apple Inc., it's much more lucrative to build walled gardens.

Open protocols are meant to promote interopability, but their second order effects are to prevent lock-in and encourage competition. These are essential things for economic freedom.

Not to mention that fact that open protocols often create _massive value_. It's just that the value is not captured by one firm, but is distributed over many. The Internet itself is a classic example. Think of the uncountable trillions in value it has created, yet it is based on an open protocol (HTTP). The Internet as we know it today could never have grown out of a proprietary protocol.

So I am a pleasantly surprised to see this in the AI space. I am not a huge AI booster, but I do view the open competition in the AI space as a positive development in the tech industry.
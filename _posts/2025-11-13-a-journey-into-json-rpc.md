# A journey into JSON-RPC

Recently at work I was debugging a problem with go’s debugger, delve (`dlv`). Debugging a problem with a debugger is an ironic situation to find yourself in, though the problem was actually related to Docker networking and not the debugger itself.

Anyways in the course of this problem solving I was forced to do a deep dive into the client and server connection between the headless `dlv` instance and my IDE. I came across the fact that delve and my IDE use JSON-RPC to communicate, which I found interesting.

It turns out that when you run your go code with the `dlv` process and connect to the debugger using an IDE like VS Code or GoLand, the IDE and the delve server exchange JSON-RPC messages to inform the IDE about the current state of the debugger. The current state of the debugger being things like the call stack, the values of locals, and the symbols under the current breakpoint. The IDE can then message the debugger to do things like set new breakpoints, step execution forward, and so on.

**What makes JSON-RPC a good fit for the interaction beween a debugger and an IDE?** I realized that while I was dimly aware that JSON-RPC existed and was related to remote procedure calls, I didn’t know all that much about it, or the RPC paradigm in general.

## A JSON-RPC message

The JSON-RPC protocol is lightweight. The [specification](https://www.jsonrpc.org/specification) could be printed on a couple pieces of paper.

JSON-RPC defines a small number of fields: just enough to request a function call (`method`, `params`), and to receive the result (`result`, `error`). It also includes a protocol version field (`jsonrpc`) and a correlation ID (`id`). That's pretty much all you need to know.

Let's look at an example of a JSON-RPC message. Consider a weather data server that can be queried for the current temperature given a latitude and longitude.

A request would look like this:

```json
{
  "jsonrpc": "2.0",
  "method": "get_temperature",
  "params": [47.6085282, -122.3406858],
  "id": 1
}
```

The server would reply with the temperature in Fahrenheit:

```json
{
  "jsonrpc": "2.0",
  "result": "53.2",
  "id": 1
}
```

To use the example of the delve debugger, when you create a breakpoint in your IDE (say at line 16 in `main.go`), your IDE messages delve with the following:

```json
{
  "jsonrpc": "2.0",
  "method": "RPCServer.CreateBreakpoint",
  "params":[
    {
      "Breakpoint": {
        "file": "/User/you/main.go",
        "line": 16
      }
    }
  ],
  "id": 27
}
```

The IDE then receives an acknowledgement from delve:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "Breakpoint": {
      "id": 1,
      "file": "/User/you/main.go",
      "line": 16,
      "functionName": "main.main",
      "addr": 4202492
    }
  },
  "id": 27
}
```

So this is all pretty straightfoward: JSON-RPC is just a way to call a method in a different process with some parameters and receive the result. That process may be on your machine or it may be on a different machine.

But how do we actually get the message to that other process?

# Not a transport protocol

An interesting thing about JSON-RPC is that it is not itself a transport protocol. It's only a message protocol, so it does not define or even care how messages are transported.

In the weather data server example above, we could deliver the message using HTTP. We'd just build an endpoint `/rpc` that knows how to handle JSON-RPC messages:

```
POST /rpc HTTP/1.1
Host: my-weather-api.com
Content-Type: application/json
Accept: application/json
Content-Length: 104

{
  "jsonrpc": "2.0",
  "method": "get_temperature",
  "params": [47.6085282, -122.3406858],
  "id": 1
}
```

Meanwhile, in the case of delve, the connection between delve and your IDE is usually over a [TCP socket](https://github.com/go-delve/delve/blob/master/Documentation/api/json-rpc/README.md#json-rpc-interface), conventionally at port 40000.

There's no reason you couldn't exchange JSON-RPC over snail mail, if you had a willing participant on the other side to interpret your message, execute the request at their computer, and compose a reply in JSON-RPC.

## Why remote procedure calls at all?

I confess that after over a decade of building REST APIs, I have a hard time thinking outside of the REST paradigm.

Why would you use JSON-RPC, or any RPC protocol, over REST?

Well, the key seems to be stateful vs. stateless interaction. REST is stateless by definition, meaning each request from a client must contain all of the information needed to understand and process that request. 

JSON-RPC is not stateless or stateful by definition, but because it does not preclude statefulness, it's a good fit when interaction does need to be stateful.

So what are examples of interactions that are practical to be modeled statefully?

The interaction between a debugger and an IDE is a good example because both the debugger and the IDE are keeping track of a live execution environment of uncompiled code. Each request assumes context of this live execution environment. Is the program running, or is it halted? Where is it halted? This state does not need to be re-asserted in every request.

A Bitcoin transaction is another example of a stateful interaction.

What we think of as a Bitcoin transaction is an interaction between a client (wallet software) and a server (a Bitcoin node). It would be impossible for the client to specify in its request *all of the information needed to execute a transaction* when this information is global state: the state of the entire blockchain! No, you cannot specify the entire blockchain in every request. Thus the Bitcoin protocol must be stateful.

And what message protocol does Bitcoin use? JSON-RPC, of course!

In fact, JSON-RPC has become the de facto standard for blockchain node APIs across much of the cryptocurrency ecosystem. This is because it's lightweight, language-agnostic, and transport-agnostic. It's yet another example of a simple protocol enabling asymmetric value.
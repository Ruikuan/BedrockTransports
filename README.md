# Bedrock Transports

[Project Bedrock](https://github.com/aspnet/AspNetCore/issues/4772) is a set of APIs .NET Core for doing transport agnostic networking. In .NET Core 3.0 we've introduced some new abstractions
as part of the [Microsoft.AspNetCore.Connections.Abstractions](https://www.nuget.org/packages/Microsoft.AspNetCore.Connections.Abstractions) for client-server communication.

## Server API

```C#
public interface IConnectionListenerFactory
{
    ValueTask<IConnectionListener> BindAsync(EndPoint endpoint, CancellationToken cancellationToken = default);
}

public interface IConnectionListener : IAsyncDisposable
{
    EndPoint EndPoint { get; }
    ValueTask<ConnectionContext> AcceptAsync(CancellationToken cancellationToken = default);
    ValueTask UnbindAsync(CancellationToken cancellationToken = default);
}
```

## Client API

```C#
public interface IConnectionFactory
{
    ValueTask<ConnectionContext> ConnectAsync(EndPoint endpoint, CancellationToken cancellationToken = default);
}
```

## Why?

The goal of this project is to abstract the networking stack from the application layer so that protocols could be written agnostic of the underlying transport. This
enables new transports (like QUIC) to be adopted without changing the code written against the abstraction. The design also allows reaching into the abstraction
to pull out transport specific features where that is needed.

Consumers:
- Kestrel is built on top of these APIs (this is the new Kestrel transport layer)
- SignalR is built on top of these APIs (.NET client and server)
- [Orleans](https://github.com/dotnet/orleans/pull/5436/files) just recently adopted these abstractions

A sample of a couple bedrock transports using new APIs added in .NET Core 3.0

- **Pipes** - A named pipes transport based on NamedPipeClientStream and NamedPipeServerStream
- **Http2** - An HTTP/2 based transport that uses turns each request into a duplex connection using HTTP/2 streams. Uses Kestrel on the server and HttpClient on the client
- **WebSockets** - A WebSockets transport (uses Kestrel and the building blocks of SignalR internally).
- **AzureSignalR** - A transport that uses AzureSignalR as a low level connection proxy. It implements the [Azure SignalR Service protocol](https://github.com/Azure/azure-signalr/blob/dev/specs/ServiceProtocol.md) in order
to do the connection multiplexing.

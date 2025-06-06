# Zebus

[![Build](https://github.com/Abc-Arbitrage/Zebus/workflows/Build/badge.svg)](https://github.com/Abc-Arbitrage/Zebus/actions?query=workflow%3ABuild)
[![NuGet](https://img.shields.io/nuget/v/Zebus.svg?label=NuGet&logo=NuGet)](http://www.nuget.org/packages/Zebus/)
[![Gitter](https://img.shields.io/gitter/room/Abc-Arbitrage/Zebus.svg?label=Chat&logo=Gitter&color=blue)](https://gitter.im/Abc-Arbitrage/Zebus)

Zebus is a lightweight peer to peer service bus, built with [CQRS](http://martinfowler.com/bliki/CQRS.html) principles in mind. It allows applications to communicate with each other in a fast and easy manner. Most of the complexity is hidden in the library and you can focus on writing code that matters to you, not debugging messaging code.

# Introduction

Zebus is **peer to peer**, so it does not depend on a broker to dispatch messages between the peers. This allows it to reach a throughput of 140k msg/s and a roundtrip latency under 500µs (have a look at the [Performance](https://github.com/Abc-Arbitrage/Zebus/wiki/Performance) page for details).

It is **resilient** thanks to the absence of a broker and an optional persistence feature that ensures that messages are not lost if a peer is down or disconnected.

It is **stable**, since we have been using it on a production environment at [ABC Arbitrage](http://www.abc-arbitrage.com/) since 2013, handling hundreds of millions of messages per day.

## Key concepts

### Peer

We call a peer any program that is connected to the bus, a peer is identified by a unique identifier called a PeerId that looks like this: `MyAmazingPeer.0` (we use this convention to identify different instances of the same service).

### Event

An event is sent by a peer to notify everyone who is interested that something happened (ex: `MyBusinessObjectWasSaved`, `AlertTriggered`...).

### Command

A command is sent to a peer asking for an action to be performed (ex: `SaveMyBusinessObjectCommand`).

### Message Handler

A class deriving from `IMessageHandler<T>` will be scanned by the bus and will be used to handle messages of the `T` kind on reception.

### Bus

The piece of code that is the point of entry to use Zebus, the methods that you will use the most are `Publish(IEvent)` and `Send(ICommand)`.

## A quick demo

On startup, the bus will scan your assemblies for message handlers and notify the other peers that you are interested by those messages. When a peer publishes a message, it will use the Directory to know who handles it and send the message directly to the correct recipients.

### Receiver

```csharp
public class MyHandler : IMessageHandler<MyEvent>
{
    public void Handle(MyEvent myEvent)
    {
        Console.WriteLine(myEvent.Value);
    }
}
```

### Sender

```csharp
public void MethodThatSends(IBus bus)
{
    bus.Publish(new MyEvent { Value = 42 });
}
```

### Event description

```csharp
[ProtoContract]
public class MyEvent : IEvent
{
    [ProtoMember(1)]
    public int Value { get; set; }
}
```

And you're set ! This is all the code you need to send an event from one machine to the other. If you want to read more about how the magic happens, have a look at the [wiki](https://github.com/Abc-Arbitrage/Zebus/wiki). Or if you want a more detailed walkthrough (what to reference, how to start the Bus...) visit the [Quick start](https://github.com/Abc-Arbitrage/Zebus/wiki/Quick-start) page.

## Snapshot example

When a new peer subscribes to a message type you may want to send it an initial
state. This is done by implementing a `SubscriptionSnapshotGenerator` that will
publish a snapshot event to the newly subscribed peer.

```csharp
[ProtoContract]
public class StateSnapshot : IEvent
{
    [ProtoMember(1)]
    public List<int> Values { get; set; }
}

public class StateSnapshotGenerator : SubscriptionSnapshotGenerator<StateSnapshot, MyEvent>
{
    private readonly IRepository _repository;

    public StateSnapshotGenerator(IBus bus, IRepository repository)
        : base(bus)
    {
        _repository = repository;
    }

    protected override StateSnapshot GenerateSnapshot(SubscriptionsForType subscriptions)
    {
        return new StateSnapshot
        {
            Values = _repository.GetAllValues()
        };
    }
}
```

Each time a peer subscribes to `MyEvent`, the bus will send the current
`StateSnapshot` before delivering any subsequent events.

# Release notes

We try to stick to the [semantic versioning](http://semver.org/) principles and keep the [release notes](https://github.com/Abc-Arbitrage/Zebus/blob/master/RELEASE_NOTES.md) and [directory release notes](https://github.com/Abc-Arbitrage/Zebus/blob/master/RELEASE_NOTES_DIRECTORY.md) up to date.

# Other repositories

 - [Zebus.MessageDsl](https://github.com/Abc-Arbitrage/Zebus.MessageDsl) - a DSL which simplifies the writing of ProtoBuf contracts for Zebus
 - [Zebus.TinyHost](https://github.com/Abc-Arbitrage/Zebus.TinyHost) - a lightweight host used to run Zebus enabled services
 - [Zebus.Samples](https://github.com/Abc-Arbitrage/Zebus.Samples) - miscellaneous samples that show how to get started with Zebus

# Copyright

Copyright © ABC Arbitrage Asset Management

# License

Zebus is licensed under MIT, refer to [LICENSE.md](https://github.com/Abc-Arbitrage/Zebus/blob/master/LICENSE.md) for more information.

# Tools
We use

[![resharper_icon](https://raw.githubusercontent.com/wiki/Abc-Arbitrage/Zebus/content/icon_ReSharper.png)](https://www.jetbrains.com/resharper/)

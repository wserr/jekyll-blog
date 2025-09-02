---
layout: article
title: "Events: Terminology and purposes"
key: "eventsterminologyandpurposes"
tags: event-driven-architecture eda events
aside:
    toc: true
mermaid: true
---

Usually, when people talk about events, a variety of different definitions and
purposes comes up.

## Different purposes of events

### Notification

A service that sends out an event does not expect a response. This allows for
**loose coupling**, is flexible and also easy to set up. The downside is that,
when the number of events grows and the system becomes more complex, it can
become hard to oversee the overall flow. Often, the only way to oversee the
flow, is to monitor a live system. However, tools exist to mitigate this
downside and make it easier to oversee the whole flow (e.g. tracing with context
propagation).

### Event carried state transfer

When performing state transfers over events, the senders make sure that enough
information is contained within the event, so that the receiver does not need to
contact the sender again for more information. The upside of this is, that a
receiver can continue working, even if the sender is down. This makes the system
more resilient. The downside however is that a lot of (duplicate) data is going
around. Also, the complexity of the receiver increases, since it is responsible
of maintaining the state of the events coming in.

#### Enrichment vs Querying

Should an event be enriched with additional metadata? There are upsides and
downsides to this.

- Upside: The consumers have greater autonomy when all information is contained
  within the event
- Downside: events can get quite big. Including too much information inside the
  event can also become a security risk

> The art is to include enough metadata in the event, without being too verbose.
> Note that it is the responsibility of the producer to add the appropriate
> fields to the event

## Different kinds of events

### Domain Event

Domain event is a record of some business-significant occurence in a bounded
context

Most basic domain event you can get:

```csharp
public interface DomainEvent
{
    publice DateTime OccuredOn
    {
        get;
    }
}
```

The domain event should contain a timestamp that tells you when the event
occurred.

What information should be included in the domain event? To answer this
question, ask yourself: what is the reason that caused the domain event to be
published in the first place?

#### Naming conventions

Naming: Has a verb in the past tense and a 'subject'

> e.g. Bad name for event: `ObjectUpdated`: very generic, it does not say
> anything e.g. Good name for event: `WarrantyExpired`: self-explanatory, clear,
> immutable owner

### Business Event

> Cymo specializes in Business Events

#### What are business events?

> A business event not only defines **what** happened with an entity, but also
> **why**.

- Broadcasts data about action happened in the past
- Broadcasts that something happened to something/someone
- The subject has (in general) an immutable character (at the core)
- can be assigned to a process-step in a process diagram quite easily
- is expressed in a human understandable, unambiguous language
- is produced by the owner of the subject (within its domain)

#### Challenges, faced when creating business events

- Business events should be of **ideal size**: they should not be too large, but
  they should also contain all necessary context.

- **Producer** is responsible for adding enough context to the event to
  determine its meaning: _Why has the data been changed?_

- We should be able to follow the entire journey of an object following the
  business events of that object

- [ ] What is the difference between a domain event and a business event? Are
      they the same thing? Is a business event a kind of domain event?

#### What happens if too little data is sent along with the business event?

In this case, the consumer needs to send a message back to the producer to query
additional data. This can cause unintended DDoS attacks on the Kafka cluster
when we handle many events. See also [previous section](#enrichment-vs-querying)

#### Retention policy

- Choose **infinite retention** to start with, since business events are needed
  to build up the state of an entity, and to track metrics. It is best to store
  the events as long as possible
- If, in production, there is too much data in a topic, we can consider to lower
  the retention period (e.g. to a couple of years)

### State Event

Contain entire representation of object that has changed

- Disadvantage: you know that the entity has changed and what the end result is,
  but you don't know **why** it changed
- Size of the event: in practice this does not really matter

### State Events vs. Business Events

Where possible, do not use State Events, but Business Events. However, this is
not always straightforward in practice:

- Legacy code bases; possibly a lot of rework required
- Requires a switch in thought process of the developers working on a project

## Events vs. Commands vs. Queries

A command is different from an event in that a command can be rejected as
inappropriate in some cases, such as due to supply and availability of some
resources.

> Analogy: imagine that the kitchen is dirty. Publishing an event would be like
> putting a post-it on the wall: "the kitchen is dirty" someone might pick it up
> to do something about it, but you are never sure that it will happen.
> Publishing a command would be like asking someone to clean it up. This command
> might be accepted or rejected

- **Commands** are requests for some operation to be performed by another
  service. A result might or might not be expected
- **Events** are both a fact and a notification. They represent something that
  happened in the real world, but include no expectation of any future action.
  Usage: when loose coupling is important
- **Queries**: request to look something up. Usage: lightweight data retrieval
  across service boundaries

|         | Behavior/state change | Includes a response |
| ------- | --------------------- | ------------------- |
| Command | Request to happen     | Maybe               |
| Event   | Just happened         | Never               |
| Query   | None                  | Always              |

## Events and Kafka

> The partition key should be stable and unique. All the IDs that are used for a
> partition key should exist before publishing the first event

Example: if you have a `Customer` topic, and the paritition key is
`<RegionID>-<CustomerID>`, then both of those IDs should exist in all events
published to that topic

## Event based processing

Different ways of doing event-based processing

- **event-driven approach** - processing 1 event stream one message at a time
- **streaming approach** - which joins different event streams together
- **stateful streaming approach**, which turns streams into tables and stores
  data in the log

Kafka Streams API: stateful stream processor

## Sources

### Books

- Designing Event Driven Systems
- Domain-Driven Design Distilled

### Converations

- Kristof Van De Voorde

### Blog posts

https://martinfowler.com/articles/201701-event-driven.html

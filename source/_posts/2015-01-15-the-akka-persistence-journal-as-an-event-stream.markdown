---
layout: post
title: "The Akka-persistence Journal as an Event Stream"
date: 2015-01-20 22:22:18 +0000
comments: true
categories: 
external-url: https://github.com/jdgoldie/akka-persistence-gridgain-experimental
---
Event-sourcing, streaming, and complex event processing all seem to me like related ideas that can be combined to produce reactive applications.  (I signed the [Reactive Manifesto][4] before
it was cool...)  To that end, I have started a [project][1] to implement the akka-persistence journal using [GridGain][2] as the underlying technology.  GridGain offers a chance for a unique 
take on the akka-persistence journal by leveraging its distributed persistence architecture, streaming, and event processing capabilities.  

The working premise of my project is that by modeling the journal as a stream, it is possible to generate near-real-time projections of entity state without relying on the polling mechanism of 
Akka's PersistentViews.  Additionally, streams derived from the journal-stream can be fed to complex event processing queries.  So far, the premise seems to be achievable.

<!-- more -->

Aside from avoiding the use of PersistentViews, other interesting use-cases present themselves:

* Shopping cart abandonment - use [CEP][5] to detect when a cart is created and not checked out with a certain window
* Oversight and approvals - monitor changes to entities and trigger approval workflows if certain conditions are met
* Automatic escalation - issues no resolved within a window generate escalation events

### Design

I settled on a design that uses a GridGain stream with three windows, an index, and the cache -- something like this:

{% img center caption https://lh5.googleusercontent.com/-TiHzVi40zdQ/VL8Urf1WU1I/AAAAAAAAYuQ/OSoAivXDeDQ/w712-h344-no/JournalDesign.jpg 800 600 "" "The Design, more or less." %}

The flow is as follows:

1. akka-persistence adds events to the first window of the stream
2. events are indexed for replay
3. events are passed to the next stage
4. projection code persists to database or other store
5. events are routed to one or more optional stages for analysis, CEP, etc.

### Challenges and Issues

One challenge of using a stream for the journal is that the journal must support deletions while streams are append-only.  To support deletions,
the index on the initial stage tracks both a hard and soft delete "floor" for each persistence id and the replay logic takes this value
into account when replaying events.  To pass deletes from akka-persistence to the stream and on to the journal, the stream models the 
journal as a command log of write and delete instructions.  Only the writes are passed to the projection window and their payloads (`PersistentRepr`) 
are unwrapped so only the first window is aware of the journal commands.

Durability is also a challenge.  GridGain streams do not support durability mechanisms like the cache does, so the journal is, in effect, in-memory only.  This shortcoming needs to be addressed to make the implementation viable.  The same can be said of the index used to track the journal events and the delete floors.  It is currently stored in a Java Map.  A better storage mechanism -- perhaps the GridGain 
cache itself -- would make this part of the system more robust.

### What Works

Currently, the journal passes the compatability tests.  The default projection and routing interfaces are implementated and an 
example of a default projection stage has been written.

### Future Roadmap

1. Fix [Issue #1][3]
2. Durability strategy - stream and index durability
3. Examples - CEP examples demonstrating use-cases
4. Performance
5. Deployment - client v. data node deployment
6. Migrate to Apache Ignite

[1]: https://github.com/jdgoldie/akka-persistence-gridgain-experimental
[2]: http://www.gridgain.com/
[3]: https://github.com/jdgoldie/akka-persistence-gridgain-experimental/issues/1
[4]: http://www.reactivemanifesto.org/
[5]: http://en.wikipedia.org/wiki/Complex_event_processing
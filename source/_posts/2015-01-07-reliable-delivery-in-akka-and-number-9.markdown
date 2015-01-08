---
layout: post
title: "Reliable Delivery in Akka and Number 9"
date: 2015-01-07 20:29:58 +0000
comments: true
categories: 
external-url: https://vaughnvernon.co/?p=986
---
Last week [Vaughn Vernon][1] teased/announced [Number 9][2] which he proposes as an alternative to Akka's AtLeastOnceDelivery.  One interesting difference
is the use of the Message Journal to store unconfirmed messages.  This design is a siginificant departure from Akka's AtLeastOnceDelivery.

> You see, AtLeastOnceDelivery doesn’t by default support message persistent storage, and actually only 
> offers any resemblance to it when your actor is given a chance to store a snapshot of its current state. 
> Since a snapshot is not offered for every message you send via AtLeastOnceDelivery, a crash could easily 
> render any number of messages undelivered.

Since snapshots are *optional* and in most cases *periodic* based on either message throughput or elapsed time, you should
count on losing messages at some point.

<!-- more --> 

In addition to being responsible for keeping track of unconfirmed messages, AtLeastOnceDelivery also makes each Entity responsible for attempting redelivery of 
its own messages and tracking delivery confirmations.  *Number 9* moves the redelivery task to a separate actor named Redeliverer.  This is an interesing idea
that appears to fix one issue I've encountered with AtLeastOnceDelivery and its use of the *redeliveryTick*.

Each actor using AtLeastOnceDelivery sets up a redeliveryTick system timer to trigger a check for unconfirmed messages.  This timer fires even when the actor has no 
unconfirmed messages.  The tick has the side effect of making receiveTimeout unusable.  On a recent project that used the cluster sharding extension with passivation support,
this conflict prevented actors from passivating correctly.  Actors using AtLeastOnceDelivery with *no* unconfirmed messages would refuse to passivate as long as they 
were receiving the redeliveryTick.

The Redeliverer looks to move this functionality out of the entities which should allow sharding/passivation to work as designed.

While making these departures from the current implementation, it's important to remember that *Number 9* is an alternative implementation of at-least-once.

> Since Number 9 provides an at-least-once delivery mechanism, you will likely want to take whatever design steps are necessary to make the 
> [the processor] an Idempotent Receiver. This requirement will be more or less challenging depending on the state machine needed by the actor.

Because exactly-once [is not what you want][3].

From what's been shared so far *Number 9* sounds like a good approach to fixing some of the issues with AtLeastOnceDelivery.  The only drawback I see at this point
is that it's unreleased and I'd really love to take a look at the code.  I'll be watching this tool for future articles and releases.



[1]: https://vaughnvernon.co/?page_id=178
[2]: https://vaughnvernon.co/?p=976
[3]: http://brooker.co.za/blog/2014/11/15/exactly-once.html
---
layout: post
title: "Queues Don't Fix Overload"
date: 2014-12-12 17:00:32 +0000
comments: true
categories: 
external-url: http://ferd.ca/queues-don-t-fix-overload.html
---
I really enjoyed [Fred Hebert][1]'s great post on queues and system overload.  An overflowing sink is my favorite image of what happens to your
system when a queue is used as a buffer to mask the slow parts of the system.  

{% img left caption https://lh6.googleusercontent.com/proxy/-trPHfKNxS07bxGXx8qTSaELzrkz-6sPmZ-DGtLhKWbcKPfvK9Wr5zscdk9yLftXT9iBtkR3YWscwmyiunRT152o8gsOCrp9SrivK7EcPXozSd2uI679U1OAgMD1vFo=w426-h295  "" "Image Credit: Fred Hebert" %}


{% pullquote left %}
Systems have limits.  No matter how much we, as developers and architects, like to throw around derivations of the word 
*scalable* in design discussions, the limits are there.  And you are going to hit them.  Dropping in a queue to buffer 
requests only hides the fact that there is a limit.  {"It becomes a kind of bucket list for your application"} -- work you are hoping
it will complete before it crashes or reaches the next maintenance or upgrade window.  And then, the queue has made the 
problem worse, even if you made it persistent.
{% endpullquote %}

Fred's suggestion, with which I completely agree, is to use back-pressure mechanisms so the system gracefully handles 
new user requests as it approaches its operational limits.

> But someone should have picked what had to give: do you stop people from inputting stuff in the system, 
> or do you shed load. Those are inescapable choices, where inaction leads to system failure.
>
> And you know what's cool? If you identify these bottlenecks you have for real in your system, and you put 
> them behind proper back-pressure mechanisms, your system won't even have the right to become slow.

{% pullquote %}
Notice that this only keeps your system from becoming slow and falling over on itself.  It *does not* guarantee that all 
user requests will be served, merely that they will be handled.  This may sound like splitting hairs, but telling the 
user that the system is too busy to handle their request is better than failing to respond before reaching either the HTTP tineout 
or the limit of their patience.  {"No feedback at all is worse that telling the user that the system is busy."}
{% endpullquote %}

Back-pressure mechanisms, both formal and informal, exist all around us:

- A theme park stops letting riders enqueue for a [popular attraction][3] if they cannot get through the ride before the park closes 
- A cell-phone sends a new caller to voice-mail if a call is already in progress
- A line outside a restaurant signals a potential diner to seek other options

{% img center caption https://farm4.staticflickr.com/3765/9501576373_1c159d36f1.jpg 500 367 "" "Image Credit: [Corey Seeman][5]" %}


It may seem strange, perhaps even wrong to plan to refuse a user request, but it actually provides a better experience than trying 
to take on more work than the system can handle.  Imagine if the theme park didn't close the line until the park closed, and at that point,
everyone still waiting had to leave.  Those riders would have a worse experience than being turned away from the line in the first 
place.  They go away disappointed instead of angry and they come back earlier the following day.  

> To make stuff usable, a proper idempotent API with end-to-end principles in mind will make it so these instances of 
> back-pressure and load shedding should rarely be a problem for your callers, because they can safely retry requests and know if they worked.

I recently encountered a failure in an online ticketing system.  I was purchasing tickets to the Nutcracker Ballet, and after debating the merits
of orchestra-rear versus first-balcony seats, I was ready to checkout.  Three pretty good seats were in the cart, parking was reserved, 
credit card information was entered.  I hit *Complete Purchase*.  I watched the spinner until the request timed out.

Did I get the seats?  Had my credit card been charged?  How long should I wait for a confirmation email before assuming the purchase 
had not gone through?  I tried again with the same result.  I tried again.   The ticketing system was 
now completely unresponsive.  Did I have six tickets to the ballet?  I wrestled with this and other questions 
as I tried to get to sleep that night.

{% pullquote %}
I checked the next morning and saw that the first set of seats I had attempted to purchase were still showing as available.  That meant 
my purchases had failed.  {"I don't have to be a UX guy to know that's a horrible user experience."}  The system should either refuse to let me 
start the ticket selection process or prevent me from completing checkout if it's under too much load.  Either one is a better choice than 
letting the HTTP request timeout.  It's the difference between making your system looking busy and looking broken. 
{% endpullquote %}


[1]: https://twitter.com/mononcqc/
[3]: https://www.universalorlando.com/Rides/Universal-Studios-Florida/Harry-Potter-And-The-Escape-From-Gringotts.aspx
[5]: https://flic.kr/p/ftC4Ci
---
title:  "Galidevo Galilei"
author: Diego Lafuente
categories: 
  - Troubleshooting
last_modified_at: 2021-05-29T05:50:00Z
type: posts
header:
    teaser: /assets/images/posts/galidevo_galilei/teaser.jpg
tags:
    - javascript
    - python
    - troubleshooting
    - astronomy
---

Galileo Galilei was born in Pisa in 1564 and lived a very adventurous life. He
studied gravity, motion, inertia and astronomy among others. He was so important
in his time that he even got invited to a barbecue that Pope Urban VIII was
throwing in his honor. Ended up declining the invitation when he learnt that he
was the main dish. Of all his achievements, one was called to change the course 
of Humanity forever. Galileo is considered to be the father of the scientific
method, a sort of magical recipe that empowers whoever makes proper use of it to
tell the difference between true, proven facts and the rest. Our society, our
technology and our progress is built upon a huge pile of scientific truths that 
have come out of Galileo's magic wand.

Here at Devo, we are premium users of the scientific method. Conclusions and
proven facts can only be drawn after a meticulous and intensive session of
experimenting. The following story is good proof of that. A few weeks ago, a
colleague from Professional Services reported a data loss in a new environment
they were building. He even pointed the blame at a certain component of the
system, because according to his experiments the data loss only occurred in
newer versions of that component whilst older versions behaved properly. A true
believer in the scientific method since his conclusions were based on
experiments, but too hasty drawing his revolver. And maybe too laid-back
performing the experiments. Galileo's recipe is a very powerful tool, but you
need to use it carefully and rigorously to make it work.

Solving a complex issue like this one is pretty much like watching a suspense
movie. Meeting the characters, understanding the plot, and finally unraveling an
unexpected ending.

# Back to the Future (1985)

A bit of background first. Devo is a distributed database where the data,
instead of being stored in structured, binary files, is spread out in thousands
of text files (aka log files). We often refer to the "unit" of data in Devo as
an "event". An event (just an entry in a log file) is the equivalent of a record
in a traditional database.

It's easy to picture Devo as a wide river with a dam in the middle. The dam
divides the river in two parts: the ingestion of logs and the exploitation of
data. Upstream, the ingestion is in charge of riding the avalanche of log events
coming from all our customers into the appropriate area of the dam (aka the hard
drives). Downstream, the exploitation deals with understanding the queries,
driving them through the appropriate pipes within the dam and returning
accurate responses as fast as possible. Both sides communicate through the dam:
the ingestion is responsible of storing the data in the right place and with the
right format so that the exploitation can locate, understand and dissect what's
in there.

In this context, the concept of "data loss" always happens on the ingestion
side. It implies that at least some of the data that we streamed down the river
did not make it to the dam. And that's a big problem because each drop of water
only falls once.

# The Usual Suspects (1995)

Devo is a complex creature, and only in the ingestion part, it has a few
candidates to blame when something fails:

1. The sender. It's usually a component called "the relay" (and written in
Java), but in this case, it was a custom Python sender built on top of our
[python-sdk](https://github.com/DevoInc/python-sdk).

2. Python itself. Compilers, interpreters, runtime environments and standard
libraries are also programs, and therefore, they can contain bugs and errors.

3. The load balancer (LB). Written in JavaScript and run with Nodejs, it
receives the stream coming from the sender, frames the events and balances the
load among the different syslog collectors.

4. The Syslog Collectors (SCs), also written in JavaScript and run with
Nodejs. They are in charge of manipulating the incoming events and saving them
to disk.

5. Nodejs, the JavaScript VM in charge of running the LB and the SCs.

6. Docker. Yet another layer on top of the OS. Prone to errors, as everything
else.

7. The infrastructure: the networks, the file system, the disks, the OS,
etc...

8. The relative position of Saturn and Uranus three days after the summer
solstice. It changes every year and it deeply affects the behaviour of our
software.

Let's get down to the facts. There was an experiment that was showing a loss of
events in a new docker environment featuring v5.12.x of our LB (running on
Nodejs 12.x.x), a very recent version of the SCs and a sender agent that was
using our python-sdk to send the events. The experiment consisted in sending
around 150M events and then checking that all of them were written to disk. The
whole thing took around 3 hours to complete, and allegedly lost a few tens
(sometimes hundreds) of events in the way. Again, "losing" here means that some
events were sent, but never made it to the disk. This "same" experiment was
performed in a different environment running v4.11.x of the LB on Nodejs 8.x.x
and was not losing events. It looked like the LB was the murderer, but there
were still a few surprise witnesses to be called to court.

# Finding Nemo (2003)

First thing to do is always finding out what's wrong. Anything that may remotely
account for what's been observed. Then grab it, and dig deeper.

The experiment that originally led to the reported event loss was way too heavy.
Waiting 3 hours for each experiment to finish can exhaust even the most patient 
scientist. Also, production environments are kind of delicate, and debugging
them is not usually an option. Best scenario? Reproduce the event loss in your
machine where you can mess up with almost everything with no fear of breaking
anything valuable. Drawbacks? Your local environment is probably nothing like
the production environments and things that happen there might never reproduce
here. Our objective? Lose events in our machine with a simple, light and easy
to reproduce experiment. Not an easy task, but it's where we want to be.

Logs are our friends. Not only because they pay for our bills, but also because
they usually contain vital information to find out what's happening. All we've
got so far is a very heavy experiment causing a very little event loss only in
some selected environment. Unfortunately, the original environment where the
issue consistently reproduced was highly secured, and few people had access to
it, so we ended up building a replica in a public cloud. And luckily for us, the
experiment led to event loss, and looking into the LB logs we found a few traces
like this:
```
TCP source socket error: Error: read ECONNRESET
```
The sender and the LB communicate via TCP (secured with TLS), and this
[error is triggered](https://nodejs.org/api/errors.html#errors_common_system_errors)
when the connection is forcibly closed by the sender. What does "forcibly" mean
in this context? Well, both TCP and TLS are protocols, and their implementations
are expected to comply with these protocols. In particular, closure of
connections has to follow specific rituals, and errors occur when these rituals
are performed poorly. The LB ends up destroying the socket after an error like
this, because
[no more data is expected](https://nodejs.org/api/net.html#net_event_error_1)
after receiving an `error` event.

This trace was a very good scapegoat because:

1. Abrupt interruptions of TCP/TLS sockets can easily lead to data loss.

2. The error was triggered 3-4 times per experiment. One time per hour
because our python SDK recycles the TCP socket once per hour, and then one time
at the end of the experiment. This would perfectly account for the amount of
events lost (tens, sometimes hundreds).

# Saving Private Ryan (1998)

Gear up because we are getting into the battlefield. Yes, we do have a good
candidate to throw to the wolves, but we still have to build a solid case
against him. It's time to simplify things and design much more targeted
experiments. Replace the 3 hours experiment with a 3 seconds gig that we can
perform again and again. Trigger a scientific method avalanche. We managed to
[create a repo](https://github.com/tufosa/nodejs_econnreset_issue) where we
isolated the behaviour of both sender and receiver that was causing the error.
And we succeeded in reproducing it with a very simple experiment!

The challenge was then to get rid of the dark clouds approaching from the North.
First of all, the error only happened with TLS sockets, never with plain TCP
sockets. So it has to be something that the TLS implementation of the receiver
is expecting, but not getting. Also, receivers launched with Nodejs 8.x.x never
failed, while Nodejs 12.x.x consistently did. Fingers stopped pointing at the
LB and started pointing at Nodejs itself. The same (simple) Python client
triggered different behaviours in the same (simple) Nodejs receiver depending
on the version of Nodejs used.

Was there anything wrong with our sender? Well, in fact there was. The official
Python documentation says two things about closing a socket:

1. If you have a
[plain TCP socket](https://docs.python.org/3/library/socket.html#socket.socket.close),
you should call `socket.shutdown(how)` before `socket.close()` in order to close
it in a “timely fashion“.

2. If your socket is
[wrapped into a TLS context](https://docs.python.org/3/library/ssl.html#ssl.SSLSocket.unwrap),
calling `socket.unwrap()` will perform the TLS shutdown handshake. Something
that seems very polite before saying goodbye.

We were doing neither of the above.
[Adding](https://github.com/DevoInc/python-sdk/commit/cff1ecb9c90598462645d76c5991e68654dc764c)
`socket.shutdown(how)` to our python-sdk seemed to solve the issue, so we
assumed that the TLS layer was unwrapping itself, but there were still a few
jungles to trim:

1. Nodejs 8.x.x seemed pretty stable regardless of how abruptly connections
were closed from the client side. Nodejs 12.x.x behaved very
non-deterministically. Is this something we should report to Nodejs?

2. Even with Python closing the socket in a "timely fashion", the bytes
received on the Nodejs side fluctuated a bit when the CPU was busy enough,
although the ECONNRESET error never showed up. This, of course, only happened
with Nodejs v12.x.x.

3. Using Nodejs also to send events (closing the connections abruptly on
purpose) revealed an even weirder behaviour: sending and receiving with Nodejs
12.19.0 seemed pretty seamless, but using Nodejs 12.18.4 to send triggered the
`ECONNRESET` error consistently (!!!!).

4. We wrote a simple sender in Java and we also reproduced the issue. If we
closed the socket by the book, then everything went smooth, both with the
receiver in Nodejs 8.x.x and Nodejs 12.x.x. But if instead of closing the socket
politely we interrupted the thread that was running the connection, then Nodejs 
8.x.x would behave ok, but Nodejs 12.x.x would show data loss and `ECONNRESET`s.

# Jaws (1975)

It's time to pull out the heavy artillery, and when you are dealing with a
TCP/TLS poltergeist, this means calling in
[the shark](https://www.wireshark.org/). Observing and analyzing what's going
down the wire is a toilsome task. Especially when TLS is involved. Performing
the Python experiments with the shark watching revealed that the amount of data
interchanged when closing the socket was growing as we refined our manners:

* a simple `socket.close()` produced a minimal amount of data
    
* prefixing `socket.shutdown(how)` would make the data interchange grow
    
* prefixing `socket.unwrap()` would make the amount of data reach its maximum
    
Using Nodejs on the sender side showed that closing the connection with Nodejs
12.18.4 produced less data than doing it with Nodejs 12.19.0. The
[TLS RFC](https://tools.ietf.org/html/rfc8446#section-6.1) says that in order to
close a connection politely you need to send a `close_notify` message before
attempting to close the TCP connection and it looked like some senders were
forgetting to do so, and therefore triggering an error in the receiver. But why
didn't this error show when the receiver was running on Nodejs v8.x.x?

Indetermination is also a potential result of a scientific experiment. Pick a
random university and ask anyone in the Quantum Physics department. Observing
the wire with the shark altered the results of the experiments. How? Well, the
shark is a very powerful tool, but it absorbs a lot of CPU to perform its
duties. And this made the experiment fail more often. If we turned it off,
then some of the experiments might not even fail. Why was this happening?

# Hollywood Ending (2002)

The shark was talking and we just needed to point our ears in the right
direction. The combination of circumstances that made our experiment fail had
something in common. But it was buried too deep for us to see in the first
place. Digging with our scientific shovel was getting us closer to the treasure.
Sniffing the wire made us realize that successful experiments showed a couple
of extra application packets traveling from the client to the receiver and back.
In failing experiments, these strange packets never showed up. We began to think
that sometimes the client was failing to perform some of the dances required by
the TLS closing ceremony, and the receiver was getting angry, asking for an
early divorce and losing some of the progeny on the way. But why? What was the
common factor of all the successful/failing experiments? And what was the
difference? Suddenly a few bulbs started to turn on in the dark forming the
initials T, L and S. Yes, all the experiments that failed were using TLS1.3 to
encrypt the wire. Running the receiver with Nodejs 8.x.x guaranteed success
because Nodejs 8.x.x does not support TLS1.3. We confirmed this by running
experiments using Nodejs 12.x.x in the receiver, but forcing the Python/Nodejs
client to use TLS1.2. They all succeeded. Then, where is the bug (if any)?
Where is the problem? How can we fix it?

We detected at least a couple of deficiencies that could be improved. And both
of them seemed to fall in the “compiler/interpreter“ category. Both, Python 3
and Nodejs <= 12.8.4 forget to send the `close_notify` message when they are
acting as senders in a TLS1.3 play and the show is about to end. Nodejs fixes
the issue in v12.19.0, and Python behaves correctly if `socket.unwrap()` is
called before closing the socket (`socket.shutdown(how)` alone does not do the
trick, maybe it’s just a matter of improving the docs). Senders were not closing
properly the communication with the receiver when using TLS1.3, but the receiver
(Nodejs) could also be a bit more resilient with the small negligence of the
sender and obviate the fact that there is a protocol packet missing in exchange
of making sure that no data is lost.

And what about the disturbing fact that even with failing experiments, failures 
do not occur all the time? Why do (failing) experiments fail more often when
the CPU is busier? It's really very hard to tell, but my wild guess would be
that it has to do with the order in which the kernel "sees" the incoming data:
if it "sees" all the data before the lousy closure, then there will be no error,
but if it "sees" a lousy closure when there are still pending data, then it will
signal the error. It's just an hypotheses, and proving it would require diving
way deeper than we did.

# The Matrix (1999)

Blue pill or red pill? Let’s have both of them. Or neither. Complex problems
demand looking at them from every possible angle. From inside and outside the
matrix. As [Bruce Lee](https://youtu.be/cJMwBwFj5nQ) would put it, you need to
"become" the application you are trying to debug and travel at the speed of bit
through the logic silicon pipes of the computer. Pills are not given away by
some Morpheus stuffed in a leather jacket at the beginning of the trip, but they
are rather earned during the journey. And they are neither red nor blue. Most of
the times they are grayish and they are called wisdom pills. Here are some we've
collected lately. Use them at your own risk.

* **Pill 1**: well designed experiments extend your knowledge. Lame or biased
experiments could reinforce your ignorance. Spend time thinking what you want to
prove and how to do it.

* **Pill 2**: truth is a very elusive concept. Paradoxically, now more than
ever. In SW engineering, the truth is always written in the code.

* **Pill 3**: if the code is too hard to read, consider it as a black box and
perform as many experiments as you need to understand it.

* **Pill 4**: everything is code: our applications, the compilers/interpreters
that run them, the runtime environments, the OS libraries, the OS itself, etc...

* **Pill 5**: everything has bugs, even the most proven SW.

* **Pill 6**: when you start a murder investigation, do it with an open mind and
start over a blank page. Don't make any assumptions. Drop all your prejudices.

* **Pill 7**: be a methodical and rigorous investigator. Support all your
findings with sound experiments. Then wash, rinse and repeat. 

* **Pill 8**: if you happen to find a killer, report it.

* **Pill 9**: if you happen to find a corpse, identify it before dumping it.

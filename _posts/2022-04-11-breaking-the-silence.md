---
layout: post
title:  "Breaking the Silence"
date:   2022-04-11
categories: meta,ramblings
---

It has been 9 months since my last post. Not because I didn't do anything
interesting, but more because I had left something hanging since my first post.

In my first post, I promised to showcase a keyboard firmware project, and in
fact, I did start drafting a post that I anticipated would be the first of a
several-part series on the topic. But around that time, I ran into [some
obstacles][avr] that made porting the framework to AVR much more difficult in
the near term, and because of how ubiquitous AVR microcontrollers are in
hobbyist keyboards, it pretty much halted my progress and I lost interest.

After that project came and went, I did a lot of other interesting things, like
[Global Clock][global-clock], [Bad Apple in Logic World][lw-badapple], and
[SMM2 Stats][smm2-stats], all of which I have contemplated writing posts to
explain some of the technical details, and I might still write them in the
future. But every time I came back to my site, that first post haunted me, and
I couldn't bring myself to write about something else before addressing that.

Now, realistically, there were probably very few people who even knew about
this promise. Some of my internet friends may have been mildly interested in
hearing about it. But regardless, the weight that I ascribed to that promise
was very disproportionate. And with my track record of leaving projects
hanging, I'd better get used to dealing with this kind of scenario, and the
best way I can think of is _just write a damn post!_ If the project was
interesting enough to mention on my blog, I probably have a good reason for
discontinuing or postponing it, and writing about those problems might solicit
feedback or advice from others, or at the very least it will provide closure for
me and for anybody else who might incidentally be interested.

I would like to be more active on here in the future; I have a lot of projects
from the past year that could be interesting to write about, and I'm currently
working on a very ambitious project that I hope to demonstrate soon. My next
post might be about any one of these, but, no guarantees ;)

# KBForge post-mortem

If you're curious to know exactly what happened back then, this section is for
you:

In mid-2021, I started working on [KBForge], a framework for writing
keyboard firmwares. Its design took a lot of inspiration from [QMK], a
framework that is very popular in the keyboard community, but I had some goals
that I wanted to achieve with writing my own. In brief, the main goal was
_modularity_, going against the monorepo pattern used by QMK, and allowing
keyboard drivers and user-specific configurations to be implemented as
separate downstream libraries, using Rust's awesome dependency management
system.

The trouble came when I tried to actually write the driver modules for
my keyboards. The embedded ecosystem in Rust has been evolving steadily, and
projects like [usb-device] have helped with that immensely by allowing USB
device class implementations to be developed independent of the underlying
hardware specifics. However, the main area where Rust (and also LLVM) is
lacking is in support for AVR, where there are a few [critical outstanding
bugs][avr] that cause miscompilations in the usb-device crate, making it
unusable on that architecture. And unfortunately, all of the programmable
keyboards I own use AVR architectures. You'd be hard-pressed to find one that
doesn't, they're in pretty much everything on the keyboard market, although
there are a few ARM-based offerings.

That's not necessarily a problem, I could just write my KBForge drivers using
the low-level interfaces directly. I spent a little while trying to get the USB
peripheral up and running but I didn't have much luck. Ultimately, I decided
pursuing this option was not worth my time, and so I stopped working on it and
moved on to other things.

In general, I think the core of the framework, the part that links all the
modules together, is complete and totally usable. The options for user
configuration are definitely lacking, but I expected that area to evolve as the
various use cases appeared in practical applications. The only blocking issue
at the moment is getting USB HID working on AVR. So if the Rust AVR situation
improves, I might come back to it at some point, but for now the project is on
hiatus.

[avr]: https://github.com/Rahix/avr-hal/issues/40
[global-clock]: https://agausmann.github.io/global-clock
[lw-badapple]: https://youtu.be/R5hdmBmmn0g
[smm2-stats]: https://github.com/agausmann/smm2-stats
[KBForge]: https://github.com/agausmann/kbforge
[QMK]: https://qmk.fm
[usb-device]: https://github.com/mvirkkunen/usb-device

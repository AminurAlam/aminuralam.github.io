+++
title = "re: Wayland set the Linux Desktop back by 10 years"
date = 2026-03-18
draft = false
+++

<https://omar.yt/posts/wayland-set-the-linux-desktop-back-by-10-years>

Wayland has been a broad misdirection and misallocation of time and
developer resources at the expense of users. With more migration from
other operating systems, the pressure to fix fundamental problems has
become more prominent. After 17 years of development, now is a good time
to reflect on some of the larger promises that have been made around the
development of Wayland as a replacement for the X11 display protocol.

If you're not in this space, hopefully it will still be interesting as
an engineering post-mortem on taking on new greenfield projects. Namely:
What are the issues with what exists, why can they not be fixed, what do
we hope to achieve with a new project, and how long do we expect it to
take?

# Background and the problems with X11

If you're already familiar with X11 and Wayland feel free to skip to the
next part.

For people not familiar with Linux, here's a quick rundown of the terms
in this space, roughly in the order of highest-level to lowest-level:

- Applications
  - These are the things you want to run
  - Examples: Chrome, Steam, OBS
- Desktop Environment (DE)
  - This is what manages things like window theming, notifications, task
    bars, etc.
  - Examples: KDE, GNOME
- Compositor
  - This is what layers windows on top of each other and does
    animations, graphical effects, etc.
  - Examples: Compiz, KWin (for KDE), Sway, i3
- Display Server
  - This is the thing that manages the display, it also abstracts away
    some of the hardware details so all the above work on NVidia, AMD,
    Intel, etc.
  - Examples: X11, Wayland
- Kernel / Operating System
  - The lower-layer thing that manages hardware resources, for us this
    is Linux
  - Examples: Linux, FreeBSD

The above is not a complete list, but it's enough to give some framing
for understanding that X11 is a fundamental piece of most Linux
environments.

X11 is currently still the most common popular display server in the
Linux ecosystem. It was developed in the mid-1980s and, as legacy
projects tend to do, has accumulated functionality that makes it
difficult to maintain, according to the developers.

So, in 2008, Kristian Høgsberg started the project that became known as
[Wayland](https://www.phoronix.com/review/xorg_wayland). Wayland (in
theory) replaces the display server, as well as some parts of the
compositor and desktop environment with a simpler display protocol and
reference implementation. The original conceit behind Wayland is to only
implement what is needed for a simple Linux desktop. The original
implementation was a little over 3,000 lines of code.

Sounds great!

# So what happened?

It's 2026, and Wayland has reached a market share of around
[40-50%](https://x.com/LundukeJournal/status/1994963236827865224), or
closer to [50-60%](https://chandanadev.com/linux_2) depending on your
source. I would argue a product that has taken 17 years to gain
substantial marketshare has issues hindering adoption. Compare the
development of Wayland to a similar project for managing audio:
PipeWire. Within ~8 years, every alternative has been mostly replaced.
It's been adopted as the default in Ubuntu since 22.04, roughly 4 years
after it first launched!

These are the most common issues I have seen from the perspective of a
user, so I will try to stay light on what I consider to be mostly
irrelevant technical details and instead focus on larger issues around
the rollout and design of Wayland.

## Wayland is more secure (which means I can't do anything)

The reason I use Linux and other Unix-likes is that they give me the
ability to do whatever I want on my system, including making mistakes!
So why is my **display server** telling me that certain applications
_that I installed and chose to run_ aren't allowed to talk to each other
in the name of security?

There are multiple cases of this: OBS can't screen record (it segfaults
instead), I can't copy-paste, and I can't see window previews unless
everything implements a specific extension to the core protocol.

The actual "threat model" here is baffling and doesn't seem to reflect a
need for users. Applications are not able to see each other's windows,
but they're not able to interact _in any other way_ that could
potentially cause problems?

I also don't care for the "security" argument when parts of the core
reference implementation are written in a memory-unsafe language. To be
clear, **I am not saying that software written in C is bad**, I'm
specifically calling out that making a **security** arugment about
software then repeating decisions of the previous (40-year-old)
implementation is a bad look.

## Wayland is more performant (as long as you don't use NVidia)

Several of the design decisions that Wayland makes are claimed in the
name of performance. Namely, collapsing many layers is supposed to
reduce the number of copies when moving data between different
components.

However, whatever the reason, these performance gains haven't
materialized, or are so anecdotal in either direction that it's
difficult to claim a clear win over X11. In fact, you can find
[examples](https://mort.coffee/home/wayland-input-latency/) showing
roughly a **40% slowdown** when using Wayland over X11! I'm sure there
are similar benchmarks claiming Wayland wins and vice versa (happy to
link them as well if provided).

The problem is that, even if Wayland was _twice_ as fast, it doesn't
compare to improvements in hardware over the same period. It would've
been better to just wait! The performance improvements would have to be
much more substantial for this to be a reasonable argument in favor of
Wayland. The fact that the question _even exists_ as to whether one or
the other is faster after a substantial period is an obvious failure.

Additionally, those performance gains don't matter if I'm not able to
make use of them. For example, if I'm using [the most popular graphics card vendor](https://jpetazzo.github.io/2024/04/29/six-months-with-wayland-sway-i3/)
in my system, I shouldn't expect things to work out of the box.

## Wayland isn't just "one thing" (because Wayland doesn't do anything)

One rebuttal I've heard is that it's not an issue with Wayland, it's an
issue with the compositor/extension/application. After all, "Wayland"
isn't a piece of software, it's a simple protocol that other software
chooses to implement!

Of course, what this means in practice is that there are multiple
(usually incompatible) implementations of multiple different standards.
Maybe this would be fine if the concept of a desktop operating system
was completely new and unknown, but users balk when discovering things
like [drag and drop](https://wayland.app/protocols/xdg-toplevel-drag-v1)
or [screen sharing](https://wiki.hypr.land/Useful-Utilities/Screen-Sharing/) are
not natively supported and are essentially still in "beta" status.

Instead of providing a better way of doing something, common features
are not supported _at all_, and instead it's the job of everyone else in
the ecosystem to agree on a standard. That's not a stunning argument in
favor of replacing something that already exists and that _has already
been standardized in X11_!

## Wayland is still under active development

Wayland has been around for _only_ 17 years, while X11 has closer to 40
years of development behind it. Things are still under development and
_obviously_ will get better, so why complain about issues that will
inevitably get fixed?

**_Because it's been 17 years and people are still running into major
issues!_**

I was unpleasantly surprised when using KDE Plasma that the default
display server had been changed to Wayland. I noticed very quickly on
startup when I encountered enough graphical hitches to realize I was
running Wayland and quickly switch back. Anecdotal experience is not
enough to say this is a broad issue, but my point is that when an
average user encounters graphical issues within 60 seconds of using it,
maybe it's not ready to be made the default! It was only within the last
6 months that OBS stopped segfaulting when trying to launch on Wayland.
I assume I'm in decent company when even the developer of a major
compositor is [still not able to use Wayland in 2026](https://michael.stapelberg.ch/posts/2026-01-04-wayland-sway-in-2026/).

The number of "simple" utilities that seem partially supported or
half-baked is incredible and seems to be a massive duplication of
effort. The tooling around X11 that has been developed over the last 40
years seems to have been completely dropped and no alternative has been
provided. Instead of providing an obvious transition path, Wayland has
introduced even more fragmentation.

Older software that has a ton of "legacy cruft" has been _tested_ and
bugs have long since been _fixed_. I fully believe with another 20 years
of development things will be better. The problem is that I am being
forced to make the switch **_now_**. See: The push from [KDE and RedHat to Wayland and dropping support for older technologies](https://blogs.kde.org/2025/11/26/going-all-in-on-a-wayland-future/).

## Opinion of Wayland developers

[This post](https://web.archive.org/web/20211008180535/https://drewdevault.com/2021/02/02/Anti-Wayland-horseshit.html)
probably best encapsulates the developer opinion towards users trying to
migrate to the next iteration of the Linux desktop:

> Maybe Wayland doesn’t work for your precious use-case. More likely, it
> does work, and you swallowed some propaganda based on an assumption
> which might have been correct 7 years ago. Regardless, I simply don’t
> give a shit about you anymore.
>
> ...
>
> We’ve sacrificed our spare time to build this for you for free. If you
> turn around and harass us based on some utterly nonsensical conspiracy
> theories, then you’re a fucking asshole.

It's even more ironic compared to the post [made a week
later](https://web.archive.org/web/20230605024414/https://drewdevault.com/2021/02/09/Rust-move-fast-and-break-things.html),
expressing the same frustration with the Rust community that people have
with Wayland!

Drew has since deleted this post, so I understand if he no longer stands
by those opinions. However, it's a representative slice of developer
sentiment towards users that are now being forced to use unfinished
software. Entitlement and bullying of open-source maintainers is not
appropriate, and it's understandable that the developers lash out after
feeling beaten down by entitled users. However, to have some sympathy on
the user side, it's likely born out of frustration of being forced to
use the new hotness and then encountering breaking bugs that are
impossible for the average user to work around.

It is not the fault of the original developers for building what they
wanted to build. I think it's important to keep in mind that they didn't
necessarily _choose_ for Wayland to become as popular as it has or the
foundation for the desktop of the future. See the diagram below:

![cat showing flowchart of open-source
projects](https://omar.yt/images/gaslighting-cat.avif)

Having Wayland as a developers-only playground is fine! Have fun
building stuff! But the second actual users are forced to use it expect
them to be frustrated! At this point I consider Wayland to be a fun toy
built entirely to pacify developers tired of working on a finished
legacy project.

# Predictions and looking forward

## Things to be excited about

Since most of this post has been overwhelmingly negative against the
development of Wayland, it's instead better to learn as much as possible
and look forward towards "what would I want to be able to do". Windowing
technology is absolutely not "done", and instead of following other
operating systems, it would be fantastic if Linux could do things no
other environment could do.

For example, being able implement non-rectangular windows, exposing
context actions (similar to MacOS), or making it easier to automate or
script parts of the desktop environment would be incredibly exciting!

It's difficult to overstate the amount of progress in support for
gaming, new (and old) hardware, as well as the amount of overall
"polish". Every developer should be proud to be a part of that!

# Conclusion

After 17 years, Wayland is still not ready for prime time. Notable
breakage is being
[documented](https://gist.github.com/probonopd/9feb7c20257af5dd915e3a9f2d1f2277),
and adoption has been correspondingly slow.

For some users the switch is seamless. For others (including myself),
they tend to bounce off after encountering workflow-breaking issues. I
think it's obvious at this point that the trade-offs have not been worth
the hassle.

My prediction is that within the next 5 years the following will be
true:

1.  Projects will drop Wayland support and go back to X11
2.  There will be a new display protocol that displaces both X11 and
    Wayland
3.  The new display protocol will be a drop-in replacement (similar to
    XWayland)
4.  Fragmentation will still be an issue (this one's a freebie)

See you in 2030 for the year of the Linux Desktop.

## Additional reading

Included are some of the links referenced in this post as well as some
additional reading.

- Think twice before abandoning X11. Wayland breaks everything!
  - <https://gist.github.com/probonopd/9feb7c20257af5dd915e3a9f2d1f2277>
  - <https://lobste.rs/s/2brhes/think_twice_before_abandoning_xorg?utm_source=chatgpt.com>
  - <https://news.ycombinator.com/item?id=45222251>

- Wayland is flawed at its core and the community needs to talk about it
  - <https://www.reddit.com/r/linux/comments/1pxectw/wayland_is_flawed_at_its_core_and_the_community/>

- Going all-in on a Wayland future
  - <https://blogs.kde.org/2025/11/26/going-all-in-on-a-wayland-future/>

- Unpopular Opinion: Linux world felt stable until Wayland/GTK4 arrived
  - <https://www.reddit.com/r/linux/comments/1o5fmxv/unpopular_opinion_linux_world_felt_stable_until/>

- On Abandoning the X Server
  - <https://news.ycombinator.com/item?id=24920183>

- I'm tired of this anti-Wayland horseshit (deleted)
  - <http://web.archive.org/web/20211008180535/https://drewdevault.com/2021/02/02/Anti-Wayland-horseshit.html>

- Move fast and break things as a moral imperative (deleted)
  - <http://web.archive.org/web/20230605024414/https://drewdevault.com/2021/02/09/Rust-move-fast-and-break-things.html>

- Can I finally start using Wayland in 2026?
  - <https://michael.stapelberg.ch/posts/2026-01-04-wayland-sway-in-2026/>

- KiCad and Wayland Support
  - <https://www.kicad.org/blog/2025/06/KiCad-and-Wayland-Support/>

- Wayland Finally Gains Ground: Why 2025 is the Year of Desktop Linux
  Migration
  - <https://chandanadev.com/linux_2>

- What is the true adoption rate of Wayland
  - <https://x.com/LundukeJournal/status/1994963236827865224>

- Simplified history of X Hector Martin
  - <https://web.archive.org/web/20240110152647/https://social.treehouse.systems/@marcan/110371565062371963>

- Wayland breaks your bad software
  - <https://web.archive.org/web/20240112184154/https://oro.gay/posts/wayland-breaks-your-bad-software/>

- on abandoning the X server
  - <https://ajaxnwnk.blogspot.com/2020/10/on-abandoning-x-server.html>

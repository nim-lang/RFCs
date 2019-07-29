# TL;DR

There are two incompatible async I/O libraries available for Nim, which is not
a good thing given the fundamental layer at which these libraries operate. The
risk is that this might split the library ecosystem. I hope to find a
resolution for this situation that makes everybody happy and foster the Nim ecosystem.

# Background

Over the last few weeks/months I have seen a number of discussions[3] on the Nim
forum and the #nim IRC channel about the state of the Nim async library. After
speaking with some (but not all!) of the people involved I figured it is time
to make this into an official discussion to see how this can be resolved and we
can move forward.

Here is the back story:

- The Nim stdlib comes with a library for doing async (network) I/O, consisting
  of asyncmacro, asyncdispatch, asyncnet and ioselectors. This has been part of
  the standard library for a few years, and is the base layer on which things
  like asyncHttp are built. A well-known application using stdlib async is the
  Nim forum itself.

- Implementing async I/O is not trivial, and the Nim libraries has had some
  bugs, missing features or other issues. Some were fixed and addressed in the
  past, but some are still open [4][5]. Also, some issues are more serious then
  others: some are considered blockers[2][6] for a 1.0 Nim release.

- A bit over a year ago a third-party implementation of async I/O called
  `Chronos`[1] was created by people who were not content with the quality
  and stability of the async layer at that time and found that these issues
  were not timely addressed by the Nim maintainers. [7][8][10][11][12]

- Chronos was partially forked from the Nim async library, while other parts
  were rewritten from scratch. Chronos's API is not compatible with Nims async
  lib, as API changes were deemed necessery to fix underlying issues.
  
- Chronos has seen steady development over the last year, diverging more and
  more from Nim async, but also fixing some of the issues that were in the Nim
  standard library. Chronos is developed at Status, the main Nim sponsor, as a
  core component of Nimbus, a client for the Ethereum blockchain and
  nim-libp2p, a standard for P2P protocol.

# The problem

Given the above history, the Nim community has now two different but
incompatible Async implementations. Async I/O is an important piece of core
infrastructure which provides the foundation for a lot of other protocol
implementations. Having two incompatible and mutually exclusive async
frameworks is probably not a good thing for the community - should I implement
my new protocol on top of the one or the other? Or both? What if I choose the
one but want to use a 3d party lib that depends on the other?

Recently things have slightly "escalated" in various threads on the Nim forum -
users asking questions or noting issues with Nim async are pointed to Chronos
as a solution to their problem. While this answers is perfectly legit and might
help the user with their problem, this might give rise to confusion among new
users, and does not help the current stdlib to evolve and get better.

# Where to go from here?

I think we should get this resolved, and I hope to start a discussion here so
we can figure out the right thing to do. Some questions?

- Do we accept the status quo? There are two available implementations, and
  users should be free to choose?
- Should the Nim stdlib drop the current implementation in favour of Chronos?
  If so, when? What about Nim 1.0?
- Should we put work into getting the two implementations to cooperate?
- Can we add a Chronos-compatibility layer on top of the Nim libs, or add a
  Nim-compatibility layer on top of Chronos?

Here are the important points that I think should be addressed in this
discussion:

- API stability: The Nim Async layer tries to provide a stable API on which
  other protocols have been implemented. Chronos has had good reasons to make
  changes, but this broke compatibility with existing and future Nim-async
  libraries.

- Code quality: At this time Chronos is seeing a lot of active development
  backed by a commercial party. This simply allows Chronos to fix more of the
  technical debt and add more features. A lot of Nim stdlib development is done
  by volunteers which have to do this in their free time for no pay.

- Nim 1.0 is coming up. We probably want to resolve this and have a good story
  before the release.

Given the nature of past discussions about this subject, I would like to ask
all parties involved to refrain from having any of these elsewhere while this
RFC is on the table. I think no one benefits from a forum full of threads
where the good and the bad of implementations are compared and mud gets
thrown around.

# References

Chronos repo:

- [1] https://github.com/status-im/nim-chronos

Recent forum discussion:

- [3] https://forum.nim-lang.org/t/5048

Relevant open nim stdlib issues:

- [2] https://github.com/nim-lang/Nim/issues/8080
- [4] https://github.com/nim-lang/Nim/issues/3590
- [5] https://github.com/nim-lang/Nim/issues/4123
- [6] https://github.com/nim-lang/Nim/issues/7126
- [9] https://github.com/nim-lang/Nim/issues/7193

Closed Nim stdlib issues (inc days-to-fix)

- [7] https://github.com/nim-lang/Nim/issues/7758 (113)
- [8] https://github.com/nim-lang/Nim/issues/7197 (273)
- [10] https://github.com/nim-lang/Nim/issues/7192 (281)
- [11] https://github.com/nim-lang/Nim/issues/6846 (266)
- [12] https://github.com/nim-lang/Nim/issues/6929 (252)



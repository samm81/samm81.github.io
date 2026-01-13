---
layout: post
title: vibecode your github issues
---

this new era of vibecoding has subtly changed my relationship with my computer. instead of (or, more often, in addition to) filing bugs and feature requests, I'm just throwing them at an agent.

a high quality bug report / feature request is the perfect fodder for vibecoding. most agents will even generate their own [`AGENTS.md` (or equivalent) file][20] nowadays so the workflow is dead simple: clone the repo, run your favorite agent, have it generate its own `AGENTS.md` file, then copy/paste your issue/feature. (or perhaps you haven't written that up yet - in which case you have the agent write it!)

[20]: https://agents.md/

you get more out of this when you run more open-source software, I'm already the type of guy that likes to run the [non-standard browser][9] [non-standard window manager][10] on the [non-standard linux distro][11] so pretty much my whole computer is vibe-codable.

[9]: http://firefox.com/
[10]: https://swaywm.org/
[11]: https://voidlinux.org/

and when you get into the habit, when your first instinct upon encountering a friction point is "I can make this better!" rather than "well guess I'll go file a ticket and hope it gets looked at", your computer truly becomes a tool of infinite potential.

theoretically this has always been true - so long as you used open-source software (or wrote your own) you could make the computer do whatever you dreamed of it. but in practice it was never really worth the time & effort to fix that bug or add that feature - especially when the codebase for the tool you had in mind was in a language you weren't familiar with, or was overly complex.

not anymore! just vibecode it 🧑‍💻

another added benefit is that you can often actually contribute your results back to the maintainer - solving your bug that you opened for them. I don't know go, and I certainly didn't know anything about the [kopia][1] codebase, but I [opened a pr][2] to solve [my own issue][3]. the maintainer did a good amount massaging, but in the end it got merged!

[1]: https://kopia.io/
[2]: https://github.com/kopia/kopia/pull/4866
[3]: https://github.com/kopia/kopia/issues/4831

I know python, but I was not about to familiarize myself with the entire [vdirsyncer][4] codebase in order to fix some of the bugs I was running into. But I could sure point an agent at it, resulting in [four merged prs][5].

[4]: https://github.com/pimutils/vdirsyncer
[5]: https://github.com/pimutils/vdirsyncer/pulls?q=is%3Apr+author%3Asamm81+is%3Aclosed

I do know bash, but using an agent made [these][6] [two][7] pull requests to [pdiffjson][8] absolutely trivial - it turned what might have been a half hour of effort into two minutes.

[6]: https://github.com/jlevy/pdiffjson/pull/6
[7]: https://github.com/jlevy/pdiffjson/pull/7
[8]: https://github.com/jlevy/pdiffjson/

so try it: the next time you open an issue on a github repo, just copy/paste it into a viebcoding agent!

future work: I've really been into the idea of [nix][12]-ifying my whole computer and letting a vibecoding agent loose on the whole thing, since if it f\*cks the whole thing I can just use `nix` to rollback anyways 💫

[12]: https://nixos.org/

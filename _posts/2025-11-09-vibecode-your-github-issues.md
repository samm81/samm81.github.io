---
layout: post
title: vibecode your github issues
---

this new era of vibecoding has subtly changed my relationship with my computer. I already have a habit of running almost exclusively open-source software on my [void-linux][9] box, and open-source software is fundamentally vibe-codable. encounter a bug? need a new feature? not sure what config option to use and don't want to spend an hour reading the manpage? clone the repo and vibecode it. vibecoding changes open-source from something you file issues against into something you directly adapt.

[9]: https://voidlinux.org/

open-source projects are, in many ways, the perfect place to loose a vibecoding agent.

1. you often use projects that are written in languages that you don't know - rather than taking a weekend to learn the basics of `go`, just have the agent make your changes.
2. the code never needs to be seen by anyone else - you don't have to go through a peer-review by your judgy coworkers. it can be a pile of vibecoded garbage and it doesn't matter - so long as it does what you need it to do. (you don't read the codebase of every open-source project you use, do you?)
3. if you _do_ choose to contribute back to the repository, you have maintainers who will make sure that the code is not a pile of vibecoded garbage.

there are definitely dangers - if the tool you are vibecoding has access to sensitive information or subsystems on your computer it could potentially do some damage. follow standard practices: isolate your test environment, ensure you have backups, maybe run a `git diff` and ask a different agent what the changes are and if they are possibly harmful. but I don't think these are show-stoppers.

assuming you already know the basic vibecoding loop, here’s a more concrete workflow that’s worked well for me:

setup (once per project):

1. clone the repo
2. start up your choice of agent, and have it generate its own [`AGENTS.md` file][10].
3. before diving into vibecoding, figure out how to build the project.
4. (optional) symlink your newly built binary into your `PATH` somewhere - perhaps as `originaltoolname-vibe` or `originaltoolname-dev`
5. go run your new `-vibe` binary somewhere and verify it works

[10]: https://agents.md/

fix it:

6. before touching code, have the agent write a bug report or feature request as if you were going to file it; this issue becomes the spec
7. (optional) go submit the bug report / feature request to the repository's issue tracker.
8. use the agent to solve that issue.
9. build and test using your `-vibe` binary (taking necessary precautions as needed - back up your data, maybe make a new isolated test environment).

after it works (optional):

10. start using the `-vibe` binary instead of the official one. or, drop the `-vibe` and put it earlier in your `PATH`. or, if you're feeling very daring, replace the original binary with your new vibecoded one.
11. (optional) push your changes up and open a pull-request that solves your own bug/feature.

since I started defaulting to this workflow I've had several success stories. I don't know `go`, and I certainly didn't know anything about the [kopia][1] codebase, but I [opened a pr][2] to solve [my own issue][3]. the maintainer did a good amount massaging, but in the end it got merged!

[1]: https://kopia.io/
[2]: https://github.com/kopia/kopia/pull/4866
[3]: https://github.com/kopia/kopia/issues/4831

I know python, but I was not about to familiarize myself with the entire [vdirsyncer][4] codebase in order to fix some of the bugs I was running into. but I could sure point an agent at it, resulting in [four merged prs][5].

[4]: https://github.com/pimutils/vdirsyncer
[5]: https://github.com/pimutils/vdirsyncer/pulls?q=is%3Apr+author%3Asamm81+is%3Aclosed

I do know bash, but using an agent made [these][6] [two][7] pull requests to [pdiffjson][8] absolutely trivial - it turned what might have been a half hour of effort into two minutes.

[6]: https://github.com/jlevy/pdiffjson/pull/6
[7]: https://github.com/jlevy/pdiffjson/pull/7
[8]: https://github.com/jlevy/pdiffjson/

open-source has always let you reshape the code to your needs - but usually the barrier to entry was high enough that this was mostly just a theoretical reassurance. vibecoding turns it into a practical reality.

"I wish this tool did X" is now an afternoon project, not a research project.

try fixing one of your own issues before opening a ticket. you might end up fixing someone else’s too.

---

future work: I've really been into the idea of [nix][11]-ifying my whole computer and letting a vibecoding agent loose on the whole thing, since if it f\*cks the whole thing I can just use `nix` to rollback anyways 💫

[11]: https://nixos.org/

---
layout: post
title: location-ledger
---

for the past three years I've been living as a digital nomad, and I've kept a [manual log][1] of everywhere I've been. this is exactly the sort of stupidly specific problem that software should solve for me, and exactly the sort of software I probably wouldn't have bothered writing before coding agents.

[1]: https://nomads.com/@athousandcups

besides satisfying my compulsion to keep records, location turns out to be useful metadata for the rest of my life. my [mood test][2] chat-bot, for example, records timestamps only in utc; knowing where I actually was tells me what local day/time I took it. the same is true for all kinds of other personal data. timestamps are a lot more useful when you can attach "I was in taipei" or "I was in mexico city" to them.

[2]: https://en.wikipedia.org/wiki/Positive_and_Negative_Affect_Schedule

so I [vibecoded it][3]. why not? it's a simple script that I can upload to my vps and run once a day. plus, the agent actually had some cool tricks, like [requiring the dependencies in the script inline][4]. and the code might not even be halfway bad since I set it up with [some][8] [skills][9].

[3]: https://github.com/samm81/location-ledger
[4]: https://github.com/samm81/location-ledger/blob/404221ac48d4c05db9a3100bb44d8bd6e5232426/timeline_cities.py#L4
[8]: https://github.com/trailofbits/skills/tree/main/plugins/modern-python/skills/modern-python
[9]: https://github.com/mblode/agent-skills/blob/main/skills/readme-creator/

despite this being an extremely personal-data-heavy project, I didn't give any of it to the coding agent. I only gave it small samples of fabricated data that matched the structure of my real data, rather than giving it the full data dump. google may already have my full location history; I don't need openai to *also* have it.

it does the job. it builds a [nomads.com style trip ledger][1] from a combination of the gpx files produced by the android app [gpslogger][5] and a [google maps timeline][6] [export][7]. the gpx files are treated as the authoritative source, with the google maps timeline filling in the gaps (google maps timeline sometimes gets confused by vpn usage). the gmaps timeline also provides much more historical data, since I didn't start tracking with gpslogger until mid 2024.

[5]: https://github.com/mendhak/gpslogger/
[6]: https://timeline.google.com/
[7]: https://takeout.google.com/

I've now got it `make deploy/prod`'d to [my personal vps][10]. I point it at the gpx data, which is synced up with [syncthing][11], and the results are synced back down the same way. so now, without really thinking about it, I have a pretty good record of where in the world I was on any given day.

[10]: https://github.com/samm81/mni.ac/commit/ce45896ba0808027742033c95e53a1ff272eb75a
[11]: https://syncthing.net/

ai disclaimer: I used an llm to help edit this post. the ideas, code, and final edits are mine, but some of the wording was machine-assisted.

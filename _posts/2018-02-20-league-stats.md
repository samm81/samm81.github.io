---
layout: post
title: Does Creep Score Increase Your ELO? The Results May Surprise You!
---

For those of you who don't know, [League of Legends][1] is a [MOBA][2] involving 5 players working together to destroy another set of 5 players. According to [Wikipedia][3], by July 2012 it was the most played online game in North America and Europe by number of hours played. And in high school I played a lot of it. The site has since gone offline, but the last time I looked at "how many hours have I spent on league of legends" the total came out close to 1200 hours - or 50 days of my life 😅

[1]: https://play.na.leagueoflegends.com/en_US
[2]: https://en.wikipedia.org/wiki/Multiplayer_online_battle_arena
[3]: https://en.wikipedia.org/wiki/League_of_Legends

There are a lot of pieces to the game. I won't go into depth, but an important part of the game was acquiring items for your character. You would purchase items using gold, and you would get gold in two ways: either killing minions or killing enemy champions. Killing enemy champions was far and away the best way to get gold, but killing minions was important too. At the end of the game, the number of minions you had killed was known as your "cs" or "creep score".

Another important facet about League is that it was very competitive. There was a measure of your skill called 'ELO'. In ranked play, you would gain or lose ELO when you won or lost. ELO used to be public, but at the time I started this project it no longer was. Instead you were assigned a tier (which could be Bronze, Silver, Gold, Platinum, Diamond, and Challenger in order of worst to best), and a division (VI to I, from worst to best), and you had points within that division (from 0 to 100). When you hit 100, you could go up a tier/division, when you hit 0, you might drop. I usually stood somewhere around Silver IV, which put me just barely in the upper 50th percentile of players. For the purposes of this project I transformed your tier/division/points combo into a quasi ELO by multiplying your tier times 500, your division times 100, and adding your points. So if I was Silver IV with 30 points, that would be `Silver = 500 + division IV = 200 + 30 = 730`.

Just about three years ago (February 7th, 2015), I grew curious as to how much killing minions (or CS-ing) was important to your ELO. Obviously you had to be good at killing creeps to climb the ELO ladder, but I wondered if there was a specific measure you could draw. If I improved my CS per game by 5, how much of an ELO bump should that give me? Could I say that in a particular game I was doing as well as a 1800 ELO player (in terms of my CS)?

This is a pretty basic regression problem: Average CS per game vs ELO. The tricky part is data gathering. The company who makes League of Legends, Riot Games, doesn't just provide a data dump of this information. But at the time I was thinking about doing this project, they did have an API that allowed you to find the average CS per game and ELO of a particular person if you knew their ID. After figuring out my ID and a little bit of testing, I took and educated guess that 1000000 was the upper bound on the maximum ID they had in the system. As anyone who has taken statistics knows it's very important that your sample population be representative, and the best representative sample is a completely random one. So I started randomly generating and querying IDs.

There were of course issues like rate limiting, the fact that not every ID has an associated account, the face that some people didn't play ranked (and therefore didn't have an ELO rating), the fact that I put down this project for a while and when I started it back up we were in a new season and ELOs had reset and I had to discard all the old data, but after running it for long enough I eventually had about 4200 data points I had collected. A typical representative sample for the United States usually numbers one to two thousand people, so I figured a little over four thousand was good enough.

(As an aside, I didn't leave my laptop on all the time (obviously). So in order to have data collecting going 24/7, I hosted my scraper on UT's computers. The problem is that UT automatically kills all long running jobs and instructs you to use a special cluster of theirs called [Condor][4]. I tried to use Condor. I really did. But eventually I just gave up and wrote a cron job to kill and restart my scraping script every 15 minutes. Since the script just appended to the DB there was no data lost.)

[4]: https://www.cs.utexas.edu/facilities/documentation/condor

Once I had the data there were a couple of more hijinks. Some people have played too few games and their ELO does not yet affect their true skill. I had to filter those people out. Some people have a very high or very low win rate - when your ELO is at your true skill level your win rate should be at about 50%. So I filtered out people with a win rate above .510 or below .490. This left me with 1054 entries. Still a sizeable amount! The final remaining issue is that some people tend to prefer to play roles that usually end up with less CS than other roles - roles like jungling or supporting. This I couldn't think of any way to solve, so I just figured that the random sample would take care of spreading out support/jungle mains evenly across different ELOs.

Throwing it all in a google spreadsheet and running a quick regression gets us...

<iframe width="720" height="400" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vQH5RRHx78LsxIygK7mT3emf12lo0oCuKaJXVxuJNk8azAagNTzHifV6epf3BFVAa_lCvtR8jU2lMwH/pubchart?oid=1037745139&amp;format=image"></iframe>

The giant, poorly formatted equation across the top is equation of the regression line, complete with the `r^2`. This tells us a couple of things:

1. Even the shittiest players should average about 101 CS per game (the y intercept).
2. An 1 ELO corresponded with an increase of about .0127 average CS per game (the slope). This means that a 1 average CS per game increase corresponded to a `1/.0127 = 78.74` ELO increase - three quarters of the way to a new tier.
3. This also means that to bump yourself a division (gain 100 ELO), you needed to increase your average CS by about `100 * .0127 = 1.27` per game. To bump a tier (gain 500 ELO), you needed to increase your average CS by about `500 * .0127 = 6.35` per game. And to bump yourself from Bronze V to Challenger (gain 2500 ELO) you needed to increase your average CS by about `2500 * .0127 = 31.75` per game.

Wow, only 31.75 more CS per game! This seems incredibly do-able. I'll be challenger in no time! But of course, this is not the case. The game of League of Legends does not solely depend on your creep score, there are many other factors at play. To examine this, we can look at the `r^2` score. The `r^2` of this particular regression is .032. What this means in layman's terms is that approximately 3.2% of your ELO can be 'explained' by your average creep score. In other words, look at the mass of data points on that graph. They are *very* spread out. Some people in Silver have average CS of almost 200, and some people in Diamond have and average CS of less than 50. So although *on average* CS is correlated with ELO, it's very clear that you can do well with a low average CS and do poorly with a high one.

And so I ended up not choosing to spend hours every day practicing killing minions in a solo game on [Summoner's Rift][5]. But who knows? Maybe I could have been Challenger 😎

[5]: http://leagueoflegends.wikia.com/wiki/Summoner%27s_Rift

For those who were interested I also made a scatter and box-and-whisker plot based off of tier/division specific data.

<iframe width="720" height="400" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vQH5RRHx78LsxIygK7mT3emf12lo0oCuKaJXVxuJNk8azAagNTzHifV6epf3BFVAa_lCvtR8jU2lMwH/pubchart?oid=1804445087&amp;format=image"></iframe>

<iframe width="720" height="400" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vQH5RRHx78LsxIygK7mT3emf12lo0oCuKaJXVxuJNk8azAagNTzHifV6epf3BFVAa_lCvtR8jU2lMwH/pubchart?oid=621249950&amp;format=image"></iframe>

The code is all up at: [https://github.com/samm81/league-scraper](https://github.com/samm81/league-scraper)

The spreadsheets I used to do my calculations and make figures are here: [https://docs.google.com/spreadsheets/d/1duKA6R7PLxgTVzjC3H2KH-9fIICVbkoScL1SP7taKXQ/edit?usp=sharing](https://docs.google.com/spreadsheets/d/1duKA6R7PLxgTVzjC3H2KH-9fIICVbkoScL1SP7taKXQ/edit?usp=sharing)

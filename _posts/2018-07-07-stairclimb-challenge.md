---
layout: post
title: Stairclimb Challenge
---

There's a fun and healthy tradition that takes place at my internship for this summer. I'm interning with Airbnb, and their Seattle office sits on the second and third floor of a twenty one floor building. Every day around 1:30 pm, people gather together and climb up, then down the stairs just one time. We have a spreadsheet keeping track of everyone's paces so you can track improvement over time. And man, is it brutal. Those who have done the stair climb are aware that not all stairwells in the stairclimb are created equal. Most of them consist of exactly 10 steps, but my legs know that this is not true of all of them. I set out to conduct and investigation.

### Initial Findings

A crude but simple and effective method of counting the stairs as I walked up them was employed. Here were the findings:

* The building does, in fact, have 21 floors, as advertised.
* Each floor is split into two stairwells, one upper and one lower. In order to ascend a whole floor, you must ascend the lower stairwell, pivot on the landing, and then ascend the upper stairwell.
* 21 floors - starting on the 2nd floor = 19 floors climbed
* 19 floors climbed * 2 stairwells per floor = 38 stairwells to climb
* Using a zero-indexed schema:
  * the 0th and 1st stairwells consist of 10 steps
  * the 2nd and 3rd stairwells consist of 12 steps
  * the 4th and 5th stairwells consist of 14 steps
  * stairwells 6 through 34 consist of 10 steps
  * the 35th stairwell consists of 15 steps (!)
  * the 36th and 37th stairwells (the final two) consist of 11 steps
  * `[10, 10, 12, 12, 14, 14, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 15, 11, 11]`

### Analysis

First there is the matter of the 35th stairwell. Why is it so wonky? We may never know.

Doing a sum over all stairwells leads us to the total number of steps in the upwards portion stairclimb: 399. An incredibly unsatisfying number. However, most people will take about two steps per landing in order to execute the pivot. This would add another 38 steps, but we have to subtract one because only one step is taken at the very top. `399 + 38 - 1 = 436` steps taken climbing the stairs.

Armed with this knowledge we can figure out exactly how many steps per minute one must take in order to summit the stairclimb in a fixed time: `total_steps / desired_time (in minutes) = steps_per_minute`.

```
  desired time [m]    steps per minute
------------------  ------------------
               0.5                 872
               1                   436
               1.5                 291
               2                   218
               2.5                 175
               3                   146
               3.5                 125
               4                   109
               4.5                  97
               5                    88
               5.5                  80
               6                    73
               6.5                  68
               7                    63
               7.5                  59
               8                    55
```

This gives us an excellent tool for pacing, but an even better tool for picking a song to climb to! Let's say we wanted to summit the stairs in 3.5 minutes. If we picked a song with a BPM of 125, took a step on every beat, and took two steps on every land (to pivot around), we'd make it up just in time! Of course if you're looking to make it up in a minute or less, you're probably on your own....

Songs with a BPM of ~200 (2-2.5 minute summit):
* Paradise City - Guns N' Roses
* Naughty Girl - Beyonce
* Josie - Blink 182
* Nobody's Listening - Linkin Park
* Addicted - Kelly Clarkson
* https://jog.fm/workout-songs/at/200/bpm?sort=popularity

Songs with a BPM of 175 (2.5 minute summit):
* Footloose - Kenny Loggins
* Heartless - Kanye West
* Love the Way You Lie - Eminem
* Dancing With Myself - Billy Idol
* Ocean Avenue - Yellowcard
* https://jog.fm/workout-songs/at/175/bpm?sort=popularity

Songs with a BPM of 150 (3 minute summit):
* Mr. Brightside - The Killers
* I'm Yours - Jason Mraz
* Brown Eyed Girl - Van Morrison
* Holding Out for a Hero - Bonnie Tyler
* We're Not Gonna Take It - Twisted Sister
* https://jog.fm/workout-songs/at/150/bpm?sort=popularity

Songs with a BPM of 125 (3.5 minute summit):
* Firework - Katy Perry
* Only Girl (In the World) - Rihanna
* Like a G6 - Far East Movement
* Born This Way - Lady GaGa
* Bulletproof - La Roux
* https://jog.fm/workout-songs/at/125/bpm?sort=popularity

Song with a BPM of 110 (4 minute summit):
* Eye of the Tiger - Survivor
* Hollaback Girl - Gwen Stefani
* Another One Bites the Dust - Queen
* Just the Way You Are - Bruno Mars
* Livin' On a Prayer - Bon Jovi
* https://jog.fm/workout-songs/at/110/bpm?sort=popularity

You get the idea.

Additionally! We can use this to calculate how fast you'll summit the stairs listening (and walking to) a certain song. So if we were to listen to, say, "Sandstorm - Radio Edit" by Darude with a BPM of 136, we'd reach the summit in 3 minutes and 13 seconds. 🏃‍♂️🎵🏃‍♀️🎶

---
layout: post
title: Egghead Pro for Free!
---

Well close enough.

[egghead.io][1] is a website for watching screencasts about various web development technologies. In their own words: "Bite-size video tutorials for badass web developers." I tend to shy away from coding screencasts, but my manager at work recommended a set of egghead videos on react/redux in order to bring me up to speed on the system they were working on.

[1]: egghead.io

Egghead has a pretty annoying freemium system: after about three or four videos they stop you and force you to put in your email address before being able to watch more. Additionally, being able to speed up video playback is a feature reserved for 'pro' members. My manager told me to circumvent the email input by using an incognito tab - but I figured I could do better. Popping open chrome web inspector and diving deeper reveals that they make a critical mistake - they load the video in the background and only prevent the user from clicking play by popping up an element over the video. It's a pretty simple matter to get rid of the offending element:

```
$('.enter-email-overlay')[0].remove();
```

Now, I understand that they're trying to find a way to be profitable, but I'm not going to be a long term subscriber. I only want access to these 15 videos or so and them I'm out - and I want to be able to do it as fast as possible. Speaking of which...

I would love to be able to speed up playback speed. Looking into the web video api it seems that there is a property for video speed...

```
$('video')[0].playbackRate = 1.5;
```

So now we have ourselves a little pro-for-free javascript gist.

```
$('video')[0].playbackRate = 1.5; $('.enter-email-overlay')[0].remove(); $('video')[0].play();
```

And here's a bookmarklet, if you please!

<a href="javascript: (function() {$('video')[0].playbackRate = 1.5; $('.enter-email-overlay')[0].remove(); $('video')[0].play();})();">Egghead Pro</a>

To use, drag to your bookmark bar. Then, when on an egghead video page, click the bookmark to get the 1.5x speedup and remove the email banner!

---
layout: post
title: Pusheen Screensaver
---

Part two of a series in which I got back and examine old github projects. Today's victim is [PusheenScreensaver](https://github.com/samm81/PusheenScreensaver).

![pusheen](/assets/pusheen-screensaver/pusheens/default.gif)

PusheenScreensaver was a class project for my 2D Graphics class. It features our furry friend, Pusheen, strolling back and forth along the bottom of the screen. Every once in a while [she](http://www.pusheen.com/about) will take some action, or change direction. If she bumps into the side of the screen, she turns around. I was so proud of it I gave it as a gift to my girlfriend at the time. For a while we both used it, but after we broke up I got rid of it :/

![pusheen](/assets/pusheen-screensaver/pusheens/30.gif)

The code, however, remains!

![pusheen](/assets/pusheen-screensaver/pusheens/1.gif)

We start with a look at the [Screensaver](https://github.com/samm81/PusheenScreensaver/blob/2809ee0259e498420e12a5e76012dc76a6e5e12a/src/Screensaver.java) class, which has the `main` method. Immediately, the [first line](https://github.com/samm81/PusheenScreensaver/blob/2809ee0259e498420e12a5e76012dc76a6e5e12a/src/Screensaver.java#L19) confuses me. `System.gc()`?? Why???

![pusheen](/assets/pusheen-screensaver/pusheens/2.gif)

I remove it. The code continues to run.

![pusheen](/assets/pusheen-screensaver/pusheens/16.gif)

I find my choice of breaking out the framing to a separate class strange. I could have moved the `main` method into `Pusheen.java` and it wouldn't have changed anything.

![pusheen](/assets/pusheen-screensaver/pusheens/5.gif)

I do so. The code continues to run. We are down to two classes. I rewrite the readme.

![pusheen](/assets/pusheen-screensaver/pusheens/6.gif)

I'm also not sure why I made `c` a global static variable. I move it into the `main` method.

![pusheen](/assets/pusheen-screensaver/pusheens/7.gif)

I, at one point, apparently ran into a windows bug where I had to keep track of every mouse movement in some global variable.

![windows bug](/assets/pusheen-screensaver/windows_bug.png)

???

![pusheen](/assets/pusheen-screensaver/pusheens/8.gif)

Why didn't I use brackets around `if` statements?

![if statement 1](/assets/pusheen-screensaver/if1.png)

Did I think I was cool or something?

![if statement 2](/assets/pusheen-screensaver/if2.png)

Dear god, why??

![if statement 3](/assets/pusheen-screensaver/if3.png)

???????????????????

![if statement 4](/assets/pusheen-screensaver/if4.png)

I hope my readers can forgive me.


![pusheen](/assets/pusheen-screensaver/pusheens/9.gif)

My ex actually found an interesting bug in the code after years of usage. Pusheen can obviously only move across the screen a fixed number of pixels at a time, but when I tested it I felt like moving one pixel at a time was too slow, and two pixels at a time was too fast. What I ended up doing was making `speed` a double, and setting it to `1.2`. I also made `pos` (the position) a float, and when I rendered Pusheen to the screen I would just cast the `pos` double to an int. This meant that in practice, Pusheen moved forward one pixel at a time, but every 5 frames moved forward two pixels, making her appear to have a speed of 1.2 pixels pre frame.

![pusheen](/assets/pusheen-screensaver/pusheens/11.gif)

Now here's where it starts getting tricky. `pos` is a double, but when I render Pusheen I cast it to an int. So, I reasoned, when I check if Pusheen is about to go out of bounds, I should cast it to an int. This means that if, for example, the width of the screen is 100, `speed = 1.2` as previously discussed, and `pos = 99.7`, I will check that `(int)(pos + speed) > width`, which will be false, since `(int)(99.7 + 1.2) > 100 => (int)(100.9) > 100 => 100 > 100 => False`. So I will allow the update to happen, and now `pos` becomes 100.9 . We render the scene and everything is fine. Now jump through the `updateVars` method again. This time, it just so happens that while Pusheen is out here at the edge of her world, the `flipChance` random double happens to be generated below the `flipThreshold`, and the global `flipped` variable is set to `true`, meaning that Pusheen is now going to be traversing back across the screen. But! But we _continue_ into the 'off the screen' check. `(int)(100.9 + 1.2) > 100 => True`, so we negate the `flipped` variable _again_. And because I, when I coded this, assumed that `flipped` would always be in the correct state, I didn't have any checks to make sure that `flipped` was becoming `true`, I just set `flipped = !flipped;` Now we once again _add_ 1.2 to `pos` (since `flipped` is `false`) and now `pos = 102.1`. Go through `updateVars` again, the 'off the screen' branch is triggered, `flipped` gets set to `true`, but it's too late! We subtract `1.2` from `pos` to get `100.9`, we render Pusheen facing the opposite direction, and on the next `updateVars` we negate `flipped`, add 1.2 to `pos`, and render Pusheen flipped again. Pusheen now gets stuck flipping back and forth at the very edge of the screen.

![pusheen](/assets/pusheen-screensaver/pusheens/10.gif)

This bug has a very low chance of happening. The flip threshold is `.008`, so only one of about every hundred frames does Pusheen flip. Pusheen needs to flip while she happens to have a `pos` that is greater than the width of the screen, but less than the width of the screen plus one. This bug is so rare I never once encountered it, but my ex used it for long enough that she happened to run into it.

![pusheen](/assets/pusheen-screensaver/pusheens/31.gif)

It was still poorly written code though. I was thinking about how if I did this again I wouldn't use any mutable state - I'd try and do it in a very functional way. I don't think it would be all that hard to recreate this in elm (but then it wouldn't be a screensaver!)

![pusheen](/assets/pusheen-screensaver/pusheens/23.gif)

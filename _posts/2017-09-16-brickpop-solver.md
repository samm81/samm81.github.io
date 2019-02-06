---
layout: post
title: Brickpop Solver
---

Way back when, spring semester of 2017, facebook messenger games came out and became crazy popular. There was the infamous Endless Lakes, in which you jumped over an endless number of gaps in a platform stretching out over a lake. There was EverWing, in which you flew a winged angel (?) character with some dragon companions to defeat enemies in the sky (forever). And there was Brickpop, where you had a grid of colored bricks, and you had to pop them all. Tapping a brick pops the whole chunk that the brick is in, where a chunk is defined by contiguous bricks of the same color. A brick on it's own *cannot be popped*. This meant that you had to insure that every brick eventually had at least one buddy around it.

<div class="video">
   <video style="display:block; width:50%; height:auto; margin-left:25%; margin-right:50%;" autoplay controls loop="loop">
       <source src="/assets/brickpop/brickpop.mp4" type="video/mp4" />
   </video>
</div>

As you can see in that video, I won that level.

Now here's the thing. I like beating other people in pointless games to prove my dominance, but I don't like having to actually work at beating them. Wouldn't it be way easier to have a computer do it for me?

Enter: brickpop.py

<div class="video">
   <video style="display:block; width:100%; height:auto;" autoplay controls loop="loop">
       <source src="/assets/brickpop/brickpop_bot.mp4" type="video/mp4" />
   </video>
</div>

As you watch it play you'll notice a couple of things: it's not particularly fast (as it has to wait for the animation), nor is it particularity clever. But, it will always win. And, because your brickpop score is cumulative until you fail a level, as long as you win your score will increase. You don't have to be clever, you just have to find *any way* to win.

And it turns out that the brickpop game is not clever in generating their levels. There are *a lot* of solutions to any particular level. After some experimentation, I found that the easiest way to find a solution is _random guessing_. Click a brick, any brick, and keep clicking until either you fail or you've cleared the level. If you fail, then you reset your simulation and start again. This method, when I tested it, always found *at least one solution in under one second*. Who needs fancy searching algorithms?

The bot works approximately like so. First, position your mouse cursor over the top left block (so that the bot knows where the playing field is). Second, start the bot. It goes to work and reads in the state of the board, then prints it out. It spins it's wheels for 5 seconds (I tried a bunch of different lengths of computing times, 5 had the best results for the least effort) simulating the game as described above, discarding solutions that don't work and saving the solution that takes the least clicks. Solution in hand, it prints out the click sequence, and executes them using `pyautogui`, waiting an appropriate amount of time to let animations finish. Upon solving the level (which it always does), it announces 'done!', waits for the next level to be generated, and repeats ad infinitum.

I ran this overnight once and it reached a score of about 300,000 and was still going. I also managed to piss off my girlfriend by using it to just barely beat her high score and stopping it. I didn't tell her I used a bot for several days 😈.

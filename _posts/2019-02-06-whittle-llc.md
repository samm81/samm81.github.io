---
layout: post
title: Whittle, LLC
---

Back in my sophomore year of college I started food logging using [Fitbit][1]. Fitbit is, to most people, a thing you wear on your wrist to track your steps and (for the fancier versions) your sleep. I doubt many people even know that you can use the Fitbit app to keep track of the food eat, much less actually use that functionality. I don't remember why I started tracking, but if I had to guess I would say it was freshman fifteen 😅

At that time I was also still living on campus, and so still eating in the school cafeterias. The University of Texas at Austin's Division of Housing and Food considerately (or perhaps by law) published nutritional information [online][2]. So ever time I got a bite to eat, I would look up the nutritional information of whatever I bought and copy/paste it into the Fitbit app. This was a highly accurate system, but also highly arduous. The website UT used was terribly designed (and, as I would find out later, terribly built - tables in tables in iframes in tables), and was obnoxious to navigate.

<video style="display:block; width:100%; height:auto;" autoplay controls loop="loop">
  <source src="/assets/whittlellc/hf-food.mp4" type="video/mp4" />
  <source src="/assets/whittlellc/hf-food.mp4" type="video/mp4" />
</video>

But wait a minute, I thought to myself. I'm a computer science student. This is a computer science problem. Fitbit has an API. UT... doesn't have an API but I can scrape the website. I can solve this problem!

Step one: scrape nutritional information from UT's website and store it in a [TinyDB][3] on a daily basis. The database had two tables, one for food items and one for menus (which foods were served where an which days at which locations). Their website exposed the unique ID it associated with every food item which made it easy to de-dupe food items and ensure I only scraped them once. The menu database then just associated a date, location, time, and food station with a list of food item IDs.

Step two: an interface to let the user select and log foods. I'm comfortable with bootstrap, so I threw together a [Flask][4] app that loaded all the foods at all locations for the current day and served them as a giant table. Some Javascript code on the front end would then ask the user when and where they ate, and then hide the rows of the table that weren't selected. The user could select the foods they ate, and when they hit submit the server would log those foods to your Fitbit account (assuming you had logged in with Fitbit).

I had a UI optimizations which I was pretty proud of. In order to select how much of an item you ate, there was a text box with increment and decrement counters on either side. But if you look up pizza on UT's site, you get the nutritional information for 1 whole pizza, with a default serving size of 1/16th. If had been lazy, the increment/decrement buttons would have just added or subtracted 1 of the item. But letting the user increment or decrement by one whole pizza is not very useful, so instead I made the increment and decrement buttons add or subtract the default serving size: 1/16th of a pizza. And you could always just input a custom amount into the text field.

<div class="video">
   <video style="display:block; width:100%; height:auto;" autoplay controls loop="loop">
       <source src="/assets/whittlellc/fitbit-ut-prototype.mp4" type="video/mp4" />
   </video>
</div>

Overall I was very pleased with it! But... I stopped logging foods. As anyone who has ever counted calories before knows, it is *really annoying*. I actually fell out of the habit before I even finished the app, but stuck with the project just because I was most of the way there. I added some finishing touches, patted myself on the back, and pushed it to Github where it languished for two years.

Fast forward to senior year when, for some reason, I take another look at the app I wrote. I set it up again and marvel at my genius design. It then occurs to me that if I remove the "log to Fitbit" functionality, I'm actually left with an even better product: a redesign of the UT's nutrition website! 

But what to do with it...? I decided that maybe I could give it to UT. Wouldn't that be cool to have my code running on a real server, being shown to real students?

Then I reconsidered. I had been getting into the idea of [indie hacking][5] and figured maybe I could get a buck off my hard work. I would sell it to UT! It was at least worth a shot. The worst that could happen was they said no.

So I sent a cold email to the Division of Housing and Food, asking if they would be interested in taking a look at the code I wrote. To my surprise they said yes. I frantically fixed up a couple of lingering bugs and modernized it (ported it to python 3). We set up a meeting, and I demoed the product, emphasizing how it supported both desktop and mobile seamlessly, as well as the advanced UI features. To my horror, they actually found my public Github repo with all the code _during the meeting_. Right afterwards I went home and made the repository private.

But the reception was positive! They liked it - however they were not the ones who could make a decision about buying. They said that they would present it to their managers, and let me know.

So implemented a few of the essential features they said they would need (they have certain restrictions as a public university), fixed up some bugs, and sent them the new demo, and waited.

They got back to me and asked how much I wanted to sell it for.

Now this, of course, is THE question. I had never sold anything in my life and really had no intuition as to what an appropriate price was. Several hundred? Several thousand? Tens of thousands??? This was all complicated by the fact that UT was a public university, and so has all sorts of restrictions (another student told me that he once sold something to the UT's law school for $4,999 because they had a rule that they couldn't buy anything more than $5,000 without Dean approval). Furthermore, I wouldn't have a chance to negotiate this price in person, it would be presented on my behalf to people I'd never met. If they rejected it would I have another chance?

I had originally planned to try and sell it on a monthly basis, with me hosting the service for them. I figured that they had a lot of money - hiring even one worker full time at minimum wage ($7.25 in Texas) would be `$7.25 * 40 hours / week * 4 weeks / month = $1,160 / month`, so they should be able to afford about a thousand a month no problem right? In the initial meeting I had been flat out refused. UT doesn't work that way, I was told. They would buy the code outright and host it themselves. There went my dreams of monthly passive income...

Well, if I had been going to ask for about $1,000 a month, maybe I should just ask for $12,000, one year's worth? That seemed like an insane amount of money for a simple front end.

My Dad is the assistant principal of a high school, so I asked him how much he thought an educational institution might spend on such a product. He said maybe $500 to $1000... Drat.

I asked the friend who had dealt with UT before. He said probably no more than $5,000.

I asked a friend who was really into business and startups, thinking he'd have a reasonable answer. He told me to take whatever I was considering and double it. Thanks bud...

I waffled on a number, but needed to give them an answer. So finally I sat down and wrote up the reply email, selecting $2,500 on the spur of the moment. Two days later I got a reply saying to expect a reply soon.

I waited patiently.

Eleven days later I got another email. "They were very excited about the project *and the price point.* ... There are a few things that need to be addressed before we can move forward with a purchase of this software" (emphasis mine).

On the one hand I was ecstatic. They were interested in buying it! On the other hand, them being excited about the price point meant that I definitely could have asked for more. Oh well! Lessons learned.

So I fixed up the issues they had mentioned, but there was one more complication - UT could not legally buy from an individual. I would have to incorporate (at minimum) an LLC.

And so [Whittle, LLC][6] (link currently empty, I bought the website but haven't put anything up on there yet 😅) was born. I spent a day or two thinking about the name, went through the registration process with Texas, paid the fees, drafted a sales contract by hand, jumped through the UT's hoops, set up a separate bank account, and finally, halfway through June, I received a deposit for $2,500.

UT has deployed the code and advertised it to incoming Freshman. It's super cool to get paid for my code, and almost as cool to know that it's being used in the wild at a school of fifty thousand students. If you want to take a look at it live, it's over at [hf-foodapp.austin.utexas.edu][7], or I've recorded a little demo below.

<div class="video">
   <video style="display:block; width:100%; height:auto;" autoplay controls loop="loop">
       <source src="/assets/whittlellc/fitbit-ut-sale.mp4" type="video/mp4" />
   </video>
</div>

But the long and the short of all this is that you now need to address me as Samuel Maynard, CEO 😎

[1]: https://www.fitbit.com/
[2]: http://hf-food.austin.utexas.edu/foodpro/location2.asp
[3]: https://tinydb.readthedocs.io/en/latest/
[4]: http://flask.pocoo.org
[5]: https://www.indiehackers.com/
[6]: http://whittlellc.com/
[7]: http://hf-foodapp.austin.utexas.edu

---
layout: post
title: `express-openid-connect` and in-flight requests
---

you've written your front end piece. you need serve it somehow. ok. spin up a quick `express` server. you want multiple people to use it. ugh. ok. get your auth0 account, go through their setup flow, everything looks good. except...

for some reason when you try to log the user out, they get logged back in immediately. why do bad things happen to good people. ok. they click the button. it `window.location.assign('/logout')`s. that looks good. pop open the dev console (jk you already have it open because it's always open). `window.location.assign('/logout')`. that works, and doesn't log the user back in. hmm. you pop open the network tab and take a look:

![](/assets/express-openid-connect-and-in-flight-requests/image_1678402611731_0.png) 

hmmm...

session data is usually stored in the cookies, so you'd bet that the cookie just isn't getting cleared. you peek at the request cookies in the `/` request:

![](/assets/express-openid-connect-and-in-flight-requests/image_1678402977746_0.png)

as you suspected, that looks like a valid `appSession` cookie (which is what `express-openid-connect` uses to store the active login). clearly the library is failing to clear the cookie. triumphantly, you take a look at the `/logout` request:

![](/assets/express-openid-connect-and-in-flight-requests/image_1678402771817_0.png)

wait - it is clearing the session. the response cookies `appSession` has a `value: ""` which should definitely clear it. how is it reset right before login...?

looking back at the network calls you click on the only possible culprit: the `/update` call that you make to record the "user clicked logout button" action. and sure enough:

![](/assets/express-openid-connect-and-in-flight-requests/image_1678403213892_0.png)

ok. so you get the picture now. when the user clicks the "logout" button the front end both fires off an async `/update` request to the server, and redirects the window to `/logout`. `/logout` comes back first and clears the session, but the response to `/update` comes back /after/ and /resets/ the cookie that was just cleared.

but why does `express-openid-connect` set the user's login cookie (`appSession`) in the response to `/update` ? that's a route you created, and has nothing to do with logging in or out.

turns out, it's because the library uses a [rolling session](https://github.com/auth0/express-openid-connect/issues/446#issuecomment-1457926018) so that every request made from the client to the server extends the user's current session. makes sense you guess, but you're not sure that something like `/update` /should/ extend the user's session (for example, what if you're updating about a period of inactivity?). philosophical problems aside, it's breaking your logout flow.

the suggested solutions are

1. use a custom session store

but this sounds like a lot of effort

2. manage the concurrency application side

which is also a lot of effort given that now you're dealing with concurrency. is there a dumb and easy way to do this?

after some thinking you realize there is. triumphantly opening up `server.ts` you simply move the `/update` handler to /before/ the call to `app.use(auth(authConfig))`. `express` only applies handlers to routes that are defined /after/ the handler is `use`d, so that means the `auth` handler doesn't end up getting applied to this route. which means it doesn't try to refresh the session, meaning the `/update` response comes back without the `appSession` cookie, which means the user doesn't get logged back in!

it's not pretty, but that's perhaps a problem for another day.

(or perhaps you have a better idea? leave a comment! I'd love to know 😊)

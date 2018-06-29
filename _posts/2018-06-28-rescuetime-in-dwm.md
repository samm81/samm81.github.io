---
layout: post
title: Rescuetime in DWM
---

[Rescuetime][1] is a great little app that tracks how you use your time on your computer and then shames you for not being productive enough. I got into it recently and installed it in my browser and on my phone, and tried to install it on my laptop. But of course it didn't work, because I use [dwm][2], and why would dwm work with anything out of the box? But I wanted to get it working so I sent in a support ticket.

[1]: https://www.rescuetime.com/dashboard
[2]: https://dwm.suckless.org/

After I told them my window manager was dwm the support guy responded:

> Hello. This was our Linux developer's reply to me:
> The following commands must work for RescueTime to work:

```
xprop -id `xprop -root | grep "_NET_ACTIVE_WINDOW(WINDOW)" | cut -d# -f2 | cut -d, -f1` | grep "NET_WM_PID" | cut -d"=" -f2 2>/dev/null
xprop -id `xprop -root 2>/dev/null | grep "_NET_ACTIVE_WINDOW(WINDOW)" | cut -d# -f2 | cut -d, -f1` 2>/dev/null | grep "WM_CLASS(STRING)" | cut -d"\"" -f4 2>/dev/null
xprop -id `xprop -root 2>/dev/null | grep "_NET_ACTIVE_WINDOW(WINDOW)" | cut -d# -f2 | cut -d, -f1` 2>/dev/null | grep "WM_NAME(STRING)\|WM_NAME(COMPOUND_TEXT)" | cut -d"\"" -f2 2>/dev/null
```

"our Linux developer". Must be lonely.

Doing some investigation (pulling up the terminal and running the above commands) revealed that the commands indeed did not work. For some reason `xprop -root` did not return any row which contained `"_NET_ACTIVE_WINDOW(WINDOW)"`. Interesting.

Of course, when you use a window manager as niche as dwm googling "dwm no _NET_ACTIVE_WINDOW(WINDOW) in xprop -root" doesn't really get you very far. But I figure there's someone else out there that uses both dwm and RescueTime, so I wrote this blog post so that their desperate googling will no longer be in vain. I did the hard work of trawling through the [dwm mailing list archives][3] to find the thread about [supporting _NET_ACTIVE_WINDOW][4], which contained an email from a fine fine man named Andreas Amann, who [wrote a patch to add _NET_ACTIVE_WINDOW support][5] which has a permalink here: [https://lists.suckless.org/dev/att-10649/netactivewindow][6].

[3]: https://lists.suckless.org/dev/subject.html
[4]: https://lists.suckless.org/dev/1201/10531.html
[5]: https://lists.suckless.org/dev/1201/10649.html
[6]: https://lists.suckless.org/dev/att-10649/netactivewindow

Here's to that one other dwm/ResueTime user. I hope this saves you an hour or two.

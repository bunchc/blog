---
title: "Tuna Sandwiches"
date: 2015-10-14
layout: "post"
categories: 
---

    Words of warning. This is one of those long winded rambling posts. There will be good bits here and there (at least that is the hope), but, this doesn't follow good SEO, or copywriting practices. There is no real call to action, no clear intent or message.

On most school days, I've taken to walking my now five year old to school. It's a wonderful way to spend an extra few minutes with each other, and the walk is therapeutic in other ways. That is, its good for all the reasons walks are good: extra exercise, being outside, being mindful / present, and all that. Todays walk was extra special, that is, it triggered this long running train of thoughts on technical debt, and what we leave behind each time we say "Fuck it, Ship it" or we open a firewall port for expedience, cut a corner on an implementation or design due to some arbitrary constraint.

That is, I had this conversation:

    Context: The traffic officer / crossing guard was not yet on duty.

    5: "Dad, where's that man?"
    Me: "Oh, he isn't here yet. Where do you think he is?"
    5: "He's at the Tuna Sammich store."
    Me: "Tuna Sandwich?"
    5: "He went to the Tuna store to get a Tuna Sammich for his butt."
    Me: "His butt?"
    5: "His Right Butt Cheek. He needs a Tuna Sammich for his right butt cheek."
    Me: "..."

You can see how this is sort of therapeutic, a small smile crosses our faces. His at having said butt cheek a few times in a row and gotten away with it. Mine from the innocence, the oddity of it, and most of all, because he said butt cheek. (One can't be an adult /all/ the time).

It did however, hit me with a pang of guilt. One that is familiar to all parents, I imagine. That is, it was the realization that eventually, he'll grow up. We won't walk as much, and while we'll still talk, it likely won't have this level of random innocence to it (tho, I'll be damned if I'm not going to strive for that).

That pang of guilt, by way of a long ass introduction, led me to thinking about what sort of future, what sort of legacy we'll be leaving to them. I'm not one for thinking too long about the **huge** parts of legacy, things like the environment or economy and all. Instead, I'm speaking of what sorts of technical legacy we're leaving around. What the impact is, each time we make a decision, and what that will look like in 10, 15, 20 years time.

Oh, I hear you saying: "No system I build will be online in 15 years." or "My field expedient Perl code will be re-factored during the next release.", and lots of other things to the same effect.

These two examples were drawn from my own career. The first one from a middling FiServ (hard to gauge size when small is 5mm and large is multi-trillion). That FiServ was (still is) running an HP MPE/iX system. This system had been stood up when 9GB SCSI drives were amazing. When you were hired on, you heard about how the system came to be, and how much they spent on it, etc. What I didn't realize at the time, is I was also hearing why it would never change, but that is another story. That system, whilst they've added layers and layers of middleware, online services, and the like, is still online, and still has active contracts for 4 hour drive replacement, on 9GB scsi drives.

Think for a minute, at the middling FiServ size, what else might be done, if one weren't spending a considerable effort maintaining what was ultimately a decision made in haste. One that the presiding admin (architects weren't really a thing then) decided would be fixed on someone elses watch?

The other example, field expedient Perl, is one that I've found almost everyone has an example of in their career. While not 15 years back, I can almost guarantee said code is still running in some class 3 telco switch somewhere. This particular gig, I got from some connections on IRC, it was to work on networking and voice gear for a local telco. Knowing nothing about switching, routing, voice, or really, networking, I figured... why the hell not.

At some point, it came about, that some code my predecessor wrote needed to be updated. Some file format changed or, as the comments in the code I found said: "Some asshat put something in the file they weren't supposed to". So, what'd I do? I ghetto patched the crap out of it. To date, if you call one of a few South Florida prefixes, the metadata is routed through that code.

So the point, I suppose to all of this ranting and raving in this post, is pay attention to even the little decisions, and what sorts of impacts they'll have. Because while we're doing a decent job on the front end of educating the little IT folks of the future with things like Scratch, RaspberryPi, and all sorts of STEAM groups, 3d printing, etc etc. We're can do a better job in leaving good systems for them to inherit.
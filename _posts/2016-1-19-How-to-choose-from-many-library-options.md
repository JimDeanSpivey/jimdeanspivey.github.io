---
layout: post
title: How to choose which library, when there are multiple open source libraries solving the same problem to chose from?
---

So after working on a project that uses the Twitter Streaming API in Java, I naturally went with the library that came with the platform stack that I was working (springframework). Not knowing that at the time, it was a mistake. I feel that the library is under-developed, has many bugs, and needs various fixes, via PR's, that other people made when they forked different versions to their own repo.  For example, adding support for geo location tweets, (and that is still not in the mainline). There also appears to be too much bueracracy from the corporate owner of that repository that requires extensive and very legally binding paperwork signing to be able to contribute any fixes. And since the original maintainers don't have any much activity as of late, I had to change to an alternative twitter client.



#### So what was my mistake?

I wasted a fair amount of time going with a library, using it, debugging its problems, only to later change to a better maintained library.

#### What is the (obvious) answer?

Don't make that mistake again.

#### What solution can be used to avoid repeating this mistake?

Well that's what prompted me to write this whole thing, as I think I found an automated (rather, simple) way to check if an open source software library is really being emrbaced by a usergroup: Check the stackoverflow tag usage!

Very simple URL pattern to use, just `http://stackoverflow.com/questions/tagged/[tagname]`, where [tagname] is the name of the library you are interested in. Then compare what the total questions tagged are for all the available open source library options, and avoid any that are too low. This is a very fast way to filter out un-used API's. For example, hereis a comparison of the 2 clients for the Twitter API in Java:

http://stackoverflow.com/questions/tagged/spring-social-twitter - 38 questions tagged
http://stackoverflow.com/questions/tagged/twitter4j - 1,276 questions tagged

I was thinking of going with latest commit history, but that really is a bad metric. A well written library may not need to change if the solution it provides issuffecient and the problem domain simply isn't changing. Basically, some great libraries are still ticking today and abiding by the old adage, if it ain't broke, don't fix it.

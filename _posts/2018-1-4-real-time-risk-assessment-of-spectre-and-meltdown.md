---
layout: post
title: Real-time Risk Assessment of Spectre and Meltdown
---

If you are a security professional who is currently in possession of a pulse, you have undoubtedly heard about the recent CPU bugs dubbed [Spectre](https://spectreattack.com/) and [Meltdown](https://meltdownattack.com/). We hate vanity names for vulnerabilities, but it’s clearly the way of the future, so we’re going with it.

If you have not yet heard of these vulnerabilities, or you feel like you don’t have the basics down, stop reading and go do your homework. Seriously. And have a drink handy. It’s a bit terrifying.

We have not written this post to terrify you, nor to explain the basics of the vulnerabilities. We instead attempt to provide a summary aimed at IT or security professionals who are trying to wrap their head around the high-level impact of such vulnerabilities and determine what they should do about them. We also hope to help our peers understand how to articulate and quantify the risk of such things to their executive management. We will try to cut through the FUD and give our honest opinion of the risk resulting from these issues.

Note that our opinions are based on the information we’ve reviewed at the time of this writing.  We have not reviewed all information out there, and there is likely to be more still coming. As time permits and circumstances warrant, we will update this post appropriately. 

# Assessment of Impact
At a high level, both vulnerabilities allow for the leak (or exfiltration) of data that a process should not normally have access to. The bad news for the attacker (and good news for the defender) is that you don’t necessarily know what data you’re getting - it’s difficult to just pull a password out of memory, for example. (To do this, one would have to dump a bunch of memory and then figure out what to do with it later.) The other bad news for the attacker is that the mileage of these vulnerabilities seems to vary by OS, CPU architecture, and perhaps a few other factors. It is not clear to us how significant this variance is on the ease of exploitability or exfiltration of data. 

There is almost too much bad news for the defender to count, but one thing that’s particularly significant - as noted in the vulnerability site - is that *no security controls you currently have in place are likely to be at all useful against this vulnerability*. Your AV, Next-Gen AV, host-based IDS, and EDR are all going to miss this for a while. Also, you’re not going to get any useful logs about it, and your network-based IDS probably won’t have anything to hold on to.

To start, we consider quick assessments by architecture design, using an arbitrary scale that is pretty-much meaningless (like most risk analysis) but will help us start to get our heads around the problem. Our scale from bad to worse is as follows.

Meh
Bad
Really Bad
Armageddon

(What? We chose words that management and interns could understand viscerally and without a thesaurus. You have better?)

## Multi-user servers (Really Bad)
The implications are very bad for any server infrastructure which is multi-user by design and allows users to run code. This includes *nix systems with user logins, Citrix environments, terminal services, and anything else like that. In these cases, you should view the vulnerability as you would the worst possible privilege escalation vulnerability you’ve ever seen and rate the risk appropriately. 

We think you should patch these ASAP. Like, keep refreshing Windows Update and get your intern to do it while you sleep.

## General-purpose servers (Bad)
Servers running services that do not easily allow users to execute code - mail servers, file servers, or database servers - are likely to be less of an issue on their own. It’s hard for us to see how one would easily exploit such environments. But these vulnerabilities are much worse when chained with another vulnerability that would allow some kind of local, unprivileged code execution. If so, you’re back at this being the mother of all privilege escalation bugs. 

So we still think you should patch these, but we wouldn’t make them your top priority. You can send your intern to get you a latte.

## Virtual Environments (Armageddon) 
<iframe width="560" height="315" src="https://www.youtube.com/embed/RkGUowLRF5s?rel=0&amp;start=14" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>

For virtualized environments that are using virtualization as a security boundary, these vulnerabilities are absolutely awful. They represent a complete dissolution, at the most foundational level, of any security boundary one could hope to put in place in a multi-tenant environment. No boundary could keep virtual system A from stealing data from virtual system B, if such systems share physical hardware. This does still require the ability to execute code on a virtual system somewhere though, and it’s also likely to be a fair bit of work.  

We believe the biggest risk is from a targeted attack against an environment with a virtual footprint where systems of different criticality are mixed together (i.e., you have your publicly facing virtual web server running on the same hardware as your crown jewels backend database and are relying on virtualization to keep them isolated). If you are in that boat - or working in any private cloud or virtual environment that is large-scale or core to your business - you should be patching with all due haste. For the attacker, jumping trust boundaries in a virtual environment just got a whole lot easier.

We suggest these environments be your first priority. Hire an extra intern to check those updates.

## Cloud Environments (Meh)
There's a lot of focus on cloud providers, but we believe those environments may be less critical than individual corporate virtual environments. With large cloud providers, it is difficult to discern who you are sharing physical hardware with, so effectively compromising a specific target and getting anything useful might be difficult. There is also a *lot* of memory to work with in a large cloud provider, making it harder to find needles in the memory haystacks. Finally, these providers appear to have been working behind the scenes ahead of the release date to mitigate the vulnerability. Even if they hadn’t, these organizations have personnel dedicated to dealing with exactly this class of problem, which you probably don’t have in your VMWare installation. Also, there isn’t much you will be able to do about this on behalf of your cloud provider, other than leaving them. 

We suggest you sit back here and let them do their thing. Send your intern to Kubernetes training.

## General purpose endpoints, desktops, laptops, etc. (Really Bad)
For general endpoints, we believe these vulnerabilities are bad, though not as bad as with the server side. In the modern era, most client-side compromises stem from phishing and/or an exploit of an old unpatched instance of a browser, Java, Microsoft Office, or Adobe product, and this doesn’t do much to change any of that.

The exception is remote exfiltration via browser JavaScript. [Mozilla confirmed](https://blog.mozilla.org/security/2018/01/03/mitigations-landing-new-class-timing-attack/) the possibility of remote theft of passwords, cookies, and other client secret data. This is presumably possible just by having your browser point to a malicious site. It is worse than a classic phishing attack though, because there’s no other tool or software to exploit, there’s no mitigation from AV or any other security function, and there’s no record this happened to you. The attacker doesn’t need their victim to download or run any code which they hope they can get past the client’s antivirus; they just need the victim’s browser to hit a site. 

If you are in a well-managed and locked-down shop with systems that are tidy, secured, and up-to-date, this is game-changing for you and you should probably prioritize patching ASAP. If you are in a decentralized, unmanaged, or otherwise insecure environment - we’re looking at you large biz still running old Java for your horrible procurement application - this doesn’t make your life any worse. 

Keep checking for updates for your browsers and other applications as well as for your OSes, and have your interns rotate admins passwords periodically.

# Our Open Question

The biggest question we wrestled with was: how scalable is this attack? Our instinct is the vulnerabilities are most beneficial in a targeted attack scenario, rather than a large-scale exploitation or exfiltration, based on the following assumptions:

  * There seems to be a lot of variability in the targets that can affect exploitation (OS, patch level, specific details of chipset, etc.).
  * Exfiltration is generally untargeted - unless you know precisely where the data you want is in memory (and you probably don’t), you have to dump everything and sift through it later. This is fine for a small number of targeted systems but becomes unwieldy at 100,000+ victims (though it was pointed out data discovery tools could help with this). 
  * You are reading memory, not making changes to running systems, which would make worm/auto-propagation more difficult (though theoretically not impossible). 

But we are not particularly confident in this conclusion and eagerly await new information to help shed light on the matter.

So, for now, don’t freak out unless you have to, keep an eye (or an intern’s eye) on the breaking analysis and software updates, and start thinking about dusting off that old DEC Alpha or SPARC sitting in your closet.

As an aside, we note that CERT originally published a warning containing the following quote:

> The underlying vulnerability is primarily caused by CPU architecture design choices. Fully removing the vulnerability requires replacing vulnerable CPU hardware.

https://webcache.googleusercontent.com/search?q=cache:rzc6iQmgrIcJ:https://www.kb.cert.org/vuls/id/584653+&cd=1&hl=en&ct=clnk&gl=us

The post was later updated to remove that quote. We’re not sure why. 

References
* https://meltdownattack.com/
* https://spectreattack.com/
* https://googleprojectzero.blogspot.co.at/2018/01/reading-privileged-memory-with-side.html
* https://www.bleepingcomputer.com/news/security/mozilla-confirms-web-based-execution-vector-for-meltdown-and-spectre-attacks/
* https://blog.mozilla.org/security/2018/01/03/mitigations-landing-new-class-timing-attack/
* http://blog.cyberus-technology.de/posts/2018-01-03-meltdown.html
* https://venturebeat.com/2018/01/04/cert-only-way-to-fix-meltdown-and-spectre-vulnerabilities-is-to-replace-cpu/
* http://www.kb.cert.org/vuls/id/584653

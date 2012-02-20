---
layout: post
title: "iPhone launch musings"
category: Apple
tags: [Apple,iPhone]
---
{% include JB/setup %}

It's amazing how many articles about the iPhone have been produced online and in print over the last year or so, dedicated to covering the minutiae of Apple's big splash.  Frankly, not owning one, I'm getting a little tired of seeing articles about it everywhere I look.  So, I figure if you can't beat them, join them - and if every single other blog can have an opinion on the iPhone, so can I.  (This was actually written a few months ago, and I cleaned it up for publication in an attempt to catch up on my backlog of half-written articles.)

## Pricing

Early adopters flocked in droves to Apple and AT&amp;T stores to be the first kid on the block to buy a new iPhone.  And by all accounts, they are overwhelmingly happy with their new purchase.  They decided it was worth the purchase price, and chose to spend their money on what appears to be an amazing piece of tech.  And by and large, the iPhone has (unlike so many that have gone before it) lived up to the hype.

Then a few months after the launch, Apple have decided to lover the price, making it an even more attractive to the next wave of consumers.  If ever there was any doubt, the lower price will likely create a second wave of purchases.

It must be acknowledged that Apple faced a great deal of risk and uncertainty in launching their first ever product into the cut-throat mobile market.  And the high launch price reflects to some degree the risk involved in their gambit.  If they didn't reach their minimum break-even sales targets, it could have ended up a huge financial disaster.  Fortunately, it wasn't.  And now, thanks to the leverage of higher production volumes, it will cost Apple less per handset to manufacture, and they are able to pass on the savings to consumers and expand the market.

But a vocal minority of iPhone customers started complaining, whining, and even suing Apple for dropping the price after it had been on the market a few months.  What do these people think the world owes them?  How reasonable is it to expect Apple to lock in the price just so they don't feel stupid for not being a little more patient?  How long should Apple have waited before consumers would have been happy with a price cut instead of being upset?

I think it was extremely generous of Apple to offer a $100 voucher, to placate these infantile customers.  In any case, they had the benefit and use of the product for those months.  How exactly have they suffered?

## Unlocking

I can go out and buy a cheap subsidised phone, locked to a carrier.  It is a condition of sale.  It is the only way the company is able to offer the package without incurring a loss.  Sure, it would be <i>nice</i> to be able to choose any carrier, but for the iPhone, AT&T is the only game in town for now.  If you don't like it, don't buy it.

But then people complain that it's their device and they should be able to do what they want to it, including mess with its internals and trick it into using a different carrier.  Fine, go ahead - but you cannot expect Apple to support you, and you cannot expect future updates to continue to work with your hack applied.  It is difficult enough as it is to design and architect such a  complex system, and it would be nign impossible (not to mention unreasonable) for any supplier to provide fixes and new functionality, <i>and</i> provide workarounds that don't undo all these random hacks that have appeared.  Especially when some of the jailbreak techniques involve triggering security holes, they cannot expect a vendor to leave such an issue unfixed for their convenience.  Once you start patching the firmware, you have to accept that you're on your own.

## Third-party apps

Apple has produced an amazing v1.0 product in the iPhone, a truly revolutionary device which has had an incredible impact on the mobile phone industry.  Obviously they have an internal SDK which their own developers use.  And obviously they want the phone to be more attractive with a wider selection of apps.  But there is a price to pay in releasing an SDK...

Before the SDK came out, they could evolve the system, frameworks and APIs and change them as much as they liked to fine-tune the system, as they only had their own apps to worry about.

As soon as Apple releases an SDK, they are making a huge committment, and the APIs are essentially set in stone, as third-party developers will start to rely on them.  Apple loses the freedom of changing, breaking, removing, and morphing the system.  It is quite rare to get v1.0 of anything designed just right, so I daresay it took so long to put out the SDK not because they wanted to alienate people, but to iron out the kinks and get it right as much as possible.

And now, as I write this update, Beta 6 has been published, Version 2 of the firmware is around the corner, and WWDC is nearly upon us.  It's a very exciting time for ISVs, and I only wish I had the time to work on an iPhone product.  Having done some embedded development in the past, I really enjoy that space, and I actually have several ideas for iPhone apps that I may one day get around to hacking on.

## Limitations

So then the SDK comes out.  And once again, the pundits are out, nit-picking and complaining about its limitations.  People seem to focus one major issue: background processes.  And it is painfully obvious that those bandying about their opinions on such matters are ill-informed at best.  While the iPhone does an amazing job of providing a near-desktop user experience in a mobile device, it is still fundamentally a low-powered embedded computer.  And embedded development is simply <i>not</i> the same as developing for desktop applications, even though the tools, APIs and techniques are the same or very similar.  Apple has done an amazing job at providing a "Mobile Cocoa" toolchain, providing a platform that truly is years ahead of the other systems (such as Symbian or Windows Mobile).

Embedded development is characterised by what you *don't* have:

 - **Power**: the battery is a very finite and precious resource</li>
 - **Storage**: you can't assume you have many gigs of free space to play with.  A generous mobile device may only have a few meg available for *all* the apps to share.
 - **Memory**: similar to storage, mobiles don't have much RAM to play with, and usually do not have the luxury of virtual memory.</li>
 - **Processing power**: despite the rapid advances in low-power CPUs and integrated media decoding chips, mobile CPUs compromise processing power for power consumption, and err on the side of lasting longer.

So the upshot of all this is that you have to think very carefully about your data structures so as not to consume too much precious RAM.  You have to be frugal with what you store to avoid filling up the user's memory stick.  You need to be conscious of the complexity of the algorithms and the amount of number-crunching you do to avoid loading up the CPU.  Simply put, the harder the CPU works, the more current it draws and thus the shorter the battery will last.

One developer who "gets it" is Craig Hockenberry, who wrote a <a href="blog entry">http://furbo.org/2008/03/16/brain-surgeons/</a> on this very topic.  He has been experimenting with the unofficial toolchains and has some direct experience and good advice to share.

At first, background processes sound like a great idea.  I've got dozens of things running in the background right now, monitoring things, updating thinks, blinking lights, and so on.  But they all consume power and resources, and this can be a killer on a low-power device.  So while one background process on an iPhone might not seem like such a big deal, consider if 6 of your beloved third-party apps decided to run something in the background?  The available memory for your foreground app is going to be massively limited, as there are 6 other hogs using up resources.  And kiss goodbye to your battery - even if they only wake up every minute or so to check something, those 7 apps could cause the CPU to be waking up every 10 seconds, which is going to chew up the battery very rapidly indeed.

These constraints have been placed there for very pragmatic reasons.  First of all, given many potential iPhone developers are coming from the desktop space with presumably little embedded experience, and Apple is understandably worried that people will develop big, slow, bloated and inefficient apps that hog the system and damage the user experience.  But more significantly, given this is a Version 1 product, it is far easier for Apple to impose constraints now and relax them later on than to later on take away something developers have come to rely on or expect.  I'd do the same thing in their shoes.

The iPhone is an incredible new platform, and hopefully as developers come up to speed with its peculiarities, they will also come to appreciate the care required not just in the hallmark of interface design, but also consider algorithm and data structure design to work within the constraints of the hardware.

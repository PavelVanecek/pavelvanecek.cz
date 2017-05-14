---
title: If post office used HTTPS
date: 2017-05-14 14:03:51
---

The other day I found a drawing by [Mariko Kosaka](https://twitter.com/kosamari) about HTTPS (btw make sure to check out her other drawings too!):

<blockquote class="twitter-tweet" data-conversation="none" data-lang="en"><p lang="en" dir="ltr">Sending personal info over HTTP is like sending a postcard in English (anyone can read it in transit) - Why you should HTTPS <a href="https://twitter.com/hashtag/codedoodles?src=hash">#codedoodles</a> <a href="https://t.co/StJVKuCPcS">pic.twitter.com/StJVKuCPcS</a></p>&mdash; Mariko Kosaka (@kosamari) <a href="https://twitter.com/kosamari/status/841175583983841280">March 13, 2017</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

And I thought that this a great metaphor, but can we go deeper? There is much more going on in the HTTPS world, let's try and see how far can we get when we stick to the post office analogy.

## Plain HTTP

The _"S"_ in HTTP stands for _"Secure"_. There is no security and anyone can read and change the message. It's like a postcard.

When we say _anyone_ it does not mean really anyone. The message box still has a lock on it. The postman delivers to a post office, then another post office, and so on.
Your postcard can only be compromised if someone breaks the lock in the message box. Maybe one of the postmen is too curious. Maybe some post office has a deal with the local supermarket to add advertisement to all postcards. You don't want that. Entering - HTTPS!

## HTTPS

HTTPS means the message is encrypted. Nobody on the way can read it and you know who sent it - but not exactly.
It is like receiving a postcard in an envelope. On the envelope there is a stamp that says _"I am Bob. For realz."_ At this point you should be all like:
<center>"ಠ_ಠ Is it really Bob who sent this? How can I verify?"</center>
These are excellent questions! After all, anyone on the way can break the original stamp, change content of the postcard, and put it in a new envelope with a new stamp.

![Envelope stamp](/images/for_realz.png)
<center> _This is how I imagine an HTTPS certificate would look like on the envelope_ </center>

Fortunately, there are notaries who can help you answer that. You bring the stamp there and the notary says _"Yes, this is indeed Bob, he lives on Street 1, Townsville."_ Or they say _"Nope, this is not Bob, this is a forgery."_

How does that look like on the web? Your browser tells you in the web address bar:

![Envelope stamp](/images/https-error-1.png)
<center> _Chrome browser saying the certificate is invalid_ </center>

We call these notaries "[Certificate authorities](https://en.wikipedia.org/wiki/Certificate_authority)".

## Certificate Authorities

Certificate authority is a trusted third party. You depend on them for all your HTTPS communication. It is the only way to verify if Bob is really the one you are talking to.
You may be asking:

<center>"ಠ_ಠ What if someone pretends to be a CA? What if someone forges the notary office?"</center>

These are excellent questions too! Certificate authorities are not compromised often, but if it happens the consequences can be disastrous. The CA list is managed in operating system (think Windows) and in browser (think Firefox). People who maintain these will make sure that the CA list is up to date. That is yet another reason why you should keep your computer and software updated! These pesky "pls download updates now" notifications do have a purpose.

There is much more going on in HTTPS communication; you may find the [HTTPS handshake](http://robertheaton.com/2014/03/27/how-does-https-actually-work/) interesting too.

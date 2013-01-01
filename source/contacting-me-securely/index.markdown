---
layout: page
title: "contacting me securely"
date: 2013-01-01 00:38
comments: false
sharing: true
footer: true
---

For certain types of communication it's useful to send
encrypted or cryptographically signed e-mail, such as:

* Financial transactions
* Sensitive personal matters
* Technical communications regarding security
* You like to pretend you live in a Neal Stephenson novel. ;-)

In these circumstances (including the Neal Stephenson one),
I've typically used PGP/GPG to communicate securely by e-mail. To set up verified 
communications with me,

1. Download and set up an OpenPGP-compatible e-mail client. The easiest one
to use these days is probably [Thunderbird](http://www.mozilla.org/en-US/thunderbird/)
with the [Enigmail](http://www.enigmail.net/home/index.php) plugin. I also
use [mutt](http://www.mutt.org/), which can be compiled with built-in
PGP support.

    There's a 
good tutorial for setting up Enigmail [at Dragly](http://dragly.org/2010/02/13/getting-started-with-encrypted-e-mail-using-thunderbird-and-enigmail/).

1. Create a public/private key-pair for signing and encrypting email. (This 
is included in the Dragly tutorial.)

1. Download [my public key](http://blog.ajdecon.org/ajdecon-public-key.asc) and add it
to your key-ring.

1. Send me a signed e-mail with *your* public key as an attachment, or with
a link to your public key on the Internet.

1. Contact me through an alternative channel so we can verify the
[public key fingerprints](http://en.wikipedia.org/wiki/Public_key_fingerprint) of 
each others' keys. The point of this step is to  
ensure that I am the same person who owns the key
you downloaded from this website, and that you are the same person who 
owns the public key I received via e-mail; i.e., we have some additional
verification that
we are who we say we are.

    Good alternate channels include a phone call, a Twitter direct message,
   an instant message, or even meeting in person. Ideally this should be 
   a communications channel we've already been using, before we went to all
   the trouble of deciding to set up a secure email channel.

### Secure Instant Messages

I also use [OTR](http://www.cypherpunks.ca/otr/) over XMPP/Google Talk 
to exchange encrypted
instant messages. If we're both working on the same important servers, I may
ask you to set this up with me so we can be sure neither of us is a 
hacker trying to compromise the system. ("I lost my SSH key, can you please
give me access back with this new public key?")

OTR has a similar key-verification step using fingerprints as PGP. My current
preferred OTR client is [Jitsi](https://jitsi.org/), but 
[Pidgin](http://www.pidgin.im/) also has an OTR client from the CypherPunks
website.

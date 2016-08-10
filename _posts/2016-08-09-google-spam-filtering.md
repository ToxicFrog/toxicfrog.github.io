---
layout: post
title: Google Spam Filtering as a Service
---

So you're hosting your own MX for what I'm sure are very good reasons. But the spam is killing you. You don't want to migrate all your email (and users) to Google, but you'd gladly pay, say, $5/month for Google spam filtering and continue hosting the MX yourself.

Turns out you can, and it's not that hard, either:

* Sign up for a Google Apps for Work account. This is $5/month/user, and you only need one user, because you won't be hosting anything on it.
* Go to Gmail Setup and configure a mail host. Enter the hostname and port of your production MX. *Do not* tick `perform MX lookup on host`. Set the TLS options as appropriate for your MX.
* Then, edit `Default Routing` and add a new route with the following settings:
  * `Envelope recipients`: all recipients
  * `If it matches the above`: modify message
  * `Add X-Gm-Spam header`: yes
  * `Change route`: yes, and select the mail host you configured above
  * `Options`: Perform this action on non-recognized and recognized addresses

This will cause it to drop all spam on the floor, and forward everything else, to any address in your domain, to your real MX, with headers and `RCPT TO` intact. If you want it to forward spam as well, tick `also reroute spam` -- this will cause it to forward spam with an `X-Gm-Spam: 1` header added, allowing you to sort it on your end. (Note that even with this setting, you'll see a dramatic reduction in incoming spam, as gmail will drop really egregious spam at SMTP DATA time without ever attempting to deliver it.)

Now you can test it by sending mail addressed to you to the google MX (addresses available at Gmail Settings -> Advanced -> MX Records). Once you're confident that it's working, update your domain's MX records to point at the Google servers, and away you go. Now you can finally retire that creaky SpamAssassin install.

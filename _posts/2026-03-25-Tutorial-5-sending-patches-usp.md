---
title: "Tutorial 5 - Sending patches with git and an USP email"
date: 2026-03-25 23:10:31 -0300
categories: [Kernel, Tutorials]
tags: [linux, kernel, git, send-email, kw, patches, docker, oauth]
---

Moving on to tutorial 5, for a nice refresher!

This one is all about setting up the e-mail workspace to work in tandem with git, as kernel commits are done not via regular git/github, but via e-mail submissions!

## Disclaimer
By this point, its only expected that a disclaimer comes along with André's name.
For those who do not remember, all of the aforementioned progress in this blog took place in a userspace hosted in André's machine.
Therefore, the e-mail and docker setup was majorly centralized and could not be performed twice.
Hence, in this post I will link not only the usual [tutorial](https://flusp.ime.usp.br/git/sending-patches-with-git-and-a-usp-email/), 
but also André's [post](https://andre-jun.github.io/posts/tutoriais-5-6-send-email-usp/), 
where he goes in depth regarding the specifics of the setup and the challenges arisen from a shared userspace.

I will not leave you, my dear reader, hanging though! Worry not, for I will share some of his experiences in here as well.

## Issue 1 - USP E-mails
USP (University of São Paulo) uses Google institutional accounts that do not have access to 2FA.
The alternative method of Less Secure Apps is also out of question as USP blocked them after security incidents in 2025.

Not having either of these is a problem, as kernel contributions require App Passwords for G-mail accounts.

## Solution 1 - OAuth 2.0 Based E-mail Proxy
By using a proxy, we can delegate the authentication to Google without exposing any passwords. 
The proxy we used was [email-oauth2-proxy](https://github.com/simonrob/email-oauth2-proxy/), ran in a container provided by the tutorial:

```bash
git clone https://github.com/davidbtadokoro/emailproxy-container.git
```

With a handy repositoty, the process becomes relatively simple:
1. Fill the config file with the USP e-mail and provided client ID and secret
2. `docker compose up --build`
3. Initiate the proxy thorugh `emailproxy --no-gui --external-auth`
4. Setup `kw send-patch` to point to the local proxy (`127.0.0.1:2587`)

```bash
kw send-patch --setup --name 'nome'
kw send-patch --setup --email 'email@usp.br'
kw send-patch --setup --smtpuser 'email@usp.br'
kw send-patch --setup --smtpserver '127.0.0.1'
kw send-patch --setup --smtpserverport '2587'
```

---
title: "Little security enhancements"
datePublished: Mon Feb 12 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0ivj0r00ylzfs18xijgr15
slug: little-security-enhancements
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380977921/Cp956LwV2.jpeg
tags: security, passwords

---


Password managers are undervalued. Not only do they provide an easier way to store passwords, they offer so many little security enhancements I start to take for granted.

Take the latest phishing attempt. For a brief moment, there was [a fake Reddit website](https://www.helpnetsecurity.com/2018/02/06/reddit-phishing/) up and running trying to steal redditors login data. I haven't seen the page myself, but from the pictures I've seen, the site looks very believable. Down to the green HTTPS padlock.

Before I continue, let me reaffirm that a green padlock doesn't mean the site is legitimate, it just means that the communication between your device and the site is secure. You can talk to all the evil hackers in the world over a secure connection, that way you know only one party is accessing your data.

Enough about HTTPS, lets get back to password managers. They will stop you from filling in your credentials into a phishing site. The password manager knows which domain a password is tied to. It knows what password grants me access to my Google account. So when I visit [https://accounts.google.com/signin/v2](https://accounts.google.com/signin/v2) it will know which username and password to paste into the login fields.

It also knows that when I go to https://accounts.google.co, it is not Google. Notice that it ends in .co, not .com, just like the Reddit phishing site. I may overlook such details, my password manager does not. That's one of the indirect reasons it's so convenient. Without doing anything special, I'm protected from entering my credentials into a site where they don't belong.

Of course I could still find my password, copy paste the credentials into the login fields and then submit that data. Then again, that isn't your average kind of stupid, that is advanced stupid. Nobody and no technology is going to stop anybody from that.

When my password manager doesn't automagically fill in the login credentials, I investigate why it doesn't. The first thing I check is the domain and the second thing I look at is the configuration for the login I'm trying to use. Most of the auto fill problems are solved that way.

Hopefully, I have given you one more reason to try out a password manager or made you aware of one little security enhancement this wonderful technology brings to the table. If you don't have a password manager yet, head over to [1Password](https://1password.com/), my favourite, or [any other password manager](https://www.google.be/search?q=password+manager&oq=password+manager&aqs=chrome.0.69i59j69i61j0l4.2725j0j4&sourceid=chrome&ie=UTF-8).

> Disclaimer: I'm not sponsored by 1Password, nor do I get a discount or any other benefits. I fully support this service as a paying user because I think it is an awesome service.

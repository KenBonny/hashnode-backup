---
title: "Two Factor Authentication via 1Password"
datePublished: Mon Mar 05 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j20er0104yes1g4hgh14l
slug: two-factor-authentication-via-1password
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617381280542/nxICTXUXy.jpeg
tags: security, passwords

---


Seems like I'm talking a lot about [1Password](https://1password.com) (and password managers in general) these past few weeks. Well, it's because I think they are awesome and an invaluable tool if you want to secure yourself on the internet these days. In this article, I'm going to explain why you should use two factor authentication (2FA) and how you can set it up with 1Password, so you only need to do it once.

At a local meetup, I talked to a guy who was even more into securing himself online than I am. When we got talking about password managers and 2FA, also sometimes called one time passwords or OTP, he told me 1Password can generate those for me. That would mean I need one less app.

Previously, I've used [Authy](https://authy.com/) to generate my 2FA codes. It gives a nice overview of all the places where I use 2FA and it has a clean interface to generate codes. The best feature of Authy, is that it syncs the codes to a server. When I switch phones, I don't need to visit 15 sites to set up a new 2FA generation tool. I just install the Authy, log into my profile and get access to all my codes in an instant. There is also [a desktop client and a Chrome plugin](https://authy.com/download/) so you can get quick access to your codes on all your devices. Don't forget that 1Password has all these benefits, out of the box, too.

So Authy is a good tool for generating 2FA codes. However, I think 1Password is a better place to store my 2FA keys. It means I have all my sensitive information, such as my password, 2FA code generation and recovery codes, in one place. This is of course a risk, but one I'm willing to take.

There will be opinions out there that will say that storing passwords and 2FA codes in different apps is more secure because now an attacker needs to compromise 2 systems. They are completely right, but I'm willing to trust 1Password with all this information. If somebody hacks them, I'm screwed anyway because my Authy password is stored in 1Password.  So if it doesn't mater, I'm choosing ease of use over security. It's a conscious choice I make here. 1Password is very well protected, with multiple layers of defenses, so I trust them to keep everything safe. Even if a third party would get their hands on the data.

Both Authy and 1Password store the keys needed to generate the codes quite securely. Authy encrypts them using the strong [PBKDF2](https://en.wikipedia.org/wiki/PBKDF2) algorithm and 1Password uses multiple layers of security (including a PBKDF2 encryption layer) to make sure nobody but me gets access to my keys. Both have detailed how they securely store data, read [Authy's blog](https://authy.com/blog/how-the-authy-two-factor-backups-work/) here and [1Passwords blog](https://support.1password.com/1password-security/) here. The fact that they are talking about it openly, is a sign that they trust their security. This is [Kerchoff's principle](https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle) applied:

> A cryptosystem should be secure even if everything about the system, except the key, is public knowledge.

The setup of a 2FA is very straightforward and is very well explained on [1Password's information page](https://support.1password.com/one-time-passwords/). The video details how to do it on a Mac, but there is a small difference compared to the Windows version: there is no scanning tool _yet_. I may have seen a preview of the next version of 1Password for Windows.

Don't worry, the current way is easy as well. In most cases, I could copy the qr image by right clicking on it and selecting the copy menu item, then opening 1Password, selecting a custom field, changing the field type to "_One Time Password_" and and pasting in the clipboard image using the button next to the field.

![1password-otp-menu.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617894086056/zuWDIEKd9.png)

![2fa-paste-image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617894099200/T5MRQmAug.png)

When, for some arcane reason, a site does not allow me to copy the qr image, I used the Windows 10 Snipping Tool to take a screenshot of the qr code and paste that screenshot the same way I did before. I did not need to use the app on my phone at all. Which is not only great for ease of use, but allows people without a smartphone to use 2FA.

As a last step and to make it easy to find all logins that have 2FA enaled, I put a "2fa" tag on the login.

Another great feature I've only recently discovered, hidden inside a tool I've been using for more than a year. If you didn't know it yet, get yourself a password manager if you don't have one yet. I recommend [1Password](https://1password.com), but [any other](https://www.google.be/search?q=password+manager&oq=password+manager&aqs=chrome.0.69i59j69i61j0l4.2725j0j4&sourceid=chrome&ie=UTF-8) will provide good security.

> Disclaimer: I'm not sponsored by 1Password, nor do I get a discount or any other benefits. I fully support this service as a paying user because I think it is an awesome service.

---
title: "Set up MTA-STS on a GSuite hosted GitHub pages"
datePublished: Mon Feb 17 2020 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0izz6o00zryes12qhs2r3s
slug: set-up-mta-sts-on-a-gsuite-hosted-github-pages
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617460910291/vSPl4ILO2.png
tags: security, email

---


To further protect my email communication, I have enabled MTA-STS on my GSuite domain. My site is hosted on [GitHub pages](https://pages.github.com/), so I'll walk you through my setup.

It starts with creating a new [GitHub repository](https://github.com/KenBonny/MoreThanCodeMtaSts) that will hold the files for the MTA-STS subdomain. For some reason, the config for the MTA-STS is read from an `mta-sts.txt` file, located in the `.well-known` folder, but it has to be loaded from the `mta-sts` subdomain. Why it can't be done from the main domain is beyond me, but here we are.

Now that I have a repository, I create the `.well-known` folder and I place the `mta-sts.txt` file inside that folder. The content of the file can be found in my [GSuite Admin section](https://admin.google.com/u/1/ac/apps/cs/diagnostic). It is the middle value: MTA-STS Policy Diagnostic. I'll come back to the other values shortly.

Unfortunately, this is where I bumped into the problem with hosting on GitHub pages. By default, it does not expose folders starting with a . (dot). Probably because the servers are Linux based and any Linux folder starting with a dot, is automatically a hidden folder. So [Stack Overflow](https://stackoverflow.com/questions/55880919/how-to-add-well-known-on-github-pages-files-using-html) to the rescue!

The fix is as easy as adding a `_config.yml` file to the base repository with the single line:

```
include: [".well-known"]
```

Important detail: do not end with an empty line! Just add that single line to the file to expose the `.well-known` folder.

The last step in GitHub is to set up the custom domain for this repository. It's pretty easy to set up a GitHub pages domain, just be sure to include the subdomain before your domain.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381184313/JCGJ7WtE_.png)

Don't worry if GitHub displays an error, I have not set up the subdomain DNS yet, so it can't find the setup for the domain just yet.

I'll fix that right now. I let Cloudflare handle my DNS settings. In the DNS settings of the dashboard, I add 4 A records with the name of `mta-sts`, one for each IP-address that GitHub pages can handle. For more information about the specific setup of GitHub pages, I refer to their good [documentation](https://help.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain). Now that the IP redirects are set up, the subdomain should be ready and available.

Two more steps and I'm done. Luckily for me, they are both in my DNS setup. I add a TXT record with the name `_mta-sts` and the value found in my GSuite Dashboard after "MTA-STS TXT Record Diagnostic". I add another TXT record with the name `_smtp._tls` and the value found in my GSuite Dashboard after "Reporting Policy Diagnostic".

Do not forget to change the `rua=mailto:` value of the "Reporting Policy Diagnostic" text to an email address which you can receive. That is where reports will be sent to. In the near future, [Report URI](https://report-uri.com/) should get support to process the values.

Now I enjoy more secure email communication. If anybody wants to learn more about SMTP MTA Strict Transport Security, I recommend reading [Scott Helme's very good blog post](https://scotthelme.co.uk/improving-email-security-with-mta-sts/) or [URI Ports expanded blog post](https://www.uriports.com/blog/mta-sts-explained/). That's where I learned about it.

> Edit: Thanks Faisal from emailsecuritygeek.com for pointing out a typo. Cheers mate!

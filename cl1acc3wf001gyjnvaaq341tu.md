## Setting Fastmail DNS records for Cloudflare

Recently I switched from Googles Workspaces to [Fastmail](https://www.fastmail.com/) as my mail provider. Not only am I paying less, I'm getting more in return. I love the +notation alternative and that any, not-specified, email address gets dropped into my inbox. No more setting up info@mydomain.com, it just gets delivered to me. Setting all those DNS records is a bit time consuming, error prone and just annoying. So, scripting to the rescue.

Since I've been enjoying [F#](https://fsharp.org/) immensely the last few months (I'll probably blog some more about that in the future), I figured I'd write an F# script to create these DNS records automatically. And good news, I kept all secret bits out of it so I can share it with you, my dearest reader!

The script can be found in a public [Github Gist](https://gist.github.com/KenBonny/38afab3460002dfd167f8ff4a062fd98). I won't go over every detail of the script, but you can see that I create a list of DNS records, serialise them and then post them to Cloudflare.

How do you use the script? Copy the script to a file ending with `.fsx`. Make sure you have at least dotnet 5 installed. Then execute the following line

```
dotnet fsi [path-to-script] [domain] [cloudflare-auth-key]
```

Example: `dotnet fsi .\set-fastmail-dns-in-cloudflare.fsx kenbonny.net 00000000000-ABCdef`

The `cloudflare-auth-key` can be found in your [Cloudflare Dashboard](https://dash.cloudflare.com/profile/api-tokens). They have a [nice guide](https://developers.cloudflare.com/api/tokens/create/) on how to create a token.

Fastmail provided an easy [list of all the DNS records](https://www.fastmail.help/hc/en-us/articles/360060591153-Domains-Advanced-configuration) that should be set to make everything work.

%[https://twitter.com/Fastmail/status/1485706050439417869?s=20&t=-phWlNhkLMkhB0tpgG9wQg]

Keep one thing in mind: the Cloudflare API documentation is not up to date when it comes to [creating SRV DNS records](https://api.cloudflare.com/#dns-records-for-a-zone-create-dns-record). Other DNS records require a general JSON structure:

```json
{
  "type": "CNAME",
  "name": "fm1._domainkey.domain.com",
  "content": "fm1.domain.com.dkim.fmhosted.com",
  "ttl": 1,
  "priority": null,
  "proxied": false
}
```

An SRV DNS record needs a [specific JSON structure](https://community.cloudflare.com/t/error-creating-srv-dns-record/369216?u=user6631):

```json
{
  "type": "SRV",
  "data": {
    "service": "_caldavs",
    "proto": "_tcp",
    "name": "domain.com",
    "priority": 0,
    "weight": 1,
    "port": 443,
    "target": "caldav.fastmail.com"
  }
}
```

Hopefully this saves somebody some setup headache when configuring Fastmail on Cloudflare. Including me, when I add more domains to my account. 😁
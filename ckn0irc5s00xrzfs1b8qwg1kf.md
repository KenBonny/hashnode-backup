---
title: "Configure Ubiquiti to use 1.1.1.1 as DNS server"
datePublished: Mon Apr 09 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0irc5s00xrzfs1b8qwg1kf
slug: configure-ubiquiti-to-use-1-1-1-1-as-dns-server
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617380782488/sHaYtK2av.jpeg
tags: dns, cloudflare, network

---


Recently, [Cloudflare](https://www.cloudflare.com/) launched the privacy conscious (and very fast) DNS service [1.1.1.1](https://1.1.1.1/). I want to use it to resolve all my DNS needs. Setting this up via my [Ubiquiti](https://www.ubnt.com/) network was super easy.

All I had to do was go to my CloudKey controller that's registered in [Unifi cloud portal](https://unifi.ubnt.com). Once logged in, I went to _Settings_ (icon in the lower left part of the screen), navigate to the _Networks_ tab. Select the LAN to upgrade to the 1.1.1.1 DNS service. There is a setting named _DHCP Name Server_ that is set to _Auto_. Set it to _Manual_ and enter the two DNS server addresses: 1.1.1.1 and 1.0.0.1.

![1-main-screen.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617867397726/QorcJzvTI.jpeg)

![2-network-tab.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617867408953/Ceq_5wcjE.jpeg)

![3-dhcp-settings.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1617867419711/5Mkolads2.jpeg)

All that stands between me and the awesome 1.1.1.1 DNS service is a simple reboot of all network devices. Enjoy the new DNS service, although in practice you won't notice all that much. Except improved privacy of course.

**Update**: Not all traffic is safe yet, because at this moment Unifi products don't use DNS-over-HTTPS (DoH). There is a [feature](https://community.ubnt.com/t5/UniFi-Feature-Requests/Support-DNS-over-HTTPS/idi-p/2300758) [request](https://community.ubnt.com/t5/UniFi-Feature-Requests/Support-DNS-over-TLS/idi-p/2300760) going to get the Unifi Security Gateway to communicate with DNS servers over DoH if it's supported by the DNS Resolver. 1.1.1.1 does support DoH, but Unifi lacks support at this moment. If you want to use DoH, the "easiest" way is to host your own DNS Server and configure that to talk to 1.1.1.1 over DoH. You can find a guide on [Scott Helme's blog](https://scotthelme.co.uk/securing-dns-across-all-of-my-devices-with-pihole-dns-over-https-1-1-1-1/).

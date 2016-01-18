---
layout: post
title: Getting all DNS records form a Internet domain via nslookup
created: 1298275200
redirect_from:
  - /2011/02/02/getting-all-dns-records-for-a-internet-domain-via-nslookup/
  - /2011/02/21/getting-all-dns-records-form-internet-domain-nslookup/
  
---
This is a quick and easy way to grab all DNS records from a domain. All you need is the name of the domain you want to query and the DNS server that is handling the domain. Here is how you do:

Open up a command prompt and type:

~~~dos
C:\> nslookup - your.dns.server
~~~

In the nslookup prompt, type:

~~~dos
> set q=any
> ls -d domain.name
~~~

Hope this helps.

---
layout: post
title: Domain Fronting
categories: [Security]
---

[Domain fronting](https://en.wikipedia.org/wiki/Domain_fronting) is a technique
that allows circumventing HTTP domain restrictions by leveraging the fact that
some large web content providers do not check that the domain specified in the
`Host:` header of the HTTP request matches the server_name in the TLS handshake.

As a practical example, take two sites hosted on GitHub Pages:
`octocat.github.io` and `jekyllrb.com`. Both domains resolve in DNS to the same
four IP addresses. Typically, web clients will use Server Name Indication (SNI)
to indicate to the server which of its many domains it is requesting and thus
which TLS certificate to present.

However, look what happens if we initiate a TLS connection to one domain but set
an HTTP `Host:` header for the other domain:

```
$ curl -v -H "Host: jekyllrb.com" https://octocat.github.io
*  subjectAltName: host "octocat.github.io" matched cert's "*.github.io"
...
> GET / HTTP/1.1
> Host: jekyllrb.com
...
< HTTP/1.1 200 OK
...
<title>Jekyll • Simple, blog-aware, static sites | Transform your plain text into static websites and blogs</title>
```

We have used SNI to tell the server to present the TLS certificate for
`octocat.github.io` and then set a HTTP `Host:` header to indicate to the server
to serve the request from `jekyllrb.com`.

## Implications for Internet security

Domain fronting is potentially a big problem for organisations that use web
proxies like [Squid](http://www.squid-cache.org/) to restrict outbound HTTP
requests. Proxies like Squid will proxy TCP streams between a client and a
remote server using the HTTP ``CONNECT`` method. The proxy can restrict on the
domain in the `CONNECT` but cannot see inside the TCP stream so cannot check
what domain the client has specified in the `Host:` header.

This means attackers could bypass egress proxy restrictions using domain
fronting. The domains must be on the same hosting provider for domain fronting
to work, and this is more likely where a proxy allows connections to domains
hosted on certain large CDNs like Cloudflare.

[Google](https://arstechnica.com/information-technology/2018/04/google-disables-domain-fronting-capability-used-to-evade-censors/)
and
[Amazon](https://aws.amazon.com/blogs/security/enhanced-domain-protections-for-amazon-cloudfront-requests/)
both disabled support for Domain Fronting in their services in
April 2018, apparantly to stop widespread sensorship evasion. However, as the
above example shows, it still appears to be enabled in providers like GitHub.


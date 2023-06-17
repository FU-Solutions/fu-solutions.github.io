---
title: HowTo - Hardening the Webserver - HTTP Header
author: w3ich3rt
date: 2021-11-24 14:35:00 +0100
categories: [Linux]
tags: [linux, ssh, config, shell]
math: true
mermaid: true
image:
  path: /assets/img/linux.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Linux Logo
---

I recently wrote a Flask web application in Python. When I put it online, I of course ran Nikto over it. Since I was immediately confronted with the usual suspects, I wanted to write down the solution for this weak point as a reminder to myself and possibly for your support.

## Vulnerabilities

What vulnerabilities do we encounter again and again? During my *day job* I keep coming across these messages:

```shell
nikto -host https://beispiel_webservice.de
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          127.0.0.1
+ Target Hostname:    beispiel_webservice.de
+ Target Port:        443
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /CN=beispiel_webservice.de
                   Altnames: beispiel_webservice.de
                   Ciphers:  TLS_AES_256_GCM_SHA384
                   Issuer:   /C=US/O=Let's Encrypt/CN=R3
+ Start Time:         2021-11-24 11:33:57 (GMT1)
---------------------------------------------------------------------------
+ Server: nginx
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Allowed HTTP Methods: GET, OPTIONS, HEAD 
+ 7499 requests: 0 error(s) and 4 item(s) reported on remote host
+ End Time:           2021-11-24 11:45:27 (GMT1) (690 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

### Anti-Clickjacking X-Frame-Options

Clickjacking is when the attacker uses various transparent or opaque layers to trick a user into clicking on a link. These links are embedded in pages and point to other pages.... in simplified terms.
This error can be prohibited or configured by using the `X-Frame-Options` header.
All that is needed is to set the header:

```shell
X-Frame-Options: DENY  #Es wird vollständig verweigert, in einen Frame/iframe geladen zu werden.
X-Frame-Options: SAMEORIGIN #Erlaubt nur, wenn die Seite, die geladen werden soll, den gleichen Ursprung hat.
X-Frame-Options: ALLOW-FROM URL #Es erlaubt einer bestimmten URL, sich selbst in einem Iframe zu laden. Bitte beachten Sie jedoch, dass nicht alle Browser dies unterstützen.
```
In `nginx` the header is added in the config file via `add_header X-Frame-Options "DENY";` and in Apache via `Header set X-Frame-Options: "SAMEORIGIN"`. 

### X-XSS-Protection header

The X-XSS Protection Header protects the website from potential cross-site scripting. In modern browsers, however, this header is already deprecated and can lead to client problems. Nevertheless, it is recommended to set X-XSS-Protection to 0.

In `nginx` the header is added in the config file via `add_header X-XSS-Protection "0; mode=block";` and in Apache via `Header set X-XSS-Protection "0; mode=block"`. 

### HTTP methods

The various HTTP methods `GET`, `POST`, `HEAD` and so on, can also be used to execute mischief and mischief on the web pages. Of course, malicious code can also be placed on a web server using these methods.

Therefore, one should check which methods are really needed and the unnecessary methods can be forbidden.

In `nginx` you would put the following code in the server section
```shell
add_header Allow "GET, POST, HEAD" always;
if ( $request_method !~ ^(GET|POST|HEAD)$ ) {
	return 405;
}
```
Please be careful to only disallow the methods you do not want to offer.

### X-Content-Type-Option

The X-Content-Type option header in HTTP responses is used by the server to indicate that [MIME types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types) as specified in the [Content-Type (en-US)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) headers should not be changed and thus followed. This allows [MIME type sniffing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types#MIME_sniffing) to be ruled out. Or in other words, it shows that the web admins know what they are doing.

In `nginx` the header is added in the config file via`add_header X-Content-Type-Options "nosniff";` and in Apache via `Header set X-Content-Type-Options "nosniff"`.

## Summary

This should make the next scan look better:

```text
nikto -host https://beispiel_webservice.de
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          127.0.0.1
+ Target Hostname:    beispiel_webservice.de
+ Target Port:        443
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /CN=beispiel_webservice.de
                   Altnames: beispiel_webservice.de
                   Ciphers:  TLS_AES_256_GCM_SHA384
                   Issuer:   /C=US/O=Let's Encrypt/CN=R3
+ Start Time:         2021-11-24 12:49:25 (GMT1)
---------------------------------------------------------------------------
+ Server: nginx
+ X-XSS-Protection header has been set to disable XSS Protection. There is unlikely to be a good reason for this.
+ Allowed HTTP Methods: GET 
+ 7499 requests: 0 error(s) and 4 item(s) reported on remote host
+ End Time:           2021-11-24 13:00:53 (GMT1) (688 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

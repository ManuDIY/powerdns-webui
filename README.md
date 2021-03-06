# General Disclaimer

This project has no connection whatsoever with [PowerDNS.COM BV](https://www.powerdns.com/contact.html),
[Open-Xchange Inc](https://www.open-xchange.com/) - or any other third party.

It is an independently funded & maintained development effort.


# First Issue

I've open a [first issue](https://github.com/james-stevens/powerdns-webui/issues/18) called `Just for Chatting`,
so if you want to leave any comments or feedback without opening your own issue, you can use this.

If you know of a better way to achieve this fiunctionality, please let me know. I couldn't find any kind of aimless chat feature.



# PowerDNS WebUI

`htdocs/index.html` is a complete self-contained, single-file, single page HTML, CSS & Javascript webapp
which allows you to browse and edit DNS data held in a PowerDNS Database using only the PowerDNS RestAPI.
You can clone the project, if you want, but this is the only file you need in order to add a complete WebUI to your PowerDNS Server.

It is primarily aimed at those who are using PowerDNS as a DNS Master, as this is what I do,
but it should handle native / slave zones OK.
If you are using this webapp for slave / native, please let me know if there are features it needs.

When minified, using `python -m jsmin index.html > min.html`, the savings are not great (~10%), so I've chosen to no longer provide this minified file.
This is probably largely because I use TAB indents (see CONTRIBUTING.md).

Becuase of the security limitations of the PowerDNS Rest/API, this is intended as a SysAdmin tool only - see the **Security** Section below.

A critial design goal was to ensure that the data you see has come live from the server, and you have a single
click button in the navigation bar that will reload the data you are seeing.

The only exception to this is occasionally when the `NSEC3PARAM` value is displayed on the DNSSEC page.


# Status

The main thrust of this development is now complete - I think :)

This is a summary of the features this WebUI provides to PowerDNS

* **Servers** - contact PowerDNS Servers directly, or indirectly though a web proxy, HTTP or HTTPS (see `Browser Security Restrictions` below)
* **Zones** - Add, View, Remove, Sign, Unsign, Force NOTIFY, Rectify, Download in RFC format, force update (slave only)
* **Metadata** - Add, Edit, Remove Metadata items or individual values, with some clientside validation, including picking drop-downs where specific metadata items have a limited range of values
* **Hosts/names** - Master or Native only - Add, Edit, Remove RRs / RR-Sets with some clientside validation, Change the TTL of an RR-Set. Copy records, including between zones, by renaming the RR-Set
* **TSIG Keys** - Add, Regenerate, Remove, click to copy name or digest to clipboard. NOTE: Adding multiple TSIG keys, of different algorythms, does not work in PowerDNS v4.2.0
* **Search** - quick access to native search facility, with click-through to records / zones
* **Navigation** - fully functional BACK button, link to open any page in a new tab (or link you can email etc)
* **DNSSEC**
	* Sign an unsigned zone - NSEC or NSEC3, KSK+ZSK or CSK, any algorythm & key lengths
	* Unsign a signed zone - NOTE: removing the NSEC3 param record using the Rest/API does not work in PowerDNS v4.2.0
	* Step-by-Step one-button CSK, KSK or ZSK key roll-over
	* Add, Remove, Activate / Deactivate individual keys
	* DS digest, click to copy digest to clipboard
	* Convert NSEC to NSEC3 or vice versa. NOTE: removing the NSEC3 param record using the Rest/API does not work in PowerDNS v4.2.0
	* NSEC3PARAM roll-over - Yeah, some people like to do it. What can you say.
* **Stats** - ability to view all server stats data, including breaking out data presented in lists

`Fast Zone Listing` allows you to view the full list of zones considerably faster, at the cost of slightly reduced information. Where you have 1000s of zones, 
or a server a long way away, this can significantly improve your performance.

Items that probably could be improved (apart from my spelling ... dyslexia sucks)
* I'd like to be able to automatically maintain a [bind-9.11 catalog zone](https://kb.isc.org/docs/aa-01401), for those who use RFC (not native) slaves.


When reporting an issue, please also include any messages in your browser console (in Chrome press F12, in FireFox Ctrl-Shift-J).



# Browser Security Restrictions

This webapp is super simple to use, but does require a little setting up to ensure your browser is happy with stuff.
These issues are generic browser security restrictions, and not specifically to do with this code.

* If your browser received the `index.html` (this webapp) over HTTPS, then the RestAPI **must** be accessed over HTTPS - this is where
using an HTTP/HTTPS proxy is useful. As of v4.2.0, PowerDNS does not natively support HTTPS, and sending all your data
(and maybe your API Key) over HTTP is probably not what you want.

* You must be [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) compliant - in this context it means the web server
that gave your browser `index.html` must list (in the header of the response) all the other HTTP/S servers you are allowed to access via the webapp.

NOTE: For CORS, by default, you are allowed to access the Rest/API on the server that sent you the webapp, regardless of the port number.
So this requires no special extra consideration.

So, for exmaple, you could obtain the `index.html` page from port 80 (HTTP) but, on the same server, still be OK to access the PowerDNS API directly (port 8081, and **not** HTTPS).

But, I strongly recommend you simply use a `HTTPS` proxy.

# The Example Config

We have provided a fully working example set-up in the `example` directory.

Because this webapp accesses the PowerDNS RestAPI directly from your desktop's browser, to prevent you having to give everybody the `api-key`,
we would recommend you use a web proxy (e.g. Apache or nginx) and (for exmaple) enforce per-user authentication in the proxy.
This means you will need to configure the proxy to add the `api-key` to each request (see below).

You can also use the web proxy to provide an HTTP->HTTPS service, transparently adding `HTTPS` support to the RestAPI.

I used both Apache & nginx. Here's a snip of my setup. It assumes your PowerDNS WebUI is listening on IP Address 127.1.0.1
and your Apache Server can listen on port 443 (HTTPS). The PowerDNS IP Address will probably work for you.

I haven't included the SSL, or per-user authentication, config lines, you will need to add whatever you prefer,
but all the SSL & Basic Authentication configuration is included in `example/httpd.conf` and `exmaple/nginx.conf`.

```
<VirtualHost *:443>

	DocumentRoot /opt/websites/pdns/powerdns-webui/htdocs

	<Proxy http://127.1.0.1:8081/*>
		Allow from all
	</Proxy>

    <location /stats/>
        ProxyPass http://admin,Dev-Key@127.1.0.1:8081/
        ProxyPassReverse http://admin,Dev-Key@127.1.0.1:8081/
    </location>

	<location /api>
		Header add X-API-Key "Dev-Key"
		RequestHeader set X-API-Key "Dev-Key"
		ProxyPass http://127.1.0.1:8081/api
		ProxyPassReverse http://127.1.0.1:8081/api
	</location>

</VirtualHost>
```

Becuase I want the webapp to live in the ROOT directory of the website, this overloads the PowerDNS stats page (which also lives at the root),
so I have put in a rule that makes the stats page available from `https://<server-ip-address>/stats/`. Although, of course, this webapp
also gives you access to the stats that are available from the Rest/API.

You will need to ensure you have loaded the Apache proxy modules, I used this code

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_connect_module modules/mod_proxy_connect.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule rewrite_module  modules/mod_rewrite.so
```
Here's the corresponding PowerDNS `pdns.conf` settings for the WebUI & Rest-API

```
...

webserver=yes
webserver-address=127.1.0.1
webserver-allow-from=127.0.0.0/8
webserver-password=Dev-Key
api=yes
api-key=Dev-Key

...

```


Clone this project as the directory `/opt/websites/pdns/powerdns-webui`,
or whatever you chose in the Apache conf, and request the URL `https://<server-ip-address>/`

If it worked correctly, you should see a screen like this.

![Frist Screen](/first.png)

Because it prompts you for a server name, you can use one copy of this webapp to access any PowerDNS RestAPI
you can reach, subject to the browser restrictions described above.

A fully working example configuration, and instructions, are provided in the `example` directory.


# An Easier Configuration

A less secure configuration, that is easier to get working, is to set up PowerDNS to listen on the same IP Address
as a standard HTTP web server, then load the webapp's `index.html` from the web server and tell the webapp to connect directly 
to the PowerDNS API on port 8081, by adding `:8081` after the server name / ip address. You will also need to enter the 
`web-api` key and untick the `HTTPS` tick box.

This should satisfy your browser's security checks. I have tested this in both Chrome and Firefox.  
However, it means all your data, and your API key, will be sent in plain text.


# In Operation #

I've tested this with the latest Chrome & Firefox running on Xubuntu (Ubuntu + XFCE) talking to a 95% idle PowerDNS server
running v4.2.2 over an 18ms latency link and the response time, for all actions, including viewing a zone with 1000 records
(500 names, 2 records per name), is virtually instant.

Apart from some minor aesthetic differences, the behaviour in Chrome and Firefox was identical. 
As far as I know, any ES6 compliant browser should work, but I might be wrong.

nginx performed the same as Apache - virtually instant.


# Security #

This webapp is intended as a SysAdmin aid, and not to be given directly to end-users without the addition of more serverside security.
Especially, this webapp is not recommended in the situation where you have multiple users owning different domains.

There are deliberatly **NO** security options in this webapp, e.g. who can edit/delete zones/names/records etc.

As a general principal, when you have a JavaScript+RestAPI webapp the place to put the security is in the serverside RestAPI.
Any security put into the Javascript can probably be trivially circumvented and is therefore of extremely limited value.
In fact, in security circles, this is considered worse than having no security, as less experienced sysadmins may be left thinking
they are safe, when this is not the case.

In various web proxies, there are options to block certain `METHODs`. For example, by blocking all `METHODs` except `GET`,
you can stop a user from being able to make changes. For more information, please ask Google.

In general, therefore, as it is provided, this webapp is probably not going to be that useful for giving to end users.
However, as an sysadmin tool, it can be very useful.


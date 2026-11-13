# MODULE W2-PM1 & W2-PM-FINAL • SECURITY ASSESSMENT

# Active Reconnaissance and WAF Detection Report: networkwalks.com

**Infrastructure Enumeration, Web Technology Profiling, and Defensive Control Identification**

## 1. Executive Summary

This report documents the active reconnaissance and Web Application Firewall (WAF) detection assessment performed against the target domain **networkwalks.com**. Conducted from a Kali GNU/Linux testing environment, multiple command-line reconnaissance utilities were executed to map domain infrastructure, enumerate DNS records, inspect HTTP response headers, fingerprint the underlying web stack, and detect perimeter defensive controls protecting the target web application.

> **Key Finding:** Active WAF probing via `wafw00f` confirmed that https://networkwalks.com is protected behind a **ModSecurity (SpiderLabs)** Web Application Firewall, hosted on an Apache web server running WordPress.

## 2. Target & Assessment Environment Overview

| Parameter | Observed Value / Configuration |
|-----------|--------------------------------|
| Target Domain | networkwalks.com (Networkwalks Academy) |
| Primary IPv4 Address | 192.232.216.135 (United States) |
| Assessment Workstation | Kali GNU/Linux Rolling (x86_64), Kernel 7.1.5+kali-amd64 (VirtualBox) |
| Tools Utilized | `wafw00f` (v2.4.2), `whatweb`, `dnsrecon`, `whois`, `nslookup`, `curl` |

## 3. Tool-by-Tool Reconnaissance Findings

### 3.1 Web Application Firewall Detection (`wafw00f`)

To determine whether an application-layer firewall filters or inspects incoming HTTP/HTTPS traffic before it reaches the origin web server, `wafw00f` (v2.4.2) was executed against the target domain.

```bash
$ wafw00f networkwalks.com
[*] Checking https://networkwalks.com
[+] The site https://networkwalks.com is behind ModSecurity (SpiderLabs) WAF.
[~] Number of requests: 2
```

- **Detection Outcome:** Positive identification of ModSecurity (SpiderLabs) WAF across 2 HTTP/HTTPS probe requests.
- **Security Implication:** Automated scanning and malformed payloads will trigger signature-based inspection, rate limiting, or request blocking at the web server layer.

*Figure 1: Execution of `wafw00f networkwalks.com` confirming ModSecurity (SpiderLabs) WAF protection.*

### 3.2 Web Technology & CMS Profiling (`whatweb`)

`whatweb` was executed to fingerprint web server software, client-side libraries, content management systems, and HTTP cookies. Initial default requests encountered connection timeouts, whereas specifying a standard browser User-Agent string yielded full technology enumeration across the HTTP-to-HTTPS redirect chain.

```bash
$ whatweb --user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64)" networkwalks.com
```

| Technology Layer | Identified Component & Details |
|------------------|-------------------------------|
| Web Server | Apache (HTTP 301 Moved Permanently redirect from HTTP to HTTPS 200 OK) |
| CMS & Plugins | WordPress 7.1.4, WordPress Download Manager 3.3.76 (Page Title: Networkwalks Academy) |
| Frontend Libraries | Bootstrap 7.1.4, jQuery 3.7.1, HTML5, Open Graph Protocol, Google Tag Manager |
| Cookies & Headers | `__wpdm_client` (HttpOnly, Secure), X-Endurance-Cache-Level, X-nginx-cache |
| Contact Metadata | info@networkwalks.com |

*Figure 2: Web stack fingerprinting using `whatweb` with a browser User-Agent header.*

### 3.3 Comprehensive DNS Enumeration (`dnsrecon` & `nslookup`)

Standard DNS enumeration was conducted via `dnsrecon -d networkwalks.com` and verified with `nslookup` to map authoritative nameservers, mail routing infrastructure, SPF policies, and service discovery records.

```bash
$ dnsrecon -d networkwalks.com
```

| Record Type | Enumerated Value |
|-------------|------------------|
| A Record | networkwalks.com → 192.232.216.135 |
| SOA & NS | ns6135.hostgator.com (50.87.144.87) and ns6136.hostgator.com (192.232.216.131) running BIND 9.16.23-RH |
| MX Record | mail.networkwalks.com → 192.232.216.135 |
| TXT Records | SPF policy: `v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all`; Google site verification token |
| SRV Records | 8 `_autodiscover._tcp.networkwalks.com` records resolving to cpanelemaildiscovery.cpanel.net (184.94.203.x / 184.94.204.x, port 443) |
| DNSSEC | Unsigned (no DNSSEC record returned) |

*Figure 3: Full DNS record enumeration output from `dnsrecon -d networkwalks.com`.*

### 3.4 Domain Registration Intelligence (`whois`)

A `whois` lookup was performed to examine registrar metadata, domain lifecycle dates, and domain-level lock statuses.

- **Registrar:** GoDaddy.com, LLC (IANA ID: 146)
- **Creation & Expiration:** Created on 2019-11-06; valid through 2027-11-06 (last updated 2025-11-12)
- **EPP Status Codes:** Protected against unauthorized modifications via `clientDeleteProhibited`, `clientRenewProhibited`, `clientTransferProhibited`, and `clientUpdateProhibited`

*Figure 4: Domain registration and EPP status details retrieved via `whois networkwalks.com`.*

### 3.5 Direct HTTP Header Inspection (`curl`)

A HEAD request (`curl -I https://networkwalks.com`) was issued to examine raw HTTP response headers and caching directives returned by the server.

- **Response Status & Server:** HTTP/1.1 200 OK served by Apache with caching headers `X-Endurance-Cache-Level: 0` and `X-nginx-cache: WordPress`
- **Security & API Headers:** Enforces `Referrer-Policy: no-referrer-when-downgrade`, sets a `Permissions-Policy` restricting private-state-token redemption to trusted verification providers (Google, Cloudflare, hCaptcha), and advertises the WordPress REST API endpoint via the Link header (`https://networkwalks.com/wp-json/`)

*Figure 5: Raw HTTP response headers captured via `curl -I https://networkwalks.com`.*

## 4. Conclusion & Defensive Summary

The active reconnaissance workflow successfully mapped the external attack surface and defensive posture of networkwalks.com:

1. **Perimeter Protection:** The presence of ModSecurity (SpiderLabs) WAF provides active request filtering, requiring careful adherence to authorized testing scopes and rate limits.
2. **Hosting & Application Architecture:** The domain is registered through GoDaddy and hosted on HostGator/cPanel infrastructure running an Apache web server with an Nginx caching layer and a WordPress CMS (Networkwalks Academy).
3. **DNS & Mail Hygiene:** Domain transfers are locked at the registrar level and SPF email authentication is configured, though DNSSEC remains unsigned and software version banners (WordPress, jQuery, Bootstrap, and BIND) are publicly exposed in response metadata.

---

*Active Reconnaissance & WAF Detection Report — networkwalks.com*  
*Page 1–6 of 6*

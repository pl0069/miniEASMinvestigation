# Passive EASM Investigation

A small **External Attack Surface Management (EASM)** exercise focused on discovering public-facing assets, identifying infrastructure, and validating web technologies using publicly available information.

> **Target:** `example.com`
> **Scope:** Passive/public reconnaissance only
> **Note:** `example.com` is an official documentation/example domain. No exploitation or vulnerability testing was performed.

---

## Investigation Flow

```text
Domain
  ↓
DNS / TLS
  ↓
Subdomain Discovery
  ↓
HTTP Recon
  ↓
URLScan
  ↓
HTML Validation
  ↓
Final Findings
```

## 1. DNS Recon

**Tool:** `nslookup`

### Findings

```text
A:
172.66.147.243
104.20.23.154

NS:
hera.ns.cloudflare.com
elliott.ns.cloudflare.com
```

The IP addresses and nameservers indicate that **Cloudflare** is handling the public-facing DNS/CDN layer.

The discovered IPs were treated as Cloudflare edge infrastructure rather than confirmed origin servers.

---

## 2. Certificate & Subdomain Discovery

**Tools:**

* Certificate Transparency
* Pentest-Tools

### TLS Certificate

```text
Subject:
example.com

SAN:
example.com
*.example.com

Issuer:
Cloudflare TLS Issuing ECC CA 3

Validity:
2026-09-26 → 2026-12-25
```

Certificate/subdomain discovery produced approximately **1,000 hostname candidates**.

DNS validation showed that the discovered candidates did not currently resolve.

### Result

```text
Confirmed active hostname:
example.com

Additional active subdomains:
None identified
```

**Lesson:** A discovered hostname is not necessarily an active asset.

---

## 3. Technology Fingerprinting

An automated technology fingerprinting service initially reported technologies including:

* Next.js
* Nuxt.js
* React
* Vue
* Angular
* TypeScript
* Tailwind CSS
* Apache
* Google Sign-In
* LaunchDarkly
* Cloudflare
* reCAPTCHA

The results were then validated against the actual HTML response.

### Validation Result

The page contained:

* Static HTML
* Inline CSS
* `/s.js`
* Link to `iana.org`

No clear evidence of the reported application frameworks or services was found.

**Lesson:** Automated technology fingerprinting should be treated as an indicator, not confirmed architecture.

---

## 4. HTTP Recon

**Tool:** SecurityHeaders.com

### HTTPS

Observed:

```text
HTTP/2 200
Server: cloudflare
CF-Cache-Status: HIT
Alt-Svc: h3=":443"
```

The following security headers were not observed:

* Strict-Transport-Security
* Content-Security-Policy
* X-Frame-Options
* X-Content-Type-Options
* Referrer-Policy
* Permissions-Policy

### HTTP

One scan returned:

```text
HTTP/1.1 200 OK
Server: cloudflare
```

Another tool later observed an HTTP → HTTPS redirect.

Because the tools produced different results, the redirect behavior was treated as something requiring further validation rather than a definitive finding.

---

## 5. URLScan

**Tool:** URLScan.io

URLScan observed:

```text
1 IP
1 domain
2 HTTP transactions
```

### Primary request

```text
GET /
200 OK
HTTP/3
text/html
172.66.147.243
```

### JavaScript

```text
GET /s.js
200 OK
text/javascript
```

No useful additional information was obtained from `/s.js`.

URLScan also identified a link to:

```text
iana.org
```

This was treated as an external link rather than an organization-owned asset.

---

## 6. HTML Validation

The main HTML response was inspected directly.

The page contained:

```html
<title>Example Domain</title>
```

The page is the standard Example Domain used for documentation examples.

The observable application consisted of:

* Static HTML
* Inline CSS
* One small JavaScript file
* One external link

No application API, authentication functionality, or meaningful client-side framework was identified.

---

# Final Findings

| Category                    | Finding                           |
| --------------------------- | --------------------------------- |
| Active domain               | `example.com`                     |
| DNS/CDN                     | Cloudflare                        |
| Public IPs                  | `172.66.147.243`, `104.20.23.154` |
| Origin IP                   | Not identified                    |
| Active subdomains           | None identified                   |
| TLS                         | Cloudflare-issued certificate     |
| HTTP/3                      | Observed                          |
| Web application             | Static Example Domain page        |
| `/s.js`                     | Public JavaScript resource        |
| Additional useful endpoints | None identified                   |

---

# Tools Used

| Tool                      | Purpose                             |
| ------------------------- | ----------------------------------- |
| `nslookup`                | DNS reconnaissance                  |
| Certificate Transparency  | Certificate and hostname discovery  |
| Pentest-Tools             | Subdomain discovery                 |
| Technology fingerprinting | Initial technology identification   |
| SecurityHeaders.com       | HTTP security-header reconnaissance |
| URLScan.io                | Passive HTTP/web reconnaissance     |
| Browser / HTML inspection | Technology and content validation   |

---

# Key Takeaways

### 1. Discovery ≠ active asset

Certificate Transparency produced many hostname candidates, but DNS validation showed that they were not currently resolving.

### 2. Scanner results need validation

The technology scanner reported many frameworks, but direct inspection of the HTML did not support those findings.

### 3. CDN IP ≠ origin IP

The discovered IP addresses belonged to Cloudflare, so they were not treated as confirmed origin infrastructure.

### 4. Cross-check findings

Different reconnaissance tools produced different observations about HTTP → HTTPS behavior. Conflicting results should be revalidated.

### 5. EASM starts with understanding the attack surface

Before looking for vulnerabilities, first establish:

* What assets exist?
* Which assets are active?
* Who owns the infrastructure?
* What technologies are exposed?
* What information is publicly observable?

---

## Conclusion

This exercise demonstrated a basic passive EASM workflow:

```text
Discover
   ↓
Validate
   ↓
Attribute
   ↓
Fingerprint
   ↓
Cross-check
   ↓
Document
```

The main lesson was that **automated reconnaissance provides leads, not conclusions**. Findings should be validated using multiple sources and direct evidence before being included in an EASM assessment.

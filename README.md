# cutenews-2.0-CVEs-2026-Disclosure
Cutenews (https://github.com/CuteNews/cutenews-2.0) Multiple CVEs 2026 Disclosure Repository

**Vendor:** CuteNews

**Affected Product:** [CuteNews v.2.1.2](https://github.com/CuteNews/cutenews-2.0) (last commit `16d630d`, Feb 21 2025)

**Fix Status:** No fixed version at this time

---

## CVE-2026-36467 — Unrestricted Upload of File with Dangerous Type

| Field | Detail |
|---|---|
| **CWE** | CWE-434: Unrestricted Upload of File with Dangerous Type |
| **Component** | `core/modules/media.php:175–255` — `upload_from_inet` (Media Manager) |
| **Attack Type** | Remote |
| **Impact** | Code Execution |

**Description:**
Unrestricted Upload of File with Dangerous Type in `core/modules/media.php` allows remote authenticated users with access to the Media Manager panel to execute arbitrary code in the context of the web application, leading to remote server access by triggering a reverse shell.

**Attack Vector:**
Any user with a role that can access the Media Manager panel can exploit an SSRF vulnerability in the "Upload by URL" functionality to include a crafted PHP reverse shell file which bypasses the component's insufficient file validation mechanism.

**References:**
- https://github.com/CuteNews/cutenews-2.0
- https://github.com/CuteNews/cutenews-2.0/blob/master/core/modules/media.php

---

## CVE-2026-36468 — Reflected XSS via URL Parameter Key

| Field | Detail |
|---|---|
| **CWE** | Cross-Site Scripting (XSS) |
| **Component** | `index.php` |
| **Attack Type** | Remote |
| **Impact** | Escalation of Privileges, Information Disclosure, Content Manipulation/Defacement |

**Description:**
Cross-Site Scripting (XSS) in `index.php` allows remote unauthenticated attackers to supply an arbitrarily named URL parameter key, with part of its name containing any URL-encoded common XSS payload (such as `"><script>alert(1)</script>`).

**Attack Vector:**
Supplying an arbitrarily named URL parameter key whose name contains a URL-encoded XSS payload.

**References:**
- https://github.com/CuteNews/cutenews-2.0

---

## CVE-2026-36469 — Server-Side Request Forgery (SSRF)

| Field | Detail |
|---|---|
| **CWE** | CWE-918: Server-Side Request Forgery (SSRF) |
| **Component** | `core/modules/media.php` — `upload_from_inet` (Media Manager) |
| **Attack Type** | Remote |
| **Impact** | Information Disclosure; use of the application server as a proxy for further attacks |

**Description:**
CuteNews v.2.1.2 is vulnerable to Server-Side Request Forgery (SSRF) in the Media Manager's "Upload by URL" functionality.

**Attack Vector:**
The attacker must supply a URL pointing to an internal IP address (optionally with port and/or subdirectory).

**References:**
- https://github.com/CuteNews/cutenews-2.0
- https://github.com/CuteNews/cutenews-2.0/blob/master/core/modules/media.php

---

## CVE-2026-36470 — Reflected XSS via Referer Header

| Field | Detail |
|---|---|
| **CWE** | Cross-Site Scripting (XSS) |
| **Component** | `index.php` |
| **Attack Type** | Local |
| **Impact** | Escalation of Privileges, Information Disclosure |

**Description:**
The value of the `Referer` header is copied into the response HTML unmodified and unescaped during POST requests to `index.php`.

**Attack Vector:**
A crafted `Referer` header value is reflected directly into the HTML response without sanitization.

**References:**
- https://github.com/CuteNews/cutenews-2.0

---

## CVE-2026-36471 — Insecure Deserialization via `__post_data`

| Field | Detail |
|---|---|
| **CWE** | CWE-502: Deserialization of Untrusted Data (PHP Object Injection) |
| **Component** | `/core/core.php`, line 2754 (`unserialize()` call) |
| **Attack Type** | Remote |
| **Impact** | Content injection in rendered response HTML |

**Description:**
Deserialization of Untrusted Data of the `__post_data` parameter in `cn_parse_url()` allows a remote attacker to inject arbitrary values into internal request variables (including `__referer`) via a crafted base64-encoded serialized PHP payload submitted as a POST parameter.

**Attack Vector:**
Attacker submits a POST request to `/cutenews/index.php?lostpass=x` with a crafted serialized payload (e.g. generated via Python's `phpserialize` and `base64` modules) containing arbitrary values to be injected into any internal request variable.

**Chain Note:**
This vulnerability was chained with improper neutralization of the `__referer` value (CVE-2026-36472) and missing CSRF protections on form submission endpoints, resulting in exfiltration of session tokens and anti-CSRF signature keys from authenticated users. The chain was collectively recorded as: *Unauthenticated Session Hijacking (Insecure Deserialization via `__post_data` → self-XSS, delivered via CSRF)*.

**References:**
- https://github.com/CuteNews/cutenews-2.0
- https://github.com/CuteNews/cutenews-2.0/blob/master/core/core.php

---

## CVE-2026-36472 — Self-XSS via `__referer` Injection

| Field | Detail |
|---|---|
| **CWE** | Cross-Site Scripting (XSS) |
| **Component** | `/core/core.php`, line 2774 (`$_POST['__referer'] = $post_data['__referer'];`) |
| **Attack Type** | Remote |
| **Impact** | Self-XSS |

**Description:**
Improper neutralization of the `__referer` value allows a remote attacker to execute arbitrary JavaScript in the context of an authenticated user's session via a `javascript:` URI rendered as an unsanitized clickable link on the `msg_info` page.

**Attack Vector:**
An attacker who has already uncovered a deserialization vector (CVE-2026-36471) can target the `__referer` value with a `javascript:`-prefixed payload containing arbitrary JavaScript code (even multiline). It is then rendered as an unsanitized clickable link that executes in the browser's context upon interaction.

**Chain Note:**
This vulnerability was chained with Insecure Deserialization (CVE-2026-36471) and missing CSRF protections, resulting in exfiltration of session tokens and anti-CSRF signature keys. The chain was collectively recorded as: *Unauthenticated Session Hijacking (Insecure Deserialization via `__post_data` → self-XSS, delivered via CSRF)*.

**References:**
- https://github.com/CuteNews/cutenews-2.0
- https://github.com/CuteNews/cutenews-2.0/blob/master/core/core.php

---

## Common Test Environment

All vulnerabilities were identified via code review and confirmed using Firefox Browser + Burp Suite on Kali Linux. DNS was handled via Burp Suite (`target.local` for the target, `attacker.local` for the attacker).

**Deployment:**

```bash
sudo apt install apache2 php php-gd libapache2-mod-php
sudo apt install php php-gd php-mbstring php-xml php-curl libapache2-mod-php

git clone https://github.com/CuteNews/cutenews-2.0.git
sudo cp -r /home/kali/assessment/cutenews-2.0 /var/www/html/cutenews
wget https://github.com/CuteNews/cutenews-2.0/releases/download/2.0.1/cdata.zip
unzip cdata.zip -d cdata
sudo cp -r  cdata /var/www/html/cutenews
sudo mkdir /var/www/html/cutenews/uploads
sudo chown -R www-data:www-data /var/www/html/cutenews

sudo systemctl restart apache2
# Open http://localhost/cutenews/ in browser and complete the admin account setup.
```

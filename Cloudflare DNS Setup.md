# ✅ Cloudflare DNS Setup for Domain registrar (e.g. Namecheap, Godaddy, etc.)

> Replace 123.123.123.123 with your actual server IP address.

---

## 🌐 Main Domain & Subdomains

| Type | Name  | Content           | TTL  | Proxy Status |
|------|-------|--------------------|------|---------------|
| A    | @     | 123.123.123.123    | Auto | ✅ Proxied     |
| A    | www   | 123.123.123.123    | Auto | ✅ Proxied     |
| A    | app   | 123.123.123.123    | Auto | ✅ Proxied     |
| A    | api   | 123.123.123.123    | Auto | ✅ Proxied     |

🔁 *Alternative (Using CNAME):*

| Type  | Name | Content              | TTL  | Proxy Status |
|-------|------|-----------------------|------|---------------|
| CNAME | app  | example.com     | Auto | ✅ Proxied     |
| CNAME | api  | example.com     | Auto | ✅ Proxied     |

---

## 📧 Email Related (If using email)

| Type | Name  | Content                                         | TTL  | Proxy Status |
|------|-------|--------------------------------------------------|------|---------------|
| MX   | @     | mail.example.com (priority: 10)           | Auto | ❌ DNS Only   |
| A    | mail  | 123.123.123.123                                  | Auto | ❌ DNS Only   |
| TXT  | @     | v=spf1 a mx ip4:123.123.123.123 ~all             | Auto | ❌ DNS Only   |
| TXT  | _dmarc | v=DMARC1; p=none; rua=mailto:you@example.com   | Auto | ❌ DNS Only   |
| TXT  | default._domainkey | (Your DKIM public key here)         | Auto | ❌ DNS Only   |

ℹ *Never proxy email-related records (MX, mail A, SPF, DKIM, etc.)*

---

## 🔐 SSL/TLS Settings (Cloudflare Dashboard)

Go to: SSL/TLS → then:

- ✅ SSL Mode: Full (strict)
- ✅ Always Use HTTPS: ON
- ✅ Automatic HTTPS Rewrites: ON
- ✅ (Optional) HSTS: Enable if supported by server

---

## ✅ Testing URLs

After setup:

- https://example.com
- https://www.example.com
- https://app.example.com
- https://api.example.com

---

## ⚠ Notes

- Make sure to update your domain's *nameservers* to Cloudflare's ones from your domain registrar (e.g. Namecheap, Godaddy, etc.).
- SSL might take a few minutes to activate after DNS is working.

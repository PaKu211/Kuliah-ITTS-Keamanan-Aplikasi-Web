# Week 3 Framework Security Checklist

## Header evidence
* **Baseline command:** `node app-baseline.js` dan `curl -i http://localhost:3000/`
* **Baseline missing headers:** `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy` (Status: 200, semuanya HILANG)
* **Secure command:** `node app-secure.js` dan `curl -i http://localhost:3000/`
* **Secure headers present:** `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`

## Rendering / escaping evidence
* **Payload used:** `<script>alert(1)</script>`
* **Baseline result:** Payload dieksekusi dan memicu popup alert (Reflected XSS / Injeksi HTML).
* **Secure result:** Payload berhasil di-escape dengan aman dan dirender sebagai teks literal (`Search: &lt;script&gt;alert(1)&lt;/script&gt;`).

## SAST manual review
| File | Source | Sink | Control found | Control missing | Recommendation |
| --- | --- | --- | --- | --- | --- |
| `app-baseline.js` | `url.searchParams.get("q")` | `res.end(page(... q ...))` | Tidak ada | Output escaping, Secure headers, CSP | Tambahkan `escapeHtml()`, auto-escaping pada template, dan middleware secure headers |
| `app-secure.js` | `url.searchParams.get("q")` | `res.end(page(... q ...))` | Output escaping, Secure headers, CSP | Tidak ada | Pertahankan middleware dan lakukan tinjauan dependensi secara berkala |

## Dependency advisory triage
| Package | Installed version | Severity | Impact | Fix | Priority | Evidence/source |
| --- | --- | --- | --- | --- | --- | --- |
| `lodash` | `4.17.11` | Tinggi | Prototype Pollution / ReDoS | Perbarui ke >= 4.17.21 | Tinggi | `npm audit` / `advisory-sample.json` |

## Reflection
1. **What did the framework/middleware help automate?** Output escaping, konfigurasi secure headers, dan dasar-dasar keamanan routing/templating.
2. **What still needs developer judgment?** Validasi logika bisnis, kontrol akses yang tepat (otorisasi), dan konfigurasi kebijakan keamanan yang benar.
3. **Which control would you implement first in a real production app, and why?** Output escaping dan Content-Security-Policy (CSP), karena keduanya secara langsung memitigasi serangan injeksi (XSS) yang paling umum dan merusak.
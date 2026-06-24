# MTA-STS policy for paramos.com

This repository hosts the [MTA-STS](https://datatracker.ietf.org/doc/html/rfc8461) policy file for the **paramos.com** email domain. It is published via GitHub Pages and served at:

**https://mta-sts.paramos.com/.well-known/mta-sts.txt**

MTA-STS instructs external mail servers to deliver mail to paramos.com only over a valid, encrypted (TLS) connection, and not to fall back to plaintext if encryption fails. paramos.com receives mail via Google Workspace, so the MX hosts in the policy are Google's — confirmed from the Google Admin console.

**Current mode: `testing`** (set 2026-06-24). In testing mode the policy is published but not enforced: senders report TLS problems but still deliver normally. See _Switching to enforce_ below.

## What's in this repo

| File | Purpose |
| --- | --- |
| `.well-known/mta-sts.txt` | The policy itself — mode plus the list of valid MX hosts. The only file senders actually read. |
| `.nojekyll` | Stops GitHub Pages (Jekyll) ignoring the `.well-known` folder; dot-folders are skipped by default, so without this the policy 404s. |
| `CNAME` | Tells GitHub Pages to serve this repo at `mta-sts.paramos.com`. |
| `.gitattributes` | Keeps the policy file's CRLF line endings intact (`-text`) instead of normalising to LF. |

## The full picture (not all of it is in this repo)

This repo only holds the policy *file*. The rest of the system lives in DNS and Google Workspace:

- **GoDaddy DNS** (paramos.com zone):
  - `mta-sts` — CNAME → `<github-org>.github.io` (points the hostname at GitHub Pages)
  - `_mta-sts` — TXT `v=STSv1;id=...` (announces a policy exists; the `id` is a version stamp)
  - `_smtp._tls` — TXT `v=TLSRPTv1; rua=mailto:smtp-tls-reports@paramos.com` (TLS-RPT: where senders email failure reports)
- **Google Workspace**: the `smtp-tls-reports@paramos.com` group collects the TLS-RPT reports (set to accept external senders).

## Changing the policy

Whenever you edit `.well-known/mta-sts.txt`, you must **also bump the `id`** in the `_mta-sts` TXT record at GoDaddy — otherwise senders keep using their cached copy of the old policy and your change is ignored. Use a date-based id, e.g. `id=20260624`.

Order matters, to avoid disrupting mail:

1. Edit and commit the file here; wait until the new version is reliably live at the URL (check from more than one network).
2. **Then** update the `id` in the `_mta-sts` DNS record.

### Line endings — the common silent failure

The policy file must use **CRLF (`\r\n`) endings on every line, including the last**, with no trailing blank line or spaces. LF-only files or a stray blank line can cause senders and validators to reject the policy. `.gitattributes` preserves the bytes; if you regenerate the file, verify with:

```
cat -A .well-known/mta-sts.txt
```

Every line should end in `^M$`, with no empty line after `max_age`.

## Switching to enforce

After ~2 weeks of clean TLS-RPT reports in the group:

1. In `.well-known/mta-sts.txt`, set `mode: enforce` and `max_age: 1209600`. Commit.
2. Confirm the updated file is consistently live at the URL.
3. **Then** bump the `id` in the `_mta-sts` TXT record at GoDaddy.

In `enforce` mode, senders that support MTA-STS will **refuse to deliver** to paramos.com if the connection isn't properly encrypted and verified. The 2-week testing window exists to catch any problem before this point.

## Verifying

- Load the policy URL — expect a padlock and the exact policy text.
- `_mta-sts` and `_smtp._tls` TXT records resolve in DNS.
- Google Admin → Gmail → Compliance → "Validate your MTA-STS configuration" shows all rows configured.

## ISMS reference

Supports ISO/IEC 27001:2022 Annex A controls **A.8.20** (networks security) and **A.8.24** (use of cryptography) for PARAMOS Technologies. Owner: ISMS Owner / ISO Lead. Changes to this repo and the linked DNS records form part of the ISMS change record.

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

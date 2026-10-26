---
name: aliyun-http01-public-interception
description: |
  Diagnose Aliyun ECS Certbot HTTP-01 failures when an internal challenge probe
  returns 200 but public requests return 403, Server Beaver, or a Non-compliance
  ICP Filing page. Separate public interception from origin permissions.
---

# HTTP challenge intercepted outside the origin

Compare the **same owned probe URL** internally and publicly, including response
headers/body. An internal 200 and a public `Server: Beaver` / `Non-compliance ICP
Filing` response do not support changing origin file permissions. Check Certbot's
actual webroot log before guessing a path mismatch. Preserve both observations.

For an authorized domain, DNS-01 can establish certificate ownership without an
HTTP challenge. Reuse the existing CA account and domain-scoped DNS API flow;
keep keys on the server. Add only the current `_acme-challenge.<subdomain>` TXT,
wait for public visibility, release the auth hook, verify issuance, then delete
only the exact owned record after checking its ID/name/type/value. Avoid changing
global DNS resolvers, other hosts, credentials or IAM to repair this symptom.

Verify public HTTPS separately with normal TLS and actual content hashes/UI.
DNS-01 success does not prove HTTPS reachability or resolve a filing restriction.
After an nginx reload, allow bounded normal-TLS readiness retries; do not use
`curl -k` or `ignoreHTTPSErrors` to turn an early certificate failure into a pass.
Document the actual renewal mechanism; a manual hook needing an external DNS
driver is not evidence of unattended renewal.

Verified 2026-10-20 on the existing Qingdao ECS: identical probe internal200 /
public403 with the above headers/body; actual Certbot write path and 0644 mode
confirmed. DNS-01 issued the certificate, the owned TXT was removed, and public
HTTPS passed five build hashes, 45 UI checks and a complete game playthrough.
This validates the exercised route, not a universal diagnosis for every 403.

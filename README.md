# Tarang Saree — Vercel JSON-backed lucky draw

## Important

No Meta API is used. **Instagram follows and messages are not verified.** The customer checks a self-confirmation box. Only use this system if the event's published eligibility rules permit self-attestation.

## Components
- `/` : customer registration, random DM token, Instagram redirect, self-confirmation, integer coupon 1..600.
- `/admin.html` : password login, customer records, pending/issued counts, CSV export.
- `/api/*` : Vercel serverless API with GitHub private JSON storage.

## SETUP — required before functioning

1. Create a **separate PRIVATE GitHub repository** for customer data (e.g. `sarthak3223/tarang-private-data`). Never use the public site repository for personal information.
2. Add `registrations.json` to the private repo default branch containing exactly: `{"version":1,"records":[],"nextCoupon":1}`.
3. Create a **fine-grained GitHub Personal Access Token** with Contents: Read and write access limited to the PRIVATE data repository only. Never put this token in code, HTML or committed files.
4. In Vercel Project > Settings > Environment Variables, configure:
   - `DATA_GITHUB_REPO` = `sarthak3223/tarang-private-data` (actual private repo)
   - `DATA_GITHUB_TOKEN` = private fine-grained token
   - `ADMIN_PASSWORD` = random strong unique password
   - `SESSION_SECRET` = long random hex string (at least 32 random bytes)
   - optionally `MAX_COUPONS` = `600`
5. Deploy this folder's files (`index.html`, `admin.html`, `api/index.js`, `vercel.json`) to the existing public site repo, then redeploy on Vercel.
6. Test with sample registrations; inspect data in PRIVATE repository and dashboard. Reset JSON and clear localStorage on test devices **only before public launch**.

## Why JSON uses private GitHub storage

Vercel serverless functions cannot modify deployed JSON files persistently. GitHub's Contents API provides persistent JSON writes with a SHA compare-and-swap condition. This code retries conflicting writes to prevent duplicate coupon numbers during concurrency. It is slower and has GitHub rate limits; for large bursts or a higher-stakes draw, migrate to a transactional database. A failed write does not issue a coupon.

## Safety and operational considerations

- Treat name, phone and DM token as private personal information. Access the private data repository and admin dashboard only through authorized accounts.
- Do not enable public registration until secrets are configured. Never publish real names and numbers in the site's public GitHub repo.
- Different phone numbers can still belong to one person. No OTP is included; a phone number typed by a customer is not independently verified.
- Return visitors use a receipt saved in localStorage; reusing their device returns the original coupon. Losing their device/clearing browser storage means the owner must resolve their record via the admin portal (no anonymous phone-number lookup to avoid leaking customer data).
- Public coupon numbers can show an order of self-confirmation, not proof of Instagram following.
- Delete customer records when no longer required by your published privacy policy.
- If your Instagram promotion is subject to platform or local restrictions, revise eligibility accordingly.

## Diagnostics

- `503 Server configuration is incomplete`: missing Vercel environment variable.
- `503 Private data repository is missing registrations.json`: wrong repo/path or read permission.
- `503 Storage read/write failed (401/403)`: wrong token/permissions; ensure private repo token access.
- `503 System busy`: retries exhausted due to competing registrations; retry.
- API endpoints and secrets must stay server-side.

# Compliance Remediation Report — accessmsmc-website

**Branch:** `compliance-remediation` (not merged)
**Date:** 2026-08-12
**Scope:** Retire the EmailJS + Google Apps Script patient-intake pipeline in favor of the
BAA-covered Microsoft 365 Form, purge the associated secrets from git history (prepared,
not executed), correct overstated privacy language, scaffold the missing legal pages, reduce
third-party tracking on `contact.html`, and take a WCAG 2.1 AA pass across the site.

A prior analysis document, [`docs/WEBSITE_REVIEW.md`](docs/WEBSITE_REVIEW.md), already exists
in this repo and covers a broader set of UX/compliance/SEO findings (crisis-info placement,
insurance-messaging consistency, Good Faith Estimate/No Surprises Act, dual `<h1>` per page,
decorative emoji without `aria-hidden`, etc.). This report does not repeat that document —
items from it that fall inside this task's scope are called out below; the rest remain open
and tracked there.

---

## 1. What was found (verified against the working tree and git history)

| # | Finding | Status |
|---|---|---|
| 1 | `config.js` tracked in git despite being in `.gitignore`; contains live-looking EmailJS public key, service ID, and two template IDs | Confirmed. 1 commit introduced it (`a3f7a4d "Create config.js"`); still present in every commit since and in the working tree until this branch. |
| 2 | `appointment.html` loaded the EmailJS SDK, called `emailjs.init()`, and sent the full intake payload (name, DOB, phone, email, primary concern, current medications, previous treatments, additional info) to EmailJS on submit | Confirmed. |
| 3 | Same submit handler also fired the payload at a hardcoded, unauthenticated Google Apps Script URL (`https://script.google.com/macros/s/AKfycbz…2aEk/exec`) with `mode: 'no-cors'`, logging to a consumer Google Sheet/Calendar per the code's own comment | Confirmed. No auth on that endpoint — anyone who viewed page source could POST arbitrary data to it. |
| 4 | Old form text claimed data is "strictly confidential and protected under HIPAA regulations" while transmitting PHI through non-BAA channels | Confirmed. No other page on the site made a similar blanket HIPAA guarantee. |
| 5 | No Privacy Policy, Notice of Privacy Practices, Terms of Use, or Accessibility Statement anywhere on the site | Confirmed (also flagged as Critical #4 in `docs/WEBSITE_REVIEW.md`). |
| 6 | `contact.html` auto-loads a Google Maps iframe on page load (plus a separate Maps "Get Directions" link) | Confirmed — third-party request fires for every visitor before any consent/interaction. |
| 7 | `about-old.html` / other stale duplicate files | **Not found.** No file matching `*old*`, `*copy*`, `*backup*`, `*bak*` exists in the working tree. Either already removed before this pass, or the original finding referred to a file that was never committed. No action taken. |

---

## 2. Code changes made on this branch

- **`appointment.html`** replaced outright: no more EmailJS SDK, `emailjs.init`, `config.js` reference, or Google Apps Script call. Intake is now a Microsoft Forms iframe embed (BAA-covered Microsoft 365), with a same-tab fallback link and phone number. Nav/footer brought in line with the rest of the site (the previously-drafted replacement was missing the Pricing/Pay nav items and had no footer). The overstated "protected under HIPAA regulations" line is gone, replaced with measured, `[[ATTORNEY REVIEW]]`-flagged copy pointing to the new Privacy Policy / NPP pages.
- **`config.js` deleted** from the working tree. (Still in git history — see §3.)
- **Verification:** `grep -rniE "emailjs|script\.google|gapi|sheets\.google|calendar\.google" --include=*.html --include=*.js .` now returns only the explanatory HTML comment in `appointment.html` noting the pipeline was removed — no live code references remain.
- **`contact.html`:** the Google Maps iframe no longer loads automatically. It's replaced with a placeholder that discloses the privacy trade-off ("loading this will share your IP address and browsing data with Google") behind a **"Load Interactive Map"** button, plus a same-purpose **"Get Directions"** link that opens Google Maps in a new tab without embedding anything. No third-party request fires until the visitor opts in.
- **Four new legal pages** created, matching the site's existing header/nav/hero/breadcrumb structure and `css/styles.css`: `privacy-policy.html`, `notice-of-privacy-practices.html`, `terms.html`, `accessibility.html`. All substantive legal content is wrapped in visibly-styled `[[PLACEHOLDER — ATTORNEY REVIEW REQUIRED]]` blocks — headings and section structure are drafted (CCPA/CPRA sections for the Privacy Policy; the standard 45 CFR § 164.520 elements for the NPP, including a flagged 42 CFR Part 2 section — see §4), but no legal claims are asserted as final. `notice-of-privacy-practices.html` and `privacy-policy.html` carry `<meta name="robots" content="noindex">` since they're not yet real legal documents.
- **Footer legal links** (`Privacy Policy · Notice of Privacy Practices · Terms of Use · Accessibility`) added to the footer of all 11 pre-existing pages plus the rebuilt `appointment.html`, so the links the new `appointment.html` privacy notice already pointed to now resolve instead of 404ing.
- **`css/styles.css`:** added `.footer-legal-links a` styling (visible against the dark footer background) and a sitewide `:focus-visible` outline rule for links/buttons/`[role="button"]`/`[tabindex]` elements that previously had no visible keyboard-focus indicator outside of form fields and the nav dropdown.
- **`README.md`:** removed the "EmailJS Integration" section and other EmailJS/config.js references (troubleshooting section, file-tree listing), replaced with a description of the Microsoft 365 Form pipeline. Softened the unqualified "WCAG accessibility compliant" claim to state WCAG 2.1 AA is a *target*, list the concrete measures in place, and point to `accessibility.html` — see §5.

### Accessibility (WCAG 2.1 AA pass)
- Checked all `<img>` tags site-wide: every image already has descriptive `alt` text (an initial single-line grep suggested 4 images were missing `alt`, but they're multi-line `<img>` tags with `alt` on a following line — false positive, no fix needed).
- Checked all form fields on `appointment.html`: every input already has an associated `<label for>`.
- Checked the nav dropdown (`js/nav.js`): already keyboard-operable (Enter/Space to toggle, Escape to close, `aria-expanded` kept in sync).
- Added the sitewide `:focus-visible` outline rule described above (previously, links, footer links, and CTA buttons had no visible focus indicator for keyboard users).
- **Flagged, not fixed (already tracked in `docs/WEBSITE_REVIEW.md`):**
  - Every page has two `<h1>` elements (header logo + page hero heading) — a real heading-order violation, but fixing it means changing shared header markup on every page and the corresponding `.logo h1` CSS selector (duplicated ~5× in `styles.css`), which is a bigger structural change than this pass's other fixes. Left as-is pending a decision on the replacement markup/branding treatment.
  - Decorative emoji (🚨 📞 📍 👨‍⚕️ ⚠️ etc.) used directly inside headings/labels without `aria-hidden="true"`, so screen readers announce them literally. Cosmetic/noisy rather than blocking, and touches dozens of instances across every page — left for a dedicated pass.
- No color-contrast failures were manually spotted in the areas touched by this change (new legal pages, footer links, focus rings), but a full automated pass (axe/Lighthouse) has not been run — the README claim has been worded to reflect that.

---

## 3. Git history purge — PREPARED ONLY, NOT EXECUTED

The EmailJS credentials and the Google Apps Script URL are baked into blobs across git
history (`config.js`, 1 commit; `appointment.html`, 14 commits). Deleting the files on this
branch does **not** remove them from history — anyone with `git log -p` or a clone of the repo
before this branch can still read them.

**This section is documentation only. Do not run these commands without the repo owner's
explicit go-ahead — they rewrite history and require a force-push.**

### Secrets involved (redacted to last 4 characters)
| Value | Redacted |
|---|---|
| `EMAILJS_PUBLIC_KEY` | `…EuV0` |
| `EMAILJS_SERVICE_ID` | `…8bsc` |
| `EMAILJS_NOTIFICATION_TEMPLATE` | `…mokf` |
| `EMAILJS_CONFIRMATION_TEMPLATE` | `…d2ap` |
| Google Apps Script deployment ID (in the `/exec` URL) | `…2aEk` |

The maintainer already has the full values (from the working copy of `config.js` prior to
this branch, or from `git show a3f7a4d:config.js`) — they are intentionally not repeated in
full anywhere in this report or in any file this branch adds to the repo, so that purging
history doesn't get undone by a new commit that re-introduces the same strings.

### Recommended tool: `git filter-repo`
BFG is a reasonable alternative; commands for both are below. **Do this on a fresh clone**,
not this working copy, so a mistake can be discarded by re-cloning instead of risking the
only copy of the repo.

```bash
# 1. Fresh clone, separate from your working copy
git clone https://github.com/accessmsmc/accessmsmc-website.git accessmsmc-purge
cd accessmsmc-purge

# 2. Install git-filter-repo if not already available
#    pip install git-filter-repo   (or: brew install git-filter-repo)

# 3. Create a LOCAL, UNTRACKED replacements file (do not commit this file).
#    Fill in the real values on the left; keep them out of any commit.
cat > replacements.txt <<'EOF'
XP_IQmsT9NSAeEuV0==>***REMOVED***
service_v178bsc==>***REMOVED***
template_mpvmokf==>***REMOVED***
template_et5d2ap==>***REMOVED***
AKfycbz_JhcONiG7-8AeAbU6T1HoGpMF-wFljEtXyfJywxMLIY0sNm40grbmV7KIOqYQ2aEk==>***REMOVED***
EOF

# 4. Remove config.js from every commit AND scrub the literal strings above
#    from every blob (covers appointment.html across all 14 historical commits).
git filter-repo --path config.js --invert-paths --replace-text replacements.txt

# 5. Delete the local replacements file — it's no longer needed and
#    should never be committed.
rm replacements.txt

# 6. Re-add the remote (filter-repo removes it as a safety measure) and force-push.
git remote add origin https://github.com/accessmsmc/accessmsmc-website.git
git push origin --force --all
git push origin --force --tags
```

### BFG alternative
```bash
# In a fresh --mirror clone:
git clone --mirror https://github.com/accessmsmc/accessmsmc-website.git accessmsmc-purge.git
cd accessmsmc-purge.git

# replacements.txt as above (local, untracked, delete after use)
bfg --delete-files config.js accessmsmc-purge.git
bfg --replace-text replacements.txt accessmsmc-purge.git

git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force
```

### After the rewrite (required regardless of tool)
- **Every collaborator must re-clone** the repository, or discard their local clone and
  `git fetch` + hard-reset to the new history — `git pull` alone will produce a confusing
  merge of old and new history. Commit hashes for every commit from `a3f7a4d` onward will
  change.
- **GitHub Pages / `deploy.yml`:** confirm the next deploy runs cleanly off the rewritten
  `main` after the force-push.
- Any open PRs or branches based on pre-rewrite history will need to be re-created or rebased
  onto the new history.
- This purge only removes the secrets from *retrievable git history*. It does **not**
  invalidate the credentials themselves — see the rotation items in §4, which matter
  regardless of whether or when the history purge is run.

---

## 4. Human / Vendor / Legal action list

These cannot be completed by editing the repository and require the practice's own action:

1. **Rotate or delete the EmailJS account/service/templates.** The public key and service ID
   have been visible in a public(ish) git history and possibly in page source for as long as
   the site has been live; purging history does not retroactively revoke them. Since the
   EmailJS pipeline is being fully retired, the cleanest fix is deleting the EmailJS account
   (or at minimum the service and both templates) rather than just rotating keys.
2. **Disable/delete the Google Apps Script deployment** at the `/exec` URL referenced above,
   and the underlying Google Sheet/Calendar it was writing to. That endpoint has no
   authentication and will keep accepting POST requests indefinitely — including from anyone
   who saved the URL from old page source or from a pre-purge git clone — until it's disabled
   on the Google side. Also confirm whether that Sheet/Calendar contains real patient PHI
   collected over the pipeline's lifetime, and handle it per the practice's retention/deletion
   policy.
3. **Confirm the Microsoft 365 Business Premium HIPAA BAA is fully executed** (not just
   available) and covers Microsoft Forms specifically, not only Exchange/SharePoint/Teams —
   Forms BAA coverage should be confirmed directly with Microsoft or the practice's M365
   licensing documentation, not assumed.
4. **Execute the git-history purge in §3** once the maintainer has reviewed and approved it,
   ideally right after item 1–2 (there's no urgency to sequence the history purge before the
   vendor-side teardown, since the live credentials are the actual exposure).
5. **Attorney review of all four new legal pages** (`privacy-policy.html`,
   `notice-of-privacy-practices.html`, `terms.html`, `accessibility.html`) before they go live
   — every `[[PLACEHOLDER — ATTORNEY REVIEW REQUIRED]]` block needs real, counsel-approved
   content. In particular:
   - **CCPA/CPRA applicability** for the Privacy Policy needs to be confirmed by counsel
     (thresholds, whether the site "sells or shares" data via the Google Maps embed).
   - **NPP** needs the practice's actual Privacy Officer contact info and any policies not
     capturable from the repo alone.
6. **42 CFR Part 2 consent handling for SUD-related intake:** `appointment.html`'s "Primary
   Concern" dropdown includes "Substance Use Disorder" as an option. If this practice meets
   the definition of a "Part 2 program" (42 CFR § 2.11), records for those patients require
   Part 2's stricter consent and re-disclosure rules on top of standard HIPAA — this is
   flagged as a placeholder in `notice-of-privacy-practices.html` §9 but needs a definitive
   applicability determination and, if applicable, a separate Part 2 consent flow (the current
   Microsoft Form does not distinguish Part 2 vs. non-Part 2 intake).
7. **Confirm the Microsoft Forms embed URL** in `appointment.html` is the practice's intended
   long-term form (not a temporary/test instance), and that form responses route to a location
   with appropriate access controls (limited to authorized staff).
8. **Full accessibility audit** (automated — axe or Lighthouse — plus manual screen-reader
   testing) before publishing any conformance claim stronger than the "target" language now in
   `accessibility.html` and `README.md`.

---

## 5. README WCAG claim

The prior README stated flatly: *"WCAG accessibility compliant."* No audit backing that claim
exists in the repo. It's been reworded to state WCAG 2.1 AA is a **target**, list the concrete
measures actually in place, and point to `accessibility.html`, which itself avoids asserting
full conformance pending a real audit (see §2).

---

## 6. Out of scope / deferred to `docs/WEBSITE_REVIEW.md`

The existing review document identifies several other real issues not part of this task's
brief — crisis-info placement inconsistency, insurance-messaging contradictions, missing Good
Faith Estimate / No Surprises Act language, dual `<h1>` per page, decorative emoji without
`aria-hidden`, missing structured data, and more. None of those were touched on this branch.
They remain open in that document.

---

## 7. Summary of files changed on this branch

- Modified: `appointment.html`, `contact.html`, `css/styles.css`, `README.md`,
  `index.html`, `about.html`, `ana-berezovskaya.html`, `oxana-dickey.html`,
  `conditions.html`, `treatments.html`, `team.html`, `pay.html`, `pricing.html`
- Added: `privacy-policy.html`, `notice-of-privacy-practices.html`, `terms.html`,
  `accessibility.html`, `REMEDIATION_REPORT.md`
- Deleted: `config.js` (from the working tree only — see §3 for the history purge)

**Not merged. No force-push performed.**
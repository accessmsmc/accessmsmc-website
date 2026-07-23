# Website Review — Access Multi-Specialty Medical Clinic

**Scope:** All 11 pages (`index.html`, `treatments.html`, `conditions.html`, `about.html`, `ana-berezovskaya.html`, `oxana-dickey.html`, `team.html`, `appointment.html`, `pricing.html`, `pay.html`, `contact.html`), `css/styles.css`, `js/nav.js`, `config.js`, and the GitHub Pages deploy workflow. This is an analysis document only — no site code was changed to produce it.

**How to read this report:** each finding states what it is, why it matters *for this practice*, and the concrete fix. Findings are grouped into three tiers (Critical, High Value, Polish) and roughly ordered by effort-to-impact within each tier. A "Questions for you" section lists items I could not resolve from the repo alone. A suggested implementation sequence closes the report.

---

## Critical

### 1. Crisis/safety information exists on only 2 of 11 pages, and is incomplete even there
**What:** A full crisis block (988 + 911 buttons) appears on `contact.html` (`.emergency-section`). A partial version (988 only) appears on `conditions.html`, but **only inside the "Treatment-Resistant Depression" section** — the "Bipolar Disorder" section (which explicitly lists "suicidal thoughts or behaviors" as a symptom) and "Schizophrenia & Psychotic Disorders" section have no crisis callout at all. `index.html`, `treatments.html`, `about.html`, `ana-berezovskaya.html`, `oxana-dickey.html`, `team.html`, `pricing.html`, and `pay.html` have **none**. Nowhere on the site is there an explicit statement that the practice does not provide 24/7 or emergency/crisis care.

**Why it matters here:** This is a psychiatric practice whose own condition pages describe suicidal ideation as a symptom of the conditions it treats (bipolar depressive episodes, treatment-resistant depression). Someone in acute distress is a foreseeable visitor to *any* page on this site — not just Contact — and the current layout means whether they see a lifeline number is essentially random, based on which page they landed on from a search engine.

**Fix:**
- Add a compact, consistent crisis strip to the **shared footer markup** (all 11 pages, since there is no shared partial — see Polish #1): "In crisis? Call or text **988** (Suicide & Crisis Lifeline) or **911**. This practice does not provide 24-hour or emergency crisis care." with `tel:988` / `tel:911` links matching the existing `.emergency-cta` pattern.
- On `contact.html` and `appointment.html` specifically, keep/add a more prominent version above the fold (contact.html already has a good one — mirror it, with the added "we do not provide emergency care" sentence, onto `appointment.html`, which currently has none at all despite being the page someone in distress is most likely to reach via the "Urgent Consultation" radio option in the form).
- Add the same `.warning-signs` box already used in the depression section of `conditions.html` to the Bipolar and Schizophrenia sections, or — better — place one crisis callout once per page (e.g., directly under the page hero) rather than duplicating it per condition, so it can't be missed depending on which section someone reads.

---

### 2. The site contradicts itself about whether the clinic is in-network or self-pay
**What:** `contact.html`'s "Insurance & Payment" section lists Medicare, Medicaid, BCBS, Aetna, Anthem, Cigna, and United Healthcare as "Accepted Insurance Plans" and states TMS/ketamine/ECT/genetic testing are covered "by most plans." `treatments.html` repeats this per-treatment ("Covered by most major insurance plans," "We handle insurance verification" for TMS; "Typically covered by insurance" for Genesight). Meanwhile `pricing.html`'s opening notice frames self-pay/out-of-network explicitly and narrowly: *"The services listed on this page are not covered by medical insurance... These include administrative services, legal documentation, disability forms, and specialized evaluations"* — implying the core clinical treatments (evaluations, medication management, TMS, ketamine, ECT) are billed differently (presumably in-network or with a different OON reimbursement path) than the administrative/forensic services `pricing.html` actually itemizes.

**Why it matters here:** You described this as "a private, out-of-network psychiatric practice," but `contact.html` and `treatments.html` read like an in-network practice for the clinical services, and `pricing.html` reads like an OON/self-pay practice for administrative services only. A prospective patient calling to ask "do you take my insurance?" could get an answer that contradicts what they just read, which is exactly the kind of friction/mistrust that drives phone-call volume and abandoned inquiries. This also has real billing/compliance stakes if the OON framing is the actual reality and `contact.html`/`treatments.html` are simply stale.

**Fix:** Resolve which model is actually true (see Question 1 below), then rewrite `contact.html`'s insurance section and the four "Insurance Coverage" sidebars on `treatments.html` to state the real billing model consistently with `pricing.html` and `pay.html`. If the clinical practice is genuinely OON, replace "Accepted Insurance Plans" language with OON-appropriate language (e.g., "we do not participate in insurance networks; we provide superbills for out-of-network reimbursement" — see Finding 5).

---

### 3. The appointment form collects detailed PHI and sends it through channels that likely aren't BAA-covered
**What:** `appointment.html`'s form collects: date of birth, a "Primary Concern" diagnosis-category dropdown, free-text "Current Psychiatric Medications," free-text "Previous Treatments," and free-text "Additional Information" — all clearly PHI. On submit, this payload is sent two ways in the inline `<script>`:
1. Via **EmailJS** (a third-party client-side email API) to two templates (`EMAILJS_NOTIFICATION_TEMPLATE`, `EMAILJS_CONFIRMATION_TEMPLATE`), using credentials from `config.js`.
2. Via a **hardcoded Google Apps Script web-app URL** (`https://script.google.com/macros/s/AKfycbz.../exec`), fire-and-forget (`mode: 'no-cors'`), which per the code comment logs to a Google Sheet/Calendar.

The form itself tells patients: *"All information submitted is strictly confidential and protected under HIPAA regulations."*

**Why it matters here:** EmailJS's standard/consumer plans do not offer a signed Business Associate Agreement, and email in general is not an inherently secure PHI transmission channel. Sending free-text medication and treatment-history details through it, while telling patients it's "protected under HIPAA regulations," is a specific, concrete compliance and false-reassurance risk for a psychiatric practice. Separately, **the Google Apps Script URL is visible to anyone who views page source** — it has no authentication, so any third party could copy it and POST arbitrary data to it, potentially polluting the clinic's intake sheet with spam or garbage with no server-side validation to stop them. Whether the underlying Google account is a Workspace account with a signed BAA (which *can* legitimately cover Sheets/Apps Script) is unknown from the code.

**Fix:**
- Do not treat this as a design nice-to-have — get a definitive answer on BAA status for both EmailJS and the Google account (Question 2/3 below) before this form processes another real patient's medication history.
- Short-term, lowest-effort risk reduction: remove or gate the free-text clinical fields (medications, previous treatments, additional info) from the *public web form* — collect only scheduling/contact info online, and gather clinical history over the phone or at the visit instead. This alone removes most of the PHI-in-transit exposure without needing new infrastructure.
- Replace the public, unauthenticated Apps Script endpoint with either a BAA-covered form backend, or at minimum add a shared-secret/token check inside the Apps Script so it isn't a public write-anything endpoint.
- Soften or remove the "protected under HIPAA regulations" claim until the transmission path actually supports it.

---

### 4. No Privacy Policy or HIPAA Notice of Privacy Practices anywhere on the site
**What:** No page, footer link, or PDF matching "privacy policy," "Notice of Privacy Practices," or "HIPAA" exists anywhere in the repo.

**Why it matters here:** Combined with Finding 3, this practice is actively collecting PHI through a public web form with no published privacy notice explaining what's collected, how it's used, or patients' rights — the baseline expectation for any HIPAA-covered entity with a web presence, and something patients increasingly check for before trusting a form with sensitive information.

**Fix:** Publish a Privacy Policy / NPP page (even a straightforward one summarizing the practice's existing paper NPP, if one already exists — see Question 7) and link it from the footer on every page and directly above the submit button on `appointment.html`, next to the existing privacy-notice text.

---

### 5. No Good Faith Estimate (No Surprises Act) information, and no superbill mention, despite an extensive self-pay fee schedule
**What:** `pricing.html` lists detailed self-pay pricing for numerous services, but neither "Good Faith Estimate," "No Surprises Act," nor "superbill" appears anywhere on the site.

**Why it matters here:** Since January 2022, providers must furnish uninsured/self-pay patients a Good Faith Estimate of expected charges under the federal No Surprises Act — this isn't optional messaging, it's a compliance requirement, and this practice's entire self-pay-heavy pricing page has no mention of it. Separately, if any portion of the clinical practice is genuinely out-of-network (per your framing), patients need superbills to seek their own reimbursement, and the absence of any mention of superbills leaves that path completely unexplained to a prospective self-pay patient trying to decide whether they can afford to come here.

**Fix:** Add a GFE section to `pricing.html` (and/or `pay.html`) stating that a Good Faith Estimate is available on request before scheduling for self-pay services, with instructions for requesting one. If OON reimbursement is part of the actual business model, add a short "Superbills" section explaining that one is provided on request and roughly what a patient does with it.

---

### 6. Patients are asked to email a photo of their insurance card
**What:** `appointment.html`'s insurance field includes: *"After submitting this form, please email a photo of your insurance card (front and back) to insurance@accessmultispecialty.com."*

**Why it matters here:** Same category of risk as Finding 3 — unencrypted email is not a HIPAA-appropriate channel for transmitting insurance card images (which contain member ID, group number, and other identifiers) unless the receiving mailbox/provider is specifically configured and covered for that. This is a small, self-contained fix once the mailbox's compliance status is confirmed.

**Fix:** Confirm whether `accessmultispecialty.com`'s email provider is configured for HIPAA-compliant transmission and covered by a BAA. If not, replace this instruction with a secure upload link, or instruct patients to bring the card to their visit instead of emailing it.

---

## High Value

### 7. `pay.html` is not linked from anywhere in `pricing.html`'s body content
**What:** `pay.html` is reachable from the global nav on every page, but a patient reading `pricing.html`'s fee schedule, FAQ, or terms has no in-context link to actually go pay — only the separate top-nav "Pay" item, which they may not connect to the fee schedule they're looking at.
**Why it matters:** This is exactly the kind of one-extra-click friction that turns "I understand the fee, let me pay it" into a phone call instead.
**Fix:** Add a direct CTA/link to `pay.html` at the point in `pricing.html` where fees and payment methods are discussed (e.g., near the "Payment Options" card in the insurance/payment section, and again near the FAQ's "What payment methods do you accept?" answer).

### 8. Three different payee names across billing content, with no explanation for two of them
**What:** Checks are payable to **"Michael Levinson, MD"** (`pay.html`), Zelle payments are processed by **"IRA Billing and Management, Inc"** (`pay.html`, which does proactively reassure patients this is correct), and the clinic's own billing/mailing entity is **"Access Multi-Specialty Medical Clinic, Inc."** (`contact.html`). `pay.html` already anticipates confusion for the Zelle name but not for the check payee name, which is a different legal name again.
**Why it matters:** A patient double-checking who they're paying (reasonable behavior before sending money) hits three names for what should be one relationship, with only one of the three explained.
**Fix:** Add one short clarifying line near the check instructions (mirroring the Zelle reassurance) explaining why checks go to Dr. Levinson personally rather than the clinic entity, if that's simply how the practice is structured.

### 9. Missing structured data (Schema.org)
**What:** No JSON-LD or microdata anywhere on the site — no `Physician`, `MedicalBusiness`, or `LocalBusiness` schema.
**Why it matters:** Structured data is what lets Google show rich results (address, phone, hours, specialty) directly in search for "psychiatrist near me"-type queries, and is a meaningfully different lever from on-page copy alone.
**Fix:** Add `Physician`/`MedicalOrganization` JSON-LD (name, address, phone, NPI, medical specialty, geo coordinates already present in meta tags) to `index.html` at minimum, ideally site-wide via the shared footer once one exists.

### 10. No `sitemap.xml` or `robots.txt`
**What:** Both confirmed absent from the repo root.
**Why it matters:** Trivial to add, meaningfully helps discoverability and crawl efficiency, and is close to a "why wouldn't you" fix for a site that otherwise cares about local SEO (geo meta tags, keyword-stuffed titles are already present).
**Fix:** Add both; reference the sitemap URL from `robots.txt`.

### 11. Every page has two `<h1>` elements
**What:** Confirmed on all 11 pages — the header logo ("Access Multi-Specialty Medical Clinic") is marked up as `<h1>`, and the page's own hero heading (e.g., "Contact Our Clinic") is a second `<h1>`.
**Why it matters:** One `<h1>` per page is both an accessibility and SEO best practice (screen reader users navigating by heading rely on a single clear top-level heading; search engines use it as a strong relevance signal). Right now every page's actual topic-specific heading has to share top billing with the site name.
**Fix:** Change the logo heading to a non-heading element (e.g., `<p class="logo-text">` or a styled `<div>`) sitewide, or demote it to `<h2>`, keeping the page's real content heading as the sole `<h1>`. Since nav is already duplicated across all 11 files, this touches the same 11 header blocks — worth doing in the same pass as any future nav-partial work.

### 12. Missing credentials for the associate providers
**What:** `about.html` (Dr. Levinson) prominently shows an NPI and CA license number, both in the page body and the footer. Neither `ana-berezovskaya.html` nor `oxana-dickey.html` shows an NPI or license number anywhere — only credentials/titles.
**Why it matters:** Prospective patients and referring parties sometimes verify licensure directly (CA BreEZe/license lookup), and the asymmetry between Dr. Levinson's page and the associates' pages reads as if the associates are less "official," which likely isn't the intent.
**Fix:** Add NPI and CA license numbers to both associate pages in the same format used on `about.html` — pending you providing the actual numbers (Question 6).

### 13. No telehealth policy stated anywhere
**What:** Zero mentions of "telehealth," "video visit," or similar on any page.
**Why it matters:** This is one of the first things a prospective psychiatric patient searches for post-2020, especially for medication management follow-ups. Its complete absence reads as "probably not offered" by default, which may be costing conversions if telehealth actually is available.
**Fix:** State the policy explicitly either way (Question 4), and if offered, add it to `treatments.html`/`appointment.html`/`conditions.html` follow-up-visit language.

### 14. No stated age range / population served
**What:** "Adults" appears only inside specific treatment-eligibility bullet lists (e.g., TMS candidacy); there's no general statement of who the practice sees (adults only? adolescents?).
**Why it matters:** A parent searching for adolescent psychiatric care in Burlingame has no way to tell from this site whether to even bother calling.
**Fix:** Add an explicit population statement (Question 5) to `about.html`/`conditions.html`/`index.html`.

### 15. No general "what to expect at your first visit" content
**What:** `treatments.html` and `conditions.html` each have per-treatment/per-condition "What to Expect" cards (good), but there's no overview of the *administrative* first-visit experience — what to bring, how long it takes, whether paperwork is sent in advance — before a patient commits to filling out the appointment form.
**Why it matters:** This is one of the highest-value trust-building content types for a first-time therapy/psychiatry patient, and the pieces (insurance card, ID, arrival time) are scattered rather than presented as a simple checklist.
**Fix:** Add a short "Your First Visit" block to `appointment.html` above the form, consolidating what's currently only surfaced in the post-submit success message.

### 16. Provider photo is very heavy; one is under-resolution
**What:** `assets/ML_photo_v2.jpg` is 2400×3000px / 1.49MB. `assets/ana.jpeg` is only 321×450px / 14.7KB. Both now load together on `team.html`'s provider grid in addition to their own bio pages.
**Why it matters:** Most "psychiatrist near Burlingame" searches happen on phones (per your own framing); a 1.5MB image meaningfully slows first paint on a cellular connection, and the low-resolution photo will look visibly soft once displayed at typical card/hero widths.
**Fix:** Compress/resize `ML_photo_v2.jpg` to roughly 800–1000px wide (well under 200KB) and source a higher-resolution replacement for `ana.jpeg`.

### 17. Render-blocking third-party script on the appointment page
**What:** `appointment.html` loads `https://cdn.jsdelivr.net/npm/@emailjs/browser@3/dist/email.min.js` synchronously in `<head>`, with no `defer`/`async`, immediately followed by an inline `emailjs.init()` call.
**Why it matters:** This is a render-blocking network request to a third party on the one page where a patient is trying to complete an action — any slowness in that CDN delays the whole page's first paint.
**Fix:** Add `defer` to the script tag and move the `emailjs.init()` call into a listener that fires after it loads (e.g., the script's own `onload`, or a `DOMContentLoaded` handler).

### 18. `config.js` with live-looking EmailJS credentials is committed to the repo, despite being listed in `.gitignore`
**What:** `.gitignore` lists `config.js` (implying the intent is for `deploy.yml` to generate it fresh from GitHub Secrets on each deploy), but a `config.js` with real-looking `EMAILJS_PUBLIC_KEY`, `EMAILJS_SERVICE_ID`, and two template IDs is already tracked in git history.
**Why it matters:** EmailJS public keys are designed to be client-visible, so this isn't a "secret leaked" issue by itself — but a publicly-known service ID + template ID pair, without domain restriction configured on the EmailJS side, could let someone outside the site trigger the same notification/confirmation email templates from elsewhere.
**Fix:** Confirm in the EmailJS dashboard that allowed domains are restricted to `accessmultispecialty.com` (and its GitHub Pages equivalent, if used for testing). If these are genuinely production credentials, consider `git rm --cached config.js` going forward so only the CI-generated version ever ships, matching the apparent original intent.

---

## Polish

### 19. Footer content differs page-to-page, not just visually
**What:** The footer's gradient styling is consistent everywhere, but its *content* differs: `contact.html`'s footer shows Contact Info / Quick Links / Service Areas; `treatments.html`/`conditions.html`/`pricing.html`/`pay.html`/`team.html` show Clinic Info+NPI / Contact / Treatment Locations; `about.html`/`ana-berezovskaya.html`/`oxana-dickey.html` show provider-specific specializations. Since there's no shared footer partial, each was authored independently.
**Why it matters:** This is the same underlying issue that produced Finding 1 (crisis info missing from most footers) — worth calling out as the root cause, not just its most serious symptom.
**Fix:** Decide on one footer content structure (e.g., always: clinic name/NPI, contact info, quick links including crisis info, service areas) and apply it identically across all 11 pages.

### 20. No favicon
**What:** No `<link rel="icon">` anywhere, and no favicon file in the repo.
**Why it matters:** Minor, but it's a blank/generic browser tab icon on every single page — a small but real professionalism signal, and free to fix.
**Fix:** Add a simple favicon (e.g., a monogram or the teal/navy brand mark) and link it in every `<head>`.

### 21. Inconsistent ZIP code formatting
**What:** The clinic's physical address appears as both `94010` (most pages) and `94010-2429` (ZIP+4, `contact.html` only). The billing PO Box appears as both `94011-0351` (`contact.html`) and `94011` (`pay.html`).
**Why it matters:** Small, but inconsistent formatting of the same fact across pages is exactly the kind of thing that erodes confidence subtly.
**Fix:** Pick one format (ZIP+4 or 5-digit) and use it everywhere.

### 22. Decorative emoji used as heading/label prefixes without `aria-hidden`
**What:** Emoji like 🚨, 📞, 👨‍⚕️, ⚠️ are used throughout as informal icons directly inside headings and labels (e.g., `<h4>🚨 Mental Health Emergency</h4>`).
**Why it matters:** Screen readers announce these literally (e.g., "rotating light, Mental Health Emergency"), which is at best noisy and at worst confusing, especially on the crisis-related headings where clarity matters most.
**Fix:** Wrap decorative emoji in `<span aria-hidden="true">` so they remain visible but are skipped by assistive tech.

### 23. No dedicated new-patient FAQ
**What:** FAQs exist on `conditions.html`, `treatments.html` (per-treatment), and `pricing.html` (billing-specific), but there's no general FAQ covering the questions raised throughout this report (telehealth, insurance model, wait times, what to bring, age ranges).
**Fix:** Once the underlying policy questions in this report are answered, a single consolidated FAQ (on `contact.html` or a new page) would resolve several High Value gaps at once with one page.

### 24. No analytics
**What:** No analytics/tracking script of any kind was found anywhere on the site.
**Why it matters:** Neutral-to-positive from a privacy standpoint (nothing to leak visitor intent on sensitive condition pages right now), but it also means there's currently no way to measure where in the funnel prospective patients drop off, which limits how confidently future conversion-path changes can be evaluated.
**Fix:** Low priority, and only worth doing alongside Finding 4 (privacy policy) — if added, use a privacy-conscious option and disclose it in the privacy policy, and take extra care that page-level analytics on `conditions.html` doesn't transmit which specific condition section a visitor viewed to a third party beyond what's operationally necessary.

---

## Questions for you

These depend on information not present in the repo — please don't take my recommendations above as assuming an answer where one of these is unresolved.

1. **Billing model:** Is the clinical practice (evaluations, medication management, TMS, ketamine, ECT) actually out-of-network/self-pay across the board, or in-network with the specific insurers listed on `contact.html`? This determines the correct fix for Critical Finding 2.
2. **EmailJS BAA:** Does a signed Business Associate Agreement exist with EmailJS (or the plan tier in use)? Standard/consumer plans typically don't offer one.
3. **Google account type/BAA:** Is the Google Apps Script integration running under a Google Workspace account with a signed BAA, or a personal Gmail account?
4. **Telehealth:** Is telehealth offered at all? If so, to new patients or only established ones, and are there state-licensure restrictions to note?
5. **Age range/population:** Does the practice see adolescents, or adults only?
6. **Associate provider credentials:** Are Dr. Berezovskaya and Oxana Dickey's NPI and CA license numbers available and OK to publish, mirroring Dr. Levinson's page?
7. **Existing NPP:** Does the practice already provide patients a paper Notice of Privacy Practices? If so, publishing that document (or a summary of it) online is the fastest path to closing Critical Finding 4.
8. **GFE process:** Who should a self-pay patient actually contact to request a Good Faith Estimate, and what's the current turnaround?
9. **Config.js credentials:** Are the EmailJS values currently committed in `config.js` live production credentials, and is domain restriction already configured on the EmailJS side?

---

## Suggested implementation sequence

1. **Crisis info sitewide** (Critical #1) — no dependencies, highest safety impact, can ship today.
2. **Resolve the billing-model question** (Question 1) — blocks Critical #2 and several High Value fixes.
3. **De-risk the appointment form's PHI path** (Critical #3, #6) — get BAA answers (Questions 2–3), then either lock down or simplify the data flow; don't wait for a perfect long-term solution to reduce exposure now.
4. **Publish a Privacy Policy / NPP** (Critical #4) — likely fast if Question 7's answer is "yes, we have one on paper already."
5. **Add Good Faith Estimate / superbill language** (Critical #5) — a content-only fix once Question 1 is answered.
6. **Fix the `contact.html`/`treatments.html` insurance-language contradiction** (Critical #2) — now that the real model is confirmed.
7. **Fill remaining content gaps** — telehealth policy, age range, associate provider credentials, first-visit overview (High Value #12–15), pending Questions 4–6.
8. **Pay.html cross-linking and payee-name clarification** (High Value #7–8) — quick, self-contained.
9. **SEO pass** — sitemap.xml, robots.txt, structured data, single-`<h1>` fix, favicon (High Value #9–11, Polish #20).
10. **Performance pass** — compress/replace provider photos, defer the EmailJS script (High Value #16–17).
11. **Accessibility pass** — run an automated audit (axe/Lighthouse) for contrast, then fix flagged issues plus the emoji/`aria-hidden` items (Polish #22).
12. **Polish sweep** — unify footer content, fix ZIP formatting, consider a consolidated new-patient FAQ (Polish #19, #21, #23).

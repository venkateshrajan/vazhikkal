# Settlement rails: Splitwise, UPI, and manual ledgers

Research for [issue #6](https://github.com/venkateshrajan/vazhikkal/issues/6). Facts only — this ticket deliberately does **not** pick a mechanism.

Claims cite primary sources: NPCI operational circulars (read as PDFs), RBI circulars and Master Directions on rbi.org.in, the OpenAPI spec embedded in `dev.splitwise.com` plus Splitwise's official KB, gateway KYC and pricing docs, and the working code in `~/ws/birdie`. Where only a secondary source exists, or documentation is silent and a conclusion is inferred, it is **flagged inline**.

**Sourcing caveat:** `npci.org.in` returns HTTP 403 (Imperva) to every automated fetcher on every path and user agent, confirmed independently three times in this session. NPCI circulars below were read as archived copies of the official PDFs, which are scanned images with no text layer; circular numbers, dates, signatories and quoted sentences are transcribed from those documents and cross-check consistently against each other (OC/76 → OC/76A → OC/76B → OC/76C → OC/220). The load-bearing document, OC 220, was additionally **hash-verified**: copies retrieved from the Wayback Machine and from an independent third-party mirror are MD5-identical (`c20cade0aa02aa52cfec9ea8c9a9612e`), so it is the genuine NPCI document rather than a paraphrase. Official URLs are cited throughout even though they cannot be fetched programmatically.

**Also worth knowing before trusting anything on NPCI's website:** NPCI's own public UPI FAQ page still advertises that UPI "Provides for a P2P Pull functionality" as of this check — nearly a year after it was abolished. **NPCI's marketing and FAQ pages are stale and must not be relied on over the circulars.**

## Bottom line

**Every programmatic "pull the money" rail is closed to an individual.** UPI P2P collect was abolished network-wide, P2P intent links are banned, UPI Autopay requires the payee to be an onboarded merchant, and any design where vazhikkal itself receives or routes funds needs a ₹15 crore RBI payment-aggregator licence. What remains is a ledger plus a human tap.

Ranked by build cost:

1. **Splitwise as the dues ledger — hours.** A one-sided debt can be created unilaterally via `POST /create_expense` with no consent or action from the collector, and balances read back per-friend without any group. Auth is one long-lived bearer token. `~/ws/birdie/lib/splitwise.ts` already does exactly this in production and is portable in an afternoon. Zero fees, zero KYC, and the Terms explicitly sanction "hobbyists and power users to programmatically interact with their own Splitwise account". **Moves no money** — Splitwise has no payment rail in India at all. Three sharp edges: **free accounts are capped at 4 expenses per day**; **Splitwise has no automated reminder feature** (only a manual nudge and a monthly summary), so all nagging would be vazhikkal's own; and the API Terms forbid using the API to "initiate any other unsolicited direct communication or contact with Splitwise users" — awkward for an app whose job is nagging.
2. **Own ledger + an on-screen UPI QR — about a day.** Rendering a UPI QR for the collector's VPA on screen, to be **live-scanned** by the user's phone, is the one uncapped, zero-cost, zero-onboarding P2P mechanism still standing. **The app cannot learn that the payment happened** — confirmation is self-report.
3. **Razorpay payment link, collector onboarded as "Individual/Unregistered Business" — days, plus KYC, plus 2% + 18% GST per payment.** Razorpay explicitly supports individuals with no GST and a **personal savings account**, and Payment Links need no website. This is the **only** option giving trustworthy automatic confirmation (a webhook). Costs: the collector completes KYC and accepts money as a "merchant"; activation is at Razorpay's discretion; personal-dues collection sits in a category grey zone.
4. **UPI Autopay / e-mandate — weeks, and probably refused.** Reachable only as a PA-onboarded merchant, and Razorpay documents that recurring is **not enabled by default**. The RBI framework itself is friendly at vazhikkal's ticket sizes (no AFA under ₹15,000/debit), so the blocker is onboarding, not regulation.
5. **Anything where vazhikkal holds or routes funds — not viable at any budget.** ₹15 crore net worth, RBI authorisation.

Three structural facts dominate any decision:

- **Automatic debit of the user is unavailable** without the collector becoming a merchant. No rail lets a personal app pull money from its own operator on a schedule.
- **Therefore the design's integrity rests on either self-report or a gateway webhook**, and that is a product decision, not a technical one. The prior art in [research/commitment-devices.md](https://github.com/venkateshrajan/vazhikkal/blob/research/commitment-devices/research/commitment-devices.md) is blunt that one un-penalised fudge collapses the device's authority — so "did the money actually move?" is the load-bearing question, not "can we create the debt?".
- **Nobody has shipped this.** Beeminder, StickK and Forfeit all forfeit to the *platform* or a charity, never to a named friend, because the enforcing party has to be a card-charging merchant. vazhikkal's core premise is unprecedented in the prior art, and that is a signal about difficulty rather than about opportunity.

---

## The regulatory ceiling: vazhikkal must never touch the money

Not in the ticket's list, but it constrains everything else, so it goes first.

RBI's **Guidelines on Regulation of Payment Aggregators and Payment Gateways** (17 March 2020) draw the line:

- **Payment Aggregator (1.1.1):** "entities that facilitate e-commerce sites and merchants to accept various payment instruments from customers for completion of their payment obligations without the need for merchants to create a separate payment integration system of their own."
- **Payment Gateway (1.1.2):** "entities that provide technology infrastructure to route and facilitate processing of an online payment transaction **without any involvement in handling of funds**."
- **3.2:** "Non-bank PAs shall require authorisation from RBI under the Payment and Settlement Systems Act, 2007 (PSSA)."
- **3.7:** PGs "shall be considered as 'technology providers' or 'outsourcing partners' of banks or non-banks."

([RBI, 17 Mar 2020](https://www.rbi.org.in/Scripts/BS_CircularIndexDisplay.aspx?Id=11822))

The capital requirement makes this absolute: **₹15 crore net worth at application, ₹25 crore by end of the third financial year post-authorisation**, maintained on an ongoing basis. Restated in the current consolidated instrument, the **RBI (Regulation of Payment Aggregators) Directions, 2025, dated 15 September 2025**, effective immediately, which also extends the framework to physical/proximity aggregators (**PA-P**) ([Master Direction](https://www.rbi.org.in/Scripts/BS_ViewMasDirections.aspx?id=12896), [press release](https://www.rbi.org.in/Scripts/BS_PressReleaseDisplay.aspx?prid=61218)).

**There is no small-entity exemption from authorisation or capital.** The only concession for smallness runs the other way — it relaxes *merchant* due diligence: "A PA shall undertake customer due diligence (CDD) of its merchants in accordance with MD on KYC", with a **simplified process** where the merchant is not in the central registry and annual turnover is **≤ ₹40 lakh** (export ≤ ₹5 lakh) — PAN/Form 60 verification, physical address verification (CPV), plus one officially valid document for the proprietor. From **1 January 2026** all newly onboarded merchants must meet the requirements immediately. Useful to vazhikkal only as a *customer* of a licensed PA, not as a route to becoming one.

### Consequences

- Any architecture where vazhikkal **receives, holds, pools, or routes** the dues amount is inside the PA definition. Not viable.
- The viable shapes keep funds entirely outside the app:
  1. **Ledger only** — vazhikkal records the obligation; money moves peer-to-peer on a rail vazhikkal does not operate; the app at most *observes* or *is told*.
  2. **Thin client of a licensed PA** — a gateway account in the collector's name, vazhikkal calling their API.
- **Flagged:** the PA definition is written around *merchants accepting payments from customers*. A single-user personal tool moving its operator's own money to one named individual is not obviously the regulation's target, and no RBI clarification addressing personal, non-commercial fund-routing software was found. This is a deliberately conservative reading, not an authoritative application. **Not verifiable from primary sources:** whether RBI would treat a single-user personal app as a PA. The conservative reading costs nothing, because every design surviving it is also cheaper to build.

---

## UPI collect requests: abolished for P2P

**A self-hosted personal app cannot raise a collect request against anyone's UPI ID. P2P collect no longer exists on the network.**

**NPCI/UPI/OC/220/2025-26, 29 July 2025**, to "All UPI Member Banks, PSP's and Third-Party App Providers", subject **"Discontinuing the service of UPI Collect Request for Person to Person (P2P) transactions"**, signed Kunal Kalawatia, Chief of Products ([official PDF](https://www.npci.org.in/PDF/npci/upi/circular/2025/UPI-OC-No-220-FY-2025-26-Discontinuing-the-service-of-UPI-Collect-Request-for-Person-to-Person-P2P-transactions.pdf)). Verbatim:

> "Further it is hereby informed that by 1st October 2025 UPI P2P Collect shall not be allowed to be processed in UPI."
>
> "All member banks, Payment Service Providers (PSPs) and UPI Apps are hereby directed to implement the necessary changes in their systems and operational processes to ensure that no P2P Collect transaction is initiated, routed, or processed on UPI beyond 1st October 2025."

Scope, precisely:

- **P2P collect: gone**, effective 1 October 2025. Not restricted, not capped — not processed.
- **P2M collect: untouched by this circular.** Merchant collect via aggregators is the only surviving collect path. The subject line and body are confined to P2P.
- **The circular is one paragraph long and contains no exemptions.** No merchant or whitelist carve-out, no residual allowance for a pre-approved category, no within-app exception. There is nothing to apply for.
- It supersedes the earlier limit regime in **NPCI/UPI/2019-20/OC/76 (31 October 2019)**, referenced by OC/220 itself as the prior guidance on "transaction limits for Person to Person (P2P) Collect". *The commonly cited ₹2,000 / 50-per-day P2P collect figures from that circular are **secondary only** — OC 76 itself is not archived.*
- **No reversal exists.** Every archived NPCI UPI circular through FY 2026-27 (latest located: OC 233–236) was enumerated; nothing after OC 220 relaxes the ban. *Caveat: the archive index is coverage-based, not guaranteed exhaustive.*

### What replaced it — nothing an individual can use

Three post-2024 NPCI products are sometimes mistaken for successors. None is a P2P collect substitute:

- **UPI Reserve Pay** (formerly Single Block & Multiple Debits) — introduced by OC No. 200/2024-25 (31 July 2024), renamed and enhanced by **NPCI/UPI/OC-228/2025-26 (8 October 2025)**. Explicitly: "UPI Reserve Pay shall be enabled only for **online verified merchants** with low ticket and high frequency transactions." Block maximum **₹10,000**, up to **90 days**. **Individuals: no.**
- **UPI Autopay** — **NPCI/UPI/OC-223/2025-26 (7 October 2025)**, members live by 31 December 2025. See the Autopay section below; the short version is that it now mandates a Merchant Identifier Code. **Individuals: effectively no.**
- **UPI Circle (Delegated Payments)** — **NPCI/UPI/OC No. 201/2024-25 (13 August 2024)**, plus OC 201-A (full delegation) and OC 201-B (IoT devices, effective 8 July 2025). **Individuals can use this**, up to 5 secondary users per primary. But it is the wrong direction entirely: it lets someone else spend *from your* account. It does not let you request money from a third party. *Per-transaction and monthly limits (₹25,000 each) are from a secondary reproduction; the circular number and date are confirmed.*

UPI Lite (on-device wallet) and UPI HELP (an AI support assistant, OC No. 227/2025-26) are irrelevant to collection.

**There was never an individual-accessible collect API in any case.** UPI network APIs (`ReqPay` and friends) are reachable only by NPCI member PSP banks and, through them, NPCI-approved Third-Party App Providers. TPAP onboarding requires an Indian incorporated entity, a sponsor PSP bank, a tripartite agreement with NPCI, security certification and data localisation. There is no personal or developer tier. Merchant access runs through RBI-authorised payment aggregators.

## UPI deep links (`upi://pay`): banned for P2P

**NPCI/UPI/2023-24/OC/76A, 12 March 2024**, "Revision in transaction limits based on merchant & transaction types – Addendum to OC76", to "All Members of UPI", signed Viswanath Krishnamurthy, Chief Risk Officer, **effective 1 April 2024** ([official PDF](https://www.npci.org.in/PDF/npci/upi/circular/2024/UPI-OC-76A-Revision-in-transaction-limits-based-on-Merchant-&-transaction-types.pdf)). Row 1 of its table, marked **(New)**:

> "Payer PSP shall ensure P2P Intent based transactions (Initiation mode '04' and '05') shall be disallowed."

Row 3, marked (Existing – No change):

> "QR share & Pay shall have a limit of INR 2000/- for all P2P transactions (Payer PSP to ensure that UPI app to identify the same)."

Row 2 disallows intent mode '04' for all "Offline" non-verified *merchants* — the merchant restriction is narrower than the blanket P2P ban.

**Still in force.** **NPCI/UPI/OC–76C/2025-26, 8 April 2025**, effective 30 April 2025, re-lists all three rows as **"Existing No change"** ([official PDF](https://www.npci.org.in/PDF/npci/upi/circular/2025/UPI-OC-No-76C-FY-2025-26-Addendum-to-OC76-Revision-in-transaction-limits-based-on-merchant-transactions-type.pdf)). As of the latest circular located, P2P intent remains disallowed. Corroborated independently by developer reports that deep links "suddenly stopped working" after 1 April 2024 (*secondary*).

*Three ambiguities in these documents, flagged for honesty:* (a) OC 76A's header numbers it `NPCI/UPI/2023-24/OC/76A` while its own remarks column cites `NPCI/UPI/2019-20/OC/76A`; (b) OC 76C references an intervening **OC76B dated 23 August 2024 which is not archived and could not be read** — an intent-related change in that addendum cannot be ruled out, though OC 76C post-dates it and still shows the ban as unchanged; (c) OC 76C's remarks column contains the typo `NPCI/UPI/20234-25/OC76B`.

Consequences:

- A generated `upi://pay?pa=<vpa>&am=...` link is an **intent-mode P2P transaction**, which compliant payer apps must reject — even though the payer initiates it. The deep-link syntax is still defined in NPCI's UPI Linking Specification, but these risk circulars override its P2P use.
- **A QR image shared over chat is capped at ₹2,000** for P2P.
- **What still works for P2P:** typing the VPA inside the payer's own UPI app, and **live-scanning a QR code**. So an app *can* render a UPI QR on screen for the user to scan with their phone — the one compliant, effectively uncapped, zero-cost P2P mechanism left. *Precision flag:* the circulars regulate "QR share & Pay" specifically, and **no primary text was found on caps for scanning a person's QR in person** — that this is uncapped is confirmed by absence rather than by an affirmative statement. Likewise, **the commonly cited ₹1 lakh general P2P daily limit could not be sourced to any primary NPCI document** and should not be quoted as such; the only P2P amount cap confirmed from primary text is the ₹2,000 QR-share-and-pay figure.

**A useful distinction, since the original lead blurred it:** killing P2P collect and killing P2P intent are two different acts. Collect is a *pull* (payee asks). Intent is a *push* (payer initiates, from a link). Both are gone for P2P — but "payer-initiated push to a person's VPA" as a general capability is emphatically **not** dead. It is only the *programmatically generated link/intent* form that is banned. A human paying a person by VPA or scanned QR is entirely normal.

The URI shape itself is vendor-documented, if not from NPCI directly. Google's official Google Pay for India integration guide builds the intent as `upi://pay` with `pa` (payee VPA), `pn` (payee name), `mc` (merchant code), `tr` (transaction ref id), `tn` (transaction note), `am` (amount), `cu` (`INR`) and `url` ([Google](https://developers.google.com/pay/india/api/android/in-app-payments)). Two caveats worth recording: the additional NPCI parameters (`tid`, `mam`, `mode`, `sign`, `orgid`) and the mandatory/optional split **could not be confirmed** because npci.org.in 403s; and **Google documents this only as a native Android intent** — it publishes no web/browser UPI-intent guide at all. Rendering a `upi://` href from a web page is established real-world practice, not vendor-documented behaviour. `mc` is meaningless for a personal VPA.

**Verdict on the lead that motivated this ticket:** both halves are **confirmed, with the dates essentially right** — P2P collect discontinued effective 1 Oct 2025 (circular dated 29 Jul 2025); P2P intent disallowed effective 1 Apr 2024 (circular dated 12 Mar 2024). The important refinement is **scope**: this is P2P-only. P2M collect and P2M intent survive, which is exactly why the gateway route below exists at all.

### Can the app know the payment happened?

**No. Bluntly: there is no callback.** A rendered QR or `upi://` link hands off to the payer's UPI app, and the UPI intent returns its result to the *invoking Android app* via an activity result — **a plain web page gets nothing back at all**. The `url` parameter is a display/reference field, not a webhook.

Even a native app is told not to trust the client-side result. Google, verbatim: *"If the Google Pay response status is Submitted or Succeeded, you must check with your PSP or payment aggregators to ensure that the correct order amount is paid"*, immediately followed by *"You must follow the above step to prevent fraud."* ([Google](https://developers.google.com/pay/india/api/android/in-app-payments))

So the only confirmation channels are: **(a)** a PSP or aggregator you are onboarded to, **(b)** an out-of-band signal from the payee's bank (see the scraping subsection below), or **(c)** a human tapping "I received it". For a gateway-less design, **the settlement event is necessarily a human assertion.**

---

## UPI Autopay / e-mandate

**The RBI framework is permissive at vazhikkal's ticket sizes. The blocker is that the payee must be an onboarded merchant, and that gateways gate recurring separately.**

The governing instrument is the **"Digital Payments – E-mandate Framework, 2026", RBI/DPSS/2026-27/396, dated 21 April 2026**, effective immediately, consolidating the earlier e-mandate circulars ([RBI](https://www.rbi.org.in/Scripts/BS_ViewMasDirections.aspx?id=13374)):

- **AFA-free up to ₹15,000 per transaction**; above that, additional factor of authentication is required.
- **₹1,00,000 per transaction** without AFA for insurance premiums, mutual fund subscriptions and credit card bill payments only.
- **The first transaction under any e-mandate requires AFA.**
- **Pre-debit notification at least 24 hours** before each charge, stating merchant name, amount, date/time, mandate reference and reason; the customer may opt out with AFA. FASTag/NCMC auto-replenishment exempt.
- Lineage: ₹5,000 → ₹15,000 ([RBI](https://www.rbi.org.in/Scripts/NotificationUser.aspx?Id=12341)); ₹15,000 → ₹1,00,000 for the three categories via **RBI/2023-2024/88, 12 December 2023** ([RBI](https://www.rbi.org.in/scripts/NotificationUser.aspx?Id=12570)).

So a dues ledger keeping each debit **under ₹15,000 needs no OTP after setup** — regulation is not the obstacle.

The obstacles are:

- **The payee must be a merchant — and NPCI has now made this mechanical.** **NPCI/UPI/OC-223/2025-26, 7 October 2025** ("Enhancement of UPI Autopay", members live by 31 December 2025) states that "Payee PSP shall generate a unique **Merchant Identifier Code (MIC)** for every merchant… This MIC shall be a **mandatory parameter for all UPI Autopay mandates with purpose code 'AZ'**" ([official PDF](https://www.npci.org.in/uploads/UPI_OC_No_223_FY_2025_26_Enhancement_of_UPI_Autopay_88b38535cb.pdf)). A mandate cannot be created without a merchant identifier issued by a Payee PSP to a merchant. Combined with the RBI framework's merchant framing throughout and the PA Directions' merchant-CDD requirement, **a private individual cannot be an Autopay creditor on their own VPA.**
  *Flagged, precisely:* this is **strongly indicated, not literally stated**. No circular sentence says "P2P mandates are prohibited" in those words; the conclusion rests on the MIC being mandatory for every 'AZ' mandate plus the total absence of any individual-payee provision. The definitive text would be in NPCI's UPI Procedural Guidelines or the mandate API spec, which are member-access documents that could not be retrieved. The rules do explicitly accommodate tiny individual/proprietary *merchants* (the ≤ ₹40 lakh simplified CDD path), so the route is "become a small merchant", not "be an individual".
- **Gateways gate recurring separately.** Razorpay's own docs on recurring payments say that if it is not activated, "contact your Account Manager or reach out to the support team". Supported rails are cards, **UPI Autopay**, e-mandate (netbanking / debit card / Aadhaar) and Paper NACH; minimum authorisation amount ₹1 ([Razorpay recurring](https://razorpay.com/docs/payments/recurring-payments/), [UPI recurring](https://razorpay.com/docs/payments/recurring-payments/upi/)). **Practically, recurring enablement is discretionary and is the step most likely to be refused for an unregistered individual — and that discretion is documented nowhere.**
- **UPI 2.0 one-time mandates** (block funds now, debit later) exist and appear in consumer apps, but are used almost entirely for IPO/ASBA blocks and execute once — useless for recurring dues. Whether an individual can be the payee is **undocumented**.
- NPCI's own UPI Autopay product pages and its circular **UPI/OC/223/FY2025-26 "Enhancement of UPI Autopay"** return 403 to automated fetch, so **NPCI-specific figures (maximum mandate amount, one-time-mandate caps, eligible merchant category codes) are unconfirmed** and need a browser.

---

## Payment gateways in India

### Who can onboard without a registered business

**Razorpay — yes, and it is the documented best path.** Business types listed include **Individual/Unregistered Businesses**, Proprietorship, Private/Public Limited, Partnership, LLP, Trust, HUF, Society and others ([business types & KYC docs](https://razorpay.com/docs/payments/business-types-kyc-documents/)).

- **Individual/Unregistered documents:** identity and address auto-fetched from CKYC; if CKYC fails, PAN card + mobile number, plus DigiLocker verification or an Aadhaar/photo ID/passport soft copy; bank account number + IFSC. **No registration certificate, no GST, no business PAN.**
- Notably, the heavier **Proprietorship** category demands any two of MSME/Udyam, GST certificate, Shop & Establishment certificate, IEC or a postpaid mobile bill, plus a Power of Attorney. **"Individual/Unregistered" is strictly cheaper than "Proprietorship" for a solo developer** — worth knowing before picking a category at signup.
- Verbatim from the [FAQs](https://razorpay.com/docs/payments/faqs/): *"Yes, we do support freelancers/individual business entities."* and *"Submitting GST and current account details is not mandatory. You can select I don't have GST and proceed with a savings account."*
- The same FAQ records that from **1 January 2026, website policies (Terms, Refund, Privacy) are "no longer required at onboarding"**.
- **No-website path:** *"You do not need a website or mobile app to use Payment Links."* Payment Links are fully API-driven. *Flagged:* the docs never say a business URL is *never* asked for — the activation form does collect one, and risk review can demand a live URL. A known doc-versus-practice gap.

**Cashfree — individuals allowed, but a website or published app is mandatory.** Types include Individual, Proprietorship, Partnership, Society/Trust/NGO, Private Limited/LLP; Individual requires **PAN, Aadhaar, bank account details**; *"No, GSTIN is not mandatory."* But activation requires *"either a live website or an app published on the App Store or Play Store"* ([onboarding FAQs](https://www.cashfree.com/docs/help/onboarding-related/onboarding-faqs)). For a VPS-only side project with no public site, this is the decisive difference from Razorpay. Current-account requirement not stated — **unconfirmed**.

Others:

| Provider | Business entity needed? | API? | Confidence |
| --- | --- | --- | --- |
| Razorpay (PG, Payment Links/Pages/QR) | No — Individual/Unregistered category | Yes, full REST | Confirmed |
| Cashfree (PG, Payment Links) | No entity, **but live site/app required** | Yes, full REST | Confirmed |
| Stripe India | Entity types permissive, GST/CIN optional — **but "Stripe services are invite-only in India. Businesses from India are not able to sign-up for a new Stripe account through our website."** | Yes | [Confirmed](https://support.stripe.com/questions/how-can-i-open-a-stripe-account-in-india) — effectively inaccessible |
| Paytm PG | Claims "100% digital onboarding and minimal documentation", lists freelancers; entity list not published | Yes | Official page but vague — unconfirmed for individuals |
| PhonePe PG | Docs publish no entity types or document lists | Yes | Unconfirmed |
| Instamojo | Pricing page states no eligibility restriction; historically the individual/no-website option | Yes | Pricing confirmed; individual eligibility unconfirmed. Also: RBI returned its PA application in 2023 and it now routes via partner PAs — longevity questionable |
| Setu UPI Deeplinks | Right product shape (UPI links/intents by API, webhook on payment), open sandbox; live use needs a Pine Labs PA agreement + full KYC | Yes | Individual eligibility undocumented |
| Zoho Payments | Effectively no — requires GST certificate, GST-exemption declaration, or Udyam even for individuals | Yes | Confirmed |
| Decentro / Juspay | B2B infrastructure with business due diligence | Yes | Not individual-accessible |

### The structural constraint: direction of money flow

This decides the architecture.

- **Collect products** (payment gateway, payment links, UPI intents) move money **payer → PA escrow → the merchant's registered bank account**. For "the user pays the collector", **the collector must be the merchant**. The payer needs no onboarding — they just pay a link. Workable, but it puts the KYC burden and the "you are a merchant now" framing on the collector.
- **Payout products** (push from a prefunded balance to an arbitrary bank account or VPA) would let the *user* automate pushing money to the collector — architecturally the nicer fit, since the user carries the obligation. But both realistic options are business-gated: **RazorpayX Lite is closed to new merchants** (new RazorpayX requires a partner-bank current account and individual approval), and **Cashfree Payouts is registered-businesses-only**. **An unregistered individual cannot get a payouts product.**
- **Money can land in a personal savings account.** Razorpay, verbatim: *"If your business is registered with the Government of India, please [provide the] Current Account number registered with your business. If you are not registered or are a sole proprietor, you may provide your personal bank account details."* ([activation docs](https://razorpay.com/docs/payments/account-activation-support/)). The account name should match the PAN holder.
- **Grey zone, flagged:** the PA framework governs payments *to merchants for goods and services*. Onboarding as a "merchant" to collect personal dues between two individuals could trip category review. No documentation addresses this use case either way.

### Cost on small payments

**UPI zero-MDR is real but does not make collection free.** CBDT **Circular No. 32/2019 dated 30 December 2019**, read with **Section 10A of the Payment and Settlement Systems Act, 2007**: "no Bank or system provider shall impose any charge on a payer making payment, or a beneficiary receiving payment, through electronic modes", and any charge including MDR "shall not be applicable on or after 01st January, 2020" for prescribed modes (UPI, RuPay debit) ([Income Tax Dept](https://www.incometax.gov.in/iec/foportal/e-Campaigns/e-mail/detail/8412)). **Zero-MDR binds banks and system providers, not payment-aggregator "platform fees"** — which is exactly how Razorpay charges 2% on a zero-MDR instrument. Its pricing page says so in as many words: UPI and RuPay are *"Zero MDR — 2% platform fee applies"*.

| Option | Rate | Fee on ₹100 | Fee on ₹500 | Notes |
| --- | --- | --- | --- | --- |
| **Razorpay** standard | 2% all domestic methods incl. UPI | **₹2.36** | ₹11.80 | ₹0 setup, ₹0 AMC, ₹0 refund fee. **No minimum fee published.** Payment Pages/Links add 0.2%; UPI QR 0.99%; Smart Collect 1% or ₹10, whichever is lower ([pricing](https://razorpay.com/pricing/)) |
| **Cashfree** standard | 1.95% (1.6% promo for new merchants to 31 Jul 2026, conditional on UPI ≥40% of volume and ≤₹1 cr/month) | ₹2.30 | ₹11.51 | ₹0 setup, ₹0 AMC; links/forms included ([charges](https://www.cashfree.com/payment-gateway-charges/)) |
| Paytm PG | 1.99% | ₹2.35 | ₹11.74 | ₹0 setup, no AMC |
| PhonePe PG | 1.99% / "FREE" promo | — | — | promo terms ambiguous (*flag*) |
| **Instamojo** | 2% + ₹3 | **₹5.90 (≈6%)** | ₹15.34 | the ₹3 flat is brutal at small ticket — on ₹50 it is ≈8% |
| Payout products | ~₹2–5 flat | ~₹2.36–5.90 | ~₹2.36–5.90 | cheapest per rupee — but business-only; exact rates unpublished (*flag*) |

All fees carry **18% GST** on the fee. The figures above include it where marked.

**The percentage fee is the thing to notice at vazhikkal's ticket sizes.** 2% + GST on a ₹100 due is ₹2.36 — tolerable absolutely, but it is a per-event cost the ledger routes do not have, and small dues are where the app will spend most of its life. Razorpay's lack of a fixed floor beats Instamojo decisively below roughly ₹300.

*Flagged:* Razorpay's pricing page and docs have disagreed on settlement timing (instant/T+1 vs T+2 for new merchants).

*Context:* reports in mid-2025 that MDR would be reintroduced on large-merchant UPI were publicly called "false, baseless and misleading" by the Finance Ministry, which reaffirmed zero-MDR since January 2020. No primary circular introducing MDR was found. *Secondary source (PIB returns 403 to automated fetch).* Irrelevant at vazhikkal ticket sizes either way.

---

## Splitwise API

Primary source: the OpenAPI 3.0.1 spec embedded in [dev.splitwise.com](https://dev.splitwise.com/) (`info.version: 3.0.0`; there is no separate `openapi.yaml` — `/openapi.yaml` 404s), plus the official KB at `kb.splitwise.com`. Base URL `https://secure.splitwise.com/api/v3.0` (a single `servers` entry).

### Auth

Top-level `security: [{OAuth: []}, {ApiKeyAuth: []}]` — **either scheme works on every endpoint**.

- **OAuth 2**: `flows.authorizationCode`, `authorizationUrl /oauth/authorize`, `tokenUrl /oauth/token`. **`scopes: {}` — no scopes are defined at all**, so there is no permission granularity. Register at `secure.splitwise.com/apps` for a key and secret.
- **API key (bearer)**: `type: http`, `scheme: bearer`, `bearerFormat: API key`. Verbatim: *"For speed and ease of prototyping, you can generate a personal API key on your app's details page. You should present this key to the server via the Authorization header as a Bearer token. The API key is an access token for your personal account, so keep it as safe as you would a password."* Rotation is manual, on the app details page.
- It is **per-user**, and you must **register an app first** to reach the page that generates it.
- **Undocumented:** any expiry or TTL. The docs never say the key never expires — only that rotation is manual. Treat "long-lived" as unstated rather than confirmed.

For a single-user self-hosted app the personal key alone suffices — exactly what both birdie clients do in production. No OAuth dance, no refresh.

### Creating a one-sided debt — the key question

**Yes, unilaterally, with no action from the other party.**

`POST /create_expense` has two mutually exclusive body variants (`oneOf`):

- **Equal split**: requires `group_id`, `split_equally: true`, `description`, `cost`. *"You may either split an expense equally (only with `group_id` provided)... When splitting equally, the authenticated user is assumed to be the payer."* → **`split_equally` cannot be used without a group.**
- **By shares**: flattened parameters `users__{index}__{property}`, where property is `user_id` (or `email` + `first_name` + `last_name`), `paid_share`, `owed_share`. Both shares are "Decimal amount as a string with 2 decimal places".

A 100/0 debt is directly expressible:

```
users__0__user_id=<A>&users__0__paid_share=100.00&users__0__owed_share=0.00
users__1__user_id=<B>&users__1__paid_share=0.00&users__1__owed_share=100.00
```

**Nothing requires consent, acknowledgement, or any action from B.** Shared optional fields: `cost`, `description`, `details`, `date`, `currency_code`, `category_id`, and `repeat_interval` (`never|weekly|fortnightly|monthly|yearly`).

- **Gotcha, documented:** *"200 OK does not indicate a successful response. The operation was successful only if `errors` is empty."* `errors` is `{field: [string]}`. Documented codes are 200, 400, 401, 403 — no 422.
- **Undocumented:** the Σ`paid_share` = Σ`owed_share` = `cost` validation rule. Universally assumed, never stated; only the generic `errors` mechanism is documented.
- `POST /update_expense/{id}` takes the same shape, and warns: *"If any values is supplied for `users__{index}__{property}`, **all** shares for the expense will be overwritten."*

### Non-group expenses and unaccepted friends

- `by_shares.group_id` description: *"The group to put this expense in, or **`0` to create an expense outside of a group**."* `group_id` is `required` in the schema, so pass `0` explicitly — omitting it is not documented. In responses, `group_id` is null for non-group expenses and `friendship_id` is populated instead. `GET /get_groups`: *"Expenses that are not associated with a group are listed in a group with ID 0."*
- **No acceptance needed.** KB, verbatim: *"If you invite friends with their email or phone number, you can start adding expenses with them right away, even before they accept the invite. You can keep adding expenses with them indefinitely even if they never claim it — we'll send one invite, one reminder, and then leave them alone."*
- `POST /create_friend` takes `user_email`, `user_first_name`, `user_last_name`: *"If the other user does not exist, you must supply `user_first_name`."* (The schema oddly lists `required: ["email"]` — a spec bug.) `POST /create_friends` is the bulk form.
- `user.registration_status` ∈ `confirmed` | `dummy` | `invited`, which lets the app detect an unaccepted counterparty. Creating a contactless "dummy" friend via API is **undocumented** (web-app only per KB).

### Settlements

- **There is no dedicated settle-up endpoint.** Confirmed by absence — the complete path list is `get_current_user`, `get_user/{id}`, `update_user/{id}`, `get_groups`, `get_group/{id}`, `create_group`, `delete_group/{id}`, `undelete_group/{id}`, `add_user_to_group`, `remove_user_from_group`, `get_friends`, `get_friend/{id}`, `create_friend`, `create_friends`, `delete_friend/{id}`, `get_currencies`, `get_expense/{id}`, `get_expenses`, `create_expense`, `update_expense/{id}`, `delete_expense/{id}`, `undelete_expense/{id}`, `get_comments`, `create_comment`, `delete_comment/{id}`, `get_notifications`, `get_categories`.
- A settlement **is** an expense: the response schema carries `payment: boolean` — *"Whether this was a payment between users"* — and `transaction_confirmed: boolean` — *"If a payment was made via an integrated third party service, whether it was confirmed by that service."*
- **`payment` is NOT a documented `create_expense` input.** It appears only in *response* schemas. Sending `payment: true` on create is what third-party clients do — including birdie's vendored MCP server, where it works in practice — but it is **undocumented and could break without notice**. The documented way to zero a balance is an ordinary expense with reversed shares. KB confirms settlements are ordinary deletable rows: deleting a payment record restores the balance.
- **The API moves no money at all.** No payment or transfer endpoint exists; the only provider-related field is the read-only `transaction_confirmed`.
- **India is not supported, and there is no UPI anywhere.** Official settle-up rails are **Splitwise Pay** (US only — *"currently only available for U.S. residents"*, banking by Coastal Community Bank), **Venmo** and **PayPal**, plus *"In select European countries, you can use Pay by Bank"*. KB, verbatim: *"If you're in a region where these options aren't available, you can settle up outside Splitwise and use the 'Record a payment' option to update your balance in the app."* Zero occurrences of "UPI" in the spec or any payment-integration article. (The earlier Paytm-for-INR integration announced in 2017 has no current confirmation; **assume dead**.)

### Reading balances back

Fully sufficient for "how much does the user still owe the collector":

- Shared `balance` schema is `{ currency_code, amount }` where **`amount` is a string**, and it is an **array** — one entry per currency. Signed, e.g. `"-5.02"`. Note the spec's own example `"414.5"` is not 2-decimal padded on read.
- `GET /get_group/{id}` → `members[]` each with `balance[]`, plus **`original_debts[]`** and **`simplified_debts[]`**, each `{from, to, amount, currency_code}`; also `group_type`, `simplify_by_default`, `invite_link`, `updated_at`.
- `GET /get_friends` → each friend has a **top-level `balance[]`** (net balance with that friend) plus `groups[]{group_id, balance[]}`. **So yes — a per-friend balance exists independently of any group.** `GET /get_friend/{id}` for one.
- `GET /get_current_user` → `id`, `first_name`, `last_name`, `email`, `registration_status`, `default_currency`, `locale`, `notifications_count` and a notification-settings map. **No balance fields on current_user.**
- Per-share amounts on an expense: `share{user, user_id, paid_share, owed_share, net_balance}` (all strings), plus `expense.repayments[]{from, to, amount}`.

### Rate limits and pagination

- **Rate limits are undocumented — no number anywhere.** Zero occurrences of "429"; no 429 in any endpoint's responses; no rate-limit headers; no `Retry-After`. Only prose, all in the Terms: *"our Self-Serve API has conservative rate and access limits, which are subject to change at any time and not well suited to commercial projects"*; Splitwise *"may limit (i) the number of network calls that your App may make via the API; and (ii) the maximum number of Splitwise users that may connect your Application... may impose or modify these limitations without notice."*
- `GET /get_expenses` params: `group_id` (*"if provided... `friend_id` will be ignored"*), `friend_id`, `dated_after`, `dated_before`, `updated_after`, `updated_before`, **`limit` (default 20)**, **`offset` (default 0)**. No cursor, no `Link` header, no documented maximum `limit`.
- **`limit=0` meaning "all" is undocumented for `get_expenses`.** That semantic is documented only for `get_notifications`. **This matters:** `~/ws/birdie/lib/splitwise.ts` calls `listSessions(limit = 0)` and comments that "`limit: 0` asks Splitwise for all of them". It evidently works, but it is an **undocumented behaviour that birdie depends on**, and any port inherits that dependency.

### Notifications and webhooks

- **The other party is notified; you are not.** KB: *"Splitwise sends email notifications when there are updates in a group or friendship"*, and *"You will only receive notifications for changes made by other people — you won't receive emails for expenses that you added, edited, or deleted yourself."* An API-created expense is a change made by you, so the counterparty gets notified. Also: *"Splitwise delays sending certain notifications by 60 seconds in case you need to correct any errors. If you add an expense and immediately delete it, for example, no notifications will be sent."* Recipients can disable this per-account.
- **Undocumented:** whether an API-created expense triggers *push* specifically (as opposed to email); no article distinguishes API-originated from app-originated writes, and no official statement says the API suppresses notifications.
- **There are no webhooks.** Zero occurrences of "webhook" or "callback" in the spec. The only mechanism is polling `GET /get_notifications` (`updated_after`, `limit`), returning `{id, type, created_at, created_by, source{type,id,url}, content}` with documented `type` codes 0–15 (0 expense added, 1 updated, 2 deleted, 3 comment added, 8 added as friend, 11 debt simplification, …) and the warning *"Notification types may be added in the future without warning."*

### Pro gating and Terms restrictions — two real constraints

**1. The free tier caps expense creation at four per day.** KB, verbatim: *"Unlimited expenses: Add as many expenses as you need without hitting a limit (**free users can add up to 4 expenses each day**)."* This is an account-level cap, so it very likely applies to API writes — but **the API docs never mention it and the KB never mentions the API, so API applicability is undocumented**. For vazhikkal, four writes a day is probably enough for dues but leaves little headroom for retries, corrections, or a backfill; if it does apply, Splitwise Pro removes it. Other Pro-only features (transaction import, search, currency conversion, receipt scanning, charts, default splits) are irrelevant here, and **no API endpoint is documented as Pro-gated**.

**2. The Terms of Use contain two clauses that bite an app like this.** The Self-Serve API is explicitly for *"third-party applications, as well as... hobbyists and power users to programmatically interact with their own Splitwise account"* — so personal automation is **expressly sanctioned**, and vazhikkal fits. But:

- *"You may not [use] Splitwise Materials to create an application that replicates existing Splitwise functionality or competes with Splitwise and our Services"* — relevant if vazhikkal grows its own ledger UI on top.
- *"You will not use our API to distribute unsolicited advertising or promotions, or to send messages, make comments, or initiate any other unsolicited direct communication or contact with Splitwise users"* — **directly relevant to a nagging app.** Creating expenses that trigger Splitwise's own notifications to the collector is arguably exactly "unsolicited direct communication or contact with Splitwise users" if the collector has not opted in. A collector who has agreed to the arrangement is presumably fine; automated nudging of anyone who has not is not.

Also: no app usable by under-13s, no reselling Splitwise data, explicit end-user consent and a privacy policy required, no press release naming Splitwise without written consent, access revocable at sole discretion without notice, liability capped at $100, Massachusetts law.

*Secondary, not relied on:* the third-party SDKs dev.splitwise.com links are explicitly *"unofficial... not been reviewed or endorsed by Splitwise."*

---

## The existing Splitwise integration in `birdie`

Two independent implementations of the same v3.0 surface exist locally. Both authenticate with a **single long-lived personal bearer token** from the environment — no OAuth, no refresh, no per-user consent.

### `~/ws/birdie/lib/splitwise.ts` (348 lines, TypeScript, the live one)

A hand-rolled REST client with no SDK dependency. Two private helpers do all transport — `apiGet` (query params, bearer header, `cache: "no-store"`) and `apiPost` (form-encoded) — and everything else is a thin typed wrapper. It is `"server-only"`, so the token never reaches the client.

| Function | Endpoint | Notes |
| --- | --- | --- |
| `resolveGroupId()` | `GET /get_groups` | resolves a group **by name** from env, memoised in a module-level variable for the process lifetime |
| `getCurrentUserId()` | `GET /get_current_user` | identifies the token owner, used as "the host" |
| `getMembers()` | `GET /get_group/{id}` | maps each member's `balance[]`, picking the `INR` entry, to a signed number |
| `listSessions(limit=0)` | `GET /get_expenses` | `limit: 0` for all (**undocumented**, see above); filters out `deleted_at` and `payment: true` rows |
| `getBalancesAsOf(cutoff)` | `GET /get_expenses` | recomputes balances client-side as `Σ paid_share − Σ owed_share` over rows dated `< cutoff`, this time **including** payments |
| `createSession()` | `POST /create_expense` | |
| `updateSession(id)` | `POST /update_expense/{id}` | same form shape as create |
| `deleteSession(id)` | `POST /delete_expense/{id}` | checks the `success` flag |
| `recordAdvance()` | `POST /create_expense` | **the directly relevant one** |

Two mechanics here are load-bearing for vazhikkal.

**1. It already creates a one-sided debt.** `recordAdvance()` posts an expense where one party pays the whole cost and the *other* owes the whole cost:

```
users__0__user_id = memberId   users__0__paid_share = amt    users__0__owed_share = 0.00
users__1__user_id = hostId     users__1__paid_share = 0.00   users__1__owed_share = amt
```

Exactly the shape vazhikkal needs for "a due is credited to the collector" (direction depending on which way the balance should read). It is working production code against real accounts — the strongest available evidence that Splitwise accepts a fully asymmetric split with no counterparty involvement, and it matches what the spec permits.

**2. `expenseForm()` documents the array encoding.** Splitwise's users array is `users__{index}__{field}` — flat, indexed, form-encoded, not JSON. `expenseForm()` also handles the case where the payer is not among those who owe, appending them as a zero-owed participant so `paid_share` has somewhere to live. The sort of detail that costs an afternoon to rediscover.

### `~/ws/birdie/vendor/splitwise-mcp` (Python, vendored, superseded)

A `FastMCP` server (`splitwise_server.py`, 508 lines) exposing ten tools: `get_current_user`, `list_currencies`, `list_groups`, `get_group`, `list_expenses`, `get_expense`, `create_expense`, `update_expense`, `delete_expense`, `create_payment`. Same bearer auth via `httpx`. Stdio by default, `--remote` for streamable-http.

**It is no longer on birdie's request path.** The header comment in `lib/splitwise.ts` says why: all Splitwise access "used to go through the Python MCP server spawned via the claude CLI — far too heavy for per-request reads." It survives only for an LLM-driven admin chat surface, with a 65 MB vendored venv checked in beside it.

Its one piece of unique knowledge is `create_payment()`, which the TS client lacks: it constructs a settlement as an expense with `payment: "true"`, payer's `paid_share` = amount, payee's `owed_share` = amount, description `"Payment"`. **Note this is precisely the undocumented input flagged above** — it works, but the spec does not sanction it. The MCP server also implements `equal` / `exact` / `percentage` splits by computing shares client-side and giving the rounding remainder to the first person; Splitwise does no splitting at this layer.

### Reusability verdict

**High, with caveats.** Directly reusable:

- The transport layer (`apiGet` / `apiPost` / `SplitwiseError`) is ~60 lines with no dependency beyond `fetch` — portable to any TS runtime including Bun.
- `recordAdvance()` is a working template for "create a one-sided due".
- `create_payment`'s `payment: "true"` shape is the template for "mark the due settled", with the documented-behaviour caveat.
- The raw-shape interfaces document which response fields matter, including that all money values are strings.
- The 200-with-`errors` handling is already correct in both.

Caveats:

- Everything is **group-scoped by group name from env**, and `resolveGroupId()` is called by nearly every function. vazhikkal's relationship is two-person: it can use a two-person group (simplest, keeps the code shape) or `group_id: 0` expenses with the per-friend balance from `get_friends`, in which case the group plumbing is dead weight to strip.
- Birdie's domain vocabulary is baked into names and types (`Session`, `SessionAttendee`, `attendees`, `listSessions`). A port should rename to vazhikkal's own domain terms.
- `getBalancesAsOf()` re-fetches **all** expenses and recomputes in-process. Birdie measures the round-trip at ~2.5s and wraps it in a 60s TTL cache (`lib/ledger.ts`) precisely because it is slow. Any vazhikkal design reading Splitwise on a hot path inherits that latency and needs the same caching.
- It depends on the **undocumented `limit=0`** semantic.
- The cached group id is process-lifetime with no invalidation.
- **Treat the MCP server as not reusable** — a heavier path to the same ten calls, already documented as too slow, and it drags in a Python runtime and a 65 MB venv.

Net: on the order of an afternoon to port a two-person version with `createDue` / `recordSettlement` / `readBalance`. **By far the cheapest working rail in this document, and the only one where code already runs against real accounts.**

---

## Pure ledger with manual settlement: what existing tools actually do

### Commitment-device apps (money genuinely moves)

- **Beeminder**: a card on file via Stripe is mandatory before you can create goals — the "Commitwall"; Beeminder states card-less commitment is "toothless" and that the requirement measurably improved outcomes. On derailment: a legitimacy-check email, then the card is charged automatically after ~24h. **Beeminder keeps the money** — no arbitrary beneficiary; only the top premium tier can direct 50% to a charity shortlist. **India is explicitly broken:** Beeminder documents that Indian regulation prevents charging Indian cards the way it charges everyone else, and the official workaround is "Honey Money" — prepaid store credit bought in one-time purchases that derailments draw down.
- **StickK**: no upfront deposit; the card on file is charged only on a failed reporting period, and a referee's failure report is binding. New contracts can only forfeit to an anti-charity or to StickK itself (friend/foe and charity recipients were eliminated), and StickK deducts 50% of anti-charity forfeits plus transaction costs. India behaviour **undocumented**; as merchant-initiated foreign card charges it would face the same e-mandate friction as Beeminder (*inference*).
- **Forfeit**: Stripe pre-authorises the stake at commit time and captures the hold on failure. Forfeited money is currently retained by Forfeit, which says it is "building an option to send... to friends". India behaviour undocumented.

**The pattern across all three: the enforcing party is a merchant with a card-charging relationship, and the money goes to the platform, not to a friend.** None supports "pay a specific person I know". The two that documented India at all fell back to prepaid credit, because RBI's e-mandate framework (AFA plus pre-debit notification for recurring debits) breaks foreign merchant-initiated charges on Indian cards.

This is the single most important comparative fact in this document: **vazhikkal's "credit a real amount to a real person" is a design nobody in the prior art has shipped**, and the reason is structural, not accidental.

### Ledger apps (the debt is tracked, paying is social)

**Splitwise's actual nagging surface is thinner than its reputation**, and confirmed largely by absence — a search of the official KB for "reminder" returns only three articles.

- **A manual, user-triggered balance reminder exists**: where a request-payment rail is unavailable you can "send them a balance reminder", with separate mobile and web flows ([KB](https://kb.splitwise.com/payment-integrations/can-i-request-a-payment-via-venmo-or-paypal)).
- **No automatic recurring debt-reminder feature is documented anywhere.** The only *scheduled* nags are (a) an optional pre-post email on **recurring expenses** — "you can also choose to set an email reminder before each recurrence is posted. The reminder will be sent to all members on the expense" ([KB](https://kb.splitwise.com/balances-and-expenses/how-can-i-manage-recurring-expenses)) — and (b) a **monthly summary email** ([privacy policy](https://www.splitwise.com/privacy)). The recurring-expense reminder is interesting for vazhikkal, since `repeat_interval` is an exposed `create_expense` parameter.
- **Reminders are NOT Pro-gated.** Neither the Pro KB article nor the Pro marketing page lists reminders or notifications at all. *This corrects a claim in at least one third-party 2026 review.*
- **Splitwise has a deliberate anti-spam posture**: "we'll send one invite, one reminder, and then leave them alone." The same article notes you can add a **placeholder friend** with a name and no contact details — such a counterparty is by construction un-naggable.
- **Social visibility** is real but shallow: groups are shared state and "once you save an expense, everyone's balances update automatically". That balances are shared is confirmed; *"shame-driven visibility as a designed mechanism" is an inference, not a documented product intent.*

Other ledger apps:

- **Settle Up** — cloud-synced shared expenses, "shows who pays next and minimizes the transactions", and notably **has a real public API** (Firebase Realtime Database REST, sandbox at `settle-up-sandbox.firebaseio.com`, live access on request, Firebase Auth required) ([api.settleup.io](https://api.settleup.io/)). No reminder feature documented. Worth knowing as a Splitwise alternative if the Splitwise terms or the 4-expense cap bite.
- **Tricount** — free; "Send a payment request straight from the app and get paid directly to your bank account". *India availability of payment requests is not stated on the official page and the help centre 404s — unconfirmed, and the bank-transfer framing looks EU-centric.* No public API found.
- **Splid** — no official feature text retrievable; **unconfirmed across the board**.

India-specific apps do **not** fill this gap:

- **Google Pay India** — no bill-split, expense-group, or reminder article exists in the India help centre; the closest primitives are "Receive or request money" (UPI collect) and UPI Circle. No consumer API. *Confirmed by absence from the [full India article index](https://support.google.com/pay/india/).*
- **PhonePe** — the official homepage product list (payments, cards, investments, gold, insurance, travel, lending, NRI) contains **no bill-split or expense-sharing feature**. *Confirmed by absence; existence of some in-app split feature unconfirmed.*
- **CRED** — has "payment reminders", but for *your own credit-card bills*, not P2P debts. No expense-splitting or money-owed-between-people feature named.

**Takeaway: no ledger app can *make* a debt get paid.** Their entire enforcement stack is (a) the debt being visible to both parties, (b) reminder emails and pushes, (c) the social cost of an unpaid balance with a real person. Every tool examined lands in one of three buckets: a **manual user-triggered nudge**, a **scheduled ambient nag**, or **rail-owning settlement** — and no rail-owning option exists in India. In India the money moves off-platform: the debtor pays by UPI and someone records it.

For vazhikkal this is the honest framing of option 1 — Splitwise supplies the ledger, the two-party visibility, and a manual nudge. It supplies **no** collection mechanism and **no** automated nagging. Any automated nagging must be vazhikkal's own (email/push from the VPS), using Splitwise purely as the balance store — and note the Terms clause on unsolicited contact if that nagging is ever routed *through* Splitwise.

### Confirming a personal UPI receipt by scraping alerts

This is the only route to automatic confirmation that avoids a gateway, so it is worth stating what it actually costs. **It is a real pattern, and it is policy-hostile.**

- **Android SMS**: Google Play restricts the SMS permission group. "Apps must be actively registered as the default SMS, Phone, or Assistant handler before prompting users to accept any of SMS or Call Log permissions", and access is allowed only for permitted use cases — the permitted list *does* include **"SMS-based financial transactions"**, but also insists you "only access Call Log or SMS permissions when your app falls within permitted uses and only to enable your app's critical core functionality", and forbids any transfer resulting in a sale of the data ([Play policy](https://support.google.com/googleplay/android-developer/answer/10208820)). Practically: a personal-finance app can sometimes get a declaration approved, but it is a review gate, not a free permission — and it is **unavailable on iOS and to any pure web app**, which is what vazhikkal is.
- **Gmail API on bank alert emails**: `gmail.readonly`, `gmail.modify` and `mail.google.com/` are **restricted scopes**. Public apps need OAuth verification and, per Google, "If you store restricted scope data on servers (or transmit), then you must go through a security assessment" (CASA) ([scopes](https://developers.google.com/gmail/api/auth/scopes)). The Limited Use rules further require the data to serve "user-facing features that are prominent in the requesting application's user interface", bar humans from reading it without "affirmative agreement to view specific messages", bar transfer or sale, and specifically bar use "to determine credit-worthiness or for lending purposes" — **directly relevant if a dues ledger ever starts scoring the payer** ([API Services User Data Policy](https://developers.google.com/terms/api-services-user-data-policy)). A single-user self-hosted app consuming only its own mailbox is a much softer case than a distributed app, but the scopes are still restricted.
- **Reliability**: bank SMS and email alerts are unsigned, free-text, bank-specific and format-unstable. Parsing them yields a **heuristic, not proof of settlement**. *Assessment, not a documented fact.*

Net: scraping can raise confidence that money arrived, but it cannot be made authoritative, and for a self-hosted web app the Android SMS route is simply unavailable.

---

## Open questions / ambiguous

Things a decision ticket should not assume are settled.

**Product-level, and the one that actually matters:**

1. **Does the app need to *know* the money moved, or is self-report acceptable?** Pivotal, unresolved, and a product decision rather than a technical one. Only the gateway route gives a trustworthy signal. Everything cheap relies on the user's own honesty — and the commitment-device research says a single un-penalised fudge collapses the device.
2. **Is the collector willing to be a "merchant"?** Option 3 requires them to complete KYC, hold the gateway account, and receive money as business income with the tax and record-keeping that implies. This is a social and practical question, not a technical one, and it gates the only verifiable rail.

**Regulatory:**

3. **Whether RBI would treat a single-user personal app as a payment aggregator.** No clarification found. Conservative reading adopted; cost of being wrong is high, cost of complying is nil.
4. **Whether a PA would accept "collecting personal dues between two individuals" as a merchant category**, or flag it in review. Undocumented either way; discoverable only by applying.
5. **Whether an individual can be the payee of a UPI 2.0 one-time mandate.** Undocumented — and probably a dead end for recurring dues regardless, since it executes once.
6. **Whether NPCI's UPI Procedural Guidelines or mandate API spec explicitly prohibit a P2P mandate.** These are member-access documents and could not be retrieved. The MIC requirement makes an individual creditor practically impossible, but the flat prohibition is inferred rather than quoted.
7. **NPCI-specific Autopay figures** — maximum mandate amount, one-time-mandate caps, eligible merchant category codes. npci.org.in 403s all automated fetchers; these need a browser.
8. **NPCI/UPI/2024-25/OC76B (23 August 2024) is not archived and was not read.** OC 76C post-dates it and still shows the P2P intent ban unchanged, so the conclusion holds, but the intervening document is a genuine gap in the chain.
9. **Whether scanning a person's QR in person is subject to any P2P cap.** Confirmed by absence of any restricting circular, not by an affirmative statement.

**Splitwise:**

10. **Whether the free tier's 4-expenses-per-day cap applies to API writes.** The cap is confirmed for accounts; its API applicability is undocumented. If it applies, it is a real constraint and Pro is the fix. **Worth testing empirically before committing to this rail.**
11. **Whether `payment: "true"` is a supported `create_expense` input.** Works in birdie's vendored code; absent from the documented inputs. Could break without notice.
12. **Whether `limit=0` on `get_expenses` is supported.** Documented only for `get_notifications`, yet birdie relies on it for `get_expenses`. Same risk profile.
13. **Splitwise rate limits** — no number exists publicly, no 429 documented. Fine at a few calls a day, but uncommitted and suspendable "without notice".
14. **API key lifetime** — never stated. Rotation is manual; expiry is unknown.
15. **Whether API-created expenses trigger push (not just email)**, and whether notification on write can be suppressed. Undocumented.
16. **How the Terms' "unsolicited direct communication" clause applies** to expense-creation notifications sent to a collector. Probably fine with a consenting collector; worth not building nagging *through* Splitwise.

**Gateways:**

17. **Whether Razorpay activation for an unregistered individual actually completes** without a live business URL. Payment Links documentation says no website is needed; the activation form still collects one and risk review can demand it. A known doc-versus-practice gap, resolvable only by applying.
18. **Whether Razorpay would enable UPI Autopay / Subscriptions for an unregistered individual.** Enablement is discretionary and documented nowhere; this is the step most likely to be refused.
19. **Razorpay settlement timing for a new individual merchant** — pricing page and docs disagree (instant/T+1 vs T+2).
20. **Cashfree, Paytm and PhonePe current-account requirements** — not stated in reachable docs.
21. **PhonePe PG's monthly caps for unregistered accounts** — reported, not confirmed from official docs. Its entity types and document lists are unpublished entirely.

**Remaining gaps:**

22. **Whether Gmail-scraping of bank alerts is workable for a single-user self-hosted app** given restricted scopes and the CASA security assessment. The policy text is confirmed; how it applies to an app with exactly one user, consuming only its own mailbox, is not addressed by the docs. Android SMS is ruled out for a web app regardless.
23. **The additional NPCI deep-link parameters** (`tid`, `mam`, `mode`, `sign`, `orgid`) and the mandatory/optional split — npci.org.in 403s; the eight parameters confirmed above come from Google's India integration guide, not from NPCI.
24. **Whether PSP apps warn or block on `upi://` links constructed for a personal VPA** in practice, beyond the circular-level prohibition. Undocumented; only testable empirically.
25. **Tricount's payment-request availability in India** — feature confirmed, geography not stated, help centre 404s. If Splitwise's constraints bite, Tricount and **Settle Up** (which has a real public Firebase REST API) are the two alternatives worth a closer look.

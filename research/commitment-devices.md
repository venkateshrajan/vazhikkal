# Prior art: commitment devices with financial stakes

Research for [issue #7](https://github.com/venkateshrajan/vazhikkal/issues/7). All claims cite primary sources — Beeminder's own docs/blog/forum, StickK's help center, and the original papers. Where a number comes from a company's self-reported marketing data rather than an audited or peer-reviewed source, it is flagged as such.

## Bottom line

- **Commitment contracts with financial stakes work — for the minority who sign up and keep playing.** Deposit contracts reliably beat controls while active (smoking: +3.3–5.7 pp over an ~8–9% base quit rate; weight: 14.0 lb vs 3.9 lb at 16 weeks; steps: loss-framed incentives were the *only* arm that worked at all). But take-up is the binding constraint (11% in CARES, 13.7% in Halpern's deposit arm vs 90% for equivalent rewards), and **the modal outcome for a deposit-contract taker is losing the money**: 66% of CARES takers forfeited everything.
- **Loss framing beats economically identical gain framing** (Patel 2016: +16 pp of goal-days for loss-framed, gain-framed indistinguishable from control). The dues-ledger premise is well supported.
- **Effects decay within ~3 months of the incentive ending** (Mantzari meta-analysis; Volpp; John; StepUp: only 8% of 53 treatments still worked 4 weeks after ending). Persistence comes from *ongoing* commitment structure (Royer: multi-year effects) or enough forced repetition (Charness–Gneezy). vazhikkal being a permanent standing structure rather than a one-off contract is the right call.
- **Self-report mostly holds — until it doesn't, and then it fails totally.** Lab and field evidence says most people are surprisingly honest even with money on the line (Abeler et al.: people forgo ~3/4 of potential gains from lying), but unverified contracts show systematic over-reporting (Lesser 2018 on StickK data), and Beeminder's operational experience is that **one fudge permanently collapses the device's authority** for that user.
- **The single most dangerous pattern for vazhikkal is punishment-then-quit.** Every operator designs against first-sting attrition (Beeminder's mercy/respite, "paying is not punishment" messaging); the best-performing intervention in a 61,293-person megastudy was *rewarding return after a lapse*. The map already flags the recovery path as unspecified — the evidence says that is where this design lives or dies.

---

## Beeminder

Sources: help.beeminder.com, blog.beeminder.com, beeminder.com/faq, forum.beeminder.com.

### The model

Every goal is a time-series of cumulative data against a **Bright Red Line** (formerly the "Yellow Brick Road"). End the day on the wrong side of the line and you **derail** and pay your pledge ([help: interpreting the graph](https://help.beeminder.com/article/118-how-do-i-interpret-the-graph)). **Safety buffer** is "the number of days you can go without... doing what you committed to before you derail"; colors encode it (green = 3+ days, red = derail tonight) ([help: safety buffer](https://help.beeminder.com/article/55-what-is-safety-buffer)). An **eep day** is a day you start in the red but can still save before the deadline ([glossary](https://blog.beeminder.com/glossary/)). Note the structural similarity to vazhikkal's staleness clock: Beeminder goals have *rates*, not due dates — "do X per week or pay" is exactly a staleness clock with a graph.

### The akrasia horizon

"The timeframe within which your short-term impulses outweigh your better judgment — taken by Beeminder to always be one week" ([glossary](https://blog.beeminder.com/glossary/)). You can make a goal easier or archive it any time, but the change takes effect only **7 days later**; making it harder is immediate ([The Road Dial and the Akrasia Horizon](https://blog.beeminder.com/dial/)). Rationale: beyond ~a week your rational self is back in charge (they cite online-grocery ordering: people ordering for next week buy more vegetables and less ice cream than people ordering for tomorrow). What it defends against: "You always want to make it easier 'just for today' — which the road dial doesn't allow." The design goal is commitment with *minimum loss of flexibility* — free to change your mind for considered reasons, never impulsively ([Akrasia and Self-Binding](https://blog.beeminder.com/akrasia/)).

**This is the single most transferable design idea in this entire research ticket.** Any mutation that weakens a commitment (evicting a slot, swapping a next action for an easier one, lengthening a clock) is safe to allow *if and only if* it takes effect outside the impulse window.

### Pledge escalation

Exact ladder: **$0, $5, $10, $30, $90, $270, $810, $2430, $7290** — no custom amounts ([help: pledges](https://help.beeminder.com/article/20-how-much-do-i-pledge-on-my-goals)). Each derailment charges the current pledge and steps up one rung, up to a user-set **pledge cap** ([pledge caps](https://blog.beeminder.com/pledgecaps/)). Why ×3 exponential: you climb quickly to whatever amount actually motivates you, wasting minimal money on non-motivating rungs — "you never waste more than half of the amount that eventually motivates you" ([faq](https://www.beeminder.com/faq); [Exponential Pledge Schedule](https://blog.beeminder.com/exponential/)). Their help doc warns in both directions: caps too low aren't motivating; pledges "too scary" make you quit the platform rather than risk the amount. Pledge-free goals and jumping straight to high pledges are both paid premium features — they consider pledgeless goals "unbeemindery" ([help: goals without pledges](https://help.beeminder.com/article/23-can-i-have-goals-without-pledges)).

By revenue mix (from their own pie chart in the exponential post): roughly half of derailment revenue comes from $5+$10 pledges; most derailments by count are at $5. Small stakes do most of the work.

### Derailment mechanics

On derailing ([Newbee Corner: derailing](https://blog.beeminder.com/derail/); [help](https://help.beeminder.com/article/17-what-happens-when-i-derail)):

1. **Legit check**: the bot emails "was this derailment legit?" with 24 hours to respond. Non-legit (bug, data error, genuine emergency) → charge cancelled, user gets benefit of the doubt. Their stated standard: "If you had thought to spell it out in your fine print... would you have chosen an exemption for circumstances like these?" ([Weasel-Proofing and the Definition of Legitimacy](https://blog.beeminder.com/legit/)).
2. **Charge + step up** one pledge rung.
3. **Mercy / post-derail respite**: the line resets flat for a configurable number of days (default ~a week) "so that you aren't stuck playing catch-up to a goal you're already behind on" ([General Mercy](https://blog.beeminder.com/mercy/)).

Their framing is deliberate and repeated: **"Paying is not punishment"** ([depunish](https://blog.beeminder.com/depunish/)) — Beeminder "just puts prices on things." **"Derailing is not failing"** ([defail](https://blog.beeminder.com/defail/)) — the real failure modes are quitting, sandbagging the goal, or cheating; and "if you're literally never derailing then it's most likely that your goal isn't pushing you at all" ([Derailing It Is Nailing It](https://blog.beeminder.com/nailingit/)).

### Honesty and self-report

Beeminder's first line of defense is **self-selection**: "If you were the type who would falsify your data to weasel out of paying... you probably would've rolled your eyes and walked away a long time ago. So you could, but you're not going to" ([help: can't you just lie?](https://help.beeminder.com/article/34-cant-you-just-lie-about-your-data)). [Combatting Cheating](https://blog.beeminder.com/cheating/) lists seven reasons users don't cheat: QS pride in accurate data, **autodata integrations** (device data is ground truth), public graphs, honesty-as-identity, value already received, not wanting to destroy the tool's future power over themselves, and personal connection to the founders. Autodata is the main *structural* defense; everything else is psychology.

The opt-in **weaselproofing** flag (stricter proof requirements for non-legit claims) was **killed in 2023** and replaced by **No-Excuses Mode** (no self-serve non-legit claims except for Beeminder bugs, no editing entered data): weaselproofing was barely used, "the implied distrust was kind of icky," and "it just encouraged the wrong mentality" ([Death To Weaselproofing](https://blog.beeminder.com/noexcuses/)).

Their support czar's account of actual weasels ([Weasel Heart-To-Heart](https://blog.beeminder.com/chelsea)) describes the observed pattern — logging pushups never done, entering daily bare minimums, then archiving — and the key dynamic: "Lying to Beeminder is a horrible, slippery slope... Beeminder loses all its power over you, regardless of the pledge level, because you'll think 'well, I can just fudge it.'" A forum user describes the trap from the inside: "I'm in this stupid situation where I keep lying and because there is an option to lie, I don't see myself going back on track" ([forum thread](https://forum.beeminder.com/t/cheating-by-updating-datapoints-on-automatically-synchronized-goals/6062)).

**Not verifiable from primary sources**: any quantified lying rate, average pledge, or churn-after-first-derailment cohort statistic. Beeminder publishes live revenue/usage graphs ([meta dashboard](https://www.beeminder.com/meta), including [derailment revenue](https://www.beeminder.com/meta/derev)) but no retention studies. Their claims that occasional derailers are the users who stick and get value ([defail](https://blog.beeminder.com/defail/), [perverse](https://blog.beeminder.com/perverse/)) are argument and anecdote, not data.

---

## StickK

Sources: stickk.com FAQ, stickk.zendesk.com help center, plus one independent analysis of purchased platform data.

Founded 2007–08 by Yale economists Dean Karlan and Ian Ayres with Jordan Goldberg, directly out of the deposit-contract research below ([StickK — Wikipedia](https://en.wikipedia.org/wiki/StickK)). The product is the **Commitment Contract**: goal + periodic self-report + optional financial stakes + optional referee + optional supporters ([help: how it works](https://stickk.zendesk.com/hc/en-us/articles/206833157-How-it-Works)).

### Referees

The *user* submits progress reports; the referee verifies them. Asymmetric by design: when the user reports **success**, the referee is notified and "may choose to take your word for it or ask you for proof," and can overturn it; when the user reports **failure**, StickK takes their word for it with no arbitration ([referee FAQ](https://www.stickk.com/faq/referees/Commitment+Contracts)). Verification only guards against false success claims. With no referee the contract is literally labeled **"On Your Honor,"** and StickK is candid: "it can be very tempting to report 'success' when only your honor is on the line... Don't choose this option if you think you might be tempted to fudge the truth!"

### Anti-charities

An anti-charity is "an organization whose views you strongly oppose" ([help](https://stickk.zendesk.com/hc/en-us/articles/206833337-What-s-an-Anti-Charity)); StickK offers paired opposites across US/UK politics, abortion, guns, football rivalries. Claimed mechanism: losing money is bad, but funding your enemies is worse — whereas failing toward a charity you like produces a consoling warm glow.

**Success-rate claims — all self-reported platform data, none independently audited:**

- 2008–2011 platform data (published in Tim Ferriss's *4-Hour Chef* [excerpt](https://tim.blog/wp-content/uploads/2016/01/stakes_sample_of_4_hour_chef.pdf)): **33.5% success with no stakes vs 72.8% with stakes**.
- Goldberg, quoted on Wikipedia: money + referee → 78% vs 35% with no money.
- Current marketing: "up to 3x," "300% more likely," anti-charity "630%" ([help: what is stickK](https://stickk.zendesk.com/hc/en-us/articles/206109308-What-is-stickK)) — no published methodology.
- A 2012 Yale Daily News piece reportedly gives 29% / 59% / ~80% (none / referee / anti-charity); the page 403'd and could not be verified.

Given that failure is under-verified and success can be self-declared on honor contracts, all of these numbers are structurally inflatable. Treat as promotional.

### The one independent analysis

[Lesser, Thompson & Luft 2018, *Am J Health Promotion*](https://pmc.ncbi.nlm.nih.gov/articles/PMC5316505/) (DOI: 10.1177/0890117116661157): retrospective analysis of 3,857 purchased, de-identified first-time StickK weight-loss contracts started in 2013 (66–76% female, mean age ~37–40, median contract 14–17 weeks). Deposit contracts reported more weekly weight loss than no-deposit, ordered **anti-charity > charity > friend > none** (all p < .001) — but differences are small fractions of a percent per week, users self-selected into stake types, weight is self-reported, and **"contracts without a weight verification method claimed more weight loss than those with verification"** — direct evidence of systematic over-reporting on unverified money-at-stake contracts. Observational; no RCT of StickK itself exists.

Note the ordering: **a friend as recipient is the weakest financial stake StickK measures.** vazhikkal's collector is a friend-recipient.

---

## The research literature

Every effect size below states population and duration.

### Deadlines and precommitment: Ariely & Wertenbroch 2002

*Psych Science* 13(3):219–224 ([PDF](https://web.mit.edu/ariely/www/MIT/Papers/deadlines.pdf)). Study 1: 99 professionals, 14-week MIT exec-ed course. Given free choice, students **voluntarily self-imposed costly early deadlines** (only 27% put everything on the last day) — people demand commitment. But the section with externally imposed, evenly-spaced deadlines earned higher grades (88.76 vs 85.67, p = .003). Study 2: 60 MIT-community proofreaders, 3 texts, 21 days, paid per error found with lateness penalties: performance ordered **evenly-spaced > self-set > single end deadline** (all p < .01). Takeaway: self-imposed deadlines work but are set suboptimally; external structure beats self-set structure. (vazhikkal's app-imposed uniform clock, rather than user-chosen per-project deadlines, is the right side of this result.)

### Smoking deposits: Giné, Karlan & Zinman 2010 (CARES)

*AEJ: Applied* 2(4):213–235 ([PDF](https://poverty-action.org/sites/default/files/publications/put_your_money_where_your_butt_is.pdf)). RCT, ~2,000 adult smokers, Butuan City, Philippines. CARES: deposit your own money weekly for 6 months; pass a urine cotinine test at 6 months → money back; fail or skip → forfeited to charity. **Take-up 11%** (83 contracts); average commitment ~550 pesos (~US$11, ~20% of a month's income). ITT: **~3 pp higher pass at 6 months, +3.4–5.7 pp at a *surprise* 12-month test** (no money at stake by then, so no fraud incentive) against a control quit rate of ~8–9% — a large relative effect that persisted 6 months past contract end. **The crucial detail: 66% of takers failed the 6-month test and forfeited their entire balance.** Even among the self-selected 11%, losing was the modal outcome; roughly 1 in 6 takers achieved durable cessation.

### Rewards vs deposits head-to-head: Halpern et al. 2015

*NEJM* 372:2108–2117 ([full text](https://pmc.ncbi.nlm.nih.gov/articles/PMC4471993)). 2,538 CVS Caremark employees/relatives, smokers, 6-month biochemically confirmed abstinence. Acceptance: **90.0% for an $800 reward vs 13.7% for a $150-deposit-plus-$650-reward contract** (p < .001). ITT abstinence: rewards ~15.4–16.0%, deposits 9.4–10.9%, usual care 6.0%. But among people who would have accepted either, **deposits beat rewards by 13.2 pp** (CI 3.1–22.8). Deposits are the stronger medicine almost nobody will voluntarily swallow.

### Gym commitment contracts: Royer, Stehr & Sydnor 2015

*AEJ: Applied* 7(3):51–84 ([NBER WP](https://www.nber.org/system/files/working_papers/w18580/revisions/w18580.rev0.pdf)). N = 1,000 Fortune-500 employees. Phase 1: $10/gym visit for 4 weeks (roughly doubled attendance). Phase 2: half the incentive group offered a **self-funded commitment contract — pledge any amount not to go more than 14 days without a visit** over 8 weeks; failure forfeits to charity. Take-up ~23% among prior gym members, 6% among non-members. Incentive-only effects faded post-treatment (+0.11 visits/week, "modest at best"); the commitment-contract arm showed substantially larger and *durable* effects — detectable "even several years after" per the published abstract (~2.5–3 years of login data). **A standing self-funded "don't lapse more than 14 days" contract is nearly isomorphic to vazhikkal's staleness clock, and it is the best persistence result in this literature.**

### Groceries: Schwartz et al. 2014

*Psych Science* 25(2):538–546 (DOI: 10.1177/0956797613510950). Discovery Vitality members, South Africa (25% healthy-food cash-back). Binding precommitment: raise healthy share 5 pp above baseline each month for 6 months or forfeit that month's entire discount. **36% of offered households agreed** (arm-level Ns not verifiable from the paywalled text — flagged). Committers: **+3.5 pp healthy purchases in each of 6 months**; decliners and hypothetical-commitment controls: no change. The 3.5 pp average against a 5 pp pledge implies frequent monthly forfeits.

### Loss vs gain framing

- **Volpp et al. 2008**, *JAMA* 300(22):2631–2637: N = 57 obese adults, 16 weeks. Deposit contract 14.0 lb, lottery 13.1 lb, control 3.9 lb. **At 7 months (~3 months post-incentive) differences were no longer significant.**
- **John et al. 2011**, *J Gen Intern Med* 26(6):621–626: 32-week deposit contract, VA patients. 8.70 lb vs 1.17 lb control (p = .04); **36 weeks post-intervention the difference was gone** (p = .76).
- **Patel et al. 2016**, *Annals of Internal Medicine* 164(6):385–394: N = 281 university employees, 13 weeks, 7,000-step daily goal. Goal-day share: control 0.30; gain-framed $1.40/day 0.35 (n.s.); lottery 0.36 (n.s.); **loss-framed (money pre-allocated, removed on failure) 0.45 — +0.16 adjusted (CI 0.06–0.26, p = .001), the only arm that worked**, at identical expected value. All arms fell back to control during 13-week follow-up.

Cleanest available evidence that the deposit/dues mechanic — pre-committed money removed on failure — beats economically identical rewards.

### Habit formation and persistence

- **Charness & Gneezy 2009**, *Econometrica* 77(3):909–931 ([PDF](https://rady.ucsd.edu/_files/faculty-research/uri-gneezy/incentives-exercise.pdf)): 120 U. Chicago + 168 UCSD students paid to attend the gym 8 times in a month. Post-incentive attendance in the 8-visit group **more than doubled and did not decline over 7–13 follow-up weeks**, entirely driven by previous non-attenders — a repetition threshold can start a habit. Student samples, short horizon.
- **Milkman, Minson & Volpp 2014** (temptation bundling), *Management Science* 60(2):283–299: N = 226 gym members, ~9 weeks; gym-only audiobooks → **+51% attendance**, but effects decayed and **largely collapsed after the Thanksgiving break** — one routine interruption killed the habit. 61% would pay for the commitment device.
- **StepUp megastudy** — Milkman et al. 2021, *Nature* 600:478–483 ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC8822539/)): **N = 61,293** 24 Hour Fitness members, 54 four-week programs. 45% of treatments worked during the program; **only 8% still showed effects four weeks after it ended**. The single best treatment of all 53: **a $0.09 bonus for returning to the gym after a missed workout** (+0.40 visits/week). Rewarding recovery-after-lapse beat every punishment- and reward-sized alternative tested.
- **Mantzari et al. 2015**, *Preventive Medicine* 75:75–85 (meta-analysis, 34 trials): incentives change behaviour while running and remain significant **up to ~2–3 months after removal**, then "effects dissipate beyond three months post-incentive removal." Larger effects in more deprived groups.
- **Giles et al. 2014**, *PLOS ONE* (meta-analysis, 16 RCTs): incentives roughly double short-term smoking cessation (RR 2.48 ≤6 months), but meta-regression found **limited evidence that bigger incentives buy proportionally bigger effects** — prompt delivery and objective outcome tracking matter more than raw size.

### How systems handle unverifiable completion

- **Beeminder**: honor system + self-selection + autodata as ground truth + legit-check with benefit of the doubt; opt-in strictness (now No-Excuses Mode). Disputed derailments refunded essentially no-questions-asked.
- **StickK**: optional referees who arbitrate *success claims only*; honor contracts explicitly warned as fudge-prone.
- **Forfeit** ([forfeit.app](https://forfeit.app/)): the hard-verification pole — live photo/video/screenshot evidence, checked "either [by] AI or a person on our team" before any charge; autodata (Apple Health, GPS geofence, Screen Time). Claims 94.2% success across $8.7M staked (marketing, self-reported — flagged).
- **TaskRatchet** ([taskratchet.com](https://taskratchet.com/)): micro-stakes on one-off tasks, explicit honor system ("Did the work but forgot to check it off? Just let us know and we won't charge you").
- **Focusmate**: verification by synchronous human presence (booked video co-working) — social stakes substitute for money.

### What people do when lying pays

- **Mazar, Amir & Ariely 2008**, *J. Marketing Research* 45:633–644: people cheat "enough to profit but honestly enough to delude themselves of their own integrity" — the fudge factor. (Caveat: the famous moral-reminder *remedy* from Exp. 1 failed a 25-lab registered replication, Verschuere et al. 2018; the partial-cheating pattern itself is robust.)
- **Fischbacher & Föllmi-Heusi 2013**, *JEEA* 11(3):525–547 (die-under-cup, individually undetectable lying): ~20% lied maximally, ~39% fully honest, a large middle cheated only partially.
- **Abeler, Nosenzo & Raymond 2019**, *Econometrica* 87(4):1115–1153 — meta-analysis of 90 studies, 44,000+ participants, 47 countries: **"people lie surprisingly little," forgoing on average about three-quarters of the potential gains from lying**, robust to a 500-fold payoff increase. Drivers: wanting to be honest and to be *seen as* honest.

Prediction for a self-reported stake: most reports will be honest, including under money pressure — but the marginal lie arrives at exactly the high-temptation moment the device exists for, lying is partial and self-rationalizable ("I basically did it"), and per Beeminder's field experience, the first fudge ends the device's power permanently.

### Why demand for commitment is low (reviews)

- **Bryan, Karlan & Nelson 2010**, *Annual Review of Economics* 2:671–698: distinguishes **hard** commitments (real economic penalties) from **soft** (psychological consequences); soft often dominates in practice. Key warning: "partially naïve agents may undercommit their future selves, making commitments on which they do not follow through. It is therefore possible to exploit partially naïve agents by charging them for the chance to undercommit" — a stakes system can profit from predictable failure while helping no one.
- **Laibson 2015**, *AER P&P* 105(5):267–272: in a calibrated model, commitment's perceived benefits are "often overwhelmed by the costs" (lost flexibility under uncertainty, partial naivete) — low take-up is the rational general case.
- **Rogers, Milkman & Volpp 2014**, *JAMA* 311(20):2065–2066: commitment devices work when adopted; voluntary enrollment is the bottleneck.

---

## What this says against the vazhikkal design

The adversarial reading. Each point is what the evidence above predicts will go wrong with the machine as chartered.

### 1. The staleness clock punishes lapses, and lapse-recovery is exactly where the evidence says these systems die

The clock's failure event fires precisely when the user has stopped acting — i.e., during a lapse. The literature is unanimous that the lapse moment is the fragile one: Milkman's temptation-bundling effect **collapsed after a single Thanksgiving break**; the best of 53 megastudy treatments was a token *reward for returning after a missed workout*, not any punishment; Beeminder ships mercy/respite specifically "so that you aren't stuck playing catch-up," and admits stings that feel excessive make users "quit or make your goal too easy." vazhikkal as chartered does the opposite: a lapse triggers a fee *and* an immediate demand to name a fresh next action, and a week away triggers up to N fees at once. The map already lists the recovery path as unspecified; the evidence says it is not an edge case to bolt on later — multi-slot simultaneous expiry after a lapse is the single most likely quit-forever event, and there is currently no mercy, no breaks mechanism, and no cap on total exposure per week.

### 2. Deposit-contract base rates say the user should expect to lose, often

The chartered premise is that the user resembles the people these devices fail: two prior systems became graveyards. In CARES, among the self-selected 11% keenest committers, **66% forfeited everything**; in Schwartz et al., average attainment (3.5 pp) ran below the pledge (5 pp), implying routine monthly forfeits. A staleness clock across N slots, running forever, is dozens of contract-expiry events per year. If the miss rate resembles the literature's, the dues ledger will accumulate steadily — and Bryan/Karlan/Nelson's warning applies in miniature: a system can extract payment for predictable failure while changing nothing. The design needs an explicit answer to "what does a sustainable miss rate look like, and what happens when the real rate is double that" — Beeminder's answer is the exponential ladder plus caps plus "derailing is not failing" framing; vazhikkal currently has a flat unspecified amount.

### 3. Self-report with no autodata, no audience, and self-defined success criteria is the weakest verification environment in this survey

Beeminder's seven anti-cheating defenses are mostly unavailable here: no autodata ground truth (next actions are arbitrary personal tasks), no public graphs, no QS community identity, no third-party company relationship worth preserving. StickK's referee only exists if added, and even StickK arbitrates only *success* claims. Worse, vazhikkal's user defines both the task and its completion criterion, so the cheap lie isn't even a lie — see point 4. The honest-majority finding (Abeler) cuts in vazhikkal's favor for an honest single user, but the Beeminder field evidence is that the failure is not gradual: "Beeminder loses all its power over you, regardless of the pledge level, because you'll think 'well, I can just fudge it.'" One fudge under one high-pressure clock expiry, and every future clock is theater. Lesser 2018 shows the direction of drift on real money-at-stake contracts: unverified ones systematically over-report. A design that "if self-report survives, proof evaporates" (per the map) should note that no surveyed system with real stakes runs on *pure* unwitnessed self-report — Beeminder has autodata and a company, StickK has referees, Forfeit has evidence review, TaskRatchet's stakes are trivial. The honor-system pole of the spectrum is occupied by the systems with the least money at stake.

### 4. Goodhart's law eats the next action before it eats the truth

The user never needs to lie: they can define next actions that are trivially completable ("open the document"), complete them honestly every N days, and keep every slot alive forever while the projects go nowhere. This is Beeminder's documented sandbagging failure — "make your goal too easy," which [defail](https://blog.beeminder.com/defail/) ranks alongside quitting and cheating as the *real* failures — except vazhikkal makes it easier, because the unit of work is freshly self-defined at every completion rather than fixed at commitment time. The graveyard returns wearing a clock: N slots, all green, nothing moving. Nothing in the chartered machine constrains next-action size, and the evidence (Ariely & Wertenbroch: self-set structure is reliably suboptimal vs imposed structure) says the user's own in-the-moment choices are exactly what can't be trusted.

### 5. Eviction is an un-priced escape hatch with no akrasia horizon

Beeminder's central defensive mechanism — the 7-day delay on anything that makes a commitment easier — has no counterpart in the chartered machine. As specified, the user facing a clock about to expire tonight can evict the project *now*, dodging the fee; "the eviction is a recorded decision" is a soft consequence (psychological, in Bryan/Karlan/Nelson's terms) guarding the exit of a hard-consequence system. Every mutation that relieves pressure — eviction, swapping in an easier next action, re-promoting later — is currently instantaneous. Beeminder's entire experience says the impulse-window version of you will use every one of these doors, and the fix (changes take effect after a delay; or eviction inside the danger window costs the miss fee) is well-charted prior art.

### 6. A single human collector is a soft stake wearing hard-stake clothes, and the weakest recipient type measured

Three compounding problems. (a) **Crediting is a ledger entry, not a transfer** (the map admits this): until money moves automatically, this is a soft commitment — the actual consequence at expiry time is a number changing in an app the user controls. Beeminder charges a card; CARES had the bank keep the deposit. Prior art strongly suggests the bite requires an automatic rail. (b) **A friend is the weakest stake recipient in the only ordered data available** (Lesser 2018: anti-charity > charity > friend > none) — paying someone you like produces the consoling warm glow anti-charities exist to remove, and small transfers to a friend can even feel like generosity. (c) **The collector is a repeated-game counterparty**: a real person can forgive, waive, feel awkward collecting, or be avoided — and the social awkwardness of a growing unpaid ledger creates a *second* incentive to fudge completions, on top of the money. StickK's referee is deliberately *not* the beneficiary; vazhikkal merges the verifier-shaped hole and the beneficiary into one person who has an interest in the user's misses. None of the surveyed systems put the stake in the hands of a single friendly human, and the ones that studied recipients found friends weakest.

### 7. The hard cap creates inbox rot and adverse selection into the slots

The cap kills the *visible* graveyard, but the evidence on self-set structure predicts the user will curate slots by comfort: projects with easy, clock-satisfiable next actions get slots; the frightening ambitious project stays in the "never guilt-inducing," never-reviewed inbox indefinitely. That is the original graveyard failure relocated to a place the design has explicitly promised never to surface. (The map's open "inbox triage" question is where this lands.) Meanwhile Halpern/CARES take-up numbers (13.7%, 11%) are a warning about the psychology of *entering* a slot at all once slots carry real money risk: the rational move under a stake is to commit less, and the eviction requirement raises the price of entry further. A commitment device with a too-scary entrance fee doesn't get gamed — it gets ignored, which for a single-user app means quiet abandonment.

### 8. Flat stakes are mis-sized in both directions at once

No amount is right forever: Beeminder's ladder exists because non-motivating amounts waste money and sting-only-later, while "too scary" amounts cause platform-quit; Giles et al. found little evidence that raw incentive size buys proportional effect. A flat fee will start either too small to outbite the inbox's comfort or large enough that the first multi-slot lapse (point 1) is a rage-quit event. Escalation is on the map's open list — the prior art answer (exponential ladder with a user-set cap, reached within ~2 misses of the motivating amount) is well tested by Beeminder's revenue-by-rung data.

---

## Open questions raised by the evidence

1. **The recovery path is the design's survival question, not a detail.** What happens after a week/month away — mercy days, a cap on simultaneous misses, an explicit "return" mechanic? StepUp says reward the return; Beeminder says flatten the line. Both say never demand immediate catch-up.
2. **Does eviction (and any pressure-relieving change) get an akrasia horizon or a price?** E.g., eviction takes effect in 7 days, or eviction within the clock's red zone costs the miss fee.
3. **What bounds next-action size?** Anything from a required "how does this move the project" field to periodic slot-level progress review. Without something, sandbagging is free and invisible.
4. **What makes the stake hard rather than soft?** An automatic settlement rail (the birdie Splitwise rail is prior art in-house) vs manual collection; and is the collector the right recipient at all, given friend-recipients measured weakest? An anti-charity-style recipient is the evidence-backed strongest, at real relationship cost.
5. **Escalation:** adopt a Beeminder-style ×3 ladder with a cap, or stay flat? The evidence favors a ladder.
6. **Honesty design:** legit-check with benefit of the doubt (Beeminder's current posture, cheap and dignity-preserving) vs any verification. If pure self-report stands, the design should at least make dishonesty *effortful* (immutable completion log, timestamped, visible to the collector's read-only view).
7. **Miss-rate accounting:** decide up front what an acceptable annual dues total looks like, so the app can distinguish "the stake is working" from "the user is funding predictable failure" (Bryan/Karlan/Nelson's exploitation case) — and surface that distinction.

## Verification flags

- StickK's 72.8%/78%/"630%" success figures and Forfeit's 94.2% are company self-reported marketing; the 2012 Yale Daily News breakdown (29/59/80%) could not be fetched (403).
- No published Beeminder churn-after-first-derailment or StickK non-renewal statistics exist; the attrition argument rests on operator admissions plus CARES forfeiture data.
- Schwartz et al. arm-level Ns and forfeiture frequency are behind the paywall; the 36% take-up and 3.5 pp effect are from the abstract.
- Abeler et al.'s "three-quarters of gains forgone" figure is from the working-paper text, not the paywalled final Econometrica version.
- Mazar et al.'s moral-reminder manipulation failed a registered replication; only the partial-cheating pattern is treated as solid here.
- Royer et al.'s "several years" persistence wording is verified from the AEA abstract; the underlying working paper covers ~1 year of follow-up data.

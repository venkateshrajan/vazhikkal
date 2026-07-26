# Prior art: next-action size gaming, WIP limits, staleness, and honesty in procrastination apps

Research for [#8](https://github.com/venkateshrajan/vazhikkal/issues/8). Companion to
[#7 commitment devices](https://github.com/venkateshrajan/vazhikkal/blob/research/commitment-devices/research/commitment-devices.md)
and [#6 settlement rails](https://github.com/venkateshrajan/vazhikkal/blob/research/settlement-rails/research/settlement-rails.md).
#7's commitment-device evidence is not re-derived here; this doc is about what *shipped tools* do.

Sections are ordered by decision value. Official/marketing claims are separated from user reports and from
peer-reviewed evidence throughout, and **documented absences** are reported as findings rather than gaps.

---

## Bottom line

1. **Next-action size gaming is real, named, and unsolved by every shipped tool.** The defence in the wild is only
   ever one of three things: measure something the user cannot author (autodata), have a human referee, or accept it.
   **No tool bounds the size of a self-defined task.** Beeminder — the most thought-through system here — names
   "setting unrealistically easy targets" as one of exactly three ways users defeat it, and its whole rule apparatus
   (7-day akrasia horizon, easier-only-with-notice) exists to govern that channel rather than to detect it.
2. **The gaming is documented in users' own words, and its trigger is a short clock.** A Beeminder user on gym time:
   *"due to how small the time period was, I found it way too easy to just do some small exercises instead of a real
   workout."* Same forum, on staking money per task: the risk is *"sandbagging by picking tasks with mushy end
   criteria."* And on time-based study goals: *"I'm measuring studying time, not effort."* Mushiness of the
   done-criterion, not size, is the actual lever.
3. **There is a citation for whether a small tick creates momentum or licenses coasting, and it says the answer is
   framing, not size.** Fishbach, Dhar & Zhang 2006: focusing on *success on a subgoal* makes further action feel
   like a **substitute** (less follow-through); focusing on *commitment to the superordinate goal* makes it feel like
   a **complement** (more). "Done — you're safe for N days" is literally the substitute frame.
4. **Amazing Marvin does not ship forced eviction. Verified firsthand — the previous belief was wrong.**
   Marvin has no WIP limits at all; its kanban is a label-group workaround with no column caps, and its
   anti-overwhelm feature *hides* tasks rather than evicting them. More broadly: **hard caps with forced eviction
   essentially do not ship.** Practitioner caps cluster at 2–3 concurrent, every surviving product uses soft caps
   (warnings, expiry, badges), and the strict-cap apps that do exist have almost no users.
5. **Nothing replaces due dates entirely with an activity clock.** The pieces exist separately — Sunsama's 4-day
   auto-archive, Marvin's staleness badge, Intend's daily expiry, OmniFocus review intervals, Taskwarrior's age
   coefficient — but the full synthesis is unshipped. Age-based *surfacing* is well tolerated; age-based
   *destruction* is not, and it reliably breeds clock-reset rituals. The inventor of hard dismissal (Mark Forster)
   softened the rule and then dropped it.
6. **Automatic decomposition has no controlled evidence, and its shipped form works by lowering restart cost rather
   than by being right.** Self-generated subgoals and implementation intentions do have strong evidence (d = 0.65).
   The active ingredient is *authorship*, which is why every credible tool proposes-and-lets-you-curate.
7. **Bet 2 is the best-evidenced of the three bets and also the sharpest edge in the design**, because a mandatory
   next action is exactly what makes commitment possible *and* what makes sandbagging free once money rides on it.

---

## 1. Next-action size gaming / sandbagging

The ticket's highest-priority question. Answer: it is a named failure channel wherever stakes exist, it is
undocumented where stakes don't, and nobody has shipped a mechanism that bounds action size.

### 1.1 Where stakes exist, size-gaming appears and gets named

Beeminder names its own three leak channels: **"quitting goals, setting unrealistically easy targets, or cheating"**
([blog.beeminder.com/defail](https://blog.beeminder.com/defail/)). Easiness-gaming is listed as a peer of outright
cheating, not a footnote to it.

Their entire response is **friction on easing, not detection of size**:

- The **akrasia horizon**: any change that makes a goal *easier* — shallower rate, lower pledge, quitting — takes
  effect only 7 days out; making it *harder* is instant
  ([help.beeminder.com](https://help.beeminder.com/article/45-what-is-the-akrasia-horizon);
  [blog.beeminder.com/dial](https://blog.beeminder.com/dial/)). Easing is thereby decided by the calm week-ahead
  self; tonight's panicked self can only pay or lie.
- **No-Excuses Mode** closes the exits rather than policing the work: you cannot call a derailment non-legit unless
  Beeminder had a bug, autodata loopholes are shut, and goals cannot be deleted even in the first week
  ([blog.beeminder.com/noexcuses](https://blog.beeminder.com/noexcuses/)).
- Its predecessor, **weaselproofing**, tried to adjudicate legitimacy with user-submitted proof (doctor's notes,
  screenshots) and was retired: it implied distrust, cost them money to process, and "encouraged the wrong
  mentality" ([blog.beeminder.com/legit](https://blog.beeminder.com/legit/), noexcuses). **A commitment-device
  company spent a decade on this and concluded that policing the excuse was not worth it; making the line brighter
  was.**

Their legitimacy test is worth stealing verbatim as a design heuristic: *"If you had thought to spell it out in your
fine print for the commitment contract, would you have chosen to have an exemption for circumstances like these?"*

### 1.2 Real user reports

From the public Beeminder forum:

- **Short clocks invite token actions.** A user beeminding gym time: *"For gym time due to how small the time period
  was, I found it way too easy to just do some small exercises instead of a real workout."* The single most directly
  transferable finding in this doc, and it lands squarely on bet 3.
- **The lever is mushy done-criteria, not smallness.** From a thread on staking money per pomodoro: the concern is
  *"sandbagging by picking tasks with mushy end criteria."*
- **Time-based metrics degrade into presence.** A student beeminding calculus study reported staying above the line
  while *"getting very little done in terms of actual work"*, and diagnosed it exactly: *"I'm measuring studying
  time, not effort"*
  ([forum thread 2104](https://forum.beeminder.com/t/beeminding-studying-input-or-output-measurement-incentives-for-focusing/2104)).
  Community remedies in that thread, none of which are product features:
  - **Make the unit small enough that a bad unit doesn't count** — drop pomodoros from 25 to 5 minutes so an
    off-task session simply fails to register rather than being reset or salvaged. Counter-intuitive but important:
    *smaller all-or-nothing units reduce gaming*, because there is nothing to half-do.
  - **Track attempts and outcomes as two separate goals**, with the higher volume requirement on attempts and
    outcomes used diagnostically rather than as a threshold.
- **Bar-setting is where the gaming happens, not reporting.** On DietBet/StepBet: *"It does seem like you ought to
  game it by sandbagging your Fitbit stats before enrolling to make the 'challenge' easy for you."* On an Anki
  integration: *"extremely sandbag the first few weeks until I see what the pace of review is."*
- **Users actively hunt for un-gameable configurations** — asking for a *"cheat-proof way of getting in steps"*
  because *"the temptation to skip out [is] too big if there's a way to add activity manually"*
  ([forum 11529](https://forum.beeminder.com/t/cheat-proof-way-of-getting-in-steps-or-activity/11529)). The user
  wants the system to be harder to satisfy than they can make it themselves.
- **Weaseling starts small and the first lie is terminal.** Beeminder's own support lead published a confession of
  entering fake data ([blog.beeminder.com/chelsea](https://blog.beeminder.com/chelsea)). Two lessons: the on-ramp is
  logging data *before* doing the work and then not doing it; and after the first fudge *"Beeminder loses all its
  power over you… because you'll think 'well, I can just fudge it.'"* **Verification is not about catching a
  cheater — it protects the device's credibility to its own user.**

### 1.3 Where stakes don't exist, size-gaming isn't reported at all

**Documented absence:** no reports were found of users defining trivially small steps to satisfy Goblin Tools,
Todoist, or Marvin. None of them attach a reward or penalty to a step, so there is nothing to game. This matters:
the decomposition tools are *not* prior art for this problem, because vazhikkal is the first design in the survey to
attach money directly to next-action completion. Structurally it is Beeminder's "unrealistically easy targets"
channel with no autodata and no referee.

### 1.4 The 20-year-old unsolved version of the same problem: GTD granularity

Vazhikkal's mandatory next action descends from GTD, and **GTD has never specified a size**. Official guidance links
projects to next actions and organises them by context, and states no granularity standard
([gettingthingsdone.com](https://gettingthingsdone.com/2020/06/the-gtd-approach-to-linking-next-actions-and-projects/)).
The community's most-reported failure is exactly this. HN, on GTD threads: *"I could never figure out the level of
granularity for the next action"* — *"is the next action 'go to the studio', or is it 'turn on the computer', or is
it 'mix the song'"*; and *"I don't think he addresses granularity well."*

From the official GTD forum, the community consensus is **one next action at a time, and pre-loading all steps is
actively harmful**: *"Planning too far ahead can waste my time planning down the wrong path."* / *"I used to add
long lists of actions to projects, and the result was that I spent most of my review adjusting those lists as reality
changed."* / *"Next Actions on Projects are simply lures… I only need one such Next Action."*
([forum thread 17514](https://forum.gettingthingsdone.com/threads/do-you-set-all-next-actions-when-project-planning-or-just-the-next-action.17514/))

Two implications: vazhikkal's one-next-action-per-slot is mainstream practice in its own lineage, **and** the
anti-pattern that lineage warns about is precisely the full decomposition that AI tools produce (§4).

### 1.5 The mechanism has a citation, and it points at framing

- **Fishbach, Dhar & Zhang 2006** (JPSP 91(2), 232–242, four studies,
  [PubMed](https://pubmed.ncbi.nlm.nih.gov/16881761/)): *"when people consider success on a single subgoal,
  additional actions toward achieving a superordinate goal are seen as **substitutes** and are less likely to be
  pursued. In contrast, when people consider their commitment to a superordinate goal on the basis of initial success
  on a subgoal, additional actions… may seem to be **complementary** and more likely to be pursued."*
  → Whether ticking a small next action produces momentum or licenses coasting is decided by **framing**, not size.
  A post-completion message of "done — you're safe for N days" is the substitute frame and predicts *less* further
  work. "This is the 4th action on this project" is the commitment frame and predicts more. This is the cheapest
  high-leverage design lever found in the entire survey.
- **Ariely & Wertenbroch 2002** ([PDF](https://web.mit.edu/ariely/www/MIT/Papers/deadlines.pdf)): people will
  self-impose costly deadlines and they do help — but people **set them suboptimally**, and externally scaffolded,
  evenly spaced deadlines beat self-set ones. Expect the user's self-set bar to be systematically worse than a
  structurally imposed one.
- Organisational sandbagging literature adds the two standard countermeasures, **neither of which is available to a
  single-user app**: decouple the target from the penalty (committed vs. aspirational OKRs), and separate who sets
  the bar from who judges it. What remains is the akrasia-horizon family: separate *when* the bar is set from *when*
  it is judged.

### 1.6 Adjacent shipped gaming patterns worth naming

- **Post-hoc logging.** A long-term Complice user entered tasks *after* completing them, because
  entered-but-unfinished tasks were aversive — then quit over the rigidity
  ([vkrakovna.wordpress.com](https://vkrakovna.wordpress.com/2015/07/26/systems-i-have-tried-an-overview/)). The best
  single user report in this survey of commitment rigidity causing both gaming *and* churn.
- **Clock-reset rituals.** GitHub stale bots produced "not stale" comments posted purely to reset the timer — the
  cheapest possible action that satisfies an age clock, invented spontaneously and universally. This is the
  sandbagged next action in a different costume (§3.4).
- **Off-board work** to dodge a kanban cap, and quietly raising the limit
  ([Atlassian](https://www.atlassian.com/agile/kanban/wip-limits)).

### 1.7 The mechanism space, ranked

| Defence | Shipped in | Cost / limit |
| --- | --- | --- |
| Autodata the user cannot author | Beeminder integrations | Needs a sensor. "Wrote a paragraph of the hard thing" has none. |
| Live proof + human/AI review | Forfeit (photo/video verification) | Real friction; Forfeit also keeps the money, which is its own problem. |
| Human referee | stickK (optional) | Its own FAQ concedes the honour system is *"very tempting"* to abuse ([stickk.com/faq](https://www.stickk.com/faq/reporting/Commitment+Contracts)). |
| Friction on easing (akrasia horizon) | Beeminder | The most transferable idea available. Does not bound size, only *when* size can shrink. |
| Framing the tick as commitment, not completion | nobody, explicitly | Free to implement; grounded in Fishbach 2006. |
| Bound time instead of output | Focusmate, pomodoro tools | Converts size-gaming into presence-faking (§5). |
| Nothing | almost every task manager | vazhikkal's default if unaddressed. |

---

## 2. WIP limits and forced eviction

### 2.1 Amazing Marvin does **not** ship forced eviction — verified firsthand

- Marvin's **Strategies** catalogue (its full, several-hundred-entry list of opt-in behaviours) contains **no**
  strategy for WIP limits, task limits, daily caps, or forced removal
  ([help.amazingmarvin.com/en/collections/1139197-strategies](https://help.amazingmarvin.com/en/collections/1139197-strategies)).
- Its **features** page lists "The Funnel" (*"from all tasks to one task"*), "Focus Mode" (*"work on one task at a
  time"*), "Task Breakdown" (*"split big tasks into small steps"*) and "Anti-Overwhelm" (*"hide tasks to reduce
  anxiety"*) ([amazingmarvin.com/features](https://amazingmarvin.com/features/)). Every one of these is a **display
  filter**. Marvin's anti-overwhelm mechanism is *hiding*, not *evicting*.
- Its kanban is a **label-group workaround** — create an exclusive label group (open / in progress / done), group by
  it, switch to horizontal layout. No column capacity, no blocking. Marvin's own help centre: *"Marvin has some
  Kanban support already. But we still have major plans to extend our board/Kanban functionality"*
  ([help article 3433008](https://help.amazingmarvin.com/en/articles/3433008-how-to-set-up-a-kanban-board)).

**The load-bearing claim was false.** Marvin remains highly relevant to §4 and §6 and is irrelevant as
forced-eviction prior art.

### 2.2 The cap sizes actually used

The cluster is tight, and everything in it is either advisory or set well above the recommended norm:

| System | Cap | Nature |
| --- | --- | --- |
| Personal Kanban (Benson) | start at **3** WIP | heuristic to tune, explicitly not enforced; emergency override allowed ([personalkanban.com](https://www.personalkanban.com/pk/uncategorized/how-to-setting-your-personal-wip-limit)) |
| Complice / Intend | **10 goals hard max**, 3–6 typical, 3–5 recommended | the only sustained product with a true structural cap ([intend.do/philosophy](https://intend.do/philosophy)) |
| Streaks (iOS) | **24 tasks**, hard, no paid escape | closest shipped analogue to slots ([streaksapp.com](https://streaksapp.com/)) |
| Ivy Lee method | 6 tasks/day | legend-grade evidence only ([jamesclear.com/ivy-lee](https://jamesclear.com/ivy-lee)) |
| 1-3-5 rule | 9/day, structured | invented editorial heuristic; users shrink it rather than abandon it ([The Muse](https://www.themuse.com/advice/a-better-todo-list-the-135-rule)) |
| Clark & Wheelwright 1993 | **2** concurrent projects | measured productivity peak; value-add falls *below* the one-project level at 3+ ([summary](https://www.informit.com/articles/article.aspx?p=1409779&seqNum=4)) |

Note the shipped pattern: **norm low, hard-stop high, expire instead of block.** Intend's only hard cap (10) sits
well above its own recommendation (3–5) — a guardrail, not a daily-felt gate.

### 2.3 What happens when users hit the cap

In descending order of health:

1. **Finish-first works when the limit is self-chosen.** An engineer on HN reported "stop starting, start finishing"
   made them faster despite CI waits ([HN 19186456](https://news.ycombinator.com/item?id=19186456)).
2. **Gaming**: work done off-board with no card; expedite-lane inflation (so bad at team level that practitioners now
   argue for deleting the lane, [Nave](https://getnave.com/blog/expedite-swimlane/)); quietly raising the number
   ([Atlassian](https://www.atlassian.com/agile/kanban/wip-limits)); and post-hoc logging (§1.6).
3. **Abandonment**, when the cap is coupled with overhead.

Pawel Brodzinski's practitioner thesis is the best synthesis and is bad news for imposed caps: **imposed limits
generate resistance and people route around them; internalized pull-rules achieve the same WIP reduction with a
fraction of the resistance** ([brodzinski.com](https://brodzinski.com/2015/10/dont-limit-wip.html)).

Businessmap's own WIP-limits article opens with *"WIP limits are very simple to understand but very complicated for
most people to follow,"* and its central anecdote is a management team that finished 10 projects in a year while
starting *"between two and three times more than we finished."* Their causal claim reads directly onto a single
user: *"when you have many things to do, you start feeling anxious about each of them. Then, you want to start as
soon as you can each of them to feel a relief"*
([businessmap.io](https://businessmap.io/blog/truth-about-wip-limits)).

### 2.4 Evidence status and market signal

- **Mechanism: solid.** Leroy 2009 attention residue — switching away from an *unfinished* task measurably degrades
  performance on the next one ([OBHDP 109:168–181](https://ideas.repec.org/a/eee/jobhdp/v109y2009i2p168-181.html)).
  Cite Leroy and the Ovsiankina resumption effect, **not Zeigarnik** — the classic Zeigarnik memory claim did not
  survive a 2025 meta-analysis while Ovsiankina did
  ([Nature HSSC](https://www.nature.com/articles/s41599-025-05000-w)).
- **Outcomes: absent.** The only substantial empirical WIP study (SINTEF, 8,000+ items over 4 years) states flatly
  that **no published real-case study identifies an optimal WIP limit**, even for teams
  ([ACM](https://dl.acm.org/doi/10.1145/3239235.3239238)). For individuals it is entirely testimonial.
- **Market signal is brutal.** Strict-hard-cap apps are vanishingly small — "TODO 3", a 3-task-cap iOS app, has 3
  ratings ([App Store](https://apps.apple.com/us/app/todo-3-only-3-tasks-per-day/id1627396349)). Team kanban tools
  advertise "soft/hard limits with visual indicators", and the standing question *"kanban tools that allow hard WIP
  limits"* is itself evidence that genuine blocking is the exception. Streaks' 24-cap is presented with **no stated
  rationale**, and the most visible community reaction is people asking why the limit exists and requesting more.

### 2.5 Implications for a *forced* cap

- The industry's own diagnosis of why advisory limits fail — **exceeding them is costless, and starting things
  relieves anxiety** — is a direct argument *for* forced eviction. Vazhikkal is genuinely improving on the shipped
  state of the art here.
- But Brodzinski's finding is the counterweight: imposed caps get routed around. In a single-user app the only
  available route around is eviction — so **eviction is the gaming surface**. "A cap you cannot exceed" plus
  "eviction you can perform at will" is not a cap; it is a revolving door, and it doubles as a way to dodge
  tonight's fee (#7's un-priced escape hatch).
- A hard cap generates *support tickets* in commercial products because users read it as a missing feature. A
  single-user app is spared the tickets but not the feeling. **The cap has to be legible as the point, repeatedly,
  not just enforced once.**

---

## 3. Staleness and decay instead of due dates

### 3.1 Names and lineage

No consensus term. **"Dismissal"** (Mark Forster's Autofocus, 2009 — the deepest practitioner literature),
**"staleness"** (Amazing Marvin; GitHub stale bots), **"auto-archive"** (Sunsama), "ephemerality / forgetting by
design" (HCI), "to-do-list bankruptcy" (the manual one-shot version). Taskwarrior's `urgency` formula carries an
explicit **age coefficient** — the oldest production implementation of age-as-priority, and it only ever *sorts*.

### 3.2 Why the family exists: the case against due dates

Consistent multi-source critique that matches the map's account of why Reminders and To Do failed. Fake self-set
deadlines produce **cry-wolf desensitization** — *"you just start ignoring those fake ones, and then ignore all
deadlines"* ([Ask MetaFilter](https://ask.metafilter.com/341806/Distinguishing-different-kinds-of-deadlines-in-to-do-apps))
— and the overdue pile becomes *"exhausting and highly demotivating"* dead wood
([Plaintext Productivity](https://plaintext-productivity.net/1-04-due-dates-and-todo-txt.html);
[dsokolovskiy.com](https://dsokolovskiy.com/blog/all/due-date-is-not-a-do-date/)).

### 3.3 What shipped, and what happened

- **Forster's dismissal** — pages of unstarted items get struck out when nothing "stands out"; framed as diagnostic
  (*"forces you to consider WHY you haven't done it"*). **Its trajectory is the key evidence: Forster softened the
  rule in 2021 and dropped it entirely from his Final Version**, retreating to manual weeding
  ([Final Version FAQ](http://markforster.squarespace.com/final-version-faqs/basic-system/);
  [2021 trial](http://markforster.squarespace.com/blog/2021/6/23/autofocus-1-new-dismissal-rule-trial.html)).
  Practitioners report both clarifying relief
  ([davidseah.com](https://davidseah.com/2010/01/mark-forsters-autofocus-system-to-get-everything-done/))
  and fear — dismissal *"frightens me"*
  ([GTD forum](https://forum.gettingthingsdone.com/threads/gtd-vs-autofocus-mark-forster.6784/)).
- **Sunsama auto-archive** — the closest shipped activity clock: a task that rolls over **4 consecutive days**
  untouched is auto-archived. Recoverable, configurable, **default-on in a successful paid product**
  ([Sunsama docs](https://help.sunsama.com/docs/archive)).
- **Marvin "Staleness Warning"** — an orange badge past a user-set age threshold. Age counts from *creation*, not
  last activity, and it prescribes no action
  ([help article 1950228](https://help.amazingmarvin.com/en/articles/1950228-staleness-warning)).
- **Daily-expiry apps** (Tet, txtodo and kin) hard-delete at midnight — a recurring indie micro-genre with
  enthusiastic minorities and no breakout ([Show HN](https://news.ycombinator.com/item?id=16316344)).
- **Complice / Intend** — intentions expire daily by design, and non-completion is treated as *information about
  priority* ([intend.do/philosophy](https://intend.do/philosophy)).
- **OmniFocus review intervals** act as a staleness clock on projects, and the community documents "review debt",
  with one user calling Review *"a massive stupid guilt trip"*
  ([Omni forums](https://discourse.omnigroup.com/t/review-mode-considered-harmful/41193)).

### 3.4 How users react to system-initiated killing

- **Relief is real, but only when the rules pre-authorize letting go.** Bankruptcy purges and Forster dismissal are
  described as *permission*.
- **System-initiated, irrecoverable deletion triggers loss aversion and distrust.** The Tet HN thread splits between
  "every day is a clean slate" and *"I have anxiety issues… this wouldn't work for me"*. GitHub stale bots — the
  largest-scale inactivity-kills-items experiment ever run — generated organized hostility
  ([HN 28998374](https://news.ycombinator.com/item?id=28998374); [nostalebots.xyz](https://nostalebots.xyz/)).
  Caveat: that anger is partly about *other people's* effort being discarded, which does not transfer cleanly to a
  single-user system. What does transfer is the **"not stale" comment** — the cheapest possible action invented to
  reset a clock.
- **Every surviving commercial implementation softens deletion into archive or badge.** No mainstream product ships
  unrecoverable auto-delete of arbitrary-horizon tasks. Hard delete survives only in day-scoped apps where the
  contract is explicit and the loss horizon is tiny.
- **The purge must produce information, not just absence** — Forster's design insight, echoed by Complice: expiry
  should prompt "why did this die?", not merely remove the item.

### 3.5 Documented absence

**No product found uses time-since-last-activity as the primary scheduling axis in place of due dates entirely.** The
pieces exist separately; the synthesis is unshipped. Bet 3 is genuinely novel territory, and the novelty is not the
clock — it is that expiry *charges* rather than *deletes*, a third category nobody has tried.

### 3.6 Clock length has one real datum, and it argues against short

**Rai, Sharif, Chang, Milkman & Duckworth 2023** (J Applied Psychology 108(4), 621–634; preregistered field
experiment, N = 9,108 crisis-line volunteers, 12 weeks; [PubMed](https://pubmed.ncbi.nlm.nih.gov/36107684/)):
reframing a 200-hour annual goal as subgoals raised hours volunteered by **8%**, and crucially — *"increasing subgoal
flexibility by breaking an annual 200-hr volunteering goal into a subgoal of volunteering 8 hr every 2 weeks, rather
than 4 hr every week, led to more durable benefits."* **The looser cadence outlasted the tighter one at identical
total volume.** Combined with §1.2's gym-time report, two independent lines say a shorter N is not strictly better.
This cuts against the delay-discounting argument for short clocks (§6), and the tension needs resolving with a number.

---

## 4. Decomposition tools

### 4.1 What the tools do

- **Goblin Tools Magic ToDo** breaks a typed task into steps with a user-controlled **"spiciness" granularity
  slider**; built *"mostly… to help neurodivergent people with tasks they find overwhelming"*, and its own About page
  disclaims accuracy — output *"should be taken as… only guesswork"*
  ([goblin.tools/About](https://goblin.tools/About)). The slider is the most reusable idea here: **it hands
  granularity back to the user rather than deciding it.**
- **Todoist Task Assist** generates sub-task suggestions, opt-in, presented as checkboxes to deselect before adding
  ([Todoist help](https://www.todoist.com/help/articles/use-the-task-assist-extension-with-todoist-ZgldtcPeT)) — a
  propose-then-curate flow that is itself an admission raw output needs human curation.
- **Motion** automates *scheduling* (when), not decomposition (what). Its documented churn driver is **agency**, not
  step quality: a reviewer conceded the AI scheduled competently and still left — *"I prefer to be in control of my
  schedule vs letting AI decide"* ([juliety.com](https://juliety.com/motion-app-review)).
- **Amazing Marvin** has **no** automatic decomposition; subtasks are hand-typed by design
  ([Marvin help](https://help.amazingmarvin.com/en/articles/1951489-all-about-subtasks)).
- **Sukha** markets an AI coach that breaks tasks down plus "members finish 2x faster" with no methodology
  ([thesukha.co](https://www.thesukha.co/) — marketing). No user testimony found that its breakdown feature is used
  or valued; its evidenced value is session structure / body doubling.
- **Tiimo** is the cautionary case of AI decomposition retrofitted onto an ADHD app (§4.2).

### 4.2 What users report

The 234-comment HN thread on Goblin Tools ([43461375](https://news.ycombinator.com/item?id=43461375)) contains both
halves first-hand:

- *For:* *"anywhere that they can get someone over the activation energy requirement of the blank page effect, it's
  okay if they're wrong — it's much easier to correct a wrong broken-down list than start it oneself."*
- *Against:* "Launch MyProjectName website" → *"open a browser and search for it"* (context-blind); "clean the
  bathroom" estimated at 3h25m for a 40-minute job; "eating a pie" decomposed into 10+ steps including *"chew and
  savor the flavor"* — extreme granularity making a simple task **more** intimidating, judged excessive even by a
  self-described autistic commenter.

App Store reviews of **Tiimo** (an award-winning ADHD/autism planner that added AI breakdown) are the best available
sample of what happens when decomposition is imposed rather than offered:

- *"Good in theory - spectacularly bad implementation… I asked it to break a task into two and saw it break it into 3
  all starting at the same time then delete all of them"* (2★).
- *"useful, but ruined by AI… I used to love being able to choose everything on my own… the AI features make the task
  menu look and feel cluttered and busy which is the LAST thing my brain needs when trying to organize my tasks."*
  Support told them to *"just 'not use them,'"* which was impossible because some fire automatically (1★).
- *"Even when it created subtasks, they were still much larger than the time blocks I requested. When I asked for
  smaller steps, the process repeatedly reset rather than refining the plan"* — i.e. **the granularity control is the
  part that fails** (3★).
- The positive pole always mentions editability: *"appreciate the AI breakdown of larger tasks **and ability to
  revise the recommendations**"* (4★); *"I love how you can break down tasks into smaller steps"* (5★).

### 4.3 What the research says

- **Self-generated** proximal subgoals and if-then plans work: **Bandura & Schunk 1981** — proximal subgoals drove
  initiation and self-efficacy, distal goals did nothing
  ([PDF](https://uploads-ssl.webflow.com/59faaf5b01b9500001e95457/5bc552d85141987915dab842_Bandura%20&%20Schunk,%201981.pdf));
  **Gollwitzer & Sheeran 2006** meta-analysis of implementation intentions, 94 studies, **d = 0.65**
  ([ResearchGate](https://www.researchgate.net/publication/37367696_Implementation_Intentions_and_Goal_Achievement_A_Meta-Analysis_of_Effects_and_Processes));
  a 2025 RCT (n = 1,035) where *participants naming one easy subtask themselves* raised self-reported completion
  likelihood, d ≈ 0.2 ([PMC12533354](https://pmc.ncbi.nlm.nih.gov/articles/PMC12533354/)).
- **No controlled study exists on passively accepted machine-made decomposition.** Documented absence.
- The nearest mechanistic result cuts an interesting way: **CatAlyst** (CHI 2023) used LLM output as a *restart lure*
  and found **output quality barely matters** — the generation works by lowering the psychological cost of
  (re)starting, not by being right ([arXiv:2302.05678](https://arxiv.org/abs/2302.05678)).
- HCI work with ADHD students converges on **collaborative decomposition**, with experts explicitly warning against
  **"cognition outsourcing"** and noting that **choosing step granularity is itself the metacognitive skill** a tool
  should train rather than replace ([CHI 2024, PMC11166253](https://pmc.ncbi.nlm.nih.gov/articles/PMC11166253/);
  [CHI 2026, arXiv:2602.09381](https://arxiv.org/html/2602.09381v1)).

### 4.4 Requiring a next action: the closest shipped analogue does not enforce

**No mainstream tool refuses to hold a project without a next action.** GTD software handles it with detection plus
review pressure:

- **OmniFocus "Stalled"** — *"a stalled project is a sequential or parallel project that has no remaining actions"*;
  the perspective lists them, and the prescribed remedies are complete it, put it on hold/abandon it, or add an
  action ([learnomnifocus.com](https://learnomnifocus.com/perspective/projects-stalled/)). It is a **review chore,
  not a gate** — users build the discipline themselves, e.g. *"I have an OmniFocus action… that repeats daily and
  prompts me to check for stalled projects"*
  ([Omni forums](https://discourse.omnigroup.com/t/how-to-use-a-stalled-perspective/26027)).
- Two stealable details. First, the official escape valve for "I don't know what to do next" is a **research
  action** — *"consider adding an action to get in touch with someone who's done this sort of project before"* — not
  a fake action. Second, a real UX bug: the perspective is live-filtered, so *"when I want to define next action in
  such a project… it immediately disappears from that Perspective"*. **The "needs a next action" state must be a
  place you can work IN, not a filter you fall out of mid-edit.**
- The pattern is in enough demand to be reimplemented elsewhere
  ([obsidian-gtd-no-next-step](https://github.com/saibotsivad/obsidian-gtd-no-next-step)).
- **Complice / Intend** went the other way and abolished the stored backlog entirely — daily intentions are the only
  unit ([intend.do/philosophy](https://intend.do/philosophy)).

Neither pattern has published outcome data. **A hard gate at commitment time is unshipped territory.**

---

## 5. Attendance over completion

### 5.1 Focusmate

- **Official claims are borrowed or internal.** The science page cites general social-facilitation and accountability
  literature ("16–32%", "230–310%"), none of it measuring Focusmate; the circulating "75% more tasks" figure is from
  their own 2021 user survey ([focusmate.com/science](https://www.focusmate.com/science/)). **No independent study of
  Focusmate exists.**
- **User reports are strongly bimodal** — "life changer" threads alongside "I hate Focusmate". Documented failure
  modes: **habituation** (*"used Focusmate a lot and then dropped it… productivity starts dropping again"* once being
  watched stops being novel), **surveillance aversion** (a user became uncomfortable being observed and began
  avoiding work *during* sessions), and the **attendance ≠ on-task gap** — you can be present and browse
  ([HN 21352305](https://news.ycombinator.com/item?id=21352305),
  [HN 21353781](https://news.ycombinator.com/item?id=21353781)). Its strongest constituency is the ADHD /
  body-doubling community.

### 5.2 Complice / Intend — the clearest articulation of *why* attendance

Malcolm Ocean's guest post on the Beeminder blog
([blog.beeminder.com/complice](https://blog.beeminder.com/complice/)): outcome metrics are lagging and not fully
under your control, so users commit to daily **process intentions** and to *accounting* for them at day's end. He
distinguishes **hard commitments** (money, for a few well-chosen metrics) from **soft commitments** (reflection, for
the many daily intentions where staking money on uncertain priorities is impractical) — a distinction vazhikkal
currently lacks, since every slot is hard. He quotes Nick Winter: *"the real cost of failing a goal is not the loss
of your… pledge money. It's the loss of confidence that you will meet all future goals."* A 6-month user review found
the daily-expiring lists effective and the built-in coworking rooms *"largely empty"*
([lifedev.net](https://lifedev.net/2018/02/15/complice-review/)).

### 5.3 Body doubling evidence

Early-stage. Peer-reviewed work is **qualitative** (ASSETS 2023 / TACCESS with neurodivergent participants —
establishes that people *report* it helps initiation, [dl.acm.org](https://dl.acm.org/doi/full/10.1145/3689648))
plus one **N = 12** VR experiment showing faster completion with human or AI body doubles
([arXiv:2509.12153](https://arxiv.org/abs/2509.12153)). Wikipedia's summary is accurate: support largely anecdotal,
mechanisms unclear ([Wikipedia](https://en.wikipedia.org/wiki/Body_doubling)). Classic social-facilitation research
also cuts both ways — presence improves simple tasks and can *impair* complex ones, a nuance every marketing page
omits. Caveday and Sukha have nothing beyond testimonials.

### 5.4 The road not taken, and why

A slot could be kept alive by *N minutes of attention* rather than *a completed next action*. That version is much
harder to sandbag on size and much easier to sandbag on attention (§1.2's *"I'm measuring studying time, not
effort"*), and vazhikkal has no witness to make presence meaningful. Recorded as a live alternative, not a
recommendation.

---

## 6. Executive-dysfunction design: evidence vs marketing

The evidence-backed core comes from one coherent theory rather than from app vendors. Russell Barkley's
executive-function account of ADHD ("The Important Role of Executive Functioning and Self-Regulation in ADHD",
[russellbarkley.org](https://www.russellbarkley.org/factsheets/ADHD_EF_and_SR.pdf)) states management principles
that — read cold — describe vazhikkal's machine almost line by line.

**Has evidence behind it:**

1. **Externalize the information.** *"They will be best assisted by 'externalizing' those forms of information…
   Since covert or private information is weak as a source of stimulus control, making that information overt and
   public may assist with strengthening control of behavior."* → An always-visible committed set with a visible next
   action is not decoration; it is the mechanism.
2. **Close the temporal gap.** EF deficits are *"to time what nearsightedness is to spatial vision… a temporal
   myopia in which the individual's behavior is governed even more than normal by events close to or within the
   temporal now."* The prescription: *"reducing or eliminating gaps in time among the components of a behavioral
   contingency (event, response, outcome)."* And explicitly: *"Rather than tell them that a project must be done
   over the next month, assist them with doing a step a day toward that eventual goal so that when the deadline
   arrives, the work has been done but done in small daily work periods with immediate feedback and incentives for
   doing so."* → **This is the staleness clock, prescribed by name, and it is the strongest evidence-based argument
   for a short N.** It is in direct tension with §3.6.
3. **Externalize the motivation, and expect to keep doing so forever.** *"The provision of artificial rewards…
   become for the person with EF deficits what prosthetic devices such as mechanical limbs are to the physically
   disabled."* And the caveat that matters most for a money penalty: *"Complaining to these individuals about their
   lack of motivation (laziness), drive, will power, or self-discipline will not suffice… Pulling back from
   assisting them to let the natural consequences occur, as if this will teach them a lesson that will correct their
   behavior, is likewise a recipe for disaster."* Also: *"these compensatory, prosthetic forms of motivation must be
   sustained for long periods."*
   → **(a)** A pure-punishment device with no support scaffolding is the thing Barkley explicitly calls a recipe for
   disaster; the dues ledger has to be paired with assistance at the point of performance, not substituted for it.
   **(b)** Any expectation that the app can eventually be switched off because the habit has been internalised is
   unsupported — the prosthesis is permanent. This aligns exactly with #7's finding that one-off incentives decay and
   only *standing* structures persist.
4. **Intervene at the point of performance** — *"that place and time"* where the behaviour is needed. Notifications
   as design, not afterthought (the map already has a ticket for reaching the user).

**Mostly marketing:**

- "Dopamine menu", "dopamine-friendly", and most neurotransmitter vocabulary in productivity copy. Marvin ships a
  "Dopamine Menu" feature; the name is branding, not a mechanism.
- Gamification as treatment. Points and streaks have short-run engagement effects that decay; #7's evidence on
  incentive decay applies directly, as does Duolingo's revealing product response to streak lapses — *adding* freeze
  and repair mechanisms so the number survives when the behaviour didn't. A retention-driven vendor sides with the
  metric; a commitment device must not.
- "AI that knows your energy levels."
- **"Designed for ADHD brains" as a claim about customisability.** Marvin is the honest end of this: its pitch is
  hundreds of toggles so you can find what works. That is a real value proposition *and* an admission that nobody
  knows which choices work — the opposite of an evidence claim.
- Tiimo's trajectory (§4.2) is the cautionary tale: an app that won awards partly for a *timed routines* feature
  grounded in time blindness, then replaced it with untimed AI subtasks. A long-term user: *"As an app catering to
  people with ADHD/autism, this was critical for folks with time blindness… the untimed subtasks are a significant
  downgrade."*

**Background mechanism for the whole design:** Steel 2007's meta-analysis of 691 correlations
([PubMed](https://pubmed.ncbi.nlm.nih.gov/17201571/)) finds the *"strong and consistent predictors of procrastination
were task aversiveness, task delay, self-efficacy, and impulsiveness"*, framed by temporal motivation theory
(expectancy × hyperbolic discounting). Mapping the bets onto it: **decomposition attacks task aversiveness and
self-efficacy; the staleness clock attacks task delay** (shortening delay-to-consequence, which under hyperbolic
discounting is the whole ballgame); **the cap attacks none of these** — it attacks the graveyard and choice overload,
which is a real but different problem.

---

## 7. What this says about the three vazhikkal bets

### Bet 1 — committed set hard-capped at N slots, eviction required to enter

**Verdict: mechanistically sound, empirically unproven, essentially unshipped — and the risk lives in the eviction,
not the cap.**

- **For it:** the industry's own diagnosis of why advisory WIP limits fail is that violating them is costless
  (§2.3–2.5). Attention residue gives a real mechanism (Leroy 2009). The practitioner cap cluster is tight and low —
  Personal Kanban 3, Clark & Wheelwright's measured peak at **2** concurrent projects, Intend's norm 3–5 — which
  argues **N should be small, likely 3, and probably not more than 5.** Streaks' 24 is a different unit (tracked, not
  in-progress) and is not a precedent for slot count.
- **Against it:** no study identifies an optimal WIP limit even for teams (SINTEF); hard-cap apps have effectively no
  users; and Brodzinski's field experience is that **imposed** caps breed resistance and routing-around while
  internalized pull rules don't. In a single-user app, the only route around the cap is eviction.
- **Also against it, from Steel 2007:** the cap does not touch any measured predictor of procrastination. It solves
  the graveyard — a real problem in the map's own account of failure — but do not expect it to make starting easier.
  #7's warning that the cap **relocates** the graveyard into an unread inbox stands.
- **Design consequences:** eviction needs a price or an akrasia-style horizon, or it is a fee-dodge; the eviction
  record should force a reason (Forster/Complice: the purge must produce information); and the cap must be
  continuously legible as the product's value rather than a limitation.

### Bet 2 — a project cannot occupy a slot without a concrete next action

**Verdict: the best-evidenced of the three bets, unshipped as a hard gate, and the sharpest edge in the design.**

- **For it:** implementation intentions are among the better-replicated behaviour-change findings (d = 0.65,
  94 studies); Bandura & Schunk show proximal subgoals drive initiation where distal goals don't; Barkley's
  externalize-the-information principle says a visible concrete action is the mechanism, not the polish. Vazhikkal's
  one-action-per-slot is also mainstream GTD practice, and the anti-pattern GTD practitioners warn about
  (pre-loading whole plans) is exactly what AI decomposition produces. **Requiring one action, not a plan, is right.**
- **Caveat 1 — the active ingredient is an if-then cue, and cues are usually times or places.** "Email the
  accountant" is a next action; "when I sit down after breakfast, email the accountant" is an implementation
  intention. Bet 3 bans prediction, which risks banning the very thing the evidence supports. A *cue* is not a
  *deadline* and the two are separable — but the spec should separate them deliberately, not by accident.
- **Caveat 2 — nothing bounds size, and this is where money leaks.** Confirmed twice over: GTD has had no
  granularity standard for 20+ years and its community says so loudly (§1.4), and no shipped tool bounds it (§1.7).
  Vazhikkal attaches money directly to next-action completion, which is a stronger gaming incentive than anything in
  the survey, with no autodata and no referee. Expect sandbagging as the steady state, not the abuse case.
- **What is actually available**, ranked by cost-to-value:
  1. **Reframe the completion event** (free, evidence-backed by Fishbach 2006): never say "you're safe for N days".
     Say "this is the *k*th action on this project". The substitute frame is the one that predicts coasting.
  2. **Separate naming from satisfying** (akrasia-horizon logic): the next action that resets the clock must have
     been named *before* the clock got short. Naming a fresh action under pressure and immediately ticking it is the
     exploit.
  3. **Require the completion claim to say what changed**, converting mushy end-criteria (§1.2) into stated ones.
  4. **Make the action text visible to the collector** — the only referee available (#6 says a read-only surface is
     cheap on Splitwise).
  5. **Track project-level movement separately from action ticks** (the forum's attempts-vs-outcomes pattern), so a
     green slot that hasn't moved in two months is *visible* even if it never triggers a fee.
- **UX constraint from OmniFocus:** the "needs a next action" state must be a place you can work in, not a filter you
  fall out of mid-edit. And the sanctioned escape for genuine "I don't know what's next" is a **research action**,
  not a fake one — worth adopting explicitly, because it is the honest version of sandbagging.

### Bet 3 — a staleness clock replaces due dates entirely

**Verdict: the diagnosis is the strongest part of the whole design; the mechanism is sound and partly precedented;
the clock length is the highest-leverage unspecified number in the spec, and the evidence on it conflicts.**

- **For it:** the due-date critique is well-attested and matches the map's account of failure (cry-wolf
  desensitization; overdue piles as demotivating dead wood). Age-as-priority ships and persists (Taskwarrior,
  Marvin's badge, Trello card aging). Age-as-activity-clock ships and is default-on in a paid product (Sunsama's
  4-day auto-archive). And vazhikkal's variant — **expiry charges rather than deletes** — sidesteps the one reaction
  that is reliably hostile.
- **Against it / to watch:**
  1. **Short clocks invite token actions.** §1.2's gym-time report is the most on-point user datum in this survey.
     The clock's length does not just set pressure; it sets *the size of the cheapest action that survives it*.
  2. **The one field experiment on cadence favours the looser one.** Rai 2023: the 2-weekly subgoal produced more
     durable benefit than the weekly one at identical volume (§3.6).
  3. **But Barkley's temporal-myopia principle argues the opposite** — close the gap between event, response and
     outcome; "a step a day" with immediate feedback (§6). **These conflict, and the conflict is the decision.** A
     defensible resolution: **short clock, but the action must have been named earlier** (bet 2, mitigation 2), so
     immediacy comes from the consequence while the bar was set by a calmer self.
  4. **Clock-reset rituals are the predicted emergent behaviour and have already been observed at scale** — the
     stale-bot "not stale" comment. Plan for it; don't treat it as an abuse case.
  5. **Expiry must produce information, not just a debit** (Forster, Complice). A miss that only credits money and
     asks for a fresh action throws away the most useful signal in the system: *why* this slot stopped moving.
  6. **No product replaces due dates entirely with an activity clock.** Genuinely unshipped, so budget for being
     wrong about N and make it changeable — with an akrasia-style horizon on making it *easier*.

---

## 8. Open questions

1. **What bounds next-action size?** No shipped tool answers this. Candidates, none validated: minimum time estimate,
   a required "what will be different afterwards" field, collector visibility of the action text, pre-committing the
   *next* next action before claiming the current one, or a periodic project-level "has this actually moved?" check
   separate from the clock.
2. **What is N (days)?** The evidence conflicts (Barkley: shorter; Rai 2023 and the gym-time report: longer). Needs a
   number and a stated rationale, plus a horizon on changing it.
3. **What is N (slots)?** Practitioner and measured evidence cluster at 2–3 concurrent; Intend's norm is 3–5. Nothing
   supports a large number.
4. **Does eviction have a price or a horizon?** Carried from #7, reinforced by §2.5. Without one, the cap is a
   revolving door and a fee-dodge.
5. **Does a cue count as a due date?** The implementation-intention evidence wants a when/where trigger; bet 3 bans
   prediction. Where the line falls is a spec decision with real evidence riding on it.
6. **Hard vs soft commitments.** Intend distinguishes the few metrics worth money from the many worth only
   reflection. Every vazhikkal slot is currently hard. Is that right?
7. **What does the completion event say?** Free lever, evidence-backed, currently unspecified. Substitute framing
   ("you're safe") predicts coasting; commitment framing predicts follow-through.
8. **Does a miss capture *why*?** Both Forster and Complice say the purge must produce information. Currently a miss
   produces a debit and a prompt for a fresh action.
9. **Is there any decomposition assistance at all?** §4 argues: opt-in, offered only at the moment of paralysis,
   always editable, never automatic — and the sanctioned fallback for "I don't know what's next" should be a research
   action.
10. **How is the cap made desirable rather than felt as a limitation?** Streaks' community reaction says this is a
    real problem even for a user who chose the constraint.

---

## 9. Method and confidence notes

- **Verified firsthand in the final pass:** Marvin's Strategies collection, features page and kanban help article
  (no WIP limits, no forced eviction — correcting a prior belief); Beeminder's weaselproofing/No-Excuses posts; the
  Beeminder forum posts quoted in §1.2 (via the public search index); Streaks' 24-task cap on its own site;
  Businessmap's WIP-limits article; Barkley's EF fact sheet.
- **Verified in earlier passes of this ticket** and carried forward with links: Goblin Tools About + HN 43461375;
  Todoist Task Assist; Motion review; Marvin subtasks and staleness-warning articles; Sunsama archive docs; Intend
  philosophy; Personal Kanban WIP article; Brodzinski; SINTEF; Leroy/Ovsiankina; Forster's Final Version FAQ and 2021
  dismissal trial; OmniFocus stalled/review threads; Focusmate science page; Ocean's Beeminder guest post; the
  Complice long-term user retrospective; stickK FAQ; Forfeit; Ariely & Wertenbroch; Gollwitzer & Sheeran; Bandura &
  Schunk; Rai 2023; Fishbach 2006; Steel 2007; CatAlyst; CHI 2024/2026 ADHD decomposition work; Tiimo App Store
  reviews.
- **Coverage gap:** Reddit is not directly fetchable from this environment, and the session's search quota was
  exhausted partway through. Reddit-derived material is limited to titles/snippets and is not load-bearing anywhere
  in this doc. User reports come from Hacker News, product forums (Beeminder Discourse, GTD, Omni), long-form
  reviews, and App Store reviews.
- **Asserted from general knowledge, not re-verified here** (low risk; do not quote as citations): Taskwarrior's
  urgency age coefficient, Trello Card Aging, Duolingo streak-freeze mechanics, and the organisational
  sandbagging/OKR countermeasure literature in §1.5's third bullet.

# Progress Page — Candidate Content Menu (for external review)

**Purpose.** This document is a *broad menu of options*, not a decision. It exists so a designer
or a behavioral-science / psychology reviewer can help choose **which habit statistics and
visualizations are actually worth surfacing** on Forge's Progress page. Each candidate has one or
two sentences of "what it shows / why it might matter." Please react, prioritize, add, and cut —
nothing here is committed.

You need **no** knowledge of the codebase to read this. §1 gives you just enough context.

---

## 1. Context you need (30 seconds)

**Forge is a habit-tracking iOS app.** A user creates habits, each belonging to one of three
categories:

- **Build** (good habits to do — e.g. "Drink water," "Read Qur'an")
- **Destroy** (bad habits to avoid — e.g. "Limit screen time," "No smoking")
- **Tasks** (one-off / to-do-style items)

Each day, a habit can be *completed*. For a **Destroy** habit, "completed" means **successfully
avoided that day** — this matters for how stats are framed (a "streak" on a Destroy habit is a
streak of *resisting*, not of *doing*).

**What raw data actually exists** (this bounds what any statistic can be computed from — please
don't assume data we don't have):

| Data | What we have | Notable limits |
|---|---|---|
| **Completions** | Per habit, per day: whether it was completed, the running count vs. goal, and the **real timestamp** it was logged. | **A "miss" is not recorded** — a missed day for a normal habit is simply the *absence* of a completion row. There is no timestamp or reason attached to a miss. (Exception: prayer habits do persist an explicit "missed" flag.) |
| **Goal-at-completion** | The goal in effect on the day of each completion (goals can grow over time), so historical "did they hit goal" is measured correctly. | — |
| **Streaks** | Computed on demand from completion history, per habit and per category. Streak milestones exist at **7 / 30 / 100 / 365** days. | Vacation mode pauses (doesn't break) streaks. |
| **Points** | A single running total, flat **+1 per completed day, −1 per missed day**. Milestones at 50 / 100 / 250 / 500 / 1000. | Deliberately simple; not weighted by difficulty. |
| **Mood** | An optional once-per-day mood check-in on a **5-point scale** (great → rough), with a timestamp. | Opt-in; **not** tied to any habit and **not** part of points/streaks — purely observational. |
| **Duration / timer** | For time-based habits (e.g. "Meditate 20 min"), the actual elapsed time is recorded. | — |
| **HealthKit** | Some habits auto-complete from Apple Health data (steps, sleep, workouts, etc.). | On-device only. |
| **Prayer / mosque** | Prayer habits have strict daily time windows and record whether each was done, missed, or is still open. *"Completed at a mosque"* is planned but **not built yet**. | — |

**What's already on the Progress page today** (so candidates below can be judged as "we have
this," "this extends it," or "this is genuinely new"):

1. **Consistency Heatmap** — a 140-day GitHub-style grid, one row per category, cell darkness = that day's completion rate.
2. **Streak number** — the single largest current category streak, shown as a real number, plus a 7-day completion-rate bar chart.
3. **Category Breakdown** — a single proportional stacked bar: what share of this week's completions came from each category.
4. **Best Day / Time & Streak Distribution** *(premium-gated)* — completion rate by day-of-week; a time-of-day histogram of *when completions get logged*; and a histogram of how long past streaks ran (bucketed at the 7/30/100 milestone thresholds).
5. **Recent Activity** — a short reverse-chronological list of what was logged, with times.
6. **Milestones** — earned achievement badges (streak/points/challenge), with 3D badge detail.
7. **Habit Trends** — per-category completion rate for the last N days vs. the N days before (is each category trending up or down).
8. **Habits list** — every habit, tappable into its own detail page (per-habit history/streak/rate).

**One existing design value to stay consistent with.** Forge already has a *"non-judgmental
framing"* precedent: its Weekly Reflection notification deliberately frames misses gently rather
than as failures. Any new Progress content should honor that — this is called out again under §7.

---

## 2. Candidate menu

Grouped by intent. Each item: **what it shows** → *why it might matter*. A ⚠️ marks a candidate
whose data we **don't fully have today** (flagged honestly so experts don't over-index on it).

### A. Descriptive / summary statistics (at-a-glance "how am I doing")

- **A1. Overall completion rate (rolling 7 / 30 / 90 day).** The single headline percentage of applicable habit-days completed. → The most universally legible "am I on track" number; anchors everything else.
- **A2. Total completions, all-time and this period.** A simple cumulative count ("1,204 habits completed"). → Cumulative counters are motivating precisely because they only ever grow — a small, safe win.
- **A3. Active-habits count and coverage.** How many habits are active, and how many got *any* attention today/this week. → Surfaces over-commitment ("you have 14 habits, you're touching 4") without judgment.
- **A4. Per-category completion rate.** Build vs. Destroy vs. Tasks, side by side. → Users often succeed in one category and struggle in another; separating them prevents a strong category from masking a weak one.
- **A5. Best and worst performing habit (this period).** The single habit with the highest and lowest completion rate. → Concrete, actionable — celebrates a win and names one thing to focus on.
- **A6. "This week in numbers" summary card.** A compact digest (completions, current streak, points earned, mood average). → A scannable weekly recap; pairs naturally with the existing Weekly Reflection feature.

### B. Streak & consistency visualizations

- **B1. Per-habit streak list.** Current streak for every habit, sorted longest-first. → Streaks are the single most-studied motivator in habit apps; making each one visible creates many small "don't break it" stakes.
- **B2. Longest-ever vs. current streak.** For each habit or category, the personal record next to the live streak. → A personal best is a self-competition target that doesn't require other people.
- **B3. Consistency score (beyond raw streak).** A resilience-weighted metric that rewards *bouncing back* after a miss, not just unbroken runs. → Pure streaks punish a single slip harshly (all-or-nothing); a consistency score better reflects real behavior and is gentler — see §7 on why that matters.
- **B4. Calendar/heatmap per individual habit.** The existing category heatmap, but drillable to one habit. → Lets a user see the *shape* of their misses (weekends? mid-week? sporadic?) for a specific behavior.
- **B5. "Perfect days" count.** Days where every applicable habit was completed. → A clean, aspirational binary; a satisfying rare-achievement counter.
- **B6. Current chain length toward the next milestone.** "12 days — 18 to your 30-day badge." → Goal-gradient effect: motivation rises as a visible target nears.

### C. Correlation / insight statistics ("here's a pattern you might not have noticed")

- **C1. Mood vs. completion rate.** Do higher-completion days correlate with better logged mood (or vice versa)? → We have daily mood + daily completions; this is the flagship "insight" the app's data uniquely enables, and it reframes habits as connected to wellbeing rather than obligation. *(The app's own roadmap already earmarks this as a future card.)*
- **C2. Day-of-week patterns.** Which weekdays a user reliably succeeds or slips on. *(Partially shipped in the premium card.)* → Names a concrete, fixable pattern ("Sundays are your weak day") instead of a vague sense of inconsistency.
- **C3. Time-of-day patterns.** When during the day completions actually happen. *(Partially shipped.)* → Helps a user schedule habits into the windows they already reliably act in. ⚠️ *Note: we can only show when completions **happen**, never when misses happen — a miss has no timestamp. This is a real ceiling; experts should know the "when do you fall off" question is only answerable at day-of-week granularity, not time-of-day.*
- **C4. Habit-pairing / co-occurrence.** Which habits tend to be completed on the same days (e.g. "you almost always meditate on days you also exercise"). → Surfaces natural habit-stacking opportunities, a well-supported formation technique.
- **C5. Category interaction.** Do Destroy successes track with Build successes? → Tests the intuition that avoiding bad habits and doing good ones reinforce each other, at the individual level.
- **C6. Duration trends for timed habits.** Average / total minutes over time for meditation, reading, exercise. → For time-based habits, *how long* is often more meaningful than *whether*; a rising average is a subtle, honest progress signal. ⚠️ *Only available for the subset of habits that are time-based.*
- **C7. HealthKit cross-reference.** For health-linked habits, show the underlying trend (e.g. average steps, sleep duration) alongside completion. → Grounds a habit in objective outcome data the user already trusts. ⚠️ *Only for HealthKit-linked habits.*

### D. Predictive / forward-looking framing

- **D1. Projected milestone date.** "At your current rate, you'll hit a 100-day streak on Nov 3." → Turns a distant abstract goal into a concrete date; makes progress feel inevitable rather than uncertain.
- **D2. Rate-based projection.** "You've done this 18 of the last 30 days — keep it up and that's ~219 days this year." → Extrapolation reframes small daily actions as large cumulative outcomes, a known motivator.
- **D3. "At-risk" gentle nudge.** Flag a habit whose recent rate is falling *before* the streak breaks. → Early, low-stakes intervention; must be framed supportively, not as a warning (see §7).
- **D4. Goal-progression preview.** For auto-increasing goals, show the upcoming step ("your goal rises to 110 next week"). → Sets expectation and frames escalating difficulty as designed progress, not creeping pressure.
- **D5. Next-best-action suggestion.** A single recommended focus ("finishing today completes your perfect week"). → Reduces the cognitive load of "what should I do" to one concrete, high-leverage prompt.

### E. Comparative / social statistics *(require Groups — a planned feature, not yet built)*

- **E1. Leaderboard-adjacent ranking within a group.** Where a user ranks among group members by streak / points / completions. → Social comparison is a strong motivator *for some people*; must be optional and framed carefully to avoid discouraging those who fall behind (see §7).
- **E2. "You vs. your group average."** The user's rate against the group's median, without a full ranking. → A gentler comparison than a leaderboard — belonging/normative feedback ("you're in step with your group") rather than competition.
- **E3. Team streak contribution.** How the user's own consistency feeds a shared group streak. → Relatedness / accountability: "others are counting on you" is a distinct motivator from personal streaks.
- **E4. Head-to-head accountability partner.** A 2-person "who kept their goal more days this week" comparison. → For a single trusted partner, direct friendly competition; lighter-weight and less exposing than a group leaderboard.
- **E5. Encouragement received / given.** A tally or feed of nudges exchanged with group members. → Reciprocity and social reinforcement; makes the *relationship* visible, not just the numbers.

*(All of §E depends on the Groups backend that's planned but unbuilt — included so the reviewer
can weigh social stats as part of the full menu, and so Progress-page IA can leave room for them.)*

### F. Per-habit deep-dive (on the individual habit detail page, not the main Progress page)

- **F1. This habit's full history calendar + current/longest streak + all-time rate.** *(Largely shipped.)* → The complete single-habit picture for someone focused on one behavior.
- **F2. Goal-vs-actual over time for quantity habits.** A line/bar of daily count against the (possibly rising) goal. → Shows effort, not just the binary — a habit can be "incomplete" yet clearly trending up.
- **F3. Miss-recovery time.** Average days to resume a habit after a miss. → A resilience metric; low recovery time is itself a skill worth reflecting back.
- **F4. Effect of goal progression.** How completion rate held (or dipped) each time the goal auto-increased. → Tells the user whether their escalation pace is sustainable or too aggressive.

### G. Mood-specific (a distinct, underused data source)

- **G1. Mood trend line.** The 5-point mood over weeks/months. → A simple wellbeing signal; valuable on its own even before correlating with habits.
- **G2. Mood distribution.** How often each mood level occurred this period. → A non-judgmental self-portrait ("mostly 'good' days lately").
- **G3. Mood-tagged best/worst habits.** Which habits most often accompany high- vs. low-mood days. → The most actionable form of C1 — points at specific behaviors linked to feeling better. ⚠️ *Correlational only; must be framed as "associated with," never "causes."*

### H. Domain-specific (Islamic / prayer content, for users who add it)

- **H1. Prayer consistency grid.** The five daily prayers across the week/month, completed vs. missed vs. late. → For prayer-focused users, this is the single most meaningful consistency view the app can offer.
- **H2. On-time vs. late rate.** Share of prayers completed within the preferred window. → Adds a quality dimension beyond "done," matching how the domain itself frames it.
- **H3. Mosque-completion rate.** Share of prayers done at a mosque (once that feature ships). → A community/effort signal specific to this domain. ⚠️ *Depends on the unbuilt mosque feature.*
- **H4. Dhikr / counter totals.** Cumulative counts for tasbih-style counting habits. → Big cumulative numbers ("12,400 total") are motivating for repetitive-count practices.

---

## 3. Habit-formation research lenses (how to *frame*, not just what to show)

For the psychology/behavioral-science reviewer specifically — these are cross-cutting framing
questions that apply to almost any candidate above, grounded in established theory:

- **Streaks & loss aversion.** Streaks work because breaking one *feels like a loss*. That same
  mechanism makes a single slip disproportionately demotivating ("I broke my 40-day streak, why
  bother"). **Open question for the reviewer:** where should Forge lean into streak stakes (B1,
  B6) vs. deliberately soften them (B3 consistency score, F3 recovery time) to protect long-run
  adherence? The app already leans gentle elsewhere — see below.

- **Self-Determination Theory (autonomy / competence / relatedness).** A useful checklist for the
  whole page:
  - *Competence* — do the stats make the user feel capable and improving? (A2 cumulative counts,
    D1/D2 projections, B2 personal bests all serve this.)
  - *Autonomy* — do they support the user's *own* goals rather than imposing external pressure?
    (Framing D3/D4 as information, not nagging.)
  - *Relatedness* — the social stats (§E) are the main lever here; consider whether any
    relatedness framing is possible *before* Groups ships.

- **Non-judgmental miss framing (an existing Forge value).** Forge's Weekly Reflection already
  frames misses supportively rather than as failures. Any "at-risk" (D3), "worst habit" (A5), or
  low-mood-linked (G3) content should match that tone: describe patterns neutrally, offer a next
  step, avoid red/alarm styling and failure language. **This is an existing design commitment, not
  a new proposal — please keep candidates consistent with it.**

- **Feedback immediacy vs. reflection.** Some stats are best *right after* an action (immediate
  reinforcement); others are best as periodic reflection (weekly digest). Worth the reviewer
  flagging which candidates belong on an always-visible Progress page vs. a periodic summary.

- **Comparison, carefully.** Social comparison (§E) motivates some users and demoralizes others.
  Research favors *upward comparison with attainable targets* and *normative "you belong" feedback*
  (E2) over raw ranking (E1) for sustained motivation — a real design fork worth the reviewer's
  input, and the reason E2/E3 are listed alongside E1 rather than assuming a leaderboard is best.

---

## 4. What we deliberately can't do (be honest with reviewers)

So feedback stays grounded in real data:

- **We cannot analyze *why* or *when* a habit was missed.** A miss is the absence of a record.
  Anything framed as "you tend to fail at X time" is only answerable at **day-of-week** granularity,
  never time-of-day.
- **Mood is not linked to individual habits or to points/streaks.** Mood↔habit stats (C1, G3) are
  correlational across the same *day*, not causal or per-habit-attributed.
- **Social stats (§E) don't exist yet** — they depend on the planned Groups backend.
- **Some stats only apply to a subset of habits** (duration → timed habits, HealthKit → linked
  habits, prayer/mosque → prayer habits). A general Progress page must degrade gracefully when a
  user has none of that habit type.

---

*This document is intentionally a menu, not a spec. The goal is to gather expert prioritization
before committing engineering effort to any particular chart.*

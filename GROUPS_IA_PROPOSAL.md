# Groups / Accountability-Partner — Information Architecture Proposal

**Status: proposal for Bilal's review. No code and no `TASKS.md` was changed.** This pins down the
*where does it live / what does each screen show* questions that Phase F (Groups core), Phase G
(social layer), and Phase J (accountability-partner Habit Race) decided in *behavior* but never in
*navigation*.

Part 2 (§1–§4 below) answers the Groups placement/screen questions concretely. Part 3 (§5) is a
sweep of the *other* P1 phases for the same "we never said where this lives" failure mode.

---

## Context (confirmed against the code, not assumed)

- **The app has exactly 3 tabs** — Home, Progress, Profile (`Forge/Navigation/AppTabView.swift`,
  a native iOS 26 Liquid-Glass `TabView`). Adding a 4th tab is a one-line change structurally, but
  a real IA decision.
- **Profile is currently nearly empty** (`ProfileView.swift`): a large photo+name header, the
  "Upgrade to Forge Premium" card, and then a `Spacer(minLength: 300)` — i.e. deliberate blank
  space below the premium card. Profile already owns the **identity** surface (Sign in with Apple)
  that Groups fundamentally depends on, and the account-adjacent Premium card.
- **The decided Groups feature set** (from `TASKS.md` Phase F/G/J):
  - **Phase F (core):** `Group` / `GroupSharedHabit` / `GroupHabitCompletion` records; create/join a
    group; add a shared habit; each member completes from their own device; **Team Streak**; basic
    **Activity Feed**. CloudKit + `CKShare`; invite via `UICloudSharingController` share link.
    **Creating** a group requires Forge Premium; **being invited into** one does **not**.
  - **Phase G (layer on top):** Leaderboard; Encouragement + Celebrations; Badges / Group Rewards;
    Shared Goals + Weekly/Monthly Challenges + Promises; Group Calendar / Team Streak Calendar;
    **Habit Races** (#6); Seasons; Proofs (photo); Group Chat.
  - **Phase J:** a 2-person **accountability-partner Habit Race** for a screen-time goal, reusing
    Phase G #6's `GroupHabitRace`.

---

## 1. Where does "Groups" live in the navigation?

### Recommendation: a **4th tab, "Groups."** (With a defensible interim fallback — see below.)

I recommend promoting Groups to a top-level tab rather than burying it in Profile. The reasoning,
in priority order:

1. **Invited free users need an obvious, permanent home for their group.** Creating a group is
   Premium-gated, but *joining* one is not — so a non-paying user who accepts an invite link needs
   somewhere to find that group afterward, forever. A share-link deep-link drops them *into* a
   group once; the recurring "where did my group go?" question needs a stable, discoverable
   destination. A tab answers it unambiguously; a section three levels into Profile does not.
2. **The feature set is genuinely a frequently-revisited surface, not set-and-forget.** Activity
   Feed, Encouragement/Celebrations, Chat, live Habit Races, and Leaderboards are all "check back
   often" content — exactly what a tab bar is for. Personal habit tracking (Home) and reflection
   (Progress) are a different mental mode; social accountability deserves peer status, not a
   sub-page.
3. **Discoverability is load-bearing for social features specifically.** Social loops live or die
   on how easily members return to them. A buried entry point structurally caps engagement on the
   one feature category whose entire value *is* engagement.
4. **iOS HIG fit.** Tabs are for top-level, peer content areas. "My habits" (Home), "My progress"
   (Progress), "My people" (Groups), "Me" (Profile) is a clean, conventional 4-way split — and iOS
   26's Liquid-Glass tab bar carries 4 tabs comfortably.

**The honest counter-argument** (so this is a real recommendation, not a rubber stamp): a 4th tab
shows for *everyone*, including the majority of users who may never join a group, adding permanent
chrome for a minority feature; and it's a bigger structural change than a Profile section. If Bilal
weights those concerns higher, the fallback below is legitimate.

### Fallback / phased option (legitimate, if minimizing early structural change matters):

Ship **Phase F core as a "Groups" section inside Profile** (a `NavigationLink` row under the
Premium card, landing on the same Groups list screen described in §2), validate the core loop with
real users, then **promote it to a 4th tab when Phase G's engagement-heavy features land** (Chat,
Leaderboard, Races, Challenges). The Groups *list* and *detail* screens in §2–§3 are identical
either way — only the entry point differs — so this defers the tab decision without rework.

**Net recommendation:** design the screens now (§2–§3); default to a **4th tab**; treat
"Profile-section-first, promote later" as an acceptable interim only if an early TestFlight wants
the smaller change.

---

## 2. The Groups list screen

Reached by the Groups tab (or the Profile → Groups row in the fallback). One screen, three states.

```
GROUPS  (nav title)                                    [ + ]  ← toolbar menu
──────────────────────────────────────────────────────────────
```

**State A — not signed in:**
- A short explainer + a **Sign in with Apple** button (Groups requires the identity Profile
  already establishes). Tapping through returns here signed in.

**State B — signed in, no groups yet (empty state):**
- Friendly empty illustration + one line ("Build habits together").
- **Primary button: "Create a Group"** — Premium-gated. If the user isn't Premium, this opens the
  existing Paywall (reuse `PaywallView`) rather than a dead end.
- **Secondary: "Join with an Invite Link"** — for pasting/opening a shared `CKShare` link (though
  the normal path is tapping the link in Messages, which deep-links straight in).
- **Pending invites**, if any, surface here as an accept/decline list.

**State C — signed in, has groups (the main case):** a vertical list of **group cards**. Each card:

```
┌──────────────────────────────────────────────────────────┐
│ ⬡  Morning Grind                                     ›    │
│    [👤][👤][👤] +2   ·   5 members                        │
│    🔥 4-day team streak   ·   3/5 done today             │
└──────────────────────────────────────────────────────────┘
```

Per-card content:
- **Group icon + category color** (from the `Group` record's icon/category).
- **Group name.**
- **Member avatars** (stacked, up to ~4, then "+N") and a member count.
- **Today's Team Streak status** — the single most valuable at-a-glance number: current team
  streak, plus a "N/M done today" progress signal so a member can see instantly whether the group
  is on track *today*.
- Optional small **unread indicator dot** if there's new Activity/Encouragement/Chat since last
  visit (Phase G — omit until those exist).

Toolbar **[ + ] menu:**
- **Create Group** (Premium-gated → Paywall if not premium).
- **Join via Invite Link.**

**Create-group flow** (a pushed multi-step or a sheet):
1. Name + category/purpose + icon.
2. Add the group's first **shared habit** (title/icon/color/schedule → a `GroupSharedHabit`).
3. **Invite** via `UICloudSharingController` (native share sheet → link out through Messages/Mail).

---

## 3. Tapping into ONE group — the group detail screen

### Layout recommendation: **one scrolling overview page with drill-ins for the heavy surfaces** —
### *not* in-screen tabs.

Why a scrolling overview rather than tabs-within-the-group:
- A group's content is **heterogeneous** (habit list, streak, feed, race, leaderboard, chat) and
  benefits from being seen *together* at a glance — in-screen tabs fragment it and hide the "how's
  my group doing overall" answer behind taps.
- It **matches Forge's own established pattern.** The Progress page is already "an overview of cards,
  each with a *See All* drill-in to a detail screen." Reusing that keeps the app consistent and
  keeps each heavy feature (full Leaderboard, Team Streak Calendar, Chat, Race detail) on its own
  focused screen where it needs the room.
- **Chat specifically wants its own full screen** (keyboard, scrollback) — never an inline card.

### The overview page, top to bottom (with each feature's phase tagged):

```
‹ Groups                Morning Grind                 [ ••• ]   ← settings menu
──────────────────────────────────────────────────────────────
  ⬡  Morning Grind
  [👤][👤][👤][👤] +1   ·   5 members
  ┌───────────────────────────┐
  │   🔥  4-DAY TEAM STREAK    │   ← hero  (Phase F)
  └───────────────────────────┘

  TODAY'S SHARED HABITS                                 (Phase F)
  ┌──────────────────────────────────────────────┐
  │ 🏃 Run 5K        you: ✓   [👤✓][👤✓][👤—][👤—] │  ← tap to log YOUR own
  │ 📖 Read          you: —   [👤✓][👤—][👤—][👤—] │
  └──────────────────────────────────────────────┘

  🏁 HABIT RACE  (if active)                            (Phase G #6 / J)
  ┌──────────────────────────────────────────────┐
  │ "First to 7 runs this week"                   │
  │ You    ▓▓▓▓▓░░ 5/7                             │
  │ Sara   ▓▓▓▓▓▓░ 6/7                             │
  └──────────────────────────────────────────────┘

  ACTIVITY FEED                                    See All › (Phase F basic → G)
  • Sara completed Run 5K · 2h ago
  • Omar earned a 30-day badge · 5h ago
  • You encouraged Sara 👏

  LEADERBOARD                                      See All › (Phase G #1)
  1. Sara   142 pts    2. You  138 pts    3. Omar 120 pts

  SHARED GOALS / CHALLENGES  (if any)              See All › (Phase G #4)
  "Group: 100 runs this month"  ▓▓▓▓░░ 63/100

  TEAM STREAK CALENDAR                                › (Phase G #5)

  💬 GROUP CHAT                                        › (Phase G #9)
──────────────────────────────────────────────────────────────
```

What renders inline vs. drills in:
- **Inline on the overview (Phase F, ship first):** the header + Team Streak hero; today's shared
  habits with per-member completion status (**and the user taps their own row here to complete** —
  this is the core daily action); a short Activity Feed preview.
- **Inline previews with "See All" drill-ins (Phase G):** Habit Race progress (if this group has an
  active race), Leaderboard top-3, Shared Goals/Challenges, Team Streak Calendar.
- **Full-screen drill-ins (Phase G):** the complete Activity Feed, the full Leaderboard, the Team
  Streak Calendar, and **Group Chat** (its own screen).
- **Encouragement / Celebrations (Phase G #2):** appear *in* the Activity Feed, plus a quick
  "👏 encourage" action on each member row — no separate screen.
- **`•••` toolbar menu:** group settings — invite more members (`UICloudSharingController` again),
  edit group (owner only), **Leave Group** (destructive, with the confirmation alert every
  destructive action in this app requires), Seasons (Phase G #7) if/when built.
- **Proofs (Phase G #8):** attach/view photo proof from within a completion or the feed — needs its
  own privacy/moderation review before building (already flagged in `TASKS.md`), so treat its exact
  placement as deferred.

**Build order maps cleanly onto this:** Phase F ships the header + shared-habit list + Team Streak +
Activity Feed preview and *nothing else renders yet*; each Phase G feature slots into its reserved
inline preview + drill-in as it lands, without re-laying-out the page.

---

## 4. Is the Phase J accountability partner a special group *type*, or an ordinary 2-member group?

### Recommendation: an **ordinary group** (2 members, one shared habit, an active Habit Race) — **not**
### a new group type — but with **one special *habit* type** inside it.

Reasoning:
- **Structurally it *is* just a group.** Two members, one `GroupSharedHabit`, and a `GroupHabitRace`
  (Phase G #6) scoped to those two. Everything the general group detail screen (§3) already renders —
  member status, the Race card, a Team Streak, an Activity Feed — is exactly what a screen-time
  accountability pairing wants. Inventing a parallel "AccountabilityPair" type would duplicate all
  of that UI for no structural gain.
- **Reuse is the whole point of Phase J's design.** `TASKS.md` already says Phase J reuses Phase G
  #6's `GroupHabitRace` "just a different triggering metric." Making the *group* ordinary keeps that
  promise; only the *metric* is new.
- **The genuinely special part is the shared habit, not the group.** A screen-time goal is a
  **Destroy-category habit** ("stayed under your limit today"), and — critically — Apple's Screen
  Time framework **never hands the app the actual minutes**; the app only knows a **pass/fail**
  threshold event. So the `GroupHabitCompletion` for this habit carries a **boolean pass/fail**, and
  the shared habit must **not** try to display or sync anyone's real usage numbers (it can't get
  them, and it would be privacy-sensitive if it could). That's a property of the *habit type*, not
  the *group*.

**So:** ordinary group + general group detail screen (no Phase-J-specific group UI), plus a special
**screen-time Destroy habit type** whose completion is a privacy-safe boolean. This means Phase J
needs **almost no new Groups UI** — it needs the Screen Time *habit* (Phase J's own hard part; see
the placement gap in §5) and then drops it into an ordinary 2-person group.

**One nuance to confirm with Bilal (flagged, not resolved):** should a 2-person group *present*
slightly differently (e.g. a "partner" framing / a head-to-head layout instead of a leaderboard
with two rows)? That's a cosmetic presentation choice on top of the same data model — worth a design
decision, but it does **not** require a separate group *type*.

---

# Part 3 — Sweep for other "we never said where this lives" gaps

The Groups gap above is one instance of a general risk: a phase fully decides **what** a feature
does but never pins **where** in the navigation/screen hierarchy it lives. I re-read every P1 phase
(A–M, including B.5 and F.5) hunting for that specific failure mode. Findings below — each is either
a **concrete proposed placement** (same style as Part 2) or, where genuinely ambiguous, an **open
question** stated plainly rather than guessed. **None of this edits `TASKS.md`; Bilal decides.**

### GAP 1 — Phase B.5 (Quran Memorization/Review): the **photo-capture & verification UI has no screen home.** ⚠️ real gap
Phase B.5 decides the *behavior* (photograph the page you worked on; verify it's a Quran page;
dedupe against prior submissions; N distinct photos to satisfy an N-page goal) but never says where
the capture happens or where submitted-page history lives.
- **Proposed placement:** tapping a Memorization/Review habit on **Home** opens a **camera-capture
  sheet** (capture → on-device verify → confirm/reject with a reason), incrementing the day's count
  on success. The per-habit **submitted-pages history** (thumbnails, dedupe record) lives in that
  habit's **detail page** (`HabitDetailView`), reachable from Progress.
- **Still an open decision (already flagged in `TASKS.md` independently):** the verification
  *method* (on-device vs. cloud) — that's a cost/accuracy decision, not a placement one, so it's out
  of scope here; only noting the *UI home* is unspecified.

### GAP 2 — Phase D (Mosque locations): the **"save/manage mosque locations" CRUD screen has no stated home.** ⚠️ real gap
Phase D / the mosque entry describe "a small CRUD screen to add/manage saved mosque locations" but
never say where it's reached from.
- **Proposed placement:** **Settings → Prayer** section (which already exists — it holds "Prayer
  Times / Calculation Method") → a **"My Mosques"** row → the CRUD list (add via "use current
  location" or manual pin; name each). This keeps all prayer configuration in one place. A secondary
  option is surfacing it inside a prayer habit's own settings detail, but Settings → Prayer is the
  more discoverable, feature-agnostic home.

### GAP 3 — Phase J (Screen Time): **two UI homes are unspecified,** one of them constrained by Apple. ⚠️ real gap
Phase J decides the feature but not (a) where the user **picks which apps count** (Apple's
`FamilyActivityPicker`), nor (b) where the **usage report renders** — and (b) is genuinely
constrained: Apple only lets those numbers render inside a sandboxed `DeviceActivityReport`
extension view, never in the app's own code.
- **Proposed placement (a):** the app-selection picker is part of the **Screen Time habit creation
  flow** (a new Destroy-habit setup step — "choose the apps to limit").
- **Proposed placement (b):** embed the `DeviceActivityReport` view inside that habit's **detail
  page**, or a dedicated **"Screen Time"** screen reached from Settings — but flag to Bilal that
  Apple's extension model dictates a lot here and the exact embedding must be verified against
  current `DeviceActivity` docs (Phase J already flags the entitlement risk; this adds the *UI
  placement* risk).
- **Open question:** does the accountability-partner group (Part 2 §4) need any Phase-J-specific
  entry point, or does the user just add the Screen Time habit to an ordinary group? Per §4's
  recommendation, the latter — but worth confirming.

### GAP 4 — Phase F.5 (Personal iCloud sync): **no sync settings / status surface.** minor gap
Sync is mostly invisible, but users expect a toggle and a "last synced / sync status" indicator, and
a data-loss-sensitive feature especially wants a visible state.
- **Proposed placement:** **Settings → "iCloud Sync"** — a toggle + last-synced timestamp + a
  plain error state if sync is failing. Low-effort, conventional.

### GAP 5 — Phase K ("always-available how-to-use guide"): **home is under-specified** ("from Settings/Profile"). minor gap
The other two Phase K pieces *are* placed: the onboarding Q&A is a first-launch pre-tab flow, and
the per-template explainer is a sheet shown when adding a template. Only the **persistent reference
guide** is vague.
- **Proposed placement:** **Settings → "Help & How to Use"** (a browsable list of feature
  explainers), optionally mirrored by a **"?"** button in a relevant toolbar. Recommend Settings as
  the canonical home; a toolbar "?" is a nice-to-have.

### GAP 6 — Phase B (premium custom dhikr, "Add your own dhikr"): **entry point is only partly specified.** minor
`TASKS.md` says it's "a new entry in the Islamic template pack's Dhikr group," which is *nearly*
placed but doesn't say whether it's a row in the Add-Section/Dhikr browse flow or elsewhere.
- **Proposed placement:** an **"+ Add your own dhikr"** row at the bottom of the **Dhikr & Tasbih
  section** in the section-add flow (`AddSectionView`-adjacent), gated behind the Islamic pack
  entitlement, opening a small create form (name + description) — reusing the custom-habit creation
  pattern the phase already cites. Mostly a confirmation, not a real gap.

### GAP 7 — Phase H (Apple Watch companion + Siri/App Intents): **any phone-side surface?** open question, low
The Watch app is its own target and Watch pairing is system-handled; Siri/App Intents register with
the system, not an in-app screen — so *most* of Phase H needs no phone-side placement.
- **Open question (not a blocker):** does Bilal want a phone-side **"Apple Watch"** and/or **"Siri &
  Shortcuts"** row in Settings (to explain/toggle the companion, or offer "Add to Siri" affordances)?
  Common in comparable apps, but optional. Stated as a question rather than a proposed placement,
  since it's genuinely a preference.

### Phases with **no** placement gap (checked, for completeness):
- **Phase A** (bug fixes) — edits existing screens; nothing new to place.
- **Phase C** (prayer consolidation + Duha) — the consolidated Adhkar-after-prayer habit opens the
  same Home glass-panel as Phase B; Duha is a normal prayer habit. Placed.
- **Phase E** (3D Milestones) — Milestones already have a home (Progress → Milestones card →
  list → detail); Phase E extends the existing badge rendering. Placed.
- **Phase I** (Widgets) — live in the system widget gallery / Lock Screen; per-widget config uses a
  widget configuration intent, not an in-app screen. No in-app placement needed.
- **Phase L** (marketing copy) — external deliverable, no in-app surface.
- **Phase M** (final polish: animation/sound/haptics) — cross-cutting, touches existing surfaces;
  nothing new to place.

---

*Both Part 2 and Part 3 are proposals only. Nothing here has been applied to `TASKS.md` or code —
Bilal folds in what he wants, or requests any of it as a follow-up build task.*

# /devils-advocate

**Asking AI "are you sure about that?" at every step.**

```bash
npx degit notmanas/claude-code-skills/skills/devils-advocate .claude/skills/devils-advocate
```

---

This is my most creative skill. Here's the problem it solves:

I use Claude Code for everything — designing UIs, planning features, writing code, making architecture calls. It's fast, it's capable, and it's confidently wrong more often than I'm comfortable with.

AI doesn't push back. It doesn't say "that's a bad idea" or "have you considered what happens when this service goes down?" It builds exactly what you asked for, even if what you asked for will fall apart in production. It's optimistic by default — and that optimism ships bugs.

I can't be at every step questioning its output. I don't have the time or, honestly, the expertise in every domain to catch what's missing. So I built a skill that does that for me.

Devil's advocate challenges *thinking*, not just code. It runs after any other skill — or standalone — and systematically pokes holes using real frameworks: pre-mortem analysis, inversion thinking, Socratic questioning. It finds the assumptions nobody stated, the failure modes nobody considered, and the edge cases AI glossed over because it was busy being helpful.

**The data backs this up:** AI-generated code has 1.7x more bugs than human-written code. 75% more logic errors. Not because AI is dumb — because it never stops to ask "should I?"

---

## Pair it with anything

This skill is designed as a review layer. Use it after any other skill, or after Claude builds something.

```
/ux-expert  →  audit the page        →  /devils-advocate  →  challenge the audit
/advisor    →  plan the strategy      →  /devils-advocate  →  poke holes in the plan
Claude Code →  build the feature      →  /devils-advocate  →  find what was missed
You         →  make an arch decision  →  /devils-advocate  →  stress-test the decision
```

Or invoke it directly:
```
/devils-advocate review the auth flow in src/auth/
/devils-advocate challenge my migration plan
/devils-advocate what could go wrong with this API design
```

---

## What it does

1. **Steel-mans first** — articulates why your current approach is reasonable before challenging it. No strawman attacks.
2. **Applies structured frameworks** — pre-mortem ("this shipped and failed — why?"), inversion ("what guarantees failure?"), Socratic probing ("you're assuming X — what if X isn't true?")
3. **Checks blind spots** — security, scalability, data lifecycle, failure modes, concurrency, deployment, and 5 more categories engineers consistently miss
4. **Catches AI-specific gaps** — happy path bias, scope acceptance, pattern attraction, confidence without correctness
5. **Delivers a verdict** — always ends with: "ship it", "ship with these changes", or "rethink this"

---

## Example

You've just had Claude design a user notification system. You run `/devils-advocate`:

```
Steel-man:
  The approach is reasonable — event-driven notifications with a
  queue, template rendering, and multi-channel delivery (email +
  in-app). Clean separation of concerns. Good choice for async.

Concern 1: No delivery guarantee or retry strategy
Severity: Critical
Framework: Pre-mortem

  What I see:
    The queue consumer processes notifications but has no dead-letter
    queue, no retry logic, and no idempotency key. If processing
    fails mid-way, the notification is lost silently.

  Why it matters:
    Users miss critical notifications (password resets, payment
    confirmations). You won't even know it happened — no alerting
    on failed deliveries.

  What to do:
    Add a dead-letter queue with 3 retries and exponential backoff.
    Add an idempotency key (notification_id + channel) to prevent
    double-sends on retry. Alert on DLQ depth > 0.

Concern 2: Template rendering happens inside the queue consumer
Severity: High
Framework: Inversion — "what guarantees this fails at scale?"

  What I see:
    Template rendering (markdown to HTML, variable substitution) runs
    in the same process that dequeues messages. A malformed template
    or missing variable will crash the consumer.

  Why it matters:
    One bad template blocks the entire notification pipeline. Every
    user's notifications stall because of one broken template.

  What to do:
    Render templates before enqueuing (fail fast at send-time, not
    process-time). Or isolate rendering in a try/catch that routes
    failures to DLQ without blocking the consumer.

Concern 3: No rate limiting per channel
Severity: Medium
Framework: Blind spot — scalability

  What I see:
    Nothing prevents sending 10,000 emails in a burst if a batch
    event triggers notifications for all users simultaneously.

  Why it matters:
    Email providers (SendGrid, SES) have rate limits. You'll hit
    them, get throttled or blacklisted, and notifications will fail
    silently or bounce.

  What to do:
    Add per-channel rate limiting in the consumer. Batch emails
    with configurable concurrency (e.g., 50/second for SES).

Verdict: Ship with changes
  The architecture is sound but the delivery pipeline has no safety
  nets. Fix concerns 1 and 2 before shipping. Concern 3 can be a
  fast-follow if you're not expecting high volume immediately.
```

---

## What's inside

```
skills/devils-advocate/
├── SKILL.md                          # Persona + process + rules
└── references/
    ├── questioning-frameworks.md     # Pre-mortem, inversion, Socratic, steel-manning
    ├── blind-spots.md               # 11 categories engineers miss
    └── ai-blind-spots.md            # Where AI specifically falls short
```

Knowledge base: ~1,100 lines across 3 reference files.

---

## What it does NOT do

- **Rewrite code.** It challenges and recommends. You (or Claude) implement the fixes.
- **Manufacture concerns.** If something is solid, it says "ship it." Not every review finds problems.
- **Replace domain expertise.** It catches structural and engineering blind spots, not business logic errors. It'll ask "did you consider failure modes?" but won't tell you which pricing model to use.

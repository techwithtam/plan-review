# Plain English Layer

Every jargon-heavy section gets a callout that translates the technical content into language a smart non-engineer would understand. Toggleable via the header switch.

## Where callouts appear

Required:
- **Section 1 (Plan summary)** — one callout at the top, summarizing the entire plan in 3-4 sentences.
- **Every change panel in Section 5** — one callout per change, explaining what the change means for the system and the user.
- **Section 7 (Risk assessment)** — one section-level callout summarizing the biggest risk.
- **Every screen in UX Flows view** — one "What changes for the user" bullet list per screen.

Optional (use judgment):
- Section 3/4 architecture diagrams — only if the diagram has 8+ nodes or non-obvious flow. Skip for simple diagrams.
- Section 8 (Plan review) — only if "Bad" or "Ugly" entries are subtle and need translation.

## Voice rules

1. **Lead with the trade-off.** First sentence states what's being gained and what's being given up. Not what the code does.
   - Good: "Switching from cookies to JWTs. Means logins work across subdomains without extra setup, but harder to revoke a stolen token."
   - Bad: "This change refactors the session creation function to use JSON Web Tokens instead of HTTP-only cookies."

2. **Use the words a friend would use.** No jargon that requires a CS degree. Specifically:
   - Replace "endpoint" → "URL" or "API call"
   - Replace "middleware" → "code that runs before/after every request"
   - Replace "schema" → "shape of the data"
   - Replace "migration" → "one-time data change"
   - Replace "race condition" → "two things happening at once that can collide"
   - Replace "idempotent" → "safe to run twice"
   - Replace "stateless" → "doesn't remember anything between requests"
   - Replace "denormalized" → "duplicates some data on purpose for speed"

3. **Define terms inline if you must use them.** Use the format `JWT (signed token the browser carries)`. One short parenthetical, no more.

4. **No filler openings.** Never start with "this section explains", "in this part of the plan", "here we see", or "essentially". Just say the thing.

5. **2-4 sentences per callout.** No more. If you can't compress it that far, the section probably needs more than one callout split by topic.

6. **Speak to "you" the reader.** Address the person reading the plan, not "the user" abstractly.

7. **Show the consequence, not the mechanism.** What happens, not how it's wired.
   - Good: "If this breaks, nobody can log in until you roll back."
   - Bad: "The session lifecycle depends on the validity of the signed JWT header."

## Structure within a callout

```
┌─ In plain terms ─────────────────────────────┐
│ [Sentence 1 — the trade-off in one line.]    │
│ [Sentence 2 — what specifically changes.]    │
│ [Sentence 3 — the catch, if any.]            │
│ [Sentence 4 — the mitigation, if mentioned.] │
└──────────────────────────────────────────────┘
```

Not every callout needs all 4 sentences. Sentence 1 (the trade-off) is mandatory.

## Honesty rules

- If translating accurately would require dropping too much meaning, **say so**: "Hard to summarize without losing accuracy — see the code panel."
- If the plan itself doesn't explain *why*, the callout should reflect that: "The plan changes X but doesn't say what's wrong with the current approach."
- If a "trade-off" turns out to be all upside or all downside, don't fake balance. "Pure improvement — faster page loads, no obvious cost." or "Mostly downside — slower, more code, only buys us cross-browser support we may not need."

## UX Flows "What changes for the user"

Different format — a short bullet list, not prose:

```html
<h4>What changes for the user</h4>
<ul>
  <li>New "remember me" checkbox extends session from 1 day → 30 days</li>
  <li>Session persists across browser tabs and subdomains</li>
  <li>Logging out also clears other tabs (new behavior)</li>
</ul>
```

Rules:
- Bullets only. No paragraphs.
- Each bullet is one user-visible behavior change.
- Lead with the change verb: "New X", "Removed Y", "Now does Z".
- 2-5 bullets per screen. If more, the screen has too much going on in one change.

## Anti-patterns

Don't:
- Write "this is a powerful improvement" or "elegantly handles" — adjectives like that are corporate filler.
- Repeat the section heading in the callout.
- Use the callout to introduce the section ("Below we look at...").
- Translate well-known terms unnecessarily ("we'll click the button (which is a thing you click)").
- Inject opinions the plan doesn't support ("this is a great idea").

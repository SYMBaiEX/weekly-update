# Weekly Update Template

Use this structure when drafting the weekly update. Skimmable in under 60 seconds. **Only include sections backed by a configured source** — don't emit "Customer signal" if Gmail wasn't configured, don't emit "Channel activity" if Slack wasn't configured, don't emit "Meetings" if Calendar wasn't configured.

```
:memo: *Weekly update draft — {{start_date}} → {{end_date}}*
_Review, edit, and forward when ready._

*TL;DR*
{{One sentence. The single most important thing a reader needs to know this week.}}

*🏆 Wins*
• {{Thing that shipped or landed. Outcome, not activity.}}
• {{...}}

*🛠️ In progress*
• {{Work underway, with rough % or milestone. Name the owner if not the user.}}
• {{...}}

*🚧 Blockers & risks*
• {{What's stuck, why, and who can unstick it. If nothing, write "Nothing this week".}}
• {{...}}

*📅 Meetings & decisions*       ← only if Calendar is a source
• {{Notable meeting or decision from this week. Who, what, outcome.}}
• {{...}}

*📣 Customer / external signal*  ← only if Gmail is a source
• {{Notable customer feedback, external mention, or inbound.}}
• {{...}}

*💬 Channel activity*            ← only if Slack is a source
• {{Key thread, decision, or signal from tracked channels.}}
• {{...}}

*📈 Metrics that moved*
• {{Only metrics that actually moved. Show the delta, not the absolute.}}
• {{...}}

*🙋 Asks*
• {{What does the user need from the reader? Decisions, intros, reviews.}}
• {{...}}

*🔜 Next week*
• {{Top 3 things the user intends to land. Commitments, not wishes.}}
• {{If Calendar is a source, incorporate upcoming meetings as context here.}}
```

## Drafting rules

- **Grounded bullets only.** Every bullet traces to a real source artifact.
- **Outcomes over activity.** "Shipped checkout v2 to 100%" beats "worked on checkout."
- **Trim ruthlessly.** Max 5 bullets per section; pick the highest-impact.
- **Expand internal codenames** on first use — readers may be outside the team.
- **Links in context.** Slack threads: `<url|[thread]>`. Drive: `<url|[doc]>`. Gmail: `<url|[email]>`. Calendar: `<url|[meeting]>`.
- **Empty-but-configured sections say `_Nothing this week_`** rather than disappearing. Unconfigured sections disappear entirely.
- **Metrics only if real.** If the user's sources don't surface metrics, drop the section.

## Section order

Fixed. Do not reorder even if a section is empty-but-configured. Skip (don't placeholder) sections whose source isn't configured.

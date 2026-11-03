# Client Brief — Meridian Support Pilot

*One page. Fill every line. This is what Priya takes to her leadership.*

---

**Client & pilot**
Meridian — an AI agent that triages and resolves customer support tickets (a coordinator routing to billing / technical / account specialists). Live 3 weeks; closing tickets that aren't actually resolved.

**What's actually breaking it**
The model isn't the problem — the instructions around it are. The coordinator was told, in so many words, that each ticket gets exactly one specialist, and that once the specialist reports back, the ticket is "handled" and should be closed as resolved. It was also told that escalating to a human is a last resort that counts against the team.

So when a customer writes in with two problems, the agent picks one, fixes it, and closes the ticket anyway. On T-4471 the customer had a company-wide login outage *and* a $1,200 overcharge. The agent fixed the login issue, only acknowledged the refund, and marked the ticket resolved. It did what it was told.

**The fix**
One file: the coordinator's instructions (`system-prompt-coordinator.txt`). No model change, no code change.
1. List every problem in the ticket, and bring in a specialist for each one (most urgent first).
2. Check each specialist's result before replying. A problem is only "handled" if it was actually fixed.
3. Only claim what a specialist really did. Close as resolved only when everything is handled; otherwise escalate or ask the customer for what's missing, instead of closing the ticket.
4. Login problems with SSO/Okta always go to the account specialist, even when the error message looks like a technical one. This was a second, smaller routing mistake we found along the way.

**Proof**
Same ticket (T-4471), same model, run 5 times each:

| | Resolved | Cost per ticket |
|---|---|---|
| Before | 0 of 5 | $0.13 |
| After the fix | 5 of 5 | $0.21 |

On three tickets we hadn't tuned against (a second customer with an API problem plus a billing issue, and a security-related account ticket), the first version of the fix resolved 8 of 9. We have not re-run those three after the final routing tweak, and 5 runs is a small sample, so treat 5 of 5 as strong evidence, not a guarantee.

**What it would take**
The change itself is small: one prompt file, already written and tested. What's left is rollout work: re-run the held-out tickets after the last tweak, build a larger set of real multi-issue tickets from Meridian's history to test against (the real guard against regressions), and keep the "resolved only if everything was handled" check running as the prompts change. Constraint to hold: cost per ticket stays around $0.21, since a ticket with two problems now uses two specialists.

**The objection we'll get**
*"Why not just use a better model?"* We tried. We ran the same ticket on a larger, more expensive model (`claude-opus-4-8`) with the original instructions: still 0 of 5 resolved, at about twice the cost per ticket ($0.25 vs $0.13). The instructions weren't changed, so the one-specialist-then-close rule was still in place, and the bigger model didn't get around it. Rewriting the instructions took the same cheaper model from 0 of 5 to 5 of 5. Paying for a bigger model would have cost more and fixed nothing.

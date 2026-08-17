# Native cross-scope delegation with approval routing

An agent in one scope can already hand a task to another scope by creating a one-shot cron aimed at
that scope's channel — the action runs as a turn there, under that scope's own policy. That's a nice
way to keep a scope locked to least privilege and drive it from a broader coordinator scope, and it
works well for reads and prep.

Where it breaks is approvals. If the delegated turn hits a `require_approval` command, `runTrigger`
returns `pending_approval` and fails closed — no approval card is ever posted, because cards are only
raised on the interactive event path (`postApprovalButtons` is called only from the Slack
turn-handler). So an autonomously-delegated turn can't get a human to approve the one sensitive action
it needs; the human has to be sitting in the locked conversation driving it by hand, which defeats the
point of coordinating that scope from elsewhere.

Two things would make this native:

1. A first-class "delegate a task to another scope" primitive, instead of the one-shot-cron
   workaround — which is async-only, membership-gated, and spends a cron record each time.
2. Approval routing across the hop: when a delegated turn raises an approval, deliver it to a
   designated human (e.g. the delegator) instead of failing closed. The human still approves; they
   just don't have to be inside the locked scope to do it.

A synchronous request/response form alongside the current fire-and-deliver async would be a bonus.

The security model stays intact — a human still approves the sensitive action. This only changes
where they can be standing when they do it.

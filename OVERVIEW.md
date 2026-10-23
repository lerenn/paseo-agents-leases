Agents Leases makes sure only one agent at a time works on a shared resource. An agent takes a named lease before touching a PR, a QA environment or anything else with a name you agree on, renews it while it works, and releases it when done. The plugin is generic: which lease names and TTLs to use belongs in your agent instructions, not here.

## How it works

New agents get these MCP tools:

- `lease_acquire(name, ttl, mode?)` takes the lease for `ttl` seconds. Calling it for a lease you already hold renews it.
- `lease_renew(name, token, ttl?)` extends your lease. `lease_release(name, token)` gives it up; without a token it leaves the waiting queue.
- `lease_list()` shows every lease with its holder, mode, expiry and queue.

A lease expires if its holder does not renew it. Each lease carries a handover counter, the fencing token: it goes up on every change of holder, and renew or release with an old token are refused, so a former holder cannot act after losing the lease.

Two modes:

- `queue` (default): if the lease is held the caller is queued and ends its turn. When the lease frees up (release, expiry, archive, force release) the next waiter gets it and is woken with a message starting with "Agents Leases:" carrying its token.
- `takeover`, newest wins: the caller takes the lease at once and the previous holder is told by a message that it lost it. Useful for QA per ticket, or a PR lease per head sha. Waiters stay queued.

A holder or waiter that is archived or gone is dropped automatically and its leases pass on. Wake-up messages are stored before they are sent and retried with backoff, so a failure or a daemon restart does not lose them.

## Setup

By default every new agent gets the tools; create an agent with `PASEO_LEASES=off` to opt out, or set the scope to opt-in and use `PASEO_LEASES=on`. Agents created before the plugin was installed do not have the tools. The tools are served on `127.0.0.1:47826`; set `LEASES_PORT` on the daemon to change the port. Requires Paseo 0.11.2 or later.

## Screen

The Agents Leases sidebar item shows how many agents are waiting. The screen lists active leases (name, holder, mode, expiry, queue) with a Force release button that takes the lease from its holder and passes it to the next waiter, and an Audit tab.

## Settings

Scope, tick interval, the longest TTL an agent may ask for, delivery attempts, retention, and the templates of the two messages agents receive (`{{name}}`, `{{token}}`, `{{ttl}}`, `{{reason}}`).

## Data

Leases, queues, pending messages and the audit log are stored in `~/.paseo/plugin-data/agents-leases/state.json` on the daemon machine and survive restarts. Idle free leases and audit entries are removed after the retention period (30 days by default). Leases sends messages only to agents on the same daemon and makes no network calls.

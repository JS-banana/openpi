---
decision-status: proposed
created: 2026-09-01
last-reviewed: 2026-09-01
applies-to: OpenPI development dependencies, CI compatibility lanes, and documented Pi requirement
owner: OpenPI maintainers
related-issues: "#328"
related-prs: "#333"
supersedes: none
---

# Decision 0003: Set the Pi compatibility floor to 0.84.3

## Context

OpenPI currently documents Pi 0.84.1 and keeps its Pi development packages and lockfile on that release. Caret ranges allow newer compatible releases, but the frozen lock means normal development and CI exercise only the locked version.

Pi 0.84.3 introduced two host contracts that OpenPI now relies on: threshold compaction when a provider correctly leaves usage unknown, and the public `SimpleStreamOptions.toolChoice` surface. The Cursor provider in [PR #269](https://github.com/openpi-dev/openpi/pull/269) can preserve correct unknown usage on Pi 0.84.1, but an all-Cursor session on that host cannot reach threshold compaction because the real `AgentSession._checkCompaction` entry returns before estimating a history with no usage-backed response.

## Decision

- Document Pi 0.84.3 as OpenPI's minimum supported host version.
- Set the three Pi development dependencies to `^0.84.3`; retain `*` peer dependencies because Pi owns the runtime copies used by installed packages.
- Lock normal development to the newest compatible Pi patch currently available, 0.84.4.
- Keep the existing locked Node 22 and Node 24 CI lanes, and add one full Node 22 lane that transiently overrides and verifies the entire resolved Pi package family at 0.84.3 before running the same checks, tests, and package smoke acceptance.
- Handle future baseline changes in their own Issue and PR with an explicit minimum-version lane and a locked-version lane.

## Evidence boundary

The Pi source change [`4495469a5`](https://github.com/earendil-works/pi/commit/4495469a5) adds compaction without provider usage and is first released in Pi 0.84.3. The public tool-choice change [`e5dde9a76`](https://github.com/earendil-works/pi/commit/e5dde9a76) is also first released in 0.84.3.

For PR #269, the provider regression invokes the real `_checkCompaction` entry. Local isolated runs observed no threshold compaction on Pi 0.84.1 and one native threshold compaction call on Pi 0.84.4; both versions passed the static checks, full test suite, source-install smoke, and packed-package smoke. The new minimum-version CI lane is the acceptance evidence for exact Pi 0.84.3. Real-account provider behavior and future Pi releases remain outside this Decision's evidence.

Decision 0002's Pi 0.84.1 references remain historical evidence for that Skill lifecycle decision; this proposal does not rewrite them.

## Alternatives considered

- Keep 0.84.1 as the project floor and document the Cursor limitation indefinitely. This preserves installation breadth but leaves a core Session lifecycle incomplete for a supported provider.
- Pin every Pi package exactly to 0.84.4. This narrows compatible patch uptake and conflicts with the repository's caret-based development dependency policy.
- Change runtime peer dependencies from `*` to `>=0.84.3`. This would make npm express the floor, but conflicts with OpenPI's Pi-host package contract: the host owns those packages, while OpenPI keeps development copies only for checks and tests.
- Upgrade the dependencies inside PR #269. This would mix a project-wide runtime baseline decision into a provider feature and make either change harder to review or revert independently.

## Consequences

Development and the main locked CI lanes exercise Pi 0.84.4, while a separate lane prevents accidental use of APIs newer than the documented 0.84.3 floor. Provider code no longer needs to describe Pi 0.84.1 as fully supporting unknown-usage threshold compaction.

Users on Pi 0.84.1 or 0.84.2 must upgrade Pi to receive the documented OpenPI support boundary. Because peer dependencies remain host-owned wildcards, README and release notes must communicate this requirement; npm alone will not reject an older host. Each additional CI lane increases test time, but the minimum lane carries an independent compatibility invariant rather than duplicating an unbounded version matrix.

## Amendments

None. This record remains proposed until maintainers merge the implementation PR.

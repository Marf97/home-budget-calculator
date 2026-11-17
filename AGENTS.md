# Home Budget Calculator — Agent Instructions

## Project scope and source of truth

Read `docs/requirements.md` and the assigned Kanban task before working. The original owner ticket is `t_9a6c1e02`; its confirmed decisions and execution specification define the acceptance boundary. Do not silently broaden scope or weaken requirements. This is a single-household monthly category budget planner, not a dated transaction register. Use React/TypeScript/Vite, Node.js 24/TypeScript and SQLite, as specified in the requirements.

## Language

- Communicate with the owner in English.
- End-user UI, validation, empty states, chart labels and the user-facing run/backup guide must be Czech, with correct diacritics, `cs-CZ` formatting and CZK currency.
- Code identifiers, new internal technical naming, developer comments and developer documentation use conventional English. Preserve established public contracts and existing document language.
- Review for accidental English UI strings and locale/number parsing errors.

## Privacy and safety

- Use synthetic data only for development, tests, screenshots and review. Do not put real household financial data in source, Git, task comments, logs or reusable skills.
- No application runtime analytics, cloud processing, bank integrations, external fonts/CDNs or unsolicited external data transfer. Bundle runtime assets locally.
- Keep secrets, databases, SQLite WAL/SHM files, backups, runtime data and `.worktrees` out of Git. Never print credentials or put credentials in remote URLs.
- Host publication is loopback-only at `127.0.0.1:3000`. Container-internal `0.0.0.0` is allowed; LAN/public exposure is not. No deployment, paid services, OS/service/firewall/security configuration changes or permission weakening without explicit owner authorization.
- SQLite is not encrypted by default. Document that limit honestly. Use consistent backups or stop the application before copying a database; restore verification must use isolated synthetic data and never overwrite real household records.
- Preserve unrelated files and uncommitted work. Inspect Git status before editing. No destructive Git commands or history rewrites without explicit scoped owner approval.

## Role boundaries and Git publication

- Planner clarifies material decisions and coordinates scoped tasks; workers implement and maintain project documentation; reviewer independently verifies the actual artifacts.
- Workers may prepare local commits and fix reviewer findings, but must never push branches, create the hosted repository, publish PRs or change hosted repository settings.
- Only `reviewer` may publish within the owner's explicitly authorized task scope. Verify the authenticated GitHub identity and intended private repository before remote creation or publication; never infer credential validity from an account name alone.
- The owner authorized a narrow initial documentation/metadata-only bootstrap push to `main` in the new private repository. This is not permission to push application code directly to `main`.
- After the authorized bootstrap, `main` must have the remotely verified protection described in the bootstrap ticket: PRs required, enforcement for administrators/no bypass, and force push/deletion forbidden. Do not invent CI contexts. If the GitHub account plan or permissions cannot enforce the requested protection, block for owner input; never change visibility, spend money or claim protection that is not enforced.
- Later publication uses reviewed branches/PRs only. The owner alone merges application PRs. No agent may merge application changes into `main`, deploy, or treat agent approval as owner approval.
- Concurrent source changes require separate project-linked Git worktrees. Record exact approved predecessor commits and integrate them explicitly; do not assume the owner has already merged predecessor PRs.

## Implementation and verification

- Preserve the detailed data, integer-money, month, revision/conflict, same-origin transport and atomic paste contracts in `docs/requirements.md`.
- Keep README and documentation current when behavior, setup or commands change.
- Establish reproducible commands in the repository before citing them as available. Do not invent test/build results or substitute native tests for required real-container verification.
- Run the relevant unit, SQLite/API, interaction, typecheck, build and browser checks for the task; integration also verifies restart persistence, loopback container publication and isolated backup/restore.
- Review empty and zero-income months, null versus zero, refunds, year boundaries, aggregate precision, category history, conflicts, unsaved input and chart/table reconciliation.
- State exact commands, actual outcomes, workspace, branch and commit in handoffs. Missing infrastructure or evidence must be reported, not hidden.

## Kanban lifecycle and final approval

- Use native Kanban tools and read the full task, comments and prior attempts before work. Block precisely for missing material owner decisions or access rather than guessing.
- Workers submit completed candidates with `kanban_request_review(reviewer="reviewer")`; do not approve their own implementation cards.
- Reviewer returns correctable defects to the original implementer with `kanban_request_changes`, and approves only with independent evidence. Escalate after three unsuccessful changes-requested rounds rather than loop indefinitely.
- All implementation cards and the combined integration/acceptance card must pass independent review. Preserve the integration prerequisite of the original ticket; do not mark prerequisites complete merely to release the queue.
- The original ticket remains open for final owner review. Only the owner approves/completes final delivery; subtask approval does not authorize publication outside the agreed scope, a merge or deployment.

## Maintaining this file

Planner/reviewer may request scoped updates to these project instructions, and workers may implement authorized updates. Do not use this file to override owner decisions or weaken safety and publication boundaries. Protected instruction-file writes still require the actual Hermes approval mechanism; a ticket comment alone does not satisfy it. If that gate is denied or times out, stop and report the exact blocker rather than retrying through another tool or path.

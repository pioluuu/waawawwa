# CCS HUB agent instructions

Applies to this repository and its subdirectories. Read this file and
[PROJECT_CONTEXT.md](PROJECT_CONTEXT.md) before writing or changing code. Also
read any more-specific AGENTS.md instructions for the files being edited.

## Working agreement

- Follow the user's current scope. A request marked UI-only stays UI-only; do not
  change backend, database, API, socket, auth, or business logic for a layout fix.
- Inspect the actual files, call sites, contracts, and current git diff first.
  Documentation is context, not a substitute for checking live source.
- With workspace access, search/read/edit files directly. Ask for pasted files
  only if the required source cannot be accessed. Prefer `rg` and `rg --files`.
- Make the smallest safe, complete change. Reuse existing components, hooks,
  services, state, validation, and icon libraries. Avoid unrelated refactors,
  whole-file formatting, dependencies, and duplicate implementations.
- Preserve user edits and concurrent changes. Never overwrite or revert unrelated
  work. Inspect the diff again before reporting completion.
- After each phase, remove temporary test files and artifacts created for that
  phase once validation is complete. Preserve pre-existing repository tests and
  record the checks actually performed in the phase report.
- Use PowerShell-compatible commands in this Windows workspace. Native Python,
  Node, npm, Git, and rg commands can be invoked from PowerShell.
- Keep responses direct and concise. Report exact absolute file paths as clickable
  links, what changed, checks actually run, and material limitations or side effects.
- Apply changes directly when authorized and tools permit. Supply complete files
  or exact find/replace instructions only when the user needs manual application.
  Never present omitted code or invented search text as a complete replacement.

## Non-negotiable invariants

- Preserve server-side authentication, authorization, restrictions, rate limits,
  private storage, and moderation. Do not expose secrets in output or logs.
- Keep chat on the existing SocketProvider/socketService lifecycle. UI changes
  must not create connections, duplicate listeners, or change event contracts.
- Anonymous identities are private from the community, not from moderation.
  Preserve real sender associations internally, alias snapshots in messages and
  replies, sanitized public payloads, and the safe legacy fallback.
- Preserve Shop purchase confirmation, real $RAM accounting, ownership, expiry,
  and Premium validation. UI must call the existing purchase/equip functions.
- Schema changes need explicit additive migrations and a deployment note.
  `create_all()` does not upgrade existing tables. A migration file existing in
  git is not evidence it has been applied to any database.
- Run checks appropriate to the change. Distinguish fixture tests, real database
  tests, and live end-to-end verification. Never claim unperformed checks passed.
- When an authorized change makes PROJECT_CONTEXT.md materially inaccurate,
  update the relevant section; do not accumulate a chat transcript in the file.

User instructions for the current task take precedence over these repository
preferences. System/developer requirements still apply.

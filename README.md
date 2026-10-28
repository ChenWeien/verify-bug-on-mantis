# verify-bug-on-mantis

Cursor / Codex skill for verifying a Mantis bug in iClone or Character Creator against a matching build and application MCP.

## Files

- `SKILL.md` — verification workflow
- `agents/openai.yaml` — agent display metadata

## Usage

Provide a Mantis bug ID and a build/version path. The skill reads the issue, launches the matching iClone or Character Creator build, follows the repro steps, and reports **Verify Pass**, **Verify Fail**, or **Blocked**.

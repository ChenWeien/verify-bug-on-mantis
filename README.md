# verify-bug-on-mantis

Cursor / Codex skill for verifying a Mantis bug in iClone or Character Creator against a matching build and application MCP.

## Files

- `SKILL.md` — verification workflow
- `agents/openai.yaml` — agent display metadata

## Install

Clone this repo into a folder named `verify-bug-on-mantis` under the skills directory. Codex and Cursor both expect `SKILL.md` at `%USERPROFILE%\<app>\skills\verify-bug-on-mantis\SKILL.md`.

### Codex: `%USERPROFILE%\.codex\skills`

```powershell
git clone https://github.com/ChenWeien/verify-bug-on-mantis.git "$env:USERPROFILE\.codex\skills\verify-bug-on-mantis"
```

cmd:

```bat
git clone https://github.com/ChenWeien/verify-bug-on-mantis.git "%USERPROFILE%\.codex\skills\verify-bug-on-mantis"
```

Restart Codex, or reload the skill list, after installing.

### Cursor: `%USERPROFILE%\.cursor\skills`

```powershell
git clone https://github.com/ChenWeien/verify-bug-on-mantis.git "$env:USERPROFILE\.cursor\skills\verify-bug-on-mantis"
```

cmd:

```bat
git clone https://github.com/ChenWeien/verify-bug-on-mantis.git "%USERPROFILE%\.cursor\skills\verify-bug-on-mantis"
```

Restart Cursor, or start a new agent chat, after installing.

## Usage

Provide a Mantis bug ID and a build/version path. The skill reads the issue, launches the matching iClone or Character Creator build, follows the repro steps, and reports **Verify Pass**, **Verify Fail**, or **Blocked**.

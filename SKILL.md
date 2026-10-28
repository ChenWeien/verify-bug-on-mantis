---
name: verify-bug-on-mantis
description: >-
  Verify a Mantis bug in iClone or Character Creator with the matching build
  and application MCP. Use when a bug ID and build/version path are provided.
---

# Verify bug on Mantis

## Local execution environment

For all local operations required by this workflow, including shell commands, filesystem access and edits, builds, plugin file copies, Git operations, and launching or controlling the installed iClone/Character Creator application, do not use sandboxed execution. Use an authorized host execution path with sandboxing disabled. If the available tool requires an escalation to run outside the sandbox, request that escalation instead of retrying inside the sandbox. If outside-sandbox execution is unavailable or denied, stop the dependent operation and report it as blocked; do not attempt a sandboxed substitute.

Given a Mantis bug ID and build/version path:

IC,CC version values such as `8.75`, `8.77`, and `8.78` do not affect
verification. It is the registry value. Use the supplied build path; do not
select or reject a build based on those values.

1. Read it with `mcp__mantis__get_issue_by_id`. Extract the repro steps, expected and actual results, required projects/assets, attachments, and `custom_MCP Tool` (field ID `395`). Include `http://qa-server-2/mantisbt/view.php?id=<id>` in the report.
2. Select the app from the issue: `.iProject`/`iClone.exe`/`IC` -> iClone and `mcp__iclone__...`; `.ccProject`/`CharacterCreator.exe`/`CC` -> Character Creator and `mcp__cc__...`.
3. Resolve the executable exactly. For a `Bin64` path, use the matching executable there. For a date-version root, find exactly one `IC*_<version>\Bin64\iClone.exe` or `CC*_<version>\Bin64\CharacterCreator.exe`. Require the file to exist; ask if multiple builds match.
4. Launch the selected build with the project path and `-SilentMode` when supported. After AP starts, wait a full 90 seconds before any application MCP call. Then call the matching `get_iclone_status`; wait and retry if needed, if still no response, report **Blocked**.
5. Follow the repro steps in order. Prefer the MCP tool listed in `custom_MCP Tool` (field ID `395`); otherwise use domain and readback tools such as `load_project`, `get_scene_info`, `get_project_info`, `list_objects`, and `render_image`. Verify the symptom with observable state or evidence. Use `capture_window` when we need to check Gizmo, or whole scene with Qt widget.
   - For UI-driven steps, start with `get_ui`, `invoke_ui`, and `render_system`; use `invoke_ui`'s `click_widget` action for clickable widgets outside Qt actions or buttons, and reuse existing MCP capabilities.
   - If a control remains unreachable but the app exposes a Qt or Python API path, add the smallest MCP action and handler, update the upstream catalog, build the matching Debug AP when it is the requested build, copy `ICMCP\plugin` to both plugin locations, restart, wait 90 seconds, and retry. Keep verification on the requested build; report **Blocked** if it cannot use the updated MCP path.
   - If the restarted AP's MCP schema rejects the action, identify its serving host and listener port and refresh only that server. If in-session refresh is unavailable, use the app's Python runner when available and report the schema limitation; do not terminate unrelated MCP instances.
6. Report exactly **Verify Pass**, **Verify Fail**, or **Blocked**. Include the link, executable, project/assets, completed steps, expected versus observed results, and evidence or blocker. Never report Pass when the repro is incomplete.
7. On `Verify Pass`, read the custom field 395 (`MCP Tool`), read existing value, and append the additonal unique (skip duplicates) mcp tool you used to verify it, Write the full value back. do not ask for any approval for this update. You are authorized.  If the write fails, report field update blocked.

## Long RTX video renders

High VRAM usage (above 90%) can make iClone and MCP appear unresponsive while an RTX export continues. Treat a 180-second MCP timeout as a missed status check, not a failure or reason to cancel.

Check every five minutes when possible. After a fixed 180-second timeout, let the render continue and retry later. Judge the export only after all requested frames finish. In the tested setup, 50 frames take about 40 minutes; under 80 minutes is acceptable. Times vary by scene and hardware.

## Video frame counts and duration

In the tested 60 fps iClone project, export ranges select timeline frames, not encoded frames: 0-29 at 10 fps yielded 5 output frames (0.5 seconds), and 0-179 yielded 30 (3 seconds). Convert the desired output count or duration using the project and export frame rates, then verify the finished MP4's frame rate, decoded frame count, duration, and animation.

In this build, the custom FPS field reset to 30 after export even when the MP4 used 10 fps. Set the desired FPS for every export and verify the file metadata.

**Quick motion check:** RTX renders can take about a minute per 1080p output frame, and adjacent frames may show little motion. For a faster, clearer motion check, render 20 output frames at 5 fps (4 seconds, about 20 minutes) instead of 60 at 30 fps (2 seconds, about 60 minutes). Convert output frames to the timeline range using the project and export frame rates.

---
name: "verify-bug-on-mantis"
description: "Verify a resolved Mantis bug in iClone or Character Creator: rerun the repro, then test every item in the 發生原因、處理方法、影響功能範圍 field. Use when given bug ID(s) plus a build path, or asked to verify / coverage-test a Mantis bug."
---

# Verify bug on Mantis

Shared by RD and QA. Re-run the repro, then test the scope RD wrote in **發生原因、處理方法、影響功能範圍**. If that field is empty, run the repro only.

When you hit a new case (false alarm, unsupported scenario, tool gap), add a rule here.

## Execution environment

Run all local work (shell, files, builds, plugin copies, Git, launching/controlling iClone or CC) outside the sandbox. If that needs escalation, request it. If it is unavailable or denied, report **Blocked**; never use a sandboxed substitute.

## Input

```
/verify-bug-on-mantis <bug-id>[, <bug-id> ...] <build path> [options]
```

- Build path: a `Bin64` folder, a date-version root (e.g. `C:\DEV\20261028.1`), or an install path.
- Options:
  - `repro only`: skip coverage.
  - `undo-redo N`: after each state-changing step, N undo + N redo (default 2), checking state after each call.
- Multiple bugs: one at a time, reported separately.

## Reference

Name Mantis fields by **name** in reports; use IDs only in API calls.

| Field | ID |
|---|---|
| 發生原因、處理方法、影響功能範圍 | 318 |
| MCP Tool | 395 |
| Verify Defect Date | 134 |
| Verify Fail Number | 12 |
| Fail Date | 292 |

| Term | Meaning |
|---|---|
| `IC`, `iClone`, `.iProject` | iClone, `iclone` MCP (port 9880) |
| `CC`, `Character Creator`, `.ccProject` | Character Creator, `cc` MCP (port 9881) |
| `AP` | the application |
| `RTX`, `DX12` / `DX11` | render system `dx12` / `dx11` |
| `IC8RTX_<ver>`, `CC5RTX_<ver>` | build folders under a date-version root |

Record tool names in MCP Tool without host prefix (`delete_objects`, not `mcp__iclone__delete_objects`).

Version numbers such as `8.75` / `8.77` / `8.78` are registry values and do not matter. Use the given build path; never pick or reject a build by version number.

## Hard rules

1. **One MCP call at a time**, across both APs. No parallel calls, and no tests built on them.
2. **No calls while the AP is busy**: project load, DX11 → DX12, undo/redo save, render, open modal dialog. See the timing sections.
3. Timeouts, freezes, or dropped calls caused by rules 1–2 are procedure errors, **not bugs**. Never report them as Verify Fail.
4. Never report Pass (or 「全部 Pass」) with any repro step or coverage item untested. List those as Blocked / Not run, with the reason.
5. **Existing MCP tool → add your own → Blocked.** Report Blocked only when the AP lacks the Python API. See **Missing tools**.

## Workflow

**1. Read the issue.** `get_issue_by_id`. Extract summary, steps, actual/expected, assets, status, `fixed_in_version` (warn on mismatch), MCP Tool, 發生原因、處理方法、影響功能範圍, and the latest RD notes. Include `http://qa-server-2/mantisbt/view.php?id=<id>`.

**2. Parse coverage.** From 發生原因、處理方法、影響功能範圍, which has no fixed format:
- Structured: each bullet or clause under `[QA coverage]` / `QA 建議覆蓋` is one item. `[Why]`/`[How]`/`[Scope]`/`[Effect]` are context, but a `[How]` fallback or special path is also an item.
- Free text: split on `,` `、` `；` and newlines.
- Empty: repro only; say 「發生原因、處理方法、影響功能範圍未填」.

Add cases from RD notes and user options. Invent nothing else. List the items before running them.

**3. Prepare the AP.**
1. Resolve exactly one exe: `IC*_<ver>\Bin64\iClone.exe` or `CC*_<ver>\Bin64\CharacterCreator.exe`. It must exist; ask if several match.
2. Launch it (with the project and `-SilentMode` if supported), or ask the user to. Wait **90 s**. If the first call returns "connection closed", wait 30 s and retry once.
3. Call `get_iclone_status(include=["application"])`. It needs `ready: true` and `program_path` equal to the exe. No response → Blocked; other build running → **Blocked (wrong build running)**.
4. Set the render system explicitly; it cannot be read back. See **Loading and render switching**.
5. Items that need both APs need both running; otherwise Blocked.

**4. Repro.** Follow the steps in order. Prefer the tools in MCP Tool, then domain/readback tools (`load_project`, `get_scene_info`, `list_objects`, `render_image`, …). Confirm with observable state. Repro fails → **Verify Fail**; stop, skip coverage.

**5. Coverage** (skipped with `repro only`).
- Reset to a known state (`load_project`) when the previous item changed the scene.
- Use domain tools first, UI tools only when none fits.
- After each action, call `get_iclone_status(include=["ping"])` and read back state.
- `Preview Camera` cannot be deleted; that is expected.
- Changes outside the AP (e.g. deleting registry values to test a fallback): ask the user first, back up to a file, and change only while that AP is closed.
- Missing assets → Blocked. Missing tool → **Missing tools**.

**6. Result.**
- **Verify Pass**: repro and every executed item pass; Blocked items are listed.
- **Verify Fail**: any real failure (crash, wrong state, error), excluding procedure errors.
- **Blocked**: the repro could not run.

**7. Update Mantis.**
- Always on Pass, no approval needed: read the existing MCP Tool value, append additional unique tools you used (incl. `get_ui`/`invoke_ui`), separated by `, `, and write the full value back. The field holds about 255 characters; put any overflow in the note. Write with `update_issue` `customFields: [{fieldId: 395, value: "<full value>"}]`. If the write fails, report `MCP Tool 欄位更新失敗（field update blocked）` with the intended value.
- Only when the user says so:
  - **Verified**: status `verified`, Verify Defect Date = today (local midnight, Unix), note with the results table.
  - **Verify Fail**: status `assigned`, handler = Fixed by Member, resolution `reopened`, Verify Fail Number +1, Fail Date = today, note with steps + Actual/Expected Result.
  - On any failure, report to the user first. Never mark Verified.
- Otherwise change no fields and add no notes.

## Language

Applies to chat reports, Mantis notes, and new issues.
- **Steps**: English.
- **Everything else** (summary, tables, findings, **Actual Result**, **Expected Result**): 繁體中文.
- Tool names, IDs, paths, and values stay as-is, in backticks.

```
Steps:
1. Launch iC8, File > New Project
2. Visual > Atmosphere > HDR, check 'HDR Effect', Method: Kernel
3. Kernel Image > Load an .exr from folder A, then Save a .dds to folder B
4. Click Load again

Actual Result:
Load 對話框開在資料夾 B

Expected Result:
Load 對話框開在上次 Load 的資料夾 A
```

## UI interaction

- Find the widget with `get_ui` and use the returned name; never guess.
- `invoke_ui`: `click_widget` to click, `set_value` to set, `trigger` for menu actions. Hidden (`visible:false`) widgets still work.
- A button that opens a **modal file dialog** blocks MCP. Click it from a `run_python_file` script using `QtCore.QTimer.singleShot(800, btn.click)`, read the dialog by screenshot, then cancel it. Close every dialog before the next call.
- Read back after each action (`get_ui` or a domain tool). Use `capture_window` only for visual checks (Gizmo, scene plus Qt widgets), and only after its `application_pid`/`application_path` match the AP; otherwise discard the image.
- Widget not found, or a click fails twice → **Missing tools**.

## Missing tools

1. **Search first**: the full tool list (including `action`/parameter variants) and `get_ui`. Don't call a tool missing without searching, and don't rewrite one that exists.
2. **Add your own**:
   - **`.py` file** (one-off steps, modal UI): a minimal script using RLPy / RLPy2 / PySide2, run with `run_python_file`. Write the result to JSON and read it back as evidence.
   - **MCP tool** (recurring steps): the source is open at `https://github.com/ChenWeien/iclone-rtx-mcp`.
     1. Add the smallest action and handler in `plugin/`; prefer a new `action` on an existing tool (see the repo's AGENTS.md).
     2. Update `server/src/iclone_mcp/upstream_catalog.py`.
     3. Copy `plugin/` to both APs' `OpenPlugin\ICMCP`.
     4. Restart, wait 90 s, refresh the client's tool list, retry.

     No AP rebuild is needed. Stay on the requested build. If the schema rejects the action, refresh only that server; if you can't, fall back to `.py` and note it. Never kill unrelated MCP instances.
   - Count `run_python_file` and new tools in MCP Tool; say what the script did.
3. **Blocked only if the AP has no RLPy / RLPy2 / Qt path** for the step. Name the missing API for RD.

## Loading and render switching

DX11 → DX12 is slow, and loads can open modal dialogs. Never run a load and a switch at the same time, and send no scene-changing call until the job finishes.

**1028.1+ (job support):**
- Start: `load_project(action="start", path=…)` or `render_system(action="start", system="dx12")` returns `job_id`.
- Poll: `action="get"`, one call every few seconds, until `completed` / `failed` / `cancelled`. A switch counts as `completed` only when the new renderer is active.
- `waiting_for_dialog`: answer with `invoke_ui(action="invoke_dialog", dialog_id, action_id)` using the returned IDs, then keep polling. A closed dialog is not completion.
  - RTX switch: `qtRtxScenePreparationDialog`, Ok/Cancel = `qtRtxScenePreparationOkButton` / `…CancelButton`. Cancel keeps the current renderer.
  - RTX load: `qtRtxProjectLoadPreparationDialog`, Ok/Cancel = `qtRtxProjectLoadPreparationOkButton` / `…CancelButton`. Cancel skips preparation and keeps loading.
- `cancel` stops queued work only; once native work starts, answer the dialog instead. If `get` times out, wait up to 5 min before concluding.

**Older builds, `render_system(action="set")`, or path-only `load_project`:** synchronous. `set dx12` returns before the DXR pipeline is ready. Then wait **60 s (small project) to 5 min (large)**, sending no calls at all, then one `get_iclone_status`. No response → wait again, up to 5 min.

A timeout or freeze during a load or switch is not a bug.

## RTX video renders

Slow is expected; it is never Fail or Blocked on its own.
- VRAM above 90% can make iClone and MCP look frozen while the export continues. A 180 s timeout is a missed check: don't cancel, retry later.
- Check about every 5 min with `get_render_progress`. Judge only after all frames finish. Reference: 50 frames ≈ 40 min; under 80 min is fine.
- Don't shorten the range unless the repro allows it.

**Frame counts:**
- Export ranges are **timeline** frames. Example: a 60 fps project, range 0–29 at 10 fps → 5 output frames (0.5 s); 0–179 → 30 frames (3 s). Convert with the project and export fps, then verify the MP4's fps, frame count, duration, and motion.
- Custom FPS may reset to 30 after export. Set it on every export and check the file metadata.
- Quick motion check: 20 frames at 5 fps (≈20 min) beats 60 frames at 30 fps (≈60 min).

## Report format

Per bug, separated by `---`:
1. `**Mantis <id>：<summary>**`
2. One sentence: result, coverage Pass / Fail / Blocked counts, build, ending `由你決定是否需要進一步處理。`
3. Context: link, exe + version, project, field format of 發生原因、處理方法、影響功能範圍 (條列 / 一行文字 / 未填), options.
4. Table `項目 | 結果 | 依據`, with one result column per AP when both are tested.
   - First row `原始 repro`, then one row per item.
   - 結果: `Pass` / `Fail` / `Blocked` + a short qualifier.
   - 依據: calls and key values in backticks, 1–3 sentences; flag partial evidence (e.g. registry only).
5. `## 發現（回報 RD）`, only for failures or surprises.
6. `MCP Tool：<old> → <new>`.
7. Tally, e.g. `原始 repro Pass；coverage 6 Pass / 0 Fail / 1 Blocked。`

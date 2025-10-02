# Development Guidelines

## Core Principles
- Maintain compatibility with Jianying/CapCut 5.x `draft_content.json`; avoid relying on encrypted 6.x fields unless you can prove backward compatibility.
- Optimise for explicit, reproducible APIs over cleverness—timeline bugs are costly to trace once exported.
- Preserve legacy snake_case aliases exposed through `pyJianYingDraft.__init__`; downstream projects still import them.
- Keep cross-platform behaviour in mind: timeline editing must remain Windows/macOS/Linux friendly, while automation stays safely gated behind `ISWIN` checks.

## Workflow Expectations
1. Study similar implementations (for example `video_segment.py`, `audio_segment.py`, `template_mode.py`) before designing new APIs.
2. Reproduce the scenario with a real draft: point `DraftFolder` at your `CAPCUT_DRAFT_DIR` and inspect the JSON using existing helpers.
3. Implement with type hints and meaningful docstrings. Follow the repo’s tone—Chinese explanations are fine when they match surrounding comments.
4. Validate by running `demo.py` or the scripts under `tests/`; confirm CapCut opens the generated draft without “repair” prompts.
5. Update surfaced documentation (`README.md`, `LLM_helper_guide.md`, `.ruler/` docs) whenever behaviour or prerequisites change.

## Coding Standards
- Target Python 3.8+ and stick to language features available in that baseline.
- Represent time with the helpers in `time_util.py` (`tim`, `trange`, `Timerange`); never hard-code microsecond math.
- Surface domain errors with the dedicated exceptions in `pyJianYingDraft/exceptions.py` instead of generic exceptions.
- Avoid mutable default arguments; create new dataclass instances or use `None` sentinels, mirroring the existing constructors.
- Match the project’s naming conventions (`CamelCase` classes, snake_case functions) and keep docstrings concise but instructive.

## Template and Replacement Workflows
- Imported tracks retain their original material IDs—when cloning data, duplicate the JSON blobs rather than rebuilding structures from scratch.
- Before extending `replace_material_by_seg` or `replace_text`, confirm the shrink/extend logic handles overlapping segments and different media types.
- Always propagate metadata into `imported_materials`; missing entries will cause CapCut to refuse the draft or drop effects.

## Testing and Verification
- Smoke-test new features with `python demo.py`, adapting it to cover the new scenario when useful.
- Use `python tests/create_mock_template.py` to stress template imports and material replacement; ensure local placeholder assets exist first.
- Run `python tests/inspect_draft.py` against a real project to verify metadata assumptions and to capture `resource_id` values for documentation.
- Keep linting clean with `flake8 pyJianYingDraft tests` before opening a PR.
- For automation tweaks, run `JianyingController.export_draft` manually on Windows and capture console output for users.

## Safeguards
- Never commit real media, generated drafts, or CapCut cache files; rely on `.gitignore` and local assets.
- Route filesystem writes through `DraftFolder` to prevent orphaned directories or half-written drafts.
- Document manual prerequisites (e.g. enabling legacy UI options, preparing asset folders) wherever code depends on them.

# AGENTS.md

## Dev environment tips

- Prefer a local virtualenv: `python -m venv .venv` and activate with `. .venv\Scripts\Activate.ps1` (Windows) or `source .venv/bin/activate` (Unix).
- Install the project editable plus runtime deps: `pip install -e .` followed by `pip install -r requirements.txt`.
- Set `CAPCUT_DRAFT_DIR` (or add it to `.env`) so scripts know where to read/write CapCut drafts before running examples.
- Use `python -m pymediainfo <media>` if a material fails to load; check duration/resolution against the CapCut draft expectations.
- Keep automation code guarded with `if ISWIN:`—`JianyingController` only works on Windows with the CapCut UI available.

## Testing instructions

- Run `python demo.py` to smoke-test audio/video/text segment creation after edits.
- Use `python tests/create_mock_template.py` to exercise template workflows; populate `tests/mock_assets/` with placeholder media first.
- Inspect existing drafts via `python tests/inspect_draft.py` to confirm metadata assumptions and captured resource IDs.
- Lint before committing: `flake8 pyJianYingDraft tests` (config in `.flake8`).
- When automation changes, manually call `python -c "from pyJianYingDraft import JianyingController; JianyingController().export_draft(...)"` on Windows and verify the export completes.

## PR instructions

- Title format: `[pyJianYingDraft] <Title>`.
- Ensure `python demo.py` and any touched test scripts succeed; rerun `flake8 pyJianYingDraft tests` for style/type checks.
- Document new behaviours in `README.md`, `LLM_helper_guide.md`, or `.ruler/` notes if the workflow changes.
- Avoid committing generated drafts or proprietary media—use local assets only and keep `.gitignore` intact.

# pyJianYingDraft Common Commands

## Environment Setup
```powershell
# Create and activate a virtual environment (Windows PowerShell)
python -m venv .venv
. .venv\Scripts\Activate.ps1

# Install the library in editable mode and runtime dependencies
pip install -e .
pip install -r requirements.txt

# (Optional) install linting tools used in CI
pip install flake8
```

Set `CAPCUT_DRAFT_DIR` to the folder that CapCut/JianYing uses for drafts (or add it to `.env`). Example:
```powershell
setx CAPCUT_DRAFT_DIR "C:\\Users\\<you>\\AppData\\Local\\JianyingPro\\User Data\\Projects\\com.lveditor.draft"
```
Restart the shell so the variable becomes available to Python scripts.

## Running Examples and Utilities
```powershell
# Minimal smoke test: creates a demo draft with audio/video/text
python demo.py

# Generate a feature-rich mock template (ensure placeholder media exist)
python tests/create_mock_template.py

# Inspect an existing draft, list tracks/material IDs
python tests/inspect_draft.py
```

Add media files to `tests/mock_assets/` before running the template script, or update `PLACEHOLDER_ASSETS` inside the script.

## Linting and Static Checks
```powershell
# Run flake8 with the settings in .flake8
flake8 pyJianYingDraft tests

# Optional: confirm the package imports cleanly
python -c "import pyJianYingDraft; print(pyJianYingDraft.__version__)"
```

## Packaging
```powershell
# Build source and wheel distributions
python setup.py sdist bdist_wheel

# Install the freshly built wheel in the active environment
pip install --force-reinstall dist/pyJianYingDraft-*.whl
```

## Troubleshooting Helpers
- Use `python -m pymediainfo <path>` to confirm local media metadata matches what `VideoMaterial`/`AudioMaterial` expects.
- When automation fails, run `python -c "from pyJianYingDraft import JianyingController; JianyingController()"` to verify the CapCut window can be located (Windows only).
- Capture debug JSON with `python -c "import pyJianYingDraft as draft; print(draft.ScriptFile(1920,1080).dumps())"` if you need an empty baseline for comparisons.

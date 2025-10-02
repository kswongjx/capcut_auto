# pyJianYingDraft Common Commands

## Prerequisite: Draft Location
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

## Developing Tools on Top (No Install Required)
Create scripts in the repo root that import the library directly. Example CLI scaffold:
```python
#!/usr/bin/env python3
import argparse
from pyJianYingDraft import DraftFolder, ScriptFile
from pyJianYingDraft.time_util import tim, trange
from pyJianYingDraft.video_segment import VideoSegment

def main():
    p = argparse.ArgumentParser(description="Generate a CapCut draft from inputs")
    p.add_argument("--draft-dir", required=True, help="CapCut drafts root (CAPCUT_DRAFT_DIR)")
    p.add_argument("--video", required=True, help="Path to a local video")
    args = p.parse_args()

    df = DraftFolder(args.draft_dir)
    sf = ScriptFile(1920, 1080)
    track = sf.add_track("video", track_name="main")
    seg = VideoSegment(args.video, timerange=trange("0s", "3s"))
    sf.add_segment(seg, track_name=track.name)

    draft = df.create_draft("tool_generated")
    sf.save(draft)
    print("Draft written:", draft)

if __name__ == "__main__":
    main()
```
Run it from the repository root so `pyJianYingDraft` is importable without installing.

## Linting and Static Checks
```powershell
# Run flake8 with the settings in .flake8
flake8 pyJianYingDraft tests

# Optional: confirm the package imports cleanly
python -c "import pyJianYingDraft; print(pyJianYingDraft.__version__)"
```

## Troubleshooting Helpers
- Use `python -m pymediainfo <path>` to confirm local media metadata matches what `VideoMaterial`/`AudioMaterial` expects.
- When automation fails, run `python -c "from pyJianYingDraft import JianyingController; JianyingController()"` to verify the CapCut window can be located (Windows only).
- Capture debug JSON with `python -c "import pyJianYingDraft as draft; print(draft.ScriptFile(1920,1080).dumps())"` if you need an empty baseline for comparisons.

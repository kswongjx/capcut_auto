# Project Architecture

## Overview
pyJianYingDraft is a Python library that generates and manipulates CapCut/JianYing draft projects, providing programmatic editing and export control. It wraps the structure of `draft_content.json` and exposes helpers for timeline manipulation, template reuse, and Windows automation.

## Top-Level Layout
```text
capcut_auto/
├── pyJianYingDraft/         # Library code published to PyPI
├── tests/                   # Integration scripts for exercising the API
├── readme_assets/           # Documentation screenshots and sample media (placeholders only)
├── demo.py                  # Minimal end-to-end draft creation example
├── LLM_helper_guide.md      # English quick-start guide for AI assistants
├── .env                     # Optional CAPCUT_DRAFT_DIR configuration
├── requirements.txt         # Runtime dependencies (pymediainfo, imageio, uiautomation on Windows)
└── setup.py                 # Packaging metadata
```

## Library Modules (`pyJianYingDraft/`)
### Draft lifecycle
- `draft_folder.py`: manages CapCut draft directories, providing `create_draft`, `duplicate_as_template`, `load_template`, `inspect_material`, and guards against missing folders.
- `script_file.py`: central timeline builder that tracks materials, tracks, segments, and supports import/export, SRT ingestion, template manipulation, and save/dump helpers.
- `assets/`: ships template JSON skeletons (`draft_content_template.json`, `draft_meta_info.json`) loaded by `ScriptFile` and `DraftFolder`.

### Timeline primitives
- `segment.py`: base segment types plus shared utilities such as `ClipSettings`, `Speed`, and keyframe plumbing.
- `video_segment.py`, `audio_segment.py`, `text_segment.py`, `effect_segment.py`: concrete segment implementations offering helpers like `add_animation`, `add_transition`, `add_effect`, `add_fade`, `add_bubble`, and keyframe writers.
- `track.py`: defines `TrackType`, enforces segment alignment, and serialises tracks back into CapCut JSON.
- `time_util.py`: microsecond helpers (`tim`, `trange`, `Timerange`) shared across the API.
- `exceptions.py`: domain-specific errors (for example `SegmentOverlap`, `TrackNotFound`, `AutomationError`).

### Template and metadata support
- `template_mode.py`: wraps imported tracks, `ShrinkMode`, `ExtendMode`, and replacement strategies for template editing.
- `metadata/`: enumerations of built-in CapCut resources (fonts, transitions, filters, effects) referenced by the public API.
- `local_materials.py`: inspects local media via `pymediainfo` to build CapCut-compatible material descriptors.

### Automation (Windows only)
- `jianying_controller.py`: drives the CapCut desktop UI with `uiautomation`, exposing `JianyingController.export_draft` plus resolution/frame-rate helpers.

### Compatibility helpers
- `animation.py`, `keyframe.py`, `template_mode.py`, `util.py`: glue code that serialises CapCut schema fragments and keeps snake_case aliases working through `pyJianYingDraft.__init__`.

## Supporting Scripts
- `demo.py`: builds a 1080p draft with audio, video, stickers, transitions, and text effects; use it as the canonical smoke test.
- `tests/create_mock_template.py`: generates a feature-rich template by reading environment-driven asset paths to validate replacement APIs end to end.
- `tests/inspect_draft.py`: enumerates existing drafts, prints track summaries, and calls `ScriptFile.inspect_material()` for live debugging.
- `readme_assets/`: documentation media and placeholder directories referenced by demos (real media must be supplied locally).

## Data Flow Summary
1. Initialise `DraftFolder` with the CapCut draft root (usually via `CAPCUT_DRAFT_DIR`).
2. Create or duplicate a draft, yielding a `ScriptFile` seeded from `draft_content_template.json`.
3. Add tracks and segments with the type-specific helpers; materials register themselves on first use.
4. For template workflows, obtain `EditableTrack` instances through `get_imported_track` and apply `replace_*` utilities before saving.
5. Persist changes with `ScriptFile.save()` or `dump()`, then open the draft in CapCut or export automatically with `JianyingController`.

## CapCut Compatibility Notes
- JSON structure mirrors CapCut/JianYing 5.x; newer desktop releases encrypt `draft_content.json`, so template workflows require unencrypted drafts.
- Automation routines assume Simplified Chinese UI identifiers; adapting to other locales requires updating the matchers in `jianying_controller.py`.
- Video/audio metadata comes from local files through `pymediainfo`; run scripts on the same platform CapCut will use to avoid codec or path mismatches.

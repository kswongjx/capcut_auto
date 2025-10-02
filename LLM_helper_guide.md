LLM Helper Guide

Purpose: pyJianYingDraft generates and edits CapCut (剪映) draft files (draft_content.json) programmatically, plus optional Windows UI automation for exporting through CapCut Desktop. This guide focuses on building tools and applications on top of the repository (no installation steps).
Project Layout:
pyJianYingDraft/ – core library; key files listed below.
pyJianYingDraft/assets/ – JSON templates for fresh drafts.
pyJianYingDraft/metadata/ – large enum catalogs for fonts, transitions, filters, effects, etc.; import via pyJianYingDraft.metadata.
demo.py – runnable end-to-end example creating a draft.
README.md – Chinese feature matrix and walkthrough; consult for capabilities and asset ids.
Core Concepts:
Time is stored in microseconds. Use helpers tim("1.5s") and trange("0s","5s") from time_util.
ScriptFile orchestrates tracks (Track) and segments (VideoSegment, AudioSegment, TextSegment, StickerSegment, EffectSegment, FilterSegment). It also tracks imported template material and handles JSON (de)serialization.
DraftFolder manages on-disk draft directories (create, duplicate, inspect). create_draft() copies template assets and returns a ScriptFile with save_path set.
TrackType defines allowed segment classes and render order. Track.add_segment() prevents overlaps; handle SegmentOverlap exceptions when adjusting timings.
Materials (VideoMaterial, AudioMaterial) wrap local files, pulling metadata via pymediainfo. Each segment holds a material_id referencing these instances; ensure materials are appended to ScriptFile.materials.
Segment Toolkit:
VideoSegment/AudioSegment extend MediaSegment (in segment.py). Support fades (AudioSegment.add_fade), audio effects, transitions, filters, masks, background filling, keyframes (add_keyframe with KeyframeProperty).
TextSegment handles styled text, fonts, bubbles/effects, and text animations. Styles use normalized floats: colors in 0–1, positions via ClipSettings.
SegmentAnimations (from animation.py) encapsulate intro/outro/loop animations for both text and video; they live in extra_material_refs so the materials block exports correctly.
Template Mode & Imported Content:
ScriptFile.load_template() loads an existing CapCut draft and exposes imported_tracks/imported_materials. Use template_mode.py helpers (ShrinkMode, ExtendMode) when swapping source media while preserving timing and track layout.
import_track() returns either ImportedTrack (read-only) or editable variants that maintain original JSON, allowing safe modifications before re-export.
Metadata Catalogs:
Enums like VideoSceneEffectType, FilterType, TextIntro, FontType provide CapCut IDs plus parse_params helpers. When adding new effects, call .value.parse_params() with 0–100 parameter lists.
The metadata files are large static dictionaries; avoid editing unless refreshing from CapCut updates.
Automation (Windows Only):
jianying_controller.py automates CapCut via uiautomation. JianyingController.export_draft() opens the app, selects a draft, and triggers export; requires app in Chinese locale (control names are localized). Wrap calls in try/except for AutomationError and DraftNotFound.
Automation assumes CapCut ≤ 7.x; version-specific notes live in README.md.
Error Handling & Exceptions:
Custom exceptions in exceptions.py cover missing tracks/materials, overlaps, extension failures, automation issues. Raise/propagate these for clearer tooling feedback.
util.py offers JSON import/export utilities; rely on them when adding new serializable fields to maintain symmetry.
Tool Development Quick-Start:
Create a Python script in the repository root and import the package directly without installing. Example:
```python
from pyJianYingDraft import DraftFolder, ScriptFile
from pyJianYingDraft.time_util import trange
from pyJianYingDraft.video_segment import VideoSegment

def build_draft(drafts_root: str, video_path: str, name: str = "tool_generated"):
    df = DraftFolder(drafts_root)
    sf = ScriptFile(1920, 1080)
    track = sf.add_track("video", track_name="main")
    sf.add_segment(VideoSegment(video_path, timerange=trange("0s", "3s")), track_name=track.name)
    draft = df.create_draft(name)
    sf.save(draft)
    return draft
```

Design Patterns for Tooling:
- Keep scripts small; push timeline logic into `ScriptFile` helpers (KISS/YAGNI).
- Depend on abstractions (`DraftFolder`, segment classes) rather than JSON structure (SOLID-D).
- Validate with `demo.py` and `tests/` scripts; avoid bespoke test harnesses initially (YAGNI).
- Reuse `time_util` helpers and enums; avoid manual microsecond math (DRY).

Development Tips:
Respect microsecond units when computing durations; use helper functions instead of raw ints for clarity.
Whenever you create segments manually, remember to append associated materials (extra_material_refs) so ScriptFile.dumps() exports consistent cross-references.
Keep ScriptFile.content structure aligned with CapCut expectations: materials, tracks, canvas_config, fps, duration. If you add new material categories, update both ScriptMaterial.export_json and import paths.
UI automation relies on timing (time.sleep) and control descriptions; if you tweak sequences, re-run locally with CapCut open to validate selectors.
Extending the Library:
New effect types → add metadata entries and corresponding segment APIs, ensuring exported JSON matches CapCut schema.
New segment behaviors → extend segment.py (or specialized subclasses) and update track acceptance logic if required.
Template processing enhancements → modify template_mode.py, preserving raw_data copies to keep non-edited properties intact.
Always provide docstrings/comments sparingly, matching existing bilingual style (code identifiers in English, comments often Chinese).
Validation & Testing:
No automated tests; validate by generating a draft (ScriptFile.dump) and opening in CapCut. For automation changes, run end-to-end with a sample draft.
demo.py is the canonical smoke test; adjust assets or craft new demos if you add capabilities.

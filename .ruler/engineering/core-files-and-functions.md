# pyJianYingDraft Core Files and Functions

## Entry Points
- `pyJianYingDraft.DraftFolder` (`pyJianYingDraft/draft_folder.py`): validates the CapCut draft root and exposes `create_draft`, `duplicate_as_template`, `load_template`, `inspect_material`, and `remove`. Always construct this first so the library knows where to read/write `draft_content.json`.
- `pyJianYingDraft.ScriptFile` (`pyJianYingDraft/script_file.py`): owns the in-memory draft. It tracks dimensions, materials, tracks, and imported template state. Use `save()` to persist to the path defined by `DraftFolder`, or `dump()` for ad-hoc JSON strings.

## Track and Material Management
- `ScriptFile.add_track(track_type, track_name=None, mute=False, relative_index=0, absolute_index=None)`: registers a new timeline track with automatic render indexing. CapCut enforces that the bottom-most video track starts at 0s.
- `ScriptFile.add_segment(segment, track_name=None)`: inserts `VideoSegment`, `AudioSegment`, `TextSegment`, or `StickerSegment` instances, auto-updating `duration` and material collections. Raises `SegmentOverlap` if time windows collide.
- `ScriptFile.add_effect(effect, timerange, track_name=None, params=None)` and `ScriptFile.add_filter(filter_meta, timerange, track_name=None, intensity=100.0)`: append global effect/filter segments; the underlying implementations live in `pyJianYingDraft/effect_segment.py`.
- `ScriptFile.import_srt(...)`: converts subtitle files into text tracks, optionally cloning style information from an existing `TextSegment`.
- `ScriptFile.get_imported_track(track_type, name=None, index=None)`: select an `EditableTrack` imported from a template for targeted replacements. Pair with `ScriptFile.import_track(...)` to copy tracks between drafts while keeping material IDs consistent.
- Replacement helpers:
  - `replace_material_by_name(material_name, material, replace_crop=False)`: swap every reference to a named audio/video material.
  - `replace_material_by_seg(track, segment_index, material, source_timerange=None, handle_shrink=ShrinkMode.cut_tail, handle_extend=ExtendMode.cut_material_tail)`: fine-grained replacement that respects new durations and collision rules.
  - `replace_text(track, segment_index, text, recalc_style=True)`: update imported text segments or templates while keeping style ranges proportional.
- Diagnostics: `ScriptFile.inspect_material()` prints resource IDs for stickers, bubbles, and animated text effects—use it before hunting through metadata enums.

## Segment Classes and Helpers
- `VideoSegment` (`pyJianYingDraft/video_segment.py`): wraps video/photo materials and exposes helpers such as `add_animation`, `add_transition`, `add_effect`, `add_filter`, `add_background_filling`, `add_mask`, and `add_keyframe`. Accepts optional `source_timerange`, `speed`, and `ClipSettings` for cropping and transforms.
- `AudioSegment` (`pyJianYingDraft/audio_segment.py`): handles fades via `add_fade`, scene/tone effects via `add_effect`, and dynamic volume through `add_keyframe`. Automatically links the corresponding `AudioMaterial` so materials are exported once.
- `TextSegment` (`pyJianYingDraft/text_segment.py`): pairs text content with `TextStyle`, `TextBorder`, `TextBackground`, and `TextShadow`. Use `add_animation`, `add_bubble`, `add_effect`, and `add_keyframe` to recreate animated captions. `TextSegment.create_from_template(...)` clones styles from imported segments.
- `StickerSegment` (`pyJianYingDraft/video_segment.py`): specialised subclass for animated stickers; share most APIs with `VideoSegment` but expect sticker metadata in `metadata` enumerations.
- `EffectSegment` and `FilterSegment` (`pyJianYingDraft/effect_segment.py`): represent dedicated tracks for timeline-wide VFX/LUTs created through `ScriptFile.add_effect`/`add_filter`.
- Supporting data classes (`pyJianYingDraft/segment.py`): `ClipSettings` (position/scale/rotation), `Timerange`/`trange`, `Speed`, `AudioFade`, plus keyframe infrastructure that all segment types consume.
- Keyframe utilities (`pyJianYingDraft/keyframe.py`): `KeyframeProperty` enumerates adjustable parameters (volume, position, opacity, etc.), while `KeyframeList` stores the actual curves.

## Template Mode and Metadata
- `template_mode.py`: defines `ImportedTrack`, `ImportedMediaTrack`, and `ImportedTextTrack` wrappers plus the `ShrinkMode`/`ExtendMode` strategies used by `replace_material_by_seg`. These classes keep the original JSON snippets (`raw_data`) so exports stay lossless.
- `metadata/`: large enumerations of built-in CapCut resources (fonts, transitions, filters, animations, effects). Each enum exposes `.value` objects with `resource_id`, `effect_id`, and parameter metadata consumed by segment helpers.
- `local_materials.py`: creates `VideoMaterial` and `AudioMaterial` instances by probing local files with `pymediainfo`, ensuring duration, resolution, and crop data match CapCut expectations.

## Automation (Windows only)
- `jianying_controller.py`: `JianyingController.export_draft(...)` opens CapCut, clicks through the UI with `uiautomation`, applies optional `ExportResolution`/`ExportFramerate`, waits for completion, and moves the rendered file. All control-finder helpers live inside the same module.

## Timing, Utilities, and Errors
- `time_util.py`: `SEC`, `tim()`, and `trange()` convert human-friendly strings (`"1.5s"`, `"800ms"`) to microseconds; `Timerange` objects power every timeline API.
- `util.py`: reflection helpers (`provide_ctor_defaults`, `assign_attr_with_json`, `export_attr_to_json`) keep template imports faithful to the original JSON shape.
- `exceptions.py`: domain-specific errors (`TrackNotFound`, `AmbiguousTrack`, `SegmentOverlap`, `MaterialNotFound`, `AutomationError`, etc.) signal misuse and should not be silently swallowed.

## Usage Patterns
- The quickest validation path is `demo.py`, which creates a three-track draft in-place and showcases animations, transitions, and text bubbles.
- `tests/create_mock_template.py` demonstrates the template workflow: load `CAPCUT_DRAFT_DIR`, ensure media placeholders, call `DraftFolder.create_draft`, add tracks/segments, then `save()`.
- `tests/inspect_draft.py` is the go-to script for inspecting existing projects; it prints track summaries and calls `ScriptFile.inspect_material()` so you can copy `resource_id` values into code.

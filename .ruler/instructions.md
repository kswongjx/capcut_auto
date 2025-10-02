# pyJianYingDraft AI Agent Instructions

## 📋 Available Instruction Files

Based on different development needs, please refer to the following specialized instruction files:

### 🏗️ Project Architecture

- **File**: `.ruler/project/project-architecture.md`
- **Use Case**: Understanding overall project architecture, tech stack, directory structure
- **When to Use**: New team member onboarding, architectural design decisions, project structure questions

### 🛠️ Development Guidelines

- **File**: `.ruler/engineering/development-guidelines.md`
- **Use Case**: Development processes, best practices, coding standards
- **When to Use**: Development methodology guidance, code review standards

### 🔧 Core Files and Functions

- **File**: `.ruler/engineering/core-files-and-functions.md`
- **Use Case**: Core file locations, function usage, API reference
- **When to Use**: Finding functions during coding, understanding core module usage

### ⚡ Common Commands

- **File**: `.ruler/engineering/common-commands.md`
- **Use Case**: Running examples, linting, troubleshooting, tool CLI scaffolding
- **When to Use**: Daily development, building tools on top, troubleshooting

## Instruction Map

- **Project Architecture** – `.ruler/project/project-architecture.md`: repository layout, CapCut draft data flow.
- **Core APIs** – `.ruler/engineering/core-files-and-functions.md`: `DraftFolder`, `ScriptFile`, segment helpers, template utilities.
- **Development Workflow** – `.ruler/engineering/development-guidelines.md`: coding standards, testing expectations, CapCut-specific caveats.
- **Command Cheat Sheet** – `.ruler/engineering/common-commands.md`: running demos, tool scaffolding, linting, and troubleshooting.

## Supporting References

- `README.md` (Chinese) and `LLM_helper_guide.md` summarise features and FAQs for quick context.
- `demo.py` shows the minimum flow for generating a draft with audio, video, and text.
- `tests/create_mock_template.py` exercises the high-level APIs for template-driven pipelines.
- `tests/inspect_draft.py` lists available drafts and prints material metadata for reverse engineering.

## Working With CapCut Drafts

1. Point `CAPCUT_DRAFT_DIR` (or configure `.env`) to your local Jianying/CapCut draft folder before running scripts.
2. The library targets CapCut/Jianying 5.x JSON format; 6+ encrypts `draft_content.json`, so avoid assuming compatibility.
3. Example assets live outside the repo—never commit proprietary footage, music, or fonts.

## Fallback Sources

- Upstream project documentation and changelog: https://github.com/GuanYixuan/pyJianYingDraft
- Inspect real drafts with `DraftFolder.inspect_material()` when material IDs or effects need confirmation.
- When automation fails, review `JianyingController` logs and verify the Windows UI tree with `uiautomation`’s inspector tools.

# cmux-ka build from source

Unfortunately Kyle can't send the binary/`.app` since it won't open on other people's Macs, due to code signing.

Kyle will most likely pull and handle merge conflicts from upstream at version releases.

> [!NOTE]
> This guide is for macOS 15 only

## Prerequisites

- Xcode (full app, not just Command Line Tools)

## Clone and setup

```bash
git clone https://github.com/kyleawayan/cmux && cd cmux
git checkout ka
./scripts/setup.sh # will probably fail at building ghostty
./scripts/download-prebuilt-ghosttykit.sh # if it failed at building ghostty
```

## Build with ad-hoc signing

```bash
xcodebuild -project GhosttyTabs.xcodeproj \
  -scheme cmux \
  -configuration Release \
  -destination 'platform=macOS' \
  -derivedDataPath ~/Library/Developer/Xcode/DerivedData/cmux-ka-release \
  INFOPLIST_KEY_CFBundleName="cmux-ka" \
  INFOPLIST_KEY_CFBundleDisplayName="cmux-ka" \
  PRODUCT_BUNDLE_IDENTIFIER="com.kyleawayan.cmux-ka" \
  CODE_SIGN_IDENTITY="-" \
  build
```

## Copy to Applications

```bash
rm -rf /Applications/cmux-ka.app && cp -R ~/Library/Developer/Xcode/DerivedData/cmux-ka-release/Build/Products/Release/cmux.app /Applications/cmux-ka.app
```

## Horizontal Workspace Bar and GIFs

To use the GIFs, you'll need to use the horizontal workspace bar. Go to the cmux settings (not Ghostty settings), then under "Sidebar Appearance", change "Tab Bar Position" to "Bottom".

The GIF selection is under the "Automation" section in the cmux settings.

## Known Issues

### White-on-white terminal text

If you have `minimum-contrast` set in `~/.config/ghostty/config`, terminal text will be invisible (white text on white background).

**Workaround:** Comment out or remove the `minimum-contrast` line from your ghostty config.

**Root cause:** A bug in ghostty's `macos-background-from-layer` feature (used by cmux) where the minimum-contrast shader incorrectly sees the background as black instead of white, causing it to force text to white.

---

## Updating to a new upstream release (for Kyle)

1. Sync `manaflow-ai/cmux` main branch with upstream on GitHub
2. Pull locally and rebase `ka` onto `main`:
   ```bash
   git checkout main && git pull
   git checkout ka
   git branch ka-pre-rebase-backup  # safety backup
   git rebase main
   ```
3. Resolve conflicts — see [Conflict resolution notes](#conflict-resolution-notes) below
4. Update submodules:
   ```bash
   git submodule update --init --recursive
   ```
5. Handle GhosttyKit — the ghostty submodule SHA likely changed. `reload.sh` calls `ensure-ghosttykit.sh` automatically, but if the checksum is missing for the new SHA, download manually:
   ```bash
   ./scripts/download-prebuilt-ghosttykit.sh
   mv GhosttyKit.xcframework ~/.cache/cmux/ghosttykit/<NEW_SHA>/
   ```
   Note: `reload.sh` needs sandbox disabled (`dangerouslyDisableSandbox`) to write to `~/.cache`.
6. Build and verify:
   ```bash
   ./scripts/reload.sh --tag ka --launch
   ```
7. Force push:
   ```bash
   git push origin ka --force-with-lease
   ```

### Conflict resolution notes

#### Known conflict-prone files

| File                               | Risk                                   | Notes                                                                                                                   |
| ---------------------------------- | -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `Sources/ContentView.swift`        | High — sidebar, tab bar, traffic light | See rules below                                                                                                         |
| `Sources/TerminalController.swift` | Medium — sidebar mutation              | Use `parseSidebarMutationTabTarget`, not `resolveTabIdForSidebarMutation` (doesn't exist). Keep ka's gif parsing block. |
| `Sources/cmuxApp.swift`            | Low — app setup                        | Keep both main's additions and ka's additions (different purposes).                                                     |

#### ContentView.swift rules

- **`syncTrafficLightInset`**: combine both conditions — `!isFullScreen` (main) AND `!isSidebarVisibleOnLeft` (ka)
- **`TabItemView` property decls**: main refactored sidebar `@AppStorage` vars into `SidebarTabItemSettingsSnapshot`. Do NOT re-add ka's old `@AppStorage` vars. Only keep ka's 5 truly new `let` params:
  - `sidebarShowFolderNameOnly`, `sidebarCardLineSpacing`, `sidebarMonoFontName`, `claudeCodeImageHeightPercent`, `claudeCodeImageAlignment`
- **`TabItemView ==` function**: add `lhs.settings == rhs.settings &&` (main) plus ka's new field comparisons

#### Post-rebase build check

`HorizontalTabBar` (ka-only view) may need `contextMenuWorkspaceIds:` and `settings:` params if main added them to `TabItemView`. Build first to catch compilation errors.

### CodeRabbit review after rebase

To review only upstream changes + conflict resolutions (not ka's own changes):

```bash
coderabbit review --base ka-pre-rebase-backup --type committed
```

If the diff exceeds 150 files, use the `compact-very-large-pr-for-coderabbit` skill below. Unstage noisy files: README translations, i18n messages, media, package-lock, submodule pointers, blog pages.

```md
---
description: Compact a large PR for CodeRabbit CLI review (150 file limit)
argument-hint: <base-branch>
---

## Context

- Base branch: `$1`
- Current branch: !`git branch --show-current`
- Total files in diff: !`git diff --name-only $1...HEAD | wc -l`

## Instructions

CodeRabbit CLI has a 150-file limit. Deleted files count as "files changed", so large PRs that delete old code hit this limit. This command excludes deleted/noisy files from the review.

1. Create a new branch from the current PR branch (e.g. `coderabbit-review-<current-branch>`)
2. Soft reset all commits to staged changes: `git reset --soft $1`
3. Identify which staged files are deletions or vendor noise (e.g. old framework dirs, `.yalc/`, `.sanity-export/`)
4. Unstage those directories: `git reset HEAD -- <paths>`
5. Verify remaining staged file count is under 150
6. Commit the staged files
7. Run: `coderabbit review --plain -t committed --base $1`
8. Present the findings grouped by severity
```

---

## Report Issues

Before reporting an issue here, check https://github.com/manaflow-ai/cmux/issues first, as my fork only does UI changes.

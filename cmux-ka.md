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

## Report Issues

Before reporting an issue here, check https://github.com/manaflow-ai/cmux/issues first, as my fork only does UI changes.

# Nepali Ad Blocker

A Chrome + Firefox extension that hides ads and popups on Nepali websites using
CSS and DOM rules (no network blocking). See [PLAN.md](PLAN.md) for the design.

Supported sites: **onlinekhabar.com**, **setopati.com**.

## Install from a release

Download the zip for your browser from the
[latest release](https://github.com/smnbhattarai/chrome-nepali-news-portal-ads-blocker/releases/latest).

### Chrome (and Edge, Brave, other Chromium browsers)

1. Download `nepali-adblocker-<version>-chrome.zip` and extract it to a folder you
   will keep. Chrome loads the extension from that folder, so deleting or moving it
   removes the extension.
2. Open `chrome://extensions` (`edge://extensions` in Edge).
3. Turn on **Developer mode** (top-right toggle).
4. Click **Load unpacked** and select the extracted folder (the one containing
   `manifest.json`).
5. Optional: pin the extension from the puzzle-piece menu in the toolbar.

To update, extract the new release over the same folder and click the reload icon on
the extension's card in `chrome://extensions`.

### Firefox (128 or newer)

Release builds are not signed by Mozilla, so release Firefox can only load them
temporarily:

1. Download `nepali-adblocker-<version>-firefox.zip` (not the `-sources` zip).
2. Open `about:debugging#/runtime/this-firefox`.
3. Click **Load Temporary Add-on…** and select the zip.

Firefox removes temporary add-ons when it restarts. To keep it installed, use Firefox
Developer Edition or Nightly, set `xpinstall.signatures.required` to `false` in
`about:config`, and install the zip via **about:addons → ⚙ → Install Add-on From
File…**.

## Build from source

Build the extension first (requires Node 22+):

```bash
npm install
npm run build            # Chrome  -> .output/chrome-mv3
npm run build:firefox    # Firefox -> .output/firefox-mv3
```

### Chrome (and Edge, Brave, other Chromium browsers)

1. Open `chrome://extensions` (`edge://extensions` in Edge).
2. Turn on **Developer mode** (top-right toggle).
3. Click **Load unpacked** and select the `.output/chrome-mv3` folder.
4. Optional: pin the extension from the puzzle-piece menu in the toolbar.

The extension stays installed across restarts. After rebuilding, click the reload
icon on its card in `chrome://extensions` to pick up the changes.

### Firefox (128 or newer)

1. Open `about:debugging#/runtime/this-firefox`.
2. Click **Load Temporary Add-on…**.
3. Select `.output/firefox-mv3/manifest.json`.

Firefox removes temporary add-ons when it restarts, so repeat these steps after each
restart. Release Firefox only installs signed extensions permanently; to keep an
unsigned build installed, use Firefox Developer Edition or Nightly, set
`xpinstall.signatures.required` to `false` in `about:config`, and install the zip
from `npm run zip:firefox` via **about:addons → ⚙ → Install Add-on From File…**.

After rebuilding, click **Reload** on the extension's entry in `about:debugging`.

## Development

Requires Node 22+.

```bash
npm install
npm run dev            # Chrome with the extension loaded and hot reload
npm run dev:firefox    # Firefox
npm test               # Vitest fixture tests
npm run compile        # type-check
npm run build          # .output/chrome-mv3
npm run build:firefox  # .output/firefox-mv3
npm run zip            # store-ready zips (also zip:firefox)
```

## Releasing

Pushing a `v*` tag runs [`.github/workflows/release.yml`](.github/workflows/release.yml),
which type-checks, runs the tests, builds both zips and publishes them as a GitHub
release with generated notes. The tag must match the version in `package.json`, or
the workflow fails without releasing.

```bash
npm version 0.2.0 --no-git-tag-version   # bump package.json
git commit -am "Release v0.2.0"
git tag v0.2.0
git push origin master --tags
```

## Rules

Rules live in [`rules/rules.json`](rules/rules.json). The same file is bundled with
the extension and fetched remotely every 12 hours, so selector fixes can ship without
a release:

1. Edit `rules/rules.json` and **bump `version`**. Clients only accept newer versions.
2. Run `npm test` (validates the file and runs the fixture tests).
3. Push to `master`.

Before remote updates work, replace the placeholder in `REMOTE_RULES_URL`
([`src/config.ts`](src/config.ts)) with this repo's raw GitHub URL.

Adding a **new site** needs a release: add its domain to
[`src/sites.ts`](src/sites.ts) (content-script matches are fixed in the manifest),
add its rules, and add a fixture test.

| Field | Effect |
|---|---|
| `hide` | Hidden with injected CSS at `document_start` |
| `remove` | Removed from the DOM (only for elements that block interaction) |
| `popups` | `{ overlay, close? }`: clicks `close` if present, hides `overlay` |
| `unlockScroll` | Resets `overflow: hidden` on `html`/`body` after a popup is dismissed |
| `textMatch` | `{ selector, contains[] }`: hides matching blocks containing the text |
| `linkMatch` | Hides links whose `href` contains a string and that wrap media |

# Contributing

Thanks for helping. This applies to all repositories of [esp-claw (community)](https://github.com/esp-claw) unless a repository has its own `CONTRIBUTING.md`.

## Before you start

- For a bug, open an issue with the bug report template. For a larger change, open an issue or a draft pull request first so we can agree on the approach.
- Security problems: do not open an issue. See [SECURITY.md](SECURITY.md).

## Building

- **Firmware** ([esp-claw](https://github.com/esp-claw/esp-claw)): ESP-IDF project in `application/edge_agent`. See its README (`idf.py build`, `idf.py flash monitor`).
- **App SDK** ([app-sdk](https://github.com/esp-claw/app-sdk)): needs clang with the wasm32 target, `wasm-ld`, Python 3 and OpenSSL 3. Run `tests/selftest.sh`, `shellcheck` on the shell scripts and `ruff check tools/`.
- **Apps** (`app-*`): clone with `--recursive`, then `./sdk/build-app.sh .`, `python3 sdk/tools/manifest_check.py .` and `python3 sdk/tools/wasm_check.py build`.

## Pull requests

- Keep each pull request focused on one change, and describe what changed and how you tested it (on hardware if it touches the firmware or an app's behaviour).
- CI must be green before a pull request is merged.
- Never commit secrets: API tokens, Telegram bot tokens, Wi-Fi passwords, signing keys (`*.pem`) or device configs that contain them.
- Third-party GitHub Actions are pinned to full commit SHAs with the version as a comment. Keep it that way; Dependabot proposes updates.
- A `Signed-off-by` line (DCO) is not required.
- By contributing you agree that your contribution is licensed under the repository's license.

## Submitting an app

1. Create your app from [app-template](https://github.com/esp-claw/app-template) ("Use this template") and follow its README. Its CI uses the reusable app-sdk workflow.
2. Grant only the permissions the app uses in `manifest.json`. Apps that change things through an API should ask with `claw_confirm()` or rely on the device's confirmation for mutating requests.
3. Make sure CI is green, then open an issue in [app-sdk](https://github.com/esp-claw/app-sdk/issues) with a link to your repository if you would like it listed or moved into the esp-claw organisation. Apps published here are reviewed and signed with the esp-claw community key by a maintainer; CI never signs.

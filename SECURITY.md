# Security policy

This policy covers all repositories of [esp-claw (community)](https://github.com/esp-claw), an unofficial community fork of [espressif/esp-claw](https://github.com/espressif/esp-claw). It is not affiliated with Espressif. For a vulnerability that also exists in the upstream code, please report it to Espressif as well.

## Supported versions

| Component | Supported |
|---|---|
| Firmware ([esp-claw](https://github.com/esp-claw/esp-claw)) | the latest `master` |
| App SDK ([app-sdk](https://github.com/esp-claw/app-sdk)) | the latest `main` and the current major workflow tag (`v1`) |
| Apps (`app-*`) | the latest release of each app |

Fixes go into the latest version only. There are no backports.

## Reporting a vulnerability

Do not open a public issue. Report privately through GitHub: open the affected repository, go to **Security > Advisories** and click **Report a vulnerability** ([GitHub documentation](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability)). If the vulnerability affects several repositories, report it in the one where it originates.

<!-- MAINTAINER: private vulnerability reporting must be enabled for this to work
     (org Settings > Code security > Private vulnerability reporting, or per repository).
     If a security contact email exists, add it here as a fallback. -->

Please include the firmware commit or app version, the configuration involved (without secrets) and the steps to reproduce. This is a volunteer project: expect an acknowledgement within a few days, and we will coordinate a fix and disclosure date with you.

### Scope

In scope:

- the firmware in esp-claw (all code this fork adds or changes, and upstream code as shipped here)
- the WASM app runtime, the package format, signature verification and the installer
- app-sdk tools (`ecapp.py`, `manifest_check.py`, `wasm_check.py`) and its reusable CI workflow
- the apps published under github.com/esp-claw

Out of scope:

- attacks that need physical access to the device (see known limitations below)
- vulnerabilities in third-party services (Telegram, LLM providers, Cloudflare, GitHub and others) or in vendored libraries that are not caused by how esp-claw uses them; report those upstream
- the LLM doing something unwanted that it was asked to do by an allowed Telegram user
- apps signed by keys other than the esp-claw community key

## Threat model

The device runs an LLM agent that acts on messages from Telegram. The main threats are strangers sending commands, the LLM being tricked by content it reads (prompt injection), credential theft, and malicious or buggy apps. The firmware addresses them as follows.

**Telegram sender allowlist.** The bot only processes messages from the Telegram user IDs in `tg_allowed_user_ids` (at most 16). An empty list makes the device ignore all Telegram messages.

**Confirmations for actions with side effects.** These ask for an explicit button press in the Telegram chat the request came from, and time out after 60 s (app install: 120 s):

- `ssh_exec`: every command, unless it matches an allow rule that the user added (`/ssh allow`, or from a confirmation prompt). Commands with substitutions, variables, subshells, background jobs, input redirection or output redirection other than to `/dev/null` or stdout/stderr never match a rule.
- `http_request` with a credential profile: every method other than `GET` and `HEAD`, except `POST` endpoints listed in the profile's `read_only_posts`.
- WASM apps: install and remove (`/app install`, `/app remove` and the LLM tools `apps_install` and `apps_remove`), API mutations through credential profiles (same rule as above), and the app's own `claw_confirm()` prompts.

Requests that need a confirmation are refused when they do not come from an allowed Telegram chat, for example from the web chat.

Firmware updates are not done over the air: the firmware has no OTA update path and is flashed over USB.

**SSH.** The device authenticates with its own key pair (`/ssh pubkey` shows the public key) and checks the server's host key against fingerprints pinned in the configuration. A host key that does not match is refused.

**Credential profiles.** API tokens live in the device config (`api_profiles`), and the device injects them into requests. The LLM and WASM apps only name a profile: `http_profiles` lists name, base URL and hosts without the auth section, and the web UI masks secrets. Each profile is pinned to a list of hosts. Requests and redirects must use https and stay on those hosts. On a redirect to a different host than the original one, the device removes the auth header before following it.

**Signed WASM apps.** Apps run in the WAMR interpreter, each in its own linear memory.

- The device installs and loads only `.ecapp` packages signed with Ed25519 by a key in `app_trusted_keys` (at most 8). On install and on every load it checks the format, SDK version, name and version, the SHA-256 hashes of manifest and module, the key ID and the signature. `app_allow_unsigned` disables this for development and must stay off in normal use.
- Packages are downloaded over https only (at most 512 KB), staged, and the previous version is restored if the new one fails to load.
- The manifest declares permissions (`log`, `kv`, `confirm`, `ping`, and up to 4 named `http_profiles`). The runtime refuses to load a module that imports a host function its manifest does not grant, and refuses HTTP calls to profiles the manifest does not name. Apps have no network or file access beyond these host functions.
- Each call runs under a wall-clock budget (`limits.call_timeout_ms`, at most 30 s). When it runs out, the call is stopped and the instance discarded. After 3 consecutive traps or timeouts the app is disabled. Memory, results, HTTP responses, key-value storage and logs have fixed limits per app or per call.
- CI never signs. Maintainers sign CI-built files locally with keys kept outside the repositories.

**Web UI.** The device serves its web UI and API over plain HTTP on port 80. It is meant for the local network only and must not be exposed to the internet (no port forwarding or tunnel). HTTPS is planned.

### Known limitations

- **No flash encryption or secure boot by default.** Anyone with physical access can read the flash, including the NVS with Wi-Fi passwords, the Telegram bot token, LLM keys, `api_profiles` and the SSH private key, or flash other firmware. Enabling ESP-IDF flash encryption and secure boot is up to you.
- **The web UI has no login.** Anyone who can reach the device on the network can use the web UI and its API: change the configuration (secrets are masked when read but can be overwritten, including `api_profiles`, `tg_allowed_user_ids`, `ssh_hosts`, `app_trusted_keys` and `app_allow_unsigned`), list, upload, download and delete files, restart the device and chat with the agent. Keep the device on a trusted network.
- **WebDAV uses HTTP Basic auth over plain HTTP.** It is disabled while `webdav_password` is empty. When enabled, the password crosses the LAN unencrypted.
- **Prompt injection is mitigated, not prevented.** Web pages, API responses, files and app output can contain instructions that the LLM may follow. The confirmations above stop the listed side effects from happening without your consent, but read-only actions (GET requests through profiles, reading files, searches) run without asking, and their results can be sent back to the chat. Read confirmation prompts before approving them.
- **SSH allow rules and `read_only_posts` skip confirmations by design.** Only add them for commands and endpoints that are safe to run unattended.
- **Telegram itself is trusted.** Anyone who controls an allowed Telegram account, or the bot token, can command the device.

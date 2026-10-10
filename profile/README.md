# esp-claw (community)

An unofficial community fork of [espressif/esp-claw](https://github.com/espressif/esp-claw), the AI agent runtime for ESP32. It is not affiliated with or endorsed by Espressif.

The firmware runs on an ESP32-S3 (8 MB flash, 8 MB PSRAM) and is used through Telegram.

## What this fork adds

- **Telegram**
  - Sender allowlist and voice-message transcription.
  - Rich messages (`sendRichMessage`) and a "thinking" draft that shows the tool currently running.
- **Tools**
  - `ssh_exec`, with a Telegram confirmation for every command and opt-in rules for commands that may run without asking.
  - Wake-on-LAN.
  - Credential profiles for HTTP APIs: the model never sees the tokens, and every change needs a confirmation. Cloudflare and Google Cloud skills use them, and the agent can add endpoints to those skills.
- **Memory**
  - Facts extracted at the end of each session, then reconciled with existing memories (add, update, delete or skip).
  - A nightly consolidation, plus recall that combines keywords and embeddings.
- **Storage**
  - A WebDAV share of the device storage that reports free space.
  - Clean-up for attachments and old sessions.
- **WASM apps** (in progress)
  - Sandboxed apps using [WAMR](https://github.com/bytecodealliance/wasm-micro-runtime). Each app declares its permissions in a manifest, and the firmware enforces them.
  - Apps reach APIs only through credential profiles, so the firmware keeps the tokens.
  - Signed packages and an installer are next.

## Repositories

| Repository | Contents |
|---|---|
| [esp-claw](https://github.com/esp-claw/esp-claw) | The firmware, forked from espressif/esp-claw |
| [app-sdk](https://github.com/esp-claw/app-sdk) | SDK for WASM apps: host API headers, build script, package/sign tool, manifest and module checks, reusable CI workflow |
| [app-template](https://github.com/esp-claw/app-template) | Template repository for new apps ("Use this template") |
| [app-hello](https://github.com/esp-claw/app-hello) | Example app: echo, benchmark, key-value demo, runaway-protection test |
| [app-ping](https://github.com/esp-claw/app-ping) | Pings a host from the device (ICMP) |
| [app-cloudflare](https://github.com/esp-claw/app-cloudflare) | Cloudflare zones, DNS (read/write), cache purge, tunnels, workers, analytics |
| [app-github](https://github.com/esp-claw/app-github) | GitHub notifications, PRs, issues, CI runs, failed-job logs, releases (read-only) |
| [app-supabase](https://github.com/esp-claw/app-supabase) | Supabase projects, service health, usage, branches, logs (read-only) |
| [app-search](https://github.com/esp-claw/app-search) | Web and news search via Tavily or Brave |

Apps are signed `.ecapp` packages (Ed25519). The device installs them from a release URL with `/app install <url>` after a confirmation in Telegram, and only loads apps signed by a trusted key.

## License

The firmware keeps the upstream Apache-2.0 license. The SSH tool links [libssh](https://www.libssh.org/), which is LGPL-2.1, statically. Check its terms before you distribute binaries.

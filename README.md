<p align="center">
  <img src="docs/assets/banner.svg" alt="Dex: life assistant, security sidekick, desk buddy" width="100%">
</p>

<p align="center">
  <a href="LICENSE"><img alt="License: Apache 2.0" src="https://img.shields.io/badge/license-Apache%202.0-0E9AA7?style=flat-square"></a>
  <img alt="Status: early development" src="https://img.shields.io/badge/status-phase%200-F2A541?style=flat-square">
  <img alt="Hardware: M5Stack CoreS3" src="https://img.shields.io/badge/hardware-M5Stack%20CoreS3-14213D?style=flat-square">
  <img alt="LLM: Mistral (EU) or Ollama" src="https://img.shields.io/badge/LLM-Mistral%20(EU)%20%7C%20Ollama-5B4BDB?style=flat-square">
  <img alt="PRs welcome" src="https://img.shields.io/badge/PRs-welcome-0E9AA7?style=flat-square">
</p>

<p align="center">
  <a href="#what-is-dex">About</a> |
  <a href="#privacy-and-security">Privacy &amp; security</a> |
  <a href="#how-it-works">How it works</a> |
  <a href="#roadmap">Roadmap</a> |
  <a href="#setup-wizard-and-companion-app">Setup wizard</a> |
  <a href="#choosing-a-brain">Choosing a brain</a> |
  <a href="#contributing">Contributing</a>
</p>

> [!NOTE]
> Dex is in early development. Anything marked **planned** below describes where the project is headed, not what works today. Follow the [roadmap](#roadmap) to see progress.

## What is Dex

Dex is an open-source desk robot: a friend who remembers your day, a life assistant, and a security sidekick for your home network.

Built on [Stack-chan](https://github.com/stack-chan/stack-chan), and also available as a phone app — same brain, memory and skills either way. Everything runs on a small home server you own. Default LLM is Mistral, an EU provider; switch to a fully local model with one config change.

## Privacy and security

Dex has a microphone, remembers your conversations and can run tools — every design choice starts from what that requires.

<p align="center">
  <img src="docs/assets/privacy.svg" alt="Dex privacy at a glance: runs at home, EU-based LLM option (Mistral, Paris), switch to fully local anytime" width="100%">
</p>

- The robot never talks to an AI provider directly and never stores API keys or identity — lose the robot, lose nothing.
- Every request is classified before it leaves Dex core; security findings, credentials, journal and health data never go to a cloud model.
- Default LLM is Mistral, headquartered in the EU. Go fully local at any time with one config change.

```mermaid
flowchart LR
    A[You say something] --> B{Privacy router}
    B -->|Casual chat, general questions| C[Mistral API]
    B -->|Network data, security findings,<br/>credentials, journal, health| D{Local model<br/>available?}
    D -->|Yes| E[Ollama on your server]
    D -->|Not yet| F[Local skill only,<br/>or Dex says it will<br/>wait for local mode]
    C --> G[Dex replies]
    E --> G
    F --> G
```

Sensitive categories are defined in `dex.yaml` and can only be made stricter from the web app, never looser, without re-authenticating.

| Risk | Mitigation |
| :-- | :-- |
| API key extracted from the robot | Keys live only on the server in `.env`. The robot holds no secrets beyond Wi-Fi. |
| Secrets committed to Git | `dex.yaml` and `.env` are gitignored; `dex.example.yaml` is committed instead. Pre-commit hook plus CI secret scanning catch mistakes. |
| Insecure defaults left unchanged | Shipped config defaults to strictest privacy, read-only skills, no machine control. No default passwords — credentials are generated on first boot and shown once. |
| Robot used as a pivot into the home network | Robot sits on its own VLAN and can only reach the voice gateway port. |
| Server exposed to the internet | No port forwarding. Remote access only through Tailscale or WireGuard. |
| Prompt injection from web pages, feeds or emails | Tools are allowlisted and read-only by default. State-changing actions need a physical head-touch. |
| Sensitive data sent to a cloud model | Privacy router enforces local-only categories. Zero-cloud mode in phase 8. |
| Always-on microphone | Wake word gating, a visible LED while streaming, and a mute option. |
| Unauthorized network scanning | Network skills only run against subnets listed in `dex.yaml` that you own. |
| LLM given command execution | Allowlisted commands only, no raw shell, a disposable VM, physical confirmation per command, local model only. |
| Multi-user data leakage | Per-user memory isolation; authentication required before any account beyond the owner can be created. |
| Automatic firmware OTA from vendor | Vendor update checks disabled in custom firmware; updates come only from the self-hosted OTA URL. |

Found a vulnerability? Report it privately via [SECURITY.md](SECURITY.md), not a public issue.

## Features

| Friend | Life assistant | Security sidekick |
| :-- | :-- | :-- |
| Consistent personality, voice and expressive face | Reminders, timers and a morning briefing | Alerts when an unknown device joins your Wi-Fi |
| Remembers past conversations (stored on your SSD) | Calendar and weather at a glance | Daily digest of actively exploited CVEs that affect your software |
| Reacts with head movements, LEDs and moods | Searches your own notes and documents | Summaries of Pi-hole, Suricata or Wazuh alerts |
| Wake word, so it only listens when called | Home and PC control through the server | Certificate expiry checks for your services |
| Physical head-touch to confirm actions | Works offline for local skills | Study buddy for certs and CTF practice |

## How it works

<p align="center">
  <img src="docs/assets/architecture.svg" alt="Dex architecture diagram" width="100%">
</p>

Thin robot, smart server. The robot never talks to an AI provider directly.

- **Dex robot.** Stack-chan on an M5Stack CoreS3 — streams audio, plays speech, renders face and head movement.
- **Voice gateway.** Self-hosted [xiaozhi-esp32-server](https://github.com/xinnan-tech/xiaozhi-esp32-server) — wake word, local speech-to-text and text-to-speech.
- **Dex core.** Holds Dex's personality, memory lookups, tool routing and the privacy router.
- **Memory.** SQLite conversation log plus a vector index built with a local embedding model.
- **Skills.** Small, allowlisted tools for life and security tasks — no raw shell.
- **Web app.** Setup wizard and dashboard, served by your own server.
- **LLM backend.** Any OpenAI-compatible endpoint — Mistral by default, swappable to Ollama or others.

Identity and memory live only on the server, never in firmware. Lose the robot, lose nothing — a new body reconnects to the same Dex.

## Roadmap

<details open>
<summary><b>Phase 0: Foundation</b></summary>

- [ ] Flash and test the Stack-chan firmware on the CoreS3
- [ ] Set up a Linux home server with Docker
- [ ] Put the robot on an isolated IoT VLAN that can only reach the voice gateway
- [ ] Create the repository structure, license and security policy
</details>

<details>
<summary><b>Phase 1: Server and voice</b></summary>

- [ ] Run xiaozhi-esp32-server locally and point the robot at a self-hosted OTA URL, never the vendor's
- [ ] Local speech-to-text with Whisper
- [ ] Local text-to-speech with Piper
- [ ] Confirm audio round-trips entirely inside the home network
</details>

<details>
<summary><b>Phase 2: Brain and API</b></summary>

- [ ] Dex core exposes an OpenAI-compatible endpoint that the voice gateway talks to
- [ ] First version of Dex's persona prompt and mood-to-face mapping
- [ ] Privacy router sits in front of every LLM call
- [ ] Mistral wired in behind the router, provider swappable with one line in `dex.yaml`
</details>

<details>
<summary><b>Phase 3: Memory and tasks</b></summary>

- [ ] Conversation log in SQLite
- [ ] Local embeddings (for example `nomic-embed-text`) and a vector index on the SSD
- [ ] Reminders and to-dos, stored in SQLite
- [ ] CSV import and export for reminders and to-dos (CSV is a transfer format only, never the store)
</details>

<details>
<summary><b>Phase 4: Skills</b></summary>

- [ ] Life: reminders, timers, calendar, weather, morning briefing
- [ ] Security: new-device alerts, CVE digest from the CISA KEV catalog, certificate checks
- [ ] Tool allowlist with read-only defaults
- [ ] Head-touch confirmation for any action that changes something
</details>

<details>
<summary><b>Phase 5: App and second body</b></summary>

- [ ] PWA with Dex's face, microphone input and a WebSocket connection to Dex core
- [ ] Same brain, same memory, whichever body you're talking to
- [ ] WireGuard or Tailscale for remote access to the home server
- [ ] Single-user only; accounts come in the next phase
</details>

<details>
<summary><b>Phase 6: Hardening and shared use</b></summary>

Each user runs their own server — this hardens a single install, not a multi-tenant service.

- [ ] One owner per install by default, with authentication in front of the app and the API
- [ ] Optional extra accounts for household members
- [ ] Invite codes only; open signup is never supported
- [ ] Per-user memory isolation enforced at the database layer
- [ ] Per-user privacy settings
- [ ] Rate limiting and audit logging
</details>

<details>
<summary><b>Phase 7: Smart home</b></summary>

- [ ] Home Assistant integration using scoped tokens
- [ ] Confirmation required before any state-changing action (lights, locks, plugs)
- [ ] Read-only status queries need no confirmation
</details>

<details>
<summary><b>Phase 8: Fully local</b></summary>

- [ ] Ollama on a GPU-equipped server
- [ ] Mistral open-weight models so Dex keeps a familiar personality
- [ ] Zero-cloud mode that blocks all outbound LLM traffic
</details>

<details>
<summary><b>Phase 9: Machine control</b></summary>

- [ ] Disposable Linux VM that Dex can control, reverted between sessions
- [ ] Strict allowlist of commands; no raw shell access
- [ ] Physical confirmation required before every command runs
- [ ] Full audit log of everything the VM executed
</details>

## Hardware

### Required

| Part | What it does | Estimated price | Notes |
| :-- | :-- | :-- | :-- |
| M5Stack StackChan kit (K151) | CoreS3 robot body with servos, camera, mic, speaker, 12 RGB LEDs and touch panel | €120-130 | The complete kit; nothing else needed for the robot |
| Home server | Runs the voice gateway, Dex core and memory | €0-150 | Free if you already have a Linux box, NAS or Proxmox host; a used mini PC with 16 GB RAM covers it otherwise |
| Storage | Memory, vector index, logs | €0-25 | 64 GB minimum, 100 to 200 GB comfortable |

### Optional

| Part | What it does | Estimated price | Notes |
| :-- | :-- | :-- | :-- |
| M5Stack CoreS3 (alone) | Core board for building your own Stack-chan body | €60-70 | Alternative to the full kit above |
| Travel router (for example GL.iNet) | Lets the robot reach your server away from home | €50-80 | Only needed if you carry the robot; the phone app needs nothing |
| GPU for local models (Phase 8) | Runs a local LLM | €250+ | 8 GB VRAM minimum, 12 GB+ recommended |

### Running costs

| Part | What it does | Estimated price | Notes |
| :-- | :-- | :-- | :-- |
| LLM API | Ongoing usage for every conversation | €0 | Mistral's free tier; a few euros a month on paid providers |
| Electricity | Powers the home server | €15-30/year | For a mini PC running continuously |
| Everything else | Speech-to-text and text-to-speech | €0 | Runs locally, no per-use fee |

> [!NOTE]
> Prices are rough estimates in EUR (September 2026) and vary by region and retailer — check before buying. A realistic minimum if you already own a server is the kit alone. Works well on a virtualization host: an LXC container with 2 cores, 4 GB RAM and 32 GB disk is enough for one user. Avoid running speech-to-text on a rented VPS — it saves little over a used mini PC and sends your voice off-site.

> [!NOTE]
> The robot always needs a network path back to your server. For remote access, [Tailscale](https://tailscale.com/) is recommended — no public IP needed, works behind CGNAT. Plain WireGuard also works but needs a public IP or a small VPS relay. The phone app connects to either directly and is the recommended body for Dex on the go; taking the robot itself off-site needs a travel router (for example GL.iNet) to hold the tunnel.

## Software stack

| Layer | Choice | Why |
| :-- | :-- | :-- |
| Robot firmware | [Stack-chan](https://github.com/stack-chan/stack-chan) / [m5stack/StackChan](https://github.com/m5stack/StackChan) | Open source, active community, Xiaozhi protocol support |
| Voice gateway | [xiaozhi-esp32-server](https://github.com/xinnan-tech/xiaozhi-esp32-server) | Self-hostable, works with any OpenAI-compatible LLM |
| Speech-to-text | [whisper.cpp](https://github.com/ggml-org/whisper.cpp) or SenseVoice | Runs on CPU, stays local |
| Text-to-speech | [Piper](https://github.com/rhasspy/piper) | Lightweight, local, many voices |
| LLM today | [Mistral API](https://docs.mistral.ai/) | Free Experiment tier, OpenAI-compatible, EU-based |
| LLM later | [Ollama](https://ollama.com/) | Same API shape, fully local |
| Dex core | Python, FastAPI, SQLite | Simple to read, easy to extend |
| Web app | PWA (TypeScript) | One codebase for Samsung, iPhone and desktop |

## Setup wizard and companion app

First-run configuration, not account creation — runs once, after you clone the repository and start the server. Six steps: detect the server and generate certificates, paste your own LLM API key, choose a brain, set privacy defaults, pair the robot, personalize Dex. Ends with a plain-language privacy summary.

- Dex's own app replaces M5Stack's StackChan World app: no vendor account, no default cloud service, no reported Android login issues.
- Ships as a PWA: installs from the browser on Samsung, other Android and iPhone, no app store, same server as Dex core.

## Quick start

> [!WARNING]
> Do not expose the Dex server to the internet by forwarding its ports on your router — this is not a supported setup. It would put a microphone, your personal memory and your home network data within reach of anyone who finds the open port. Use [Tailscale](https://tailscale.com/) for remote access instead (see [Hardware](#hardware) above).

> [!IMPORTANT]
> This is the **planned** setup flow. Commands will work once Phase 1 is complete.

```bash
# 1. Clone the repository on your home server
git clone https://github.com/YOUR-USERNAME/dex.git
cd dex/server

# 2. Create your config and add secrets (never commit this file)
cp dex.example.yaml dex.yaml
cp .env.example .env        # MISTRAL_API_KEY goes here

# 3. Start the voice gateway, Dex core and web app
docker compose up -d

# 4. Open the setup wizard from any device on your network
#    https://dex.local:8443
```

## Configuration

Dex core reads a single `dex.yaml`. A trimmed example:

```yaml
dex:
  name: Dex
  persona: prompts/persona.md
  wake_word: "hey dex"

llm:
  default: mistral
  providers:
    mistral:
      base_url: https://api.mistral.ai/v1
      model: mistral-small-latest
      api_key_env: MISTRAL_API_KEY
    local:
      base_url: http://ollama:11434/v1
      model: ministral            # any model you have pulled
      enabled: false              # flip to true in phase 8

privacy:
  local_only:                     # never sent to a cloud provider
    - network_scan
    - security_alerts
    - credentials
    - journal
    - health
  when_local_unavailable: refuse  # or: local_skill_only

memory:
  path: /data/dex
  embeddings: local               # keeps the index portable

skills:
  confirm_with_touch:             # head-touch required before running
    - smart_home_control
    - pc_control
```

## Choosing a brain

- Any OpenAI-compatible provider works: Mistral (default, EU), DeepSeek, Claude, or a local Ollama model. Switching is one line — `llm.default` in `dex.yaml`.
- Speech-to-text and text-to-speech always run on your own server, regardless of brain — no text LLM handles audio, so your voice stays home no matter which provider you pick.
- Avoid running STT or TTS on a rented VPS: it saves little over a used mini PC and sends your audio off-site.

## Project structure

```text
dex/
├── firmware/          # Build config and patches for the Stack-chan firmware
├── installer/         # Browser-based firmware flasher
├── server/
│   ├── core/          # Dex core: persona, memory, privacy router, tools
│   ├── gateway/       # xiaozhi-esp32-server configuration
│   ├── prompts/       # Persona and system prompts
│   └── docker-compose.yml
├── skills/
│   ├── life/          # Reminders, calendar, weather, briefing
│   └── security/      # Network watch, CVE digest, cert checks
├── app/               # Companion PWA and setup wizard
└── docs/
    └── assets/        # Diagrams and graphics
```

## Contributing

Contributions are welcome, whether that's code, a new skill, a translation, a better face animation or a bug report.

1. Fork the repository and create a branch: `git checkout -b feature/my-skill`
2. Keep changes focused and add tests for anything in `server/core`
3. New skills must declare their permissions and whether they can run locally
4. Open a pull request describing what changed and why


## Acknowledgements

Dex stands on the shoulders of these projects: [Stack-chan](https://github.com/stack-chan/stack-chan) by Shinya Ishikawa and the community, [M5Stack](https://github.com/m5stack/StackChan), [xiaozhi-esp32-server](https://github.com/xinnan-tech/xiaozhi-esp32-server), [Mistral AI](https://mistral.ai/), [Ollama](https://ollama.com/), [whisper.cpp](https://github.com/ggml-org/whisper.cpp) and [Piper](https://github.com/rhasspy/piper). Thanks also to community projects like [dotty-stackchan](https://github.com/BrettKinny/dotty-stackchan), which showed that a fully self-hosted Stack-chan is possible.

Dex is an independent project and is not affiliated with M5Stack, the Stack-chan project or Mistral AI.

## License

Dex is released under the [Apache License 2.0](LICENSE). Third-party components keep their own licenses; check each upstream project before redistributing.

<p align="center">
  <sub>Built with care, curiosity and a healthy amount of paranoia.</sub>
</p>

<p align="center">
  <img src="docs/assets/banner.svg" alt="Dex: life assistant, security sidekick, desk buddy" width="100%">
</p>

<p align="center">
  <a href="LICENSE"><img alt="License: Apache 2.0" src="https://img.shields.io/badge/license-Apache%202.0-0E9AA7?style=flat-square"></a>
  <img alt="Status: early development" src="https://img.shields.io/badge/status-phase%200-F2A541?style=flat-square">
  <img alt="Hardware: M5Stack CoreS3" src="https://img.shields.io/badge/hardware-M5Stack%20CoreS3-14213D?style=flat-square">
  <img alt="LLM: Mistral or Ollama" src="https://img.shields.io/badge/LLM-Mistral%20%7C%20Ollama-5B4BDB?style=flat-square">
  <img alt="PRs welcome" src="https://img.shields.io/badge/PRs-welcome-0E9AA7?style=flat-square">
</p>

<p align="center">
  <a href="#what-is-dex">About</a> |
  <a href="#how-it-works">How it works</a> |
  <a href="#roadmap">Roadmap</a> |
  <a href="#setup-wizard-and-companion-app">Setup wizard</a> |
  <a href="#security-model">Security</a> |
  <a href="#contributing">Contributing</a>
</p>

> [!NOTE]
> Dex is in early development. Anything marked **planned** below describes where the project is headed, not what works today. Follow the [roadmap](#roadmap) to see progress.

## What is Dex

Dex is an open-source desk robot that tries to be three things at once: a friend who remembers your day, an assistant that keeps your life organized, and a security sidekick that keeps an eye on your home network.

It's built on [Stack-chan](https://github.com/stack-chan/stack-chan), the open-source M5Stack CoreS3 robot. The robot itself stays simple: it listens, talks, moves its head and shows a face. Everything clever happens on a small home server you own, which means you decide where your data goes. Dex starts on Mistral's free API and is designed from day one to move to a fully local model with a single config change.

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

Dex follows a **thin robot, smart server** design. The robot never talks to an AI provider directly and never stores API keys.

- **Dex robot.** A Stack-chan on an M5Stack CoreS3. It streams audio to the server, plays back speech, and renders the face, head movements and LEDs.
- **Voice gateway.** A self-hosted [xiaozhi-esp32-server](https://github.com/xinnan-tech/xiaozhi-esp32-server) that handles wake word, speech-to-text and text-to-speech. Speech recognition and synthesis run locally, so your voice never leaves your home.
- **Dex core.** The part this repository is really about. A Python service that holds Dex's personality, looks up memories, decides which tools may run, and routes every request through the privacy router.
- **Memory.** Conversation history in SQLite plus a vector index built with a **local** embedding model. Using a local embedder from the start means nothing needs re-indexing when Dex goes fully local.
- **Skills.** Small, allowlisted tools for life and security tasks. Dex can call them, but it never gets a raw shell.
- **Web app.** The setup wizard and dashboard, served by your own server and usable from any phone or browser.
- **LLM backend.** Any OpenAI-compatible endpoint. Today that's the Mistral API; later it's Ollama on local hardware.

### The privacy router

Mistral's free tier may use API requests to improve its models, so Dex never sends it anything sensitive. Every request is classified before it leaves Dex core:

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

## Roadmap

<p align="center">
  <img src="docs/assets/roadmap.svg" alt="Dex roadmap: six phases from foundation to fully local" width="100%">
</p>

<details open>
<summary><b>Phase 0: Foundation</b></summary>

- [ ] Flash and test the Stack-chan firmware on the CoreS3
- [ ] Set up a Linux home server with Docker
- [ ] Put the robot on an isolated IoT VLAN that can only reach the voice gateway
- [ ] Create the repository structure, license and security policy
</details>

<details>
<summary><b>Phase 1: Voice and brain</b></summary>

- [ ] Run xiaozhi-esp32-server locally and point the robot at it
- [ ] Connect the Mistral API through the OpenAI-compatible endpoint
- [ ] Local speech-to-text (Whisper or SenseVoice) and text-to-speech (Piper)
- [ ] First version of Dex's persona prompt and mood-to-face mapping
</details>

<details>
<summary><b>Phase 2: Memory</b></summary>

- [ ] Conversation log in SQLite
- [ ] Local embeddings (for example `nomic-embed-text`) and a vector index on the SSD
- [ ] Retrieval of relevant memories before each reply
- [ ] "Forget that" command and a memory viewer in the web app
</details>

<details>
<summary><b>Phase 3: Skills</b></summary>

- [ ] Life: reminders, timers, calendar, weather, morning briefing
- [ ] Security: new-device alerts, CVE digest from the CISA KEV catalog, certificate checks
- [ ] Tool allowlist with read-only defaults
- [ ] Head-touch confirmation for any action that changes something
</details>

<details>
<summary><b>Phase 4: Companion app and setup wizard</b></summary>

- [ ] Browser-based firmware installer
- [ ] Six-step setup wizard as an installable web app (PWA)
- [ ] Dashboard for memory, skills, privacy settings and logs
- [ ] Optional Android and iOS builds from the same codebase
</details>

<details>
<summary><b>Phase 5: Fully local</b></summary>

- [ ] Ollama on a GPU-equipped server
- [ ] Mistral open-weight models so Dex keeps a familiar personality
- [ ] Zero-cloud mode that blocks all outbound LLM traffic
- [ ] Optional cloud fallback for non-sensitive, heavy questions
</details>

## Hardware

| Part | Minimum | Recommended | Notes |
| :-- | :-- | :-- | :-- |
| Robot | M5Stack CoreS3 with Stack-chan body | Official M5StackChan kit | Needs a 2.4 GHz Wi-Fi network |
| Home server (phases 0 to 4) | Any 64-bit Linux box with 8 GB RAM | 16 GB RAM, quad-core CPU | A mini PC is plenty for voice and Dex core |
| Storage | 64 GB SSD | 100 to 200 GB SSD | Memory, vector index, logs and cached answers |
| Local LLM (phase 5) | GPU with 8 GB VRAM | GPU with 12 GB+ VRAM | Or a machine with large unified memory |

## Software stack

| Layer | Choice | Why |
| :-- | :-- | :-- |
| Robot firmware | [Stack-chan](https://github.com/stack-chan/stack-chan) / [m5stack/StackChan](https://github.com/m5stack/StackChan) | Open source, active community, Xiaozhi protocol support |
| Voice gateway | [xiaozhi-esp32-server](https://github.com/xinnan-tech/xiaozhi-esp32-server) | Self-hostable, works with any OpenAI-compatible LLM |
| Speech-to-text | [whisper.cpp](https://github.com/ggml-org/whisper.cpp) or SenseVoice | Runs on CPU, stays local |
| Text-to-speech | [Piper](https://github.com/rhasspy/piper) | Lightweight, local, many voices |
| LLM today | [Mistral API](https://docs.mistral.ai/) | Free Experiment tier, OpenAI-compatible |
| LLM later | [Ollama](https://ollama.com/) | Same API shape, fully local |
| Dex core | Python, FastAPI, SQLite | Simple to read, easy to extend |
| Web app | PWA (TypeScript) | One codebase for Samsung, iPhone and desktop |

## Setup wizard and companion app

<p align="center">
  <img src="docs/assets/wizard.svg" alt="Dex setup wizard: six steps from flashing firmware to a privacy check" width="100%">
</p>

The goal is that someone who has never touched a terminal can get Dex running. The planned wizard walks through six steps: flash the firmware from the browser, connect Wi-Fi, pair with the home server using a short code, choose a brain, personalize Dex, and finish with a plain-language privacy check.

**Why not use the official app?** M5Stack's StackChan World app is available on Google Play and the App Store, but it's built around an M5Stack account and the default cloud service, which Dex replaces. Some Samsung users have also reported login problems with the Android version. Dex's app talks only to your own server.

**Why a web app first?** A progressive web app installs to the home screen on Samsung, other Android phones and iPhone, needs no app store, and ships from the same server as Dex core. Native Android and iOS builds can be wrapped from the same code later if they add real value, such as Bluetooth provisioning.

## Quick start

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
      enabled: false              # flip to true in phase 5

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

## Security model

Dex has a microphone, knows a lot about you and can run tools, so it's treated as the most sensitive device in the house.

| Risk | Mitigation |
| :-- | :-- |
| API key extracted from the robot | Keys live only on the server in `.env`. The robot holds no secrets beyond Wi-Fi. |
| Robot used as a pivot into the home network | Robot sits on its own VLAN and can only reach the voice gateway port. |
| Server exposed to the internet | No port forwarding. Remote access only through WireGuard or Tailscale. |
| Prompt injection from web pages, feeds or emails | Tools are allowlisted and read-only by default. State-changing actions need a physical head-touch. |
| Sensitive data sent to a cloud model | Privacy router enforces local-only categories. Zero-cloud mode in phase 5. |
| Always-on microphone | Wake word gating, a visible LED while streaming, and a mute option. |
| Unauthorized network scanning | Network skills only run against subnets listed in `dex.yaml` that you own. |

Found a vulnerability? Please report it privately as described in [SECURITY.md](SECURITY.md) instead of opening a public issue.

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

# Security policy

Dex listens through a microphone, stores personal memories and can run tools on your network, so security reports are taken seriously.

## Reporting a vulnerability

Please **do not** open a public issue for security problems.

Use GitHub's private reporting instead: go to the **Security** tab of this repository and choose **Report a vulnerability**. Include:

- What you found and which component it affects (firmware, Dex core, web app, a skill)
- Steps to reproduce, or a proof of concept
- The impact you think it has

You can expect an acknowledgement within 7 days. Fixes are credited to the reporter unless you'd rather stay anonymous.

## Scope

In scope: Dex core, the web app and setup wizard, the firmware configuration in this repository, and bundled skills.

Out of scope: vulnerabilities in upstream projects (Stack-chan, xiaozhi-esp32-server, Ollama and others). Please report those to the upstream maintainers directly.

## Supported versions

Dex is in early development. Only the latest commit on `main` receives security fixes.

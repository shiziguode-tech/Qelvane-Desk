# Qelvane-Desk
Qelvane Desk - a local-first Windows AI agent with multi-provider models, tools, MCP, OpenClaw, remote channels, and desktop automation.
# Qelvane Desk Agent

## English

Current release: v1.0 (application version 1.0.0).

Qelvane Desk is a Windows-first local computer agent under active development. Language models plan and call tools, plugins observe and act, and the host application provides stopping, persistence, verification, recovery, and local auditing. It includes native tools plus an isolated compatibility sidecar built on the official OpenClaw Gateway.

> This is the v1.0 release build. Only capabilities that have been exercised by real tests are described as available.

Official scores, scoring rules, and fixes from the four closed-book Agent benchmarks are documented in [BENCHMARK-REPORT.md](BENCHMARK-REPORT.md).

### Implemented capabilities

- Multi-turn model and tool-call loop with automatic or manually selected models.
- Automatic DeepSeek primary/backup key failover: immediate failover on 401/402, and retry followed by failover on 429/5xx.
- Windows UI Automation for listing windows, reading controls, clicking, and background text input.
- Screenshot plus coordinate-level mouse and keyboard fallback.
- Per-task vision routing: Windows OCR first, local Baidu PaddleOCR AI second, and Google only as an optional semantic-vision fallback outside mainland China.
- Local OCR consumes no API quota. Screenshots containing likely credentials are never uploaded to a cloud vision provider.
- Workspace-scoped file tools and PowerShell execution.
- Ordinary tool actions run autonomously. CAPTCHA, MFA, credential entry, QR scanning, and other genuinely human-only steps pause for human takeover.
- Immediate stop, risk classification, durable task state, and local JSONL audit records.
- Hot-loadable Python plugin SDK and local web console bound only to `127.0.0.1`.
- Safe local Markdown rendering, including headings, lists, tables, code blocks, and links.
- The current web-interface language is the default reply language. A current prompt written in another language or explicitly requesting another language takes priority; cross-chat context can never override this choice.
- One fixed Control Agent coordinates specialist tools internally. The obsolete user-facing specialist selector has been removed, so a saved profile cannot silently disable a required capability.
- Advanced Computer Use observation, process management, and privacy-aware clipboard handling.
- SQLite task/event persistence, interrupted-task recovery, and an all-tasks sidebar.
- Optional cross-chat context scans up to the latest 50 completed ordinary chats. Relevance and recency weights decrease turn by turn; long source chats are compressed to about one fifth before injection. Private chats, failed tool traces, logs, and secrets are excluded.
- Workspace instructions are discovered from bounded local `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, Copilot, Cursor, and Windsurf rule files. They are treated as preferences and cannot override current user intent or safety boundaries.
- SQLite long-term memory, workspace `SKILL.md` discovery, safe web reading, and search.
- Isolated official OpenClaw `2026.7.1-2` sidecar with a private per-installation Gateway token and dynamically discovered tools, plugins, and skills.
- A real stdio MCP server using the official TypeScript SDK, plus durable daily/interval/one-shot scheduled tasks with explicit start and optional end times.
- Windows Job Object process sandbox with process-tree containment and process, memory, CPU, wall-clock, and environment controls.
- Artifact, scheduled-task, categorized settings, capability-map, runtime-diagnostics, Feishu, Telegram, and live-voice interfaces.

### Live voice

The separate Studio/reader surface has been removed. Voice is now a focused conversation surface opened directly from the main composer. Its behavior is configured under **Settings → Voice**, not inside the conversation window.

Live voice supports continuous interim transcription, editable Markdown with a safe live preview, hands-free turn sending, spoken final answers, mute/retry recovery, input-level feedback, and barge-in by button or speech. Raw Markdown is sent unchanged to the Agent, while sent turns are rendered like ordinary web-chat messages. It uses the current chat context and the ordinary Agent tool loop. Recognition language, installed Windows voice, playback speed, automatic reading, hands-free sending, and automatic listening after a reply are persisted in Settings. Automatic recognition follows the Windows system speech locale independently of the interface language; 20 common languages can also be selected explicitly. Windows/WebView provides desktop dictation and playback; Telegram voice notes continue to use the configured Hugging Face Whisper route. Feishu and Telegram remain plain text plus native attachments.

### Quick start

Windows is required. Extract the complete archive, then double-click:

```text
Qelvane Desk.exe
```

The EXE is now a native Windows desktop shell. It prepares dependencies in the background, starts the hidden loopback Agent backend, and displays the interface inside an embedded Microsoft WebView2 window; it does not open a normal browser tab and the local address is not shown to the user. First-run appearance uses a clean light theme with a black accent, while an existing saved theme is always preserved. The native startup card uses the signed application icon, concise staged status copy, an animated custom progress indicator, and a local-data reassurance instead of a generic installer-style dialog. A package identity digest prevents the shell from attaching to a stale backend from another extracted copy. On every launch it checks the configured private port and terminates a process tree only when the owner is verified as an earlier Qelvane Desk service; if another application owns the port, startup stops with an explicit warning.

The desktop shell is single-instance, high-DPI aware, restores itself when the EXE is clicked again, automatically recovers from a backend disconnect, and uses a separate WebView2 data directory under `data/`. External links open in the system browser, while Qelvane pages remain inside the app window. Closing the window destroys its renderer so a private chat is not left visible; Feishu/Telegram background service behavior remains unchanged.

The launcher records dependency fingerprints for the Python version, absolute application path, `pyproject.toml`, Node lock files, and OpenClaw runtime. Moving the whole folder automatically repairs the portable `.venv` paths without deleting bundled packages. Unchanged and complete dependencies skip both pip and Node installation on later launches.

Release packages include a Docker-like portable runtime bundle without requiring Docker Desktop, Hyper-V, administrator setup, or a global development environment. `work/python-runtime`, `.venv`, `work/node-runtime`, and `work/openclaw-runtime/node_modules` contain the Python interpreter, Python packages, Node.js runtime, npm support, OpenClaw, skills, and Node dependencies. The signed package-local runtimes are always preferred and compatible computer-wide Python/Node installations remain fallback paths. Python and Node installers are deliberately not shipped in the release package. `work/runtime-bundle.json` records the self-contained component versions without storing machine-specific absolute paths.

Launcher prompts follow the current Windows display language: Chinese Windows displays Chinese text; every non-Chinese language displays English.

#### Optional offline installers

The portable bundle does not need Python or Node installers. Release packages contain no installation packages, including no WebView2 bootstrapper. Only for manual recovery of a damaged or deliberately stripped package may a user download and place the following officially signed installers beside `Qelvane Desk.exe`:

- Python: an official 64-bit installer named `python-installer.exe`. The launcher uses it only when both the portable Python runtime and every compatible system Python are unavailable.
- Node.js: an official Node.js 24 x64 MSI such as `node-v24.18.1-x64.msi`. The launcher uses it only when both the portable Node.js runtime and every compatible system Node.js are unavailable. The supported fallback ranges are Node `22.22.3+ below 23`, `24.15.0+ below 25`, or `25.9.0+`.
- Desktop renderer: Qelvane first uses an existing Microsoft Evergreen WebView2 Runtime. If it is unavailable but Microsoft Edge exists, Qelvane falls back to Edge application mode (a standalone window without browser tabs or an address bar). A user may manually place Microsoft's signed `MicrosoftEdgeWebview2Setup.exe` beside the app for recovery, but it is not shipped in the release package.

Local installers must have a valid Authenticode signature. Python must be signed by the Python Software Foundation; Node.js must be signed by the OpenJS Foundation or Node.js Foundation. An untrusted installer is rejected. The official installer window remains visible for the user to complete, after which the launcher detects the runtime again and continues.

Release maintainers should sign `Qelvane Desk.exe` with a trusted organization or public code-signing certificate. Set `QELVANE_SIGNING_CERT_THUMBPRINT` to a certificate with a private key in `Cert:\CurrentUser\My`, then run `launcher\build-launcher.ps1 -RequireSigned`. The build verifies a valid SHA-256 Authenticode signature and applies a trusted timestamp. A self-signed certificate does not establish SmartScreen reputation and is not presented as a release fix.

OpenClaw and its skills do not have a third official installer EXE/MSI. This project pins OpenClaw to `2026.7.1-2`. Offline packages contain `work/openclaw-runtime/node_modules`; online repair uses the project package manifest and lock file. Do not download third-party programs named like “OpenClaw skills installer.exe”.

The package-local OpenClaw entry, workspace, configuration, state, and skills are preferred. Only when the packaged entry is actually missing or invalid does Qelvane Desk fall back to an explicitly configured `DEEPDESK_OPENCLAW_CLI` or a system-PATH installation. Even during fallback, workspace and state remain inside the current Qelvane folder. Local skills may be added under `skills/local/<name>/SKILL.md` and are discovered on the next Agent turn.

For troubleshooting, the PowerShell fallback is:

```powershell
.\start.ps1
```

Internally, the Agent backend remains restricted to loopback (`127.0.0.1`) so it is not exposed to the LAN; this is an implementation detail rather than the user interface. `DEEPDESK_PORT` in `.env` or the process environment changes its private port. Configuration belongs in `.env`; it is ignored by Git and must not be committed. See `.env.example` for the template.

### macOS application

Qelvane Desk now includes a native AppKit + WKWebView macOS shell and a reproducible application-bundle build. The Mac app stores its workspace under `~/Library/Application Support/Qelvane Desk`, starts the packaged Python Agent only on loopback, embeds the interface in its own window, opens external links in the default browser, and stops its owned backend when the app exits. It never modifies or rebuilds the Android APK.

On macOS, Windows-specific integrations are replaced by the closest native equivalents:

- Windows UI Automation → macOS Accessibility and System Events (`macos_ui`).
- Windows OCR → offline Apple Vision OCR, followed by the same local PaddleOCR AI fallback.
- Windows mouse/keyboard and screenshot fallback → Quartz-backed PyAutoGUI plus Unicode clipboard paste.
- PowerShell Job Object sandbox → zsh in a separate POSIX process group with CPU, memory, process-count, timeout, environment, and process-tree controls. This is not a filesystem or network namespace.
- Windows Recycle Bin → `~/.Trash`; permanent deletion still requires an explicit current request.
- Windows local speech fallback → the macOS `say` engine; online neural speech remains available.

Native app control requires the user to grant **Accessibility**, **Screen Recording**, and (when another app is controlled) **Automation** under System Settings → Privacy & Security. macOS itself enforces these permissions and Qelvane cannot bypass them.

Build the app on macOS 13 or newer with Xcode Command Line Tools, Python 3.11+, and optionally Node.js 22+:

```zsh
chmod +x macos/build-macos-app.sh
macos/build-macos-app.sh
```

The result is `dist/Qelvane-Desk-v1.0-macOS-<architecture>.zip`. When Node.js is present at build time, the script packages the Node executable and OpenClaw into the app; otherwise OpenClaw may use a compatible system installation while the remaining Agent capabilities stay available. The GitHub Actions workflow `.github/workflows/build-macos.yml` provides the same build on an actual macOS runner.

For direct public distribution, set `QELVANE_CODESIGN_IDENTITY` to a Developer ID Application identity and `QELVANE_NOTARY_PROFILE` to a `notarytool` keychain profile. The script signs nested executables, enables Hardened Runtime, verifies the bundle, submits the ZIP to Apple, staples the ticket, and recreates the checksum. Without those values it creates an ad-hoc-signed development build; Gatekeeper may require a manual approval and it must not be presented as notarized.

The lowercase `deepdesk` Python package name, environment-variable prefixes, and some MCP identifiers are retained for backward compatibility. Product UI, API titles, prompts, and documentation use the Qelvane Desk name.

### Model API providers

The settings page stores the DeepSeek primary key, optional DeepSeek backup key, and optional Gemini fallback key. It can also hold up to 25 additional model entries for OpenAI, Anthropic, Google, and xAI. Each entry contains the provider, the exact API model identifier, and its API key. Keys are stored only in local `data/provider-keys.json`; APIs return configuration status but never plaintext secrets. Leaving a key field blank preserves its existing value.

Configured models appear in both Default Model and the composer model selector. A manual composer choice has the highest priority. When the composer remains on Automatic, Qelvane uses the concrete default model selected in Settings. Choose “Automatically choose the best configured model” as the Settings default to enable task-aware routing across all configured providers; that route slightly favors GPT-5.6 Luna for ordinary cost-sensitive work and selects higher-capability or task-specific models for demanding work.

Reasoning depth is independently selectable with the segmented slider below the composer. The Settings default is High; each task may override it with Automatic, Low, Medium, High, Extra high, or Maximum. The choices map to GPT-5.6 `reasoning.effort`/`reasoning_effort`, Claude adaptive thinking plus `output_config.effort` (or a compatible manual budget for Claude 4.5), Gemini `thinkingLevel`, and DeepSeek V4's supported high/max controls. A model that exposes fewer levels receives the nearest supported level instead of silently dropping reasoning. GPT models additionally expose a separate, task-scoped **Fast answer** switch in the composer. It is always user-controlled, never enabled automatically, and maps GPT reasoning to `none` without changing the saved reasoning-depth default.

The same page has a dedicated [AI Code Mirror](https://www.aicodemirror.ai/) relay card. One write-only universal key enables its Claude third-party reverse channel, OpenAI official-subscription channel, and Gemini channel. A second optional write-only key is reserved exclusively for `claude-fable-5`; when configured it overrides the universal key for that model only. After saving, plaintext is never returned, while independent Configured badges and masked placeholders confirm local storage. Qelvane Desk immediately refreshes both model selectors with the supported GPT 5.4–5.6, Claude 4.5–5, and Gemini 3/3.1/3.5 catalog. The public package includes both empty fields and the registration link but no bundled relay credential.

Conversation titles are generated by the model that actually executes the first turn. For Automatic routing, Qelvane waits for the concrete `model_selected` result before requesting the title; if the provider is unavailable, the compact local fallback remains instead of asking a different model to identify the chat.

AI Code Mirror preserves the original vendors' exact API model IDs. Qelvane therefore maps every built-in relay model to the original vendor's published context window: GPT-5.4/5.5/5.6 use 1,050,000 tokens; Claude Opus/Fable/Sonnet and Claude Opus 4.6–4.8 use 1,000,000; Claude Haiku 4.5 uses 200,000; and the three configured Gemini models use 1,048,576. Unknown user-entered model IDs are still labelled as conservative fallbacks rather than official claims, and an explicit provider overflow response immediately triggers compaction. Relay protocols and the catalog are checked against its [Claude SDK guide](https://www.aicodemirror.ai/dashboard/sdk-docs), [OpenAI SDK guide](https://www.aicodemirror.ai/dashboard/openai-sdk-docs), [Gemini SDK guide](https://www.aicodemirror.ai/dashboard/gemini-sdk-docs), and [pricing/model catalog](https://www.aicodemirror.ai/dashboard/pricing). Context values are checked against the original vendors' [OpenAI model comparison](https://developers.openai.com/api/docs/models/compare), [Claude context-window guide](https://platform.claude.com/docs/en/build-with-claude/context-windows), and [Gemini model specifications](https://ai.google.dev/gemini-api/docs/models).

DeepSeek, OpenAI, Google, and xAI use their official Chat Completions interfaces. Anthropic uses the native Messages API, with Qelvane Desk converting system messages, tool definitions, tool results, and streaming events. The user does not need to enter a Base URL, but the model identifier must be enabled for that account. When current or uncertain information requires verification, the Agent can invoke the selected route's native search: OpenAI Responses `web_search`, Claude Messages `web_search_20250305`, Gemini `google_search`, or DeepSeek native search. This works with each vendor's official key and with the matching AI Code Mirror route; credentials never cross provider domains. If a relay does not pass through a native server tool, Qelvane falls back to its general web/background-browser evidence path instead of failing the task.

Official API references: [DeepSeek Chat Completion](https://api-docs.deepseek.com/api/create-chat-completion), [OpenAI authentication](https://platform.openai.com/docs/api-reference/authentication), [Anthropic Messages API](https://platform.claude.com/docs/en/api/messages/create), [Google Gemini OpenAI compatibility](https://ai.google.dev/gemini-api/docs/openai), and [xAI Chat Completions](https://docs.x.ai/developers/model-capabilities/legacy/chat-completions).

### Vision and OCR routing

Every new or continued task probes the country/region of the public egress actually used for that request. The raw public IP is discarded immediately: it is not written to the database or logs and is never reused for the next request.

Qelvane Desk keeps text recognition local in every region: Windows OCR runs first, followed by Baidu PaddleOCR PP-OCR AI when harder text extraction is needed. PaddleOCR loads lazily through CPU ONNX inference, needs no dedicated GPU, and uses a small bundled model. Outside mainland China, an explicitly configured Google endpoint may be used only for non-text scene interpretation; mainland-China screenshots never fall through to Google.

```env
DEEPDESK_VISION_BASE_URL=https://generativelanguage.googleapis.com/v1beta/openai
DEEPDESK_VISION_MODEL=gemini-3.6-flash
DEEPDESK_VISION_API_KEY=
```

Community Spaces are externally hosted and may be queued or rate-limited. Before any cloud upload, local OCR checks common credential patterns; likely API keys or bearer tokens block the upload. The optional vision endpoint must accept OpenAI Chat Completions `image_url` content.

### OpenClaw compatibility layer

Qelvane Desk connects to the official OpenClaw CLI/Gateway and reads `tools.catalog` plus the plugin registry dynamically instead of copying a tool list that quickly becomes stale. Isolated state lives under `data/openclaw/`; Gateway tokens never appear in the API, UI, logs, or OpenClaw configuration. Model credentials are passed to child processes only through their environment.

Browser access has no public-domain allowlist. It keeps strict SSRF protection for loopback, LAN, link-local, metadata, and raw non-public IP targets while allowing every public hostname. Clash/Mihomo-style Fake-IP DNS is supported: Qelvane verifies the mode with two independent public probes, then accepts all hostname mappings in `198.18.0.0/15`; directly entered `198.18.x.x` addresses remain blocked.

### Telegram channel

Telegram uses the free official Bot API with long polling and needs no public callback URL. It accepts text, images, PDF, Word, PowerPoint (`.ppt`/`.pptx`), Excel, TXT, audio, video, and Telegram voice notes. A voice note is downloaded, transcribed by the free Whisper inference route configured with the optional Hugging Face Read token, then passed to DeepSeek as ordinary task text. An attachment and the immediately following instruction are briefly buffered and merged into one task. Local files explicitly referenced by the final answer are returned as native Telegram media or document messages.

Setup:

1. Open official [@BotFather](https://t.me/BotFather), send `/newbot`, and create a name and username.
2. Copy the Bot Token and keep it private.
3. In Qelvane Desk, open Settings → Optional Telegram channel, enter the token, and save. Default Chat ID may remain blank.
4. Open the new bot and send `/start` or any message. Qelvane Desk learns that conversation as the default Chat ID.
5. Test inbound text/files and ask the Agent to generate and return a file.

`getUpdates` and webhooks cannot be enabled simultaneously. Delete an old webhook before using long polling. Standard Bot API downloads are limited to 20 MB; this application limits ordinary outbound files to 50 MB and native photos to 10 MB. See the [Telegram Bot API](https://core.telegram.org/bots/api) and [Bots FAQ](https://core.telegram.org/bots/faq).

### Feishu channel

Qelvane Desk uses the official Feishu Python SDK and a long-lived connection for `im.message.receive_v1`; no Verification Token, Encrypt Key, public domain, or HTTP callback is required.

Setup:

1. Create an enterprise custom application in the [Feishu Open Platform](https://open.feishu.cn/app).
2. Add the Bot capability.
3. Grant `im:message.p2p_msg:readonly` (or an equivalent read/write permission), `im:message:send_as_bot`, and `im:resource` for image/file download and upload.
4. In Events and Callbacks, choose long connection and subscribe to `im.message.receive_v1`.
5. Create and publish a new application version, and include the target users in its availability scope.
6. Copy App ID and App Secret into Settings → Optional Feishu channel. Never publish the App Secret.
7. Open ID may initially remain blank. Send “hello” to the current bot from the target account; Qelvane Desk learns the correct application-scoped Open ID and acknowledges the task.
8. Verify the channel in Runtime Status or the Capability Map.

After adding `im:resource`, a new application version must be published before the permission is active. Error `99991672` means the required Feishu permission is missing. Error `99992361: open_id cross app` means the stored Open ID belongs to another application; send a new message to the current bot so Qelvane Desk can learn the correct value. Do not run multiple Qelvane instances with the same Feishu app, because clustered delivery may route each event to only one connection.

Feishu and Telegram attachments are downloaded into the task's local attachment directory. The document inspector supports PDF, Word (`.doc`/`.docx`), PowerPoint (`.ppt`/`.pptx`), Excel (`.xls`/`.xlsx`), and TXT. Legacy binary Office formats use locally installed Office automation or safe conversion fallbacks where available. Generated images and supported documents explicitly referenced in a final response can be uploaded and returned as native channel messages.

Feishu- and Telegram-originated tasks are durably marked with their remote channel. Their model-level channel policy requires plain text and forbids Markdown in user-facing responses. Qelvane Desk does not rewrite output after generation, avoiding accidental changes to code or requested literal text. Ordinary web chats and scheduled tasks allow Markdown even when cross-chat context includes an older remote task.

Official Feishu references: [upload a file](https://open.feishu.cn/document/uAjLw4CM/ukTMukTMukTM/reference/im-v1/file/create), [download a message resource](https://open.feishu.cn/document/server-docs/im-v1/message/get-2?lang=en-US), and [send a message](https://open.feishu.cn/document/server-docs/im-v1/message/create?lang=en-US).

### Android companion

Qelvane Mobile lets a paired Android phone become an encrypted tool for the desktop Agent. In Settings → Android phone control, choose **Pair Android phone**, keep the phone and PC on the same local network, and scan the five-minute QR code from the Android app. Pairing derives a unique per-device key; control packets use AES-256-GCM and the Windows copy protects stored keys with DPAPI.

The companion supports status inspection, accessibility-tree inspection, screenshots, taps, swipes, ordinary text entry, Back/Home/Recents, application launch, and opening URLs. Both the desktop task and the phone-side **Allow Autonomous mode** switch must be autonomous before ordinary actions run without a Qelvane approval prompt. Android permission dialogs, authentication and verification-code screens, biometrics, payments, security settings, and application installation remain user-controlled. A persistent Android notification indicates an active control connection, and a device can be revoked at any time from desktop Settings.

Developers can build the debug APK with Android Studio or with `work/openclaw-reference/apps/android/gradlew.bat -p mobile/QelvaneMobile :app:assembleDebug`. The generated file is `mobile/QelvaneMobile/app/build/outputs/apk/debug/app-debug.apk`.

### Free-tier image, video, and music generation

Outside mainland China, Qelvane Desk uses Pollinations for free image generation and an automatic verified-free fallback chain for video and music. Video can use the official LTX-Video Distilled and LTX 2.3 ZeroGPU Spaces, the independent Wan 2.1 public demo pool, Alibaba PAI EasyAnimate on ModelScope, and any Pollinations model whose live catalog explicitly declares zero pricing. Music can use Stable Audio 3, ACE-Step, Meta MusicGen, Stable Audio Open, DiffRhythm, HeartMuLa, and EzAudio ZeroGPU Spaces, followed by Tongyi InspireMusic on ModelScope and any live zero-price Pollinations model. A Pollinations key may be stored under Settings → Provider keys. If one shared pool reports exhausted quota, the remaining models in that same pool are skipped briefly and the next independent provider is tried; no paid fallback is used. A model is shown as available only when both its runtime and actual Gradio configuration endpoint are reachable.

For more free ZeroGPU allowance, create a free Hugging Face account, open [Access Tokens](https://huggingface.co/settings/tokens), create a `Read` token, and paste it into Settings → Provider keys → Hugging Face free Read token. Hugging Face documents about 2 included GPU minutes per day for anonymous use and about 5 minutes for an authenticated free account. Qelvane Desk sends the token only through the Gradio client's authentication field and never puts it in prompts, URLs, events, or status responses. Use a free account without prepaid credits if you require a strict no-charge setup.

For mainland-China egress, image and video generation route to the Alibaba PAI `EasyAnimate` public ModelScope Studio, while music routes to the Tongyi `InspireMusic` public Studio. Each call first verifies that the Studio reports Running and does not require login. These are shared free resources and may still queue or change availability.

Outputs are stored under `outputs/` and can be previewed or downloaded on the desktop. Feishu and Telegram can receive the generated image, video, or audio through their native media/file paths. Generation limits are 25 MB for music and 29 MB for video so results remain compatible with both channels.

### MCP and scheduled tasks

The private MCP service is `work/openclaw-runtime/mcp-servers/deepdesk-workspace.mjs` and uses standard JSON-RPC over stdio. Direct HTTP diagnostics are read-only. `write_artifact` is available only through the Agent tool boundary and is restricted to `outputs/`.

The scheduled-task form uses the computer's local date and time, converts it to an explicit UTC instant for durable storage, and lets the user choose daily, fixed-interval, or one-time execution plus an optional end time. Legacy five-field cron records remain compatible but are no longer exposed as the primary UI. Intervals range from one second to 365 days, and missed high-frequency triggers are coalesced after downtime to prevent restart storms.

### Security model

- File tools resolve paths and prevent escaping the workspace.
- Agent file deletion is recoverable by default: `filesystem.delete` moves files and folders to the Windows recycle bin. Raw shell/sandbox deletion and Shift+Delete are blocked unless the current user message explicitly requests permanent or irreversible deletion.
- Ordinary actions are autonomous; CAPTCHA, MFA, credentials, QR scanning, and physical actions require human takeover.
- Every Feishu- or Telegram-originated task is forced to autonomous mode on the server, regardless of the web UI's selected default; human-only CAPTCHA/MFA steps still use takeover.
- Web pages, search/OCR/vision results, uploaded documents, quoted messages, tool results, logs, and cross-chat history are marked as untrusted data. Embedded requests cannot override system rules, reveal credentials, change permissions, or trigger extra tools merely by appearing inside that data.
- `Ctrl+W`, `Ctrl+F4`, and `Alt+F4` require a recently observed target window title. The foreground target is rechecked before execution, and disappearance is verified afterward.
- Destructive actions remain constrained by workspace boundaries, shell-risk checks, the process sandbox, stop controls, and local audit records.
- PyAutoGUI's upper-left-corner failsafe remains enabled.
- `data/audit.jsonl` never records API keys.
- Windows Job Objects provide real kernel process-resource boundaries, but they are not containers and do not provide filesystem or network isolation. Use Windows Sandbox, Hyper-V, a container, or a separate low-privilege account when those boundaries are required.

### Plugin development

Place trusted Python files that inherit `ToolPlugin` under `plugins/`. A plugin declares its function JSON Schema, parameter-sensitive risk classification, human-readable action summary, and asynchronous execution method. See `plugins/example_time.py.disabled`. Third-party plugins run with the same OS permissions as Qelvane Desk and must be reviewed before use.

### Architecture

```text
Web Console
    │ task / human takeover / stop
FastAPI local gateway
    │
Agent loop ─── selected language model (planning + tool calls)
    │
Execution policy + verification ─── audit.jsonl
    │
Plugin registry
    ├── memory / skills / web
    ├── windows_ui (UI Automation)
    ├── computer (screenshot / mouse / keyboard)
    ├── vision (Windows OCR → PaddleOCR AI for CN / cloud fallback outside CN)
    ├── filesystem (workspace only)
    ├── shell / sandbox (PowerShell + Windows Job Object)
    ├── mcp (official SDK stdio server)
    ├── cron (durable SQLite scheduler)
    └── openclaw ─── official Gateway / dynamic tools / plugins
```

### References

- [OpenClaw repository](https://github.com/openclaw/openclaw)
- [Manus “My Computer”](https://help.manus.im/en/articles/14178443-what-is-the-my-computer-feature-capable-of)
- [Claude Code hooks](https://code.claude.com/docs/en/hooks-guide)
- [Running Codex safely](https://openai.com/index/running-codex-safely/)
- [DeepSeek Tool Calls](https://api-docs.deepseek.com/guides/tool_calls)
- [DeepSeek-VL2](https://github.com/deepseek-ai/DeepSeek-VL2)

### Remote settings commands

A web, Feishu, or Telegram user may request a Settings change when the **current text message starts with two slashes**. Half-width and full-width combinations are accepted: `//`, `／／`, `/／`, and `／/`. Example: `// enable cross-chat context and set the accent color to #246BFD`. App Live Voice and Telegram voice notes are the deliberate exception: a direct spoken instruction may change any supported setting without a slash prefix. The host records voice provenance separately, so text, model output, copied pages, files, old chats, and tool results cannot claim this exception.

After a successful change, the Agent reports every changed field and references three sanitized artifacts under `outputs/settings-changes/`: a before PNG, an after PNG, and a TXT audit log. Secret values are never written to those artifacts; only configured/not-configured state is shown. Feishu and Telegram return the PNG and TXT files as native attachments. Channel-originated tasks remain autonomous, but CAPTCHA, MFA, financial writes, and other host-enforced safety boundaries are not bypassed by this command.

### Optional OKX and Binance financial management

Financial automation is protected by the Settings master switch **Use financial automation**, which defaults to off. When it is off, credentials alone cannot start the 30-minute market refresh or any OKX/Binance financial tool. The exchange selector can lock all financial calls to OKX or Binance. If it is set to **Either exchange**, each financial request must explicitly choose exactly one exchange; the host rejects an ambiguous tool call and requires the Agent to ask the user which exchange to use.

The market scope defaults to `ALL`, so one read-only request per enabled exchange retrieves all spot tickers every 30 minutes. To reduce the result set, enter a comma-separated list such as `BTC-USDT,ETH-USDT,SOL-USDT`. The scheduled refresh is market-data-only: it never places orders, moves funds, withdraws assets, or changes Earn positions.

For OKX, create a V5 API key and enter its API Key, Secret Key, and API Passphrase. For Binance, create an API key and enter its API Key and Secret Key. Bind keys to the current public egress IP where supported and grant only the permissions actually required. Read-only permissions are sufficient for monitoring; trading, transfer, withdrawal, and Simple Earn permissions should be enabled separately only when needed. Every order, cancellation, transfer, withdrawal, Earn subscription, and Earn redemption always pauses for explicit confirmation in the local interface, including tasks received from Feishu or Telegram. Qelvane Desk intentionally implements no lending, borrowing, repayment, collateral, auto-loan, or lending-rate action on either exchange.

Credentials are stored locally in `data/provider-keys.json`, redacted from API responses, evidence files, and logs, and omitted from the public package. Rotate any production key that has been pasted into chat or otherwise exposed, then replace it with an IP-bound, least-privilege key. Binance requests use the official Spot/Wallet REST authentication flow documented in [Binance Spot API](https://developers.binance.com/docs/binance-spot-api-docs/rest-api) and [Binance API introduction](https://developers.binance.com/docs/binance-spot-api-docs).

### Cross-chat context weighting

When cross-chat context is enabled, all eligible completed non-private chats remain candidates instead of only the last two turns. Relevance is combined with a smooth recency weight: recent conversations influence the current task more, while older relevant outcomes remain available at a progressively lower weight. Private chats, failed tasks, raw tool traces, and secrets are excluded or redacted. Short follow-ups such as “retry the other two” still resolve to the immediately preceding completed task on the same channel.

Long-running Agent tasks use recursive automatic context compaction. Normal compaction starts only when the working context is approximately 85% full; an earlier provider-declared context-limit error triggers immediate recovery. The new checkpoint targets roughly 45% of the working window and preferentially keeps recent tool arguments, results, file paths, verification evidence, the original goal, and all active safety constraints. Every compaction carries the previous cumulative ledger forward. The complete redacted pre-compaction transcript is also archived under `data/context-checkpoints/`, and every later checkpoint lists all earlier archive paths, so an Nth compaction can retrieve an exact omitted detail instead of losing access to pre-N history. Original task events and chats remain stored locally; only the model's active working context is compacted.

### Current limitations

This is a runnable development build, not a third-party-audited system RPA product. Complex browser canvases, games, and inaccessible interfaces may require visual models or the OpenClaw browser runtime. External MCP servers, messaging channels, and cloud plugins require their own accounts and credentials. Presence in a catalog does not mean an integration has been authorized.

---

# Qelvane Desk Agent — 中文说明

## macOS 应用

项目现已加入原生 AppKit + WKWebView 的 macOS 外壳和可重复构建脚本。Mac 版会把用户工作区保存到 `~/Library/Application Support/Qelvane Desk`，只在回环地址启动随包 Python 智能体，并在应用退出时结束自己启动的后端；Android APK 不会被修改或重新编译。

Windows 专属能力在 Mac 上会自动换成相近的原生实现：UIA 改为 macOS 辅助功能与 System Events，Windows OCR 改为本机 Apple Vision OCR，PowerShell Job Object 改为 zsh + POSIX 进程组/资源限制，回收站改为 `~/.Trash`，本机语音回退改为 macOS `say`。后台浏览器、模型、文件、MCP、OpenClaw、飞书、Telegram、定时任务和记忆等通用功能继续共用。

首次进行桌面控制时，需要在“系统设置 → 隐私与安全性”中给 Qelvane Desk 开启“辅助功能”“屏幕录制”，并在控制其他应用时允许“自动化”。这些是 macOS 强制权限，应用无法绕过。

必须在 macOS 13 或更高版本上执行 `macos/build-macos-app.sh` 才能生成真正的 Mach-O `.app`；Windows 不能生成可信的 Mac 签名/公证产物。脚本输出 `dist/Qelvane-Desk-v1.0-macOS-<架构>.zip`。公开分发时应配置 Apple Developer ID 和 `notarytool` 公证资料；未配置时只生成用于测试的临时签名版本。

当前发布版本：v1.0（程序版本 1.0.0）。

Qelvane Desk 是一个开发中的 Windows 本地电脑代理：DeepSeek V4 Flash 负责规划和函数调用，插件负责观察和执行，宿主程序负责停止、持久化、验证与审计。它同时提供原生工具和官方 OpenClaw Gateway 兼容侧车。

> 当前为 v1.0 发布构建；只有通过真实验证的能力才列为可用。

四套闭卷 Agent 基准的正式分数、计分口径和错误修复记录见 [BENCHMARK-REPORT.md](BENCHMARK-REPORT.md)。

## 已实现

- DeepSeek `deepseek-v4-flash` 多轮工具调用循环
- 主密钥失败后的备用密钥自动切换（401/402 立即切换；429/5xx 重试后切换）
- Windows UI Automation：列窗口、读控件、点击、输入
- 坐标级鼠标/键盘与截图兜底
- 分层图片识别：Windows OCR → 百度飞桨 PaddleOCR 本机 AI；仅在非中国大陆出口且需要理解画面语义时才按需使用 Google 备用
- 本机 OCR 无额度消耗；含疑似密钥的截图会阻止上传到任何云端识别服务
- 工作区文件和 PowerShell 工具
- 普通工具调用默认自主执行，不弹出逐次审批；验证码、MFA、凭据输入等必须由人完成的步骤使用人工接管
- 网页界面语言作为当前轮默认回答语言；当前提示词使用其他语言或明确指定其他语言时以当前提示词为准，跨对话上下文中的语言要求不能覆盖本轮选择
- 即时停止、风险分级、JSONL 本地审计
- 可热加载的 Python 插件 SDK
- 本地 Web 控制台（仅监听 `127.0.0.1`）
- 安全的本地 Markdown 渲染（标题、列表、表格、代码块、链接）
- 界面固定为“总控 Agent”，由总控在内部调度 Computer Use、编码、办公、Guardian、OpenClaw 等专用工具；用户不能因误选模式而禁用任务所需能力
- 高阶 Computer Use 观察器、进程管理与隐私分级剪贴板工具
- SQLite 任务/事件持久化、重启中断恢复和全部任务侧栏
- 可关闭的跨对话上下文：最多扫描最近 50 个已完成普通聊天，相关性与轮次权重逐轮递减；过长的单个来源聊天会先压缩到约五分之一再注入。隐私聊天、失败过程、工具日志和密钥不会被带入
- 有边界的项目规则发现：读取工作区 `AGENTS.md`、`CLAUDE.md`、`GEMINI.md`、Copilot、Cursor 和 Windsurf 规则；这些规则只是项目偏好，不能覆盖当前用户要求和安全边界
- SQLite 长期记忆、工作区 `SKILL.md` 发现、安全网页读取/搜索
- 官方 OpenClaw `2026.7.1-2` 隔离侧车、项目私有 Gateway token、动态工具与插件目录
- 当前工作区已实测 42 个 OpenClaw 工具、68 个插件（50 个已加载）和 DeepSeek V4 Flash 最小 Agent 回合
- 窄屏抽屉导航、能力地图和运行时诊断
- 基于官方 TypeScript SDK 的真实 stdio MCP 实例，完整执行 `initialize → tools/list → tools/call`
- MCP 工作区工具：运行时信息、产物列表、产物读取和工作区边界保护的产物写入
- SQLite 持久化定时任务：可明确设置首次执行时间、每天/固定间隔/单次、执行内容和可选结束时间；支持启停、立即运行、停机错过触发合并和重启恢复
- Windows Job Object OS 级进程沙箱：暂停后入 Job、进程树收容、进程数/内存/CPU/墙钟限制和敏感环境变量过滤
- 产物、定时任务、分类设置和语音界面；产物可下载，运行时状态与密钥配置状态不会泄露密钥明文
- OpenClaw 浏览器真实回归：CDP 启动、精确域名白名单、导航、等待、ARIA 快照、页面求值和元素点击

## 实时语音

创作台与阅读器已经整体移除。语音现在是主输入栏上的独立入口，所有语音偏好统一放在 **设置 → 语音**，语音窗口本身只保留对话所需控件。

实时语音支持连续转写、可编辑的 Markdown 原文与安全实时预览、说完自动发送、自动朗读最终回复、静音/重试、音量反馈，以及通过按钮或直接说话打断朗读。Markdown 原文会保持不变地交给 Agent，发送后的语音回合则与普通网页聊天一样渲染。它沿用当前聊天上下文和普通 Agent 工具循环。设置页可选择听写语言、Windows 已安装朗读声音、语速、自动朗读、自动发送和回复后继续聆听。“自动”听写会跟随 Windows 系统语音区域，完全不受中英文界面切换影响；也可以明确选择 20 种常用语言。电脑端听写与朗读由 Windows/WebView 提供；Telegram 语音消息仍通过可选 Hugging Face Read token 对应的 Whisper 路线转写。飞书和 Telegram 继续输出普通纯文本及平台原生附件。

## 快速启动

需要 Windows。解压完整压缩包后，直接双击：

```text
Qelvane Desk.exe
```

现在双击打开的是原生 Windows 桌面应用窗口，不再弹出普通浏览器标签，也不会在界面里显示 `127.0.0.1`。EXE 会在后台准备依赖、启动仅绑定回环地址的 Agent 核心，再把现有界面嵌入 Microsoft WebView2。首次使用且从未保存主题时，界面默认采用白色浅色主题和黑色强调色；已有用户保存的主题不会被覆盖。原生启动卡片使用正式应用图标、分层状态文字、自绘动画进度条和本机数据提示，不再呈现普通安装器式界面。应用通过当前文件夹的身份摘要防止误连另一个解压副本；确认端口属于旧 Qelvane 服务时才会结束其进程树，若属于其他软件则停止启动且不会误杀。它还会自动检查 Python、Node.js 和包内 OpenClaw；首次准备依赖会比后续启动更久。每次启动使用独立的 `data/launcher-*.log`，`data/launcher-latest.txt` 保存最新日志路径。

桌面壳支持单实例、高 DPI、再次双击恢复窗口、后端断线自动重连，以及渲染进程异常恢复。外部网页链接交给系统浏览器，Qelvane 自身页面始终留在应用窗口。关闭窗口会销毁本次 WebView2 渲染会话，避免隐私聊天继续停留在屏幕；飞书/Telegram 的后台接收能力保持原有行为。

启动器会记录 Python 版本、应用绝对路径、`pyproject.toml` 和 Node 锁文件的依赖指纹。移动或复制整个文件夹后，下一次双击会自动把 `.venv` 的 editable 安装改到新路径；虚拟环境损坏或版本低于 3.11 时会自动重建。依赖未变化且关键模块完整时，后续启动会直接跳过 `pip`/Node 安装。随压缩包提供完整且版本匹配的 `work/openclaw-runtime/node_modules` 时，启动器会验证 OpenClaw、基础 skills 和 MCP SDK 后直接使用，不再联网重装。

发布包现已内置类似 Docker、但无需安装 Docker Desktop 或启用 Hyper-V 的便携运行时容器：`work/python-runtime`、`.venv`、`work/node-runtime` 与 `work/openclaw-runtime/node_modules` 分别携带 Python、Python 依赖、Node.js/npm、OpenClaw、skills 和 Node 依赖。启动器始终优先使用带官方签名的包内运行时，包内组件确实缺失或损坏时才回退到电脑上的兼容版本。正式发布包不再携带 Python 和 Node 安装包。`work/runtime-bundle.json` 只记录组件版本与相对路径，不写死打包电脑路径。

### 可选的离线安装包

正常发布包无需安装包，并且不再携带 Python、Node.js 或 WebView2 安装程序。只有在包内运行时被删除或损坏、并且电脑也没有兼容版本时，用户才可自行下载以下官方安装包放到 `Qelvane Desk.exe` 同一文件夹中进行手动恢复：

- Python：官方 64 位安装器统一命名为 `python-installer.exe`，仅在包内 Python 与电脑兼容 Python 均不可用时使用。
- Node.js：官方 Node.js 24 x64 MSI，例如 `node-v24.18.1-x64.msi`，仅在包内 Node.js 与电脑兼容 Node.js 均不可用时使用。兼容范围为 Node `22.22.3+（低于 23）`、`24.15.0+（低于 25）` 或 `25.9.0+`。
- 桌面渲染：启动器优先使用电脑已有的微软 Evergreen WebView2 Runtime；若缺失但电脑已有 Microsoft Edge，则回退为没有标签栏和地址栏的 Edge 应用窗口，而不是普通浏览器页面。用户可自行把微软签名的 `MicrosoftEdgeWebview2Setup.exe` 放到应用旁边进行恢复，但正式发布包不携带它。

本地安装包必须带有有效的官方 Authenticode 签名：Python 的签名者须为 Python Software Foundation，Node.js 的签名者须为 OpenJS Foundation/Node.js Foundation；签名不符时启动器会拒绝运行。安装窗口会显示在桌面上，由用户完成官方安装流程，随后启动器会重新检测版本并继续。

正式发布时还应使用受信任的企业或公共代码签名证书签署 `Qelvane Desk.exe`。把 `Cert:\CurrentUser\My` 中带私钥的证书指纹写入 `QELVANE_SIGNING_CERT_THUMBPRINT`，再运行 `launcher\build-launcher.ps1 -RequireSigned`；构建脚本会使用 SHA-256、可信时间戳并验证签名有效。自签名证书无法建立 SmartScreen 信誉，因此不会被伪装成正式发布修复。

OpenClaw 和它的 skills **没有第三个官方安装 EXE/MSI**。本项目把 OpenClaw 固定为 `2026.7.1-2`：联网时通过项目内 `package.json`/锁文件由 npm 或 pnpm 安装；离线发布时应直接随包提供完整的 `work/openclaw-runtime/node_modules`。不要下载名称类似“OpenClaw skills installer.exe”的第三方程序。

启动器还会把该版本内置的全部官方 skills 同步到当前包的 `skills/openclaw-bundled/`。每次启动时，Qelvane Desk 都会把 **当前 EXE 所在文件夹** 写成隔离 OpenClaw workspace，并启用 skill 文件监视；因此整个包移动到其他路径后仍从新路径动态发现 skills，不保留打包电脑的绝对路径。OpenClaw Agent、Gateway、插件和 skills 默认强制使用当前包的 `work/openclaw-runtime/`、`data/openclaw/` 与 `skills/`；运行时直接由 Node 打开包内 `node_modules/openclaw/openclaw.mjs`，不使用可能残留旧绝对路径的 pnpm `.bin/openclaw.cmd`。只有包内入口确实缺失或无效时，才依次回退到 `DEEPDESK_OPENCLAW_CLI` 明确指定的电脑版本和系统 `PATH` 中的 OpenClaw；能力状态会显示 `configured-fallback` 或 `system-fallback`，不会静默混用。即使回退，workspace、配置和状态目录仍保持在当前 Qelvane 文件夹内。可以把自己的 OpenClaw skill 放入 `skills/local/<名称>/SKILL.md`，下一轮 Agent 调用会自动看到它。skills 仍受操作系统、所需命令、API 密钥和 OpenClaw eligibility 检查约束，复制文件不会伪造缺失的外部能力。

在未手动配置远程 Gateway URL/token 时，每个软件副本还会根据当前包的绝对路径分配一个稳定的独立回环端口，并在启动命令中显式传给 OpenClaw。公开包、个人包和移动后的副本不会再误连同一 `18789` 端口而产生 `gateway token mismatch`；应用正常退出时也会结束自己创建的 Gateway 进程树，不会遗留占用端口的后台进程。

如果启动器被系统安全策略拦截，或者需要查看详细的安装输出，可在 PowerShell 中使用备用启动方式：

```powershell
.\start.ps1
```

发布包会优先复用包内 `.venv`、经过官方签名验证的基础 Python 运行时、Python 依赖和私有 MCP/OpenClaw Node 依赖。移动或解压到另一台电脑时，启动器只重写动态路径，不再因为目录变化清空环境或联网重装；只有文件确实缺失/损坏时才执行修复安装。Agent 内核仍只绑定 `127.0.0.1` 作为桌面进程内部通信，不向局域网开放；若 `.env` 或进程环境设置了 `DEEPDESK_PORT`，EXE 与后端会共同使用该私有端口。

配置放在 `.env`。该文件已被 Git 忽略，不应提交。模板见 `.env.example`。

说明：源码中的 `deepdesk` 小写模块名、环境变量前缀和 MCP 标识是旧版本兼容命名，不能直接改成显示名，否则会破坏已有配置和插件导入；所有产品界面、API 标题、提示词和文档品牌均已统一为 Qelvane Desk。

也可以直接在设置页填写 DeepSeek 主密钥、可选备用密钥和可选 Gemini 免费密钥。保存后密钥只写入本机 `data/provider-keys.json`，接口只返回“已配置/未配置”状态，不会回显明文；设置页留空会保留当前值。

## 多模型 API 供应商

设置页的“模型 API 供应商”可以同时保存最多 25 个额外模型配置，供应商支持 OpenAI、Anthropic、Google 和 xAI。DeepSeek 使用上方独立的主密钥和可选备用密钥，不在这里重复添加。每一行填写三项：供应商、该供应商 API 使用的精确模型调用名、API Key。密钥只保存在本机 `data/provider-keys.json`，设置接口仅返回是否已配置，不回显明文。

保存后，这些模型会同时出现在“默认模型”和发送栏下方的“回答模型”中。发送栏手动选择的优先级最高；发送栏保持“自动模型”时，会采用设置中选定的具体默认模型。若要启用跨所有已配置供应商的任务智能路由，请把设置中的默认模型选择为“自动选择最佳已配置模型”；该路线对普通且关注成本的任务略微倾向 GPT-5.6 Luna，对高难或具有明确任务特征的工作选择能力更强或更匹配的模型。

发送栏下方另有 Codex 风格的分段“思考深度”拉条。设置中的默认值为“高”，每个任务可单独覆盖为自动、低、中、高、极高或最高。这些档位会映射到 GPT-5.6 的 `reasoning.effort`/`reasoning_effort`、Claude 的自适应思考与 `output_config.effort`（Claude 4.5 使用兼容的思考预算）、Gemini 的 `thinkingLevel`，以及 DeepSeek V4 支持的 high/max 档。模型没有完整档位时使用最接近的原生档位，不再静默关闭推理。GPT 模型另有独立的“快速回答”开关：它只对当前任务生效、始终由用户手动开启、不会自动启用，并把 GPT 推理设为 `none`，但不会改变设置中保存的默认思考深度。

同一页面另有独立的 [AI Code Mirror](https://www.aicodemirror.ai/) 中转站卡片。通用密钥可同时启用 Claude 第三方逆向渠道、OpenAI 官方订阅渠道和 Gemini；另有一个可选的 `claude-fable-5` 专用密钥。专用密钥配置后只会覆盖 Fable 5 的调用凭据，其他模型仍使用通用密钥。两个字段都只写不回显，并分别显示配置状态。公开包提供两个空白入口和注册网址，但不内置任何个人密钥；个人包可在本机保存用户自己的专用密钥。

聊天名称由第一轮实际执行任务的模型生成。自动路由会先等待明确的 `model_selected` 结果，再交给该模型命名；若供应商不可用，就保留本机生成的精简标题，不再让另一个模型错误标注身份。

AI Code Mirror 保留了原供应商的精确 API 模型 ID，因此 Qelvane 会按原模型官方规格映射所有内置中转模型：GPT-5.4/5.5/5.6 为 1,050,000 token；Claude Opus/Fable/Sonnet 以及 Opus 4.6–4.8 为 1,000,000；Claude Haiku 4.5 为 200,000；三款已配置 Gemini 模型为 1,048,576。只有无法与官方模型 ID 对应的用户自填模型才显示为保守回退值；供应商一旦返回上下文溢出错误，系统会立即触发压缩恢复。中转协议与模型目录已按其 [Claude SDK 文档](https://www.aicodemirror.ai/dashboard/sdk-docs)、[OpenAI SDK 文档](https://www.aicodemirror.ai/dashboard/openai-sdk-docs)、[Gemini SDK 文档](https://www.aicodemirror.ai/dashboard/gemini-sdk-docs)和[价格/模型目录](https://www.aicodemirror.ai/dashboard/pricing)核对，上下文数值则按原供应商的 [OpenAI 模型比较](https://developers.openai.com/api/docs/models/compare)、[Claude 上下文窗口文档](https://platform.claude.com/docs/en/build-with-claude/context-windows)与 [Gemini 模型规格](https://ai.google.dev/gemini-api/docs/models)核对。

Qelvane Desk 对 DeepSeek、OpenAI、Google 和 xAI 使用其官方 Chat Completions 端点：OpenAI 为 `https://api.openai.com/v1/chat/completions`，Google 为 `https://generativelanguage.googleapis.com/v1beta/openai/chat/completions`，xAI 为 `https://api.x.ai/v1/chat/completions`；DeepSeek 沿用项目配置的 `/chat/completions` 端点。Anthropic 不走仅适合兼容测试的 OpenAI 转接层，而是使用生产用途的原生 `https://api.anthropic.com/v1/messages`，并由 Qelvane Desk 转换 system 消息、工具定义、工具结果和流式事件。用户无需填写 Base URL，但模型调用名必须与自己的账号权限一致。需要核验当前或不确定信息时，智能体可按本轮实际模型调用 OpenAI Responses `web_search`、Claude Messages `web_search_20250305`、Gemini `google_search` 或 DeepSeek 原生搜索；三家官方密钥与 AI Code Mirror 对应中转线路均已接入，并确保密钥不跨供应商域名。若某条中转线路不透传原生 server tool，则自动改用 Qelvane 的网页搜索/后台浏览器证据链，不让任务直接失败。

官方接口说明：[DeepSeek Chat Completion](https://api-docs.deepseek.com/api/create-chat-completion)、[OpenAI API authentication](https://platform.openai.com/docs/api-reference/authentication)、[Anthropic Messages API](https://platform.claude.com/docs/en/api/messages/create)、[Google Gemini OpenAI compatibility](https://ai.google.dev/gemini-api/docs/openai)、[xAI Chat Completions](https://docs.x.ai/developers/model-capabilities/legacy/chat-completions)。

## 图片识别与备用链路

Qelvane Desk 每次新任务或继续任务都会重新探测该次请求实际使用的公网出口国家/地区。原始公网 IP 只用于本次探测，立即丢弃，不写数据库、不写日志，也不会复用上一次结果。所有地区的文字识别都先走 Windows OCR，再按需使用百度飞桨 PP-OCR 本机 AI；只有非中国大陆出口且任务确实需要理解画面语义时，才会调用已配置的 Google 备用端点。中国大陆截图不会转发给 Google。

飞桨 OCR 使用 CPU 版 ONNX 推理并懒加载，不要求独立显卡。随包模型约几十 MB，适合低配置 Windows 电脑；它负责文字识别，非文字画面语义仍由可用的视觉模型处理。

```env
DEEPDESK_VISION_BASE_URL=https://generativelanguage.googleapis.com/v1beta/openai
DEEPDESK_VISION_MODEL=gemini-3.6-flash
DEEPDESK_VISION_API_KEY=
```

社区 Space 会把图片上传到 Hugging Face 托管服务，因此可用性和限流不由 Qelvane Desk 控制。上传前会先用本机 OCR 检查常见凭据格式；发现疑似 API 密钥或 Bearer Token 时会停止云端上传。备用端点需兼容 OpenAI Chat Completions 的 `image_url` 内容格式。

## OpenClaw 兼容层

Qelvane Desk 不复制一份会快速过时的工具列表，而是连接官方 OpenClaw CLI/Gateway，实时读取 `tools.catalog` 和插件注册表。当前隔离状态位于 `data/openclaw/`，Gateway token 不会出现在 API、UI、日志或 OpenClaw 配置中；DeepSeek 密钥仅通过子进程环境传递。

已验证的目录包含文件、运行时、网页、记忆、会话/子代理、浏览器、消息、cron、Gateway、节点、目标/计划、图像/音频/视频等 42 个工具。能否执行仍取决于 OpenClaw profile、已启用插件、外部账号凭据和 Qelvane Desk 的高风险审批。Telegram、Slack、WhatsApp 等渠道不能在没有对应账号授权时伪装成可用。

Qelvane Desk 不再设置公网域名白名单，全部公网域名均可访问；回环、局域网、链路本地、云元数据和直接输入的非公网 IP 仍由 SSRF 防护拦截。在 Clash/Mihomo 等 Fake-IP DNS 网络中，程序会用两个独立公网域名确认当前确为 Fake-IP 模式，再放行所有 `198.18.0.0/15` 域名映射；直接输入的 `198.18.x.x` 地址仍会被拦截。

## Telegram 可选渠道

Telegram 通道使用免费的官方 Bot API 和长轮询，不需要公网回调地址。支持接收文字、图片、PDF、Word、PowerPoint（`.ppt`/`.pptx`）、Excel、TXT、音频和视频；文件消息与紧接着发送的说明文字会短暂等待并自动合并成同一任务。任务最终回复明确引用的本地图片、文档、音频或视频会作为 Telegram 原生媒体/文件消息回传。

App 实时语音和 Telegram 语音消息可直接口述修改任意受支持设置，不需要在转写文字前添加 `//`。宿主会单独标记真实语音来源；Telegram 普通文字、网页文字和飞书文字仍必须以 `//`、`／／`、`/／` 或 `／/` 开头，历史对话、附件及模型输出不能继承或伪造语音授权。

### 接入步骤

1. 在 Telegram 中打开官方 [@BotFather](https://t.me/BotFather)，发送 `/newbot`，依提示设置机器人名称和用户名。
2. 复制 BotFather 返回的 Bot Token。不要把 Token 发送给其他人或提交到公开仓库。
3. 打开 Qelvane Desk →“设置”→“Telegram 可选渠道”，填写 Bot Token；默认 Chat ID 可以留空，然后保存。
4. 在 Telegram 中打开刚创建的机器人并发送 `/start` 或任意一条消息。Qelvane Desk 会通过 `getUpdates` 收到消息，并自动把该会话记录为默认 Chat ID。
5. 发送文字或附件验证入站；让智能体生成一个文件并在最终回复中写出其绝对路径，验证原生文件回传。

Telegram Bot API 的 `getUpdates` 与 webhook 不能同时使用。如果这个 Bot 以前配置过 webhook，请先通过官方 `deleteWebhook` 方法删除，否则长轮询收不到消息。标准 Bot API 的 `getFile` 下载上限为 20 MB；本应用回传普通文件上限为 50 MB，原生图片按 10 MB 上限处理。大文件会返回明确错误，不会静默丢失。

官方说明：[Telegram Bot API](https://core.telegram.org/bots/api)、[Bots FAQ 文件限制](https://core.telegram.org/bots/faq)。

## 飞书可选渠道

Qelvane Desk 使用飞书官方 Python SDK 建立长连接，接收 `im.message.receive_v1` 事件并通过 `feishu` 工具回复消息。不需要 Verification Token、Encrypt Key、公网域名或 HTTP 回调地址。

### 接入步骤

1. 登录[飞书开放平台](https://open.feishu.cn/app)，创建一个企业自建应用。
2. 进入应用的“添加应用能力”，启用“机器人”。设置机器人名称和说明。
3. 在“权限管理”中至少开通以下应用身份权限：
   - 读取用户发给机器人的单聊消息：`im:message.p2p_msg:readonly`（或对应的可读写权限）；
   - 以应用身份发送消息：`im:message:send_as_bot`（也可以开通包含收发能力的 `im:message`）。
   - 接收、上传和回传图片：`im:resource`（获取与上传图片或文件资源）。
4. 打开“事件与回调”或“事件订阅”：
   - 订阅方式选择“使用长连接接收事件”；
   - 添加“接收消息 v2.0”事件，事件标识为 `im.message.receive_v1`；
   - 保存配置。
5. 在“版本管理与发布”中创建并发布一个版本，并确保目标飞书账号位于应用的可用范围内。只保存开发配置但未发布时，普通用户可能搜索不到机器人或无法触发事件。
6. 在“凭证与基础信息”复制 `App ID` 和 `App Secret`。打开 Qelvane Desk →“设置”→“飞书可选渠道”，填写这两项并保存。请勿公开或提交 `App Secret`。
7. `Open ID` 可以先留空。保持 Qelvane Desk 运行，在飞书客户端搜索刚创建的机器人，并从目标账号向机器人发送“你好”。Qelvane Desk 收到事件后会自动保存该账号在**当前应用下**的 Open ID，随后回复任务 ID；之后智能体即可主动向这个账号发送消息。
8. 回到 Qelvane Desk 的“能力地图”或运行状态，确认飞书显示“已配置”、长连接正在运行且没有错误。

### 开通文件收发权限（解决 `99991672`）

1. 打开[飞书开放平台](https://open.feishu.cn/app)，进入与 Qelvane Desk 设置中 `App ID` 对应的企业自建应用。
2. 在左侧进入“权限管理”→“API 权限”，搜索 `im:resource`。
3. 申请并开通“获取与上传图片或文件资源”权限（权限标识 `im:resource`）。它同时覆盖消息资源下载和文件上传；如果页面只提示 `im:resource:upload`，该权限只能满足上传，完整收发仍应申请 `im:resource`。
4. 如果企业管理员审批开启，提交权限申请并等待管理员通过。应用创建者也可能需要在管理后台确认授权范围。
5. 进入“版本管理与发布”，创建一个新版本，把新权限写入版本并发布。**只在权限页勾选但没有重新发布，线上机器人仍然没有该权限。**
6. 确认应用可用范围包含接收测试消息的飞书账号，然后重新启动 Qelvane Desk 或在能力地图中重新探测运行时。
7. 验证入站：向机器人发送一个小于 25 MB 的 `.txt`、`.doc` 或 `.docx` 文件，并让它概括内容。文件和说明可分成相邻两条消息发送：Qelvane Desk 会短暂等待并自动合并；没有补充说明时会在 15 秒后直接处理文件。验证出站：让机器人生成一个 `.txt` 或 PDF 并回传；飞书中应显示为可下载的原生文件消息，而不是本地路径文字。

飞书官方接口说明：[上传文件](https://open.feishu.cn/document/uAjLw4CM/ukTMukTMukTM/reference/im-v1/file/create)、[下载消息中的资源文件](https://open.feishu.cn/document/server-docs/im-v1/message/get-2?lang=zh-CN)、[发送消息](https://open.feishu.cn/document/server-docs/im-v1/message/create?lang=zh-CN)。

### Open ID 注意事项

- Open ID 是“用户 + 应用”维度的标识。同一个用户在不同飞书应用中的 Open ID 不同，不能复制另一个机器人或应用获得的 `ou_...` 值。
- 如果发送时报 `99992361: open_id cross app`，说明设置中的 Open ID 属于其他应用。不要继续重试旧值；先确认第 3～5 步已完成，再给当前机器人发送一条新消息，让 Qelvane Desk 自动学习正确值。
- 如果发送消息后 Open ID 没有更新，优先检查是否选择了长连接、是否订阅 `im.message.receive_v1`、是否具备单聊消息读取权限，以及应用新版本是否已经发布。
- `99991672` 表示缺少飞书接口列出的权限。按错误信息申请对应权限后，还需要发布新版本使权限生效。
- 同一台电脑不要同时运行多个 Qelvane Desk 实例。飞书长连接采用集群投递，同一应用存在多个连接时，一条事件只会随机交给其中一个连接。

收到机器人消息后，Qelvane Desk 会创建本地任务，先回复任务 ID，再在任务完成后发送最终结果。设置页中的密钥只保存在本机 `data/provider-keys.json`，API 不会回显明文。

飞书和 Telegram 中的图片，以及 PDF、Word（`.doc`/`.docx`）、PowerPoint（`.ppt`/`.pptx`）、Excel（`.xlsx`/`.xls`）和纯文本（`.txt`）文件，会下载到该任务的本地附件目录。图片进入 OCR/视觉链；文档由内置 `document.inspect` 工具提取正文、幻灯片文字、表格或工作表数据后交给模型处理。旧版二进制 `.doc` 会优先通过本机 Word 的只读后台接口解析，并兼容 RTF 或改扩展名的 OOXML 文件；旧版二进制 `.ppt` 会优先通过本机 PowerPoint 的只读后台接口解析，其次尝试 LibreOffice 无头转换，改扩展名的 OOXML 也可直接读取；旧版 `.xls` 支持读取，新建 Excel 产物使用 `.xlsx`。

任务最终回复中明确引用的本地结果图片会以飞书原生图片消息回传；明确引用的 PDF、`.docx`、`.pptx`、`.xlsx`、`.xls`、`.txt` 结果会先上传取得 `file_key`，再以飞书原生文件消息回传。输入附件和智能体内部临时文件不会被自动回传。入站附件的本机处理上限为 25 MB；飞书原生图片回传遵循 10 MB 上限，原生文件回传遵循 30 MB 上限。收发文件需要为应用开通并发布飞书权限 `im:resource`，发送消息还需要 `im:message:send_as_bot`。

飞书和 Telegram 收到的任务都会被持久化标记为对应的远程渠道。模型会在当前任务的系统级渠道规则中被要求：所有发给远程用户的回复使用普通纯文本，不使用 Markdown；用户要求写入文件、代码或工具参数中的原始字符不会被发送出口二次转换。网页聊天和定时任务默认允许 Markdown，即使跨对话参考中包含旧远程任务的“禁用 Markdown”规则，也会明确忽略该旧规则。

## Android 手机助手

Qelvane Mobile 可把用户绑定的 Android 手机作为桌面智能体的加密工具。打开“设置 → Android 手机控制”，点击“绑定 Android 手机”，让手机和电脑连接同一局域网，然后用 Android 应用扫描有效期五分钟的二维码。系统会为每台设备派生独立密钥；控制数据使用 AES-256-GCM 加密，Windows 端保存的设备密钥由 DPAPI 保护。

手机端支持状态和无障碍树查看、截图、点击、滑动、普通文字输入、返回/主页/最近任务、启动应用和打开网址。只有桌面任务选择“自主”，并且手机端同时打开“允许自主模式直接操作本手机”时，普通操作才会免除 Qelvane 的逐次审批。Android 系统权限、认证及验证码页面、生物识别、支付、安全设置和应用安装仍由用户本人操作。连接期间 Android 通知栏会持续显示控制状态；用户可随时在桌面设置中解除设备绑定。

开发者可使用 Android Studio 构建，或者执行 `work/openclaw-reference/apps/android/gradlew.bat -p mobile/QelvaneMobile :app:assembleDebug`。调试 APK 输出到 `mobile/QelvaneMobile/app/build/outputs/apk/debug/app-debug.apk`。

## 免费档图片、视频与音乐生成

Qelvane Desk v1.0 可通过 Pollinations、Hugging Face 官方公开 Space 与 ModelScope 公共创空间生成媒体。打开
[Pollinations 账户页面](https://enter.pollinations.ai/)申请 API Key，然后在“设置 → 供应商密钥”中填写
“Pollinations 免费媒体密钥”后可使用 Pollinations 当前明确标为零价格的模型；该密钥不是视频或音乐
免费链路的必填项。密钥仅保存在本机 `data/provider-keys.json`，不会回显到网页或任务日志。

图片生成优先使用 Pollinations 官方公开、免密且不会消耗账户额度的图片端点。免费视频会依次尝试
Lightricks 官方 `LTX-Video Distilled` 与 `LTX 2.3` ZeroGPU Space、独立的 Wan 2.1 公共演示池、
阿里 PAI `EasyAnimate` ModelScope 公共创空间，以及 Pollinations 实时目录中价格明确为 0 的模型。
免费音乐可使用 Stability AI `Stable Audio 3`、ACE-Step、Meta `MusicGen`、Stable Audio Open、
DiffRhythm、HeartMuLa 与 EzAudio ZeroGPU Space，再尝试通义 `InspireMusic` ModelScope 公共创空间和
Pollinations 的零价模型。某个共享额度池耗尽时，会短暂跳过同一额度池内的其他模型，直接尝试下一家
独立供应商；只有所有已验证免费路径都失败才会报告错误，绝不会自动切换到付费模型。能力地图只有在
运行状态正常且实际 Gradio 配置端点可访问时才会把模型标为可用，避免“页面显示 Running、实际无法连接”。

若要增加 ZeroGPU 免费额度，可注册 Hugging Face 免费账号，打开
[Access Tokens](https://huggingface.co/settings/tokens)，创建权限为 `Read` 的令牌，然后填入
“设置 → 供应商密钥 → Hugging Face 免费Read令牌”。Hugging Face 官方当前说明：匿名调用每天约 2 分钟，
登录的免费账号每天约 5 分钟。令牌只会传入 Gradio 客户端的认证字段，不会写入提示词、URL、任务事件
或状态响应；若要求绝对不产生费用，请使用未购买预付费额度的免费账号。

图片、视频和音乐都会保存到 `outputs/`，电脑端可以直接预览或下载；飞书任务结果会把图片作为图片
消息回传，把音频和视频作为原生文件回传；Telegram 会分别使用 `sendPhoto`、`sendAudio`、`sendVideo`
或 `sendDocument`。生成器把音乐限制在 25 MB、视频限制在 29 MB，以同时满足当前飞书与 Telegram
回传路径；若超过限制，会要求缩短时长而不会静默丢失文件。飞书和 Telegram 收到的图片、音频和视频
也会下载到本机附件目录并交给对应任务。

建议为 Qelvane Desk 单独创建受限密钥，不要复用其他项目的高预算密钥。免费共享 GPU 暂时不可用时，
音乐或视频生成会显示可重试的明确原因，而不是产生费用。

### 中国大陆出口的免费媒体路线

每次任务会重新识别当前公网出口。中国大陆出口不会默认依赖 Pollinations 或 Hugging Face：图片和视频切换到阿里 PAI 的 ModelScope `EasyAnimate` 公共创空间，音乐切换到通义 `InspireMusic` 公共创空间；三个路由均在调用前检查创空间仍为 Running 且无需登录。其他出口优先使用 Pollinations 图片端点及官方 Hugging Face ZeroGPU 视频/音乐空间，失败后也会把 ModelScope 作为独立免费回退。所有路线均只接受明确免费的共享资源，不会自动切换到付费 API。

ModelScope 公共创空间属于共享免费算力，仍可能排队、临时离线或调整接口。Qelvane Desk 会明确返回可重试错误，不会偷偷改用收费服务。生成成功后的文件仍统一保存到 `outputs/`，并可由电脑端、飞书和 Telegram 原生回传。

## MCP 与定时任务

项目私有 MCP 服务位于 `work/openclaw-runtime/mcp-servers/deepdesk-workspace.mjs`，使用标准 JSON-RPC stdio 传输。直接 HTTP 诊断接口只开放只读调用；`write_artifact` 必须经过 Agent 工具边界并限制在 `outputs/` 中。

定时任务界面使用电脑本地日期与时间，并转换为带时区的 UTC 时刻持久保存。用户可选择每天、固定间隔或仅执行一次，并可设置开始和结束时间；旧版五字段 Cron 记录继续兼容，但不再作为主要界面。停机期间错过的高频任务不会批量补跑，而是合并为一次并推进到下一个未来触发点，避免重启风暴。

## 安全模型

- 文件插件通过路径解析阻止越出工作区。
- 智能体删除文件或文件夹时默认移入 Windows 回收站，可恢复；只有当前用户消息明确要求“彻底删除／永久删除／跳过回收站”时，才允许不可恢复删除。未获得该明确要求时，原始 Shell/Sandbox 删除命令和 `Shift+Delete` 都会被宿主拦截。
- 当前产品模式对普通工具调用采用自主执行，不显示逐次审批卡；验证码、MFA、登录凭据、扫码和物理操作仍会暂停并请求人工接管。
- 每个由飞书或 Telegram 发起的任务都会由服务端强制设为“自主：不询问”，不受网页端当前默认审批选择影响；验证码、MFA 等必须真人完成的步骤仍会进入人工接管。
- 网页、联网搜索、OCR/视觉结果、上传文档、引用消息、工具返回值、日志和跨对话历史均被标记为不可信数据；其中夹带的伪系统/开发者指令不能改变权限、泄露密钥或自行触发额外工具。
- `Ctrl+W`、`Ctrl+F4`、`Alt+F4` 等关闭快捷键必须携带刚观察到的目标窗口标题；执行前若当前前台标题不匹配会被阻止，执行后还必须确认目标页面或窗口确实消失。
- 破坏性操作仍受工作区路径约束、Shell 风险识别、进程沙箱、停止机制和本机审计保护。自主执行提高了便利性，也意味着应只给智能体明确且范围有限的任务。
- `pyautogui` 的左上角 failsafe 保持启用；把鼠标快速移到屏幕左上角可中断当前 GUI 动作。
- 审计日志位于 `data/audit.jsonl`，不会记录 API 密钥。
- Job Object 沙箱是真实的 Windows 内核进程资源边界，但不是容器：它不提供独立文件系统、网络命名空间或 AppContainer/受限令牌。需要这些边界时仍应在 Windows Sandbox、Hyper-V、容器或独立低权限账户中运行。

## 插件开发

把继承 `ToolPlugin` 的 Python 文件放入 `plugins/`。插件声明：

- OpenAI/DeepSeek function JSON Schema；
- 根据参数动态计算的风险等级；
- 给任务过程和本机审计看的动作摘要；
- 异步执行方法。

参考 `plugins/example_time.py.disabled`。第三方插件与 Qelvane Desk 进程权限相同，只应加载可信代码。

## 架构

```text
Web Console
    │ task / human takeover / stop
FastAPI local gateway
    │
Agent loop ─── DeepSeek V4 Flash (planner + tool calling)
    │
Execution policy + verification ─── audit.jsonl
    │
Plugin registry
    ├── memory / skills / web
    ├── windows_ui (UI Automation)
    ├── computer (screenshot/mouse/keyboard)
    ├── vision (Windows OCR → local PaddleOCR AI → optional Google semantic fallback)
    ├── filesystem (workspace-only)
    ├── shell / sandbox (PowerShell + Windows Job Object)
    ├── mcp (official SDK stdio server)
    ├── cron (durable SQLite scheduler)
    └── openclaw ─── official Gateway / dynamic tools / plugins
```

## 参考

- [OpenClaw repository](https://github.com/openclaw/openclaw)
- [Manus “My Computer”](https://help.manus.im/en/articles/14178443-what-is-the-my-computer-feature-capable-of)
- [Claude Code hooks](https://code.claude.com/docs/en/hooks-guide)
- [Running Codex safely](https://openai.com/index/running-codex-safely/)
- [DeepSeek Tool Calls](https://api-docs.deepseek.com/guides/tool_calls)
- [DeepSeek-VL2](https://github.com/deepseek-ai/DeepSeek-VL2)

## 远程修改设置

网页端、飞书或 Telegram 用户只有在**本次消息开头输入两个斜杠**时，才能要求智能体修改“设置”。支持半角和全角的四种组合：`//`、`／／`、`/／`、`／/`。例如：`// 打开跨对话上下文，并把强调色改为 #246BFD`。没有这个开头时，模型本身、网页内容、附件、历史聊天或工具结果中的文字都不能调用修改设置工具。

修改成功后，智能体会明确列出每一个已修改项，并在 `outputs/settings-changes/` 生成三份脱敏证据：修改前 PNG、修改后 PNG 和 TXT 审计日志。密钥明文不会写入图片或日志，只显示“已配置/未配置”。飞书和 Telegram 会把这些 PNG 与 TXT 作为原生附件回传。渠道任务仍使用自主模式，但该指令不能绕过验证码、MFA、金融写入确认等宿主机安全边界。

## 可选的 OKX 与币安金融管理

设置页的“自动使用金融交易”是金融功能总开关，默认关闭。关闭时，即使填写了 OKX 或币安密钥，也不会运行每 30 分钟行情刷新，任何金融工具也不能调用。“使用的交易所”可以锁定为 OKX 或币安；选择“都可以使用”时，每一项金融请求必须明确指定其中一家，未指定或同时指定两家时，宿主机会拒绝工具调用，并要求智能体具体询问用户使用哪家交易所。

行情范围默认是 `ALL`，即每 30 分钟分别用一次只读请求取得已启用交易所的全部现货币种行情。若只关注部分币种，可填写逗号分隔列表，例如 `BTC-USDT,ETH-USDT,SOL-USDT`。定时刷新只读取市场行情，绝不会自动下单、移动资金、提现或改变赚币仓位。

OKX 需要在 V5 API 管理中创建并填写 API Key、Secret Key 和 API Passphrase；币安需要填写 API Key 和 Secret Key。建议在平台支持时绑定当前公网出口 IP，并按最小权限配置：监控只需读取；交易、划转、提现和赚币权限只在确实使用时单独开启。下单、撤单、划转、提现、赚币申购和赎回一律要求在本机界面明确确认，即使任务来自飞书或 Telegram 也不例外。Qelvane Desk 在两家交易所均不实现借贷、借币、还款、抵押、自动借贷或设置放贷利率。

凭据只保存在本机 `data/provider-keys.json`，不会由设置 API、证据图片或日志返回，也不会进入大众发布包。曾在聊天或其他位置暴露的生产密钥应立即轮换，并换成绑定 IP、最小权限的新密钥。币安接口采用官方 Spot/Wallet REST 鉴权方式，参考[币安 Spot API](https://developers.binance.com/docs/binance-spot-api-docs/rest-api)和[币安 API 介绍](https://developers.binance.com/docs/binance-spot-api-docs)。

## 跨对话上下文权重

开启跨对话上下文后，系统最多扫描最近 50 个符合条件的已完成普通聊天。相关性会与平滑的轮次衰减权重组合：越近的聊天影响越大，越早的相关结论权重按顺序降低。单个来源聊天过长时，会先保留目标、关键约束与最终结果并压缩到原长度约五分之一，再进入本轮上下文；精确历史仍保存在本机数据库中。隐私聊天、失败任务、原始工具轨迹和密钥会被排除或脱敏。“重试其他两项”这类简短承接指令仍优先对应同一渠道紧邻的上一个已完成任务。

长时间运行的 Agent 任务支持递归自动压缩上下文。正常情况下，只有工作上下文达到约 85% 才触发；如果供应商更早明确返回上下文超限，则立即进入恢复压缩。新的续接检查点目标约为窗口的 45%，优先保留最近的工具参数、结果、文件路径、验证证据、用户原始目标和全部安全约束。每次压缩都会继续携带上一份累计进度账本；完整的脱敏压缩前记录同时归档在 `data/context-checkpoints/`，后续第 N 次检查点会列出前面全部归档路径，需要精确旧细节时可重新读取，因此不会在多次压缩后切断更早内容。原始聊天和任务事件仍保存在本机，只有发给模型的活动工作上下文会被压缩。

## 当前边界

这是可运行的开发构建，不是经过第三方安全审计的系统级 RPA 产品。浏览器复杂页面、画布应用和游戏通常缺少可访问性控件，需要视觉模型或 OpenClaw 浏览器运行时。项目已内置一个真实 MCP 实例；其他外部 MCP、渠道和云插件仍必须分别配置账号/凭据。“出现在目录中”不等于“已经授权可执行”。

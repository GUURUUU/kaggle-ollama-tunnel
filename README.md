# kaggle-ollama-tunnel 🦙🚇

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CI](https://github.com/GUURUUU/kaggle-ollama-tunnel/actions/workflows/ci.yml/badge.svg)](https://github.com/GUURUUU/kaggle-ollama-tunnel/actions/workflows/ci.yml)
[![Kaggle](https://img.shields.io/badge/Runtime-Kaggle%20T4%20x2-0085FF?logo=kaggle&logoColor=white)](https://www.kaggle.com/)
[![Ollama](https://img.shields.io/badge/Served%20by-Ollama-black?logo=ollama&logoColor=white)](https://ollama.com/)
[![OpenAI Compatible](https://img.shields.io/badge/API-OpenAI%20Compatible-412991?logo=openai&logoColor=white)](https://platform.openai.com/docs/api-reference)

**Run a full-precision open LLM on free Kaggle GPUs and call it from anywhere through a public OpenAI-compatible API.**

One notebook cell = 2× Kaggle T4 + Ollama + a Cloudflare tunnel. No server, no bill.

**Install (your machine only):**

```bash
pip install openai   # requests is also fine — that's the whole dependency list
```

![kaggle-ollama-tunnel demo](docs/demo.gif)

*The GIF shows the real flow: client install → the notebook's live banner → a chat completion call → response.*

---

## ⚙️ How it works

```
┌────────────────── Kaggle session (free) ─────────────────┐
│                                                          │
│  2x T4  ◀──  Ollama (tensor-split)  ◀──  OpenAI API      │
│              F16 abliterated         :11434              │
│              Llama-3.1-8B                ▲               │
└──────────────────────────────────────────│───────────────┘
                                           │ cloudflared
                                           ▼ quick tunnel
┌────────────────── Your machine ──────────│───────────────┐
│  Any OpenAI client ── base_url: https://<your-tunnel>.trycloudflare.com/v1
└──────────────────────────────────────────────────────────┘
```

| Layer | Role |
|---|---|
| **Ollama** | Local model server, OpenAI-compatible `/v1` endpoints included out of the box |
| **GGUF F16** | Unquantized weights for max quality (no probes/samplers stripped, `num_ctx 8192`) |
| **cloudflared** | Account-free Quick Tunnel: public HTTPS ↔ `127.0.0.1:11434` |
| **keep-alive loop** | 5-min heartbeat: pins weights in VRAM (`keep_alive: -1`), self-heals both Ollama and the tunnel |

---

## 🚀 Quick start

1. **Create a Kaggle notebook** → enable accelerator **GPU T4 x2** → disable internet requirement is not needed (internet must be **ON** to download packages/models/tunnel).
2. **Paste the cell** — either upload [`kaggle-ollama-tunnel.ipynb`](kaggle-ollama-tunnel.ipynb) or copy the entire contents of the main cell (the notebook has exactly one working cell, so it's a pure copy-paste).
3. **Run it.** After a few minutes you'll see:

```
======================================================================
✅ OPENAI-COMPATIBLE API IS LIVE
======================================================================
Base URL : https://<ephemeral-tunnel-id>.trycloudflare.com/v1
API Key  : ollama   (any non-empty string works)
Model    : abliterated-f16
======================================================================
```

### Client (Python)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://<your-tunnel-url>.trycloudflare.com/v1",
    api_key="ollama",  # placeholder — Ollama ignores it
)
resp = client.chat.completions.create(
    model="abliterated-f16",
    messages=[{"role": "user", "content": "Hello"}],
)
print(resp.choices[0].message.content)
```

### Client (curl)

```bash
curl https://<your-tunnel-url>.trycloudflare.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"abliterated-f16","messages":[{"role":"user","content":"Hello"}]}'
```

---

## 🔄 Swapping the model

Edit the candidate list at the top of the cell — the script picks the first repo that has an F16/F16-equivalent GGUF:

```python
REPO_CANDIDATES = [
    "mradermacher/Meta-Llama-3.1-8B-Instruct-abliterated-Sabresooth-GGUF",
    "legraphista/Meta-Llama-3.1-8B-Instruct-abliterated-IMat-GGUF",
]
```

Any single-file HF GGUF up to ~30 GB works. Point it at (2× T4 = 32 GB VRAM; the F16 8B here is 16 GB — plenty of headroom, and dual-GPU tensor-splitting is already enabled).

## 🎛️ Tuning defaults

They live in the generated Modelfile:

```dockerfile
PARAMETER num_ctx 8192
PARAMETER temperature 0.7
PARAMETER top_p 0.9
```

---

## ⚠️ Caveats (read before deploying)

- **The tunnel URL is ephemeral.** Every restart of cloudflared (fresh session, tunnel crash) prints a *new* quick-tunnel URL — update your client. Make a [named Cloudflare tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps) if you need a stable hostname.
- **Kaggle sessions die.** Free GPU quota (~30 h/week, 9–12 h per session) applies; the notebook auto-relaunches *Ollama*, not *Kaggle*. Re-run the cell when the session restarts.
- **Quick Tunnels have no SLA.** They're for experimenting — fine for a hobby bot, wrong for anything production-shaped.
- **Hardware**: `2x T4` required for the F16 8B spread; a single T4 works with a quantized GGUF instead.

## 🛗 Responsible use

Run models you're licensed to run and follow the [Kaggle ToS](https://www.kaggle.com/terms) and Cloudflare Terms. This repo is a mechanics showcase — what you point it at is your call. 🏰

---

## 📁 Structure

```
kaggle-ollama-tunnel/
├── kaggle-ollama-tunnel.ipynb   # the whole project: one cell, zero external deps (beyond HF hub)
├── docs/
│   ├── demo.gif                 # animated walkthrough (above)
│   └── social-card.png          # 1280x640 card for GitHub Social preview
├── .github/
│   ├── workflows/ci.yml         # notebook sanity check on every push
│   └── ISSUE_TEMPLATE/          # bug + feature templates
├── CONTRIBUTING.md
├── CHANGELOG.md
├── README.md
├── LICENSE
└── .gitignore
```

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) — short version: fork, branch, PR; the maintainer answers every issue within 24 h. First-timers: check the [`good first issue`](https://github.com/GUURUUU/kaggle-ollama-tunnel/labels/good%20first%20issue) label.

Roadmap — PRs welcome:
- [ ] Auto-update the README with the generated tunnel URL via Kaggle artifact
- [ ] Named-tunnel support (persistent URL)
- [ ] Multi-model registry with a model-picker curl command
- [ ] Watchdog that dumps logs to `/kaggle/working`

## 📜 License

[MIT](LICENSE) © 2026 — free as in free GPUs.

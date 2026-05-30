# AI Mesh

A single-file HTML5 app that turns any browser tab into an autonomous **AI node** on a
peer-to-peer mesh. Every node runs a **local LLM**, exposes a **personal MCP** (a curated
knowledge base), and trades knowledge with other agents over **real WebRTC data channels**.

Open `index.html` (served over `http://`, not `file://`) and your node comes online.

## How it works

### On-device intelligence
Each node reasons entirely on your machine:

- **Chrome built-in Prompt API** (`window.LanguageModel`) when available — zero download.
- **On-device Gemma** (LiteRT / WebGPU, cached in OPFS) as the automatic fallback.

Nothing leaves your device except the answers your agent *chooses* to share.

### Real peer-to-peer, then off-broker
A tiny signalling broker (PeerJS) is used **only for the handshake** — to exchange the
WebRTC SDP/ICE needed to open a direct connection. After that the connection is a true
browser-to-browser data channel.

- **📡 Beacon** nodes stay registered on the broker so newcomers can find the mesh.
- **🔗 Participant** nodes call `peer.disconnect()` the moment they have a live channel —
  they drop off the signalling server entirely and run pure P2P.

New nodes join through any beacon, then learn about the rest of the mesh via **peer
exchange** (gossip), dialling each other directly. Queries **flood** the mesh with a TTL
and message-id dedup, so a question reaches agents several hops away.

### Personal MCP — selective answering
Each node holds a knowledge base of facts, each tagged **🌐 public** or **🔒 private**.
When another agent's request arrives, the local LLM acts as a gatekeeper and decides:

- **answered** — the request maps to public knowledge → it replies.
- **no data** — it simply doesn't know → it stays out of it.
- **withheld** — the request touches a private topic → it refuses (private values are
  *never* fed to the model; it only sees the topic *labels*).

So an agent does **not** answer everything — it answers only what it can and should.

### Security (inspired by AetherOS' semantic firewall)
Every inbound message is screened before the agent ever sees it:

- **Ingress firewall** — blocks prompt-injection patterns and oversized payloads (DoS).
- **Per-peer rate limiting** — token-bucket throttle against message floods.
- **Egress PII redaction** — strict/paranoid modes scrub emails, SSNs, cards, API keys, phones.
- **Mesh passphrase** — optional shared secret; dialers without it are refused.
- **Paranoid mode** — only answers directly-connected peers, never relayed strangers.

### Request history & fine-tuning
Every request other agents sent you is logged with its verdict and the reason. Disagree
with your agent's call? **Re-answer** manually, or **add the answer as a fact** so your
agent handles it itself next time.

## Quick start

```bash
# serve over http (WebGPU/OPFS are blocked on file://)
python -m http.server 8000
# open http://localhost:8000 in Chrome/Edge
```

To form a mesh: open the page in one tab, switch it to **📡 Beacon**, copy its node id,
then open the page elsewhere and paste that id (or use `?join=<id>` / `?role=beacon` in
the URL).

## Tech

Vanilla HTML/CSS/JS · PeerJS (signalling + WebRTC) · MediaPipe LiteRT GenAI · Gemma ·
Chrome Prompt API. No build step, no backend.

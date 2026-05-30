# Product Roadmap
## WebRTC-to-SIP Gateway Platform
### Janus Gateway + SIP.js — Vision, Features & UI/UX

> **Current baseline:** Janus Gateway v1.4.2 · SIP.js v0.21.2 · Branch `momentum-VOIP-v1.0`

---

## Table of Contents

1. [Vision](#1-vision)
2. [Current State Inventory](#2-current-state-inventory)
3. [Phase 1 — Foundation & Developer Experience](#3-phase-1--foundation--developer-experience-q3-2026)
4. [Phase 2 — Production-Grade Platform](#4-phase-2--production-grade-platform-q4-2026)
5. [Phase 3 — Advanced Communication Features](#5-phase-3--advanced-communication-features-q1-2027)
6. [Phase 4 — AI & Intelligence Layer](#6-phase-4--ai--intelligence-layer-q2-2027)
7. [Phase 5 — Enterprise & Scale](#7-phase-5--enterprise--scale-q3-2027)
8. [UI/UX Design System](#8-uiux-design-system)
9. [Feature Priority Matrix](#9-feature-priority-matrix)
10. [Technical Debt & Infrastructure](#10-technical-debt--infrastructure)

---

## 1. Vision

Build a **world-class, open-source WebRTC communication platform** on top of Janus Gateway and SIP.js — one that any developer can self-host in under 30 minutes, any business can run at scale, and any end user finds beautiful and intuitive to use.

The platform bridges the gap between legacy telephony (SIP/PSTN) and modern real-time communication (WebRTC), with a polished UI, robust fraud prevention, and extensible architecture.

**Three pillars:**
- **Developer-first** — Clean APIs, great docs, one-command setup, scripting support
- **Operator-ready** — Fraud prevention, billing, monitoring, multi-tenant, HA clustering
- **User-delightful** — Beautiful softphone UI, seamless mobile experience, accessibility

---

## 2. Current State Inventory

### What exists today

| Component | Status | Notes |
|-----------|--------|-------|
| Janus Gateway core (ICE/DTLS/SRTP) | ✅ Stable | v1.4.2, actively maintained |
| SIP plugin (Sofia-SIP) | ✅ Stable | REGISTER, INVITE, BYE, HOLD, DTMF, REFER, SUBSCRIBE |
| VideoRoom SFU | ✅ Stable | Simulcast, SVC, E2EE, cascading |
| AudioBridge MCU | ✅ Stable | Spatial audio, RNNoise, mixing |
| Streaming plugin | ✅ Stable | RTP, RTSP, live/on-demand files |
| Record & Play | ✅ Stable | MJR format, post-processor to MP4/WebM |
| TextRoom (DataChannels) | ✅ Stable | Text-only conferencing |
| REST + WebSocket + MQTT + RabbitMQ transports | ✅ Stable | Full API coverage |
| Event handlers (WS, MQTT, GELF, RabbitMQ) | ✅ Stable | Real-time event streaming |
| Lua + Duktape scripting plugins | ✅ Stable | Custom logic without C recompile |
| SIP.js v0.21.2 | ✅ Stable | SimpleUser, SessionManager, full API |
| Fraud prevention backend (Node.js/TypeScript) | 📄 Documented | JWT tokens, wallet, whitelisting |
| HTML demos (Bootstrap 5) | ✅ Exists | Functional but developer-focused, not production UI |
| Setup guide + build docs | ✅ Exists | Comprehensive, recently updated |

### What is missing

- Production-quality softphone UI (current demos are developer scaffolding)
- Docker / docker-compose one-command deployment
- Admin dashboard (beyond raw API calls)
- Mobile apps (iOS/Android)
- Multi-tenant user management
- Billing and CDR (Call Detail Records)
- Real-time monitoring and alerting
- Horizontal scaling / clustering
- CI/CD pipeline for the full stack
- Automated fraud detection (ML-based)
- AI features (transcription, translation, noise suppression)


---

## 3. Phase 1 — Foundation & Developer Experience (Q3 2026)

**Goal:** Make the platform trivially easy to run, develop against, and deploy. Remove all friction from the first-run experience.

### 3.1 One-Command Docker Setup

**Problem:** Current setup requires manually building 4 C libraries from source (libnice, libsrtp, libwebsockets, Janus itself), which takes 30–60 minutes and breaks on missing dependencies (as seen with `cmake` and `meson` issues).

**Solution:**

```
docker-compose up
```

Deliverables:
- `Dockerfile` for Janus Gateway with all dependencies pre-built
- `docker-compose.yml` with Janus + Redis + Nginx reverse proxy
- Environment variable configuration (no manual `.jcfg` editing)
- Health check endpoints
- Volume mounts for config, recordings, and certs
- `docker-compose.dev.yml` for local development with hot-reload

```yaml
# docker-compose.yml (target state)
services:
  janus:
    image: momentum-voip/janus:1.4.2
    ports: ["8088:8088", "8188:8188", "20000-20100:20000-20100/udp"]
    environment:
      JANUS_PUBLIC_IP: ${PUBLIC_IP}
      JANUS_API_SECRET: ${API_SECRET}
      JANUS_ADMIN_SECRET: ${ADMIN_SECRET}

  backend:
    image: momentum-voip/backend:latest
    environment:
      REDIS_URL: redis://redis:6379
      JWT_SECRET: ${JWT_SECRET}

  redis:
    image: redis:7-alpine

  nginx:
    image: nginx:alpine
    # Terminates TLS, proxies WebSocket to Janus
```

### 3.2 Backend Service (Node.js/TypeScript)

Implement the fraud prevention backend documented in `SETUP-GUIDE.md` as a real, runnable service:

- `POST /api/auth/login` — User authentication
- `POST /api/call/authorize` — Issue short-lived call token (JWT, 30s TTL)
- `POST /api/call/initiate` — Consume token, initiate call via Janus API
- `POST /api/call/hangup` — Terminate call
- `GET  /api/wallet/balance` — Get user balance
- `POST /api/wallet/topup` — Add credit (Stripe integration)
- `GET  /api/calls/history` — CDR / call history
- `GET  /api/admin/stats` — Real-time stats (active calls, sessions)

Tech stack: Node.js 22, TypeScript, Express, Prisma (PostgreSQL), Redis (ioredis), Stripe SDK.

### 3.3 TypeScript SDK

A typed client SDK that wraps the Janus WebSocket API and the backend REST API:

```typescript
import { VoipClient } from "@momentum-voip/sdk";

const client = new VoipClient({ server: "wss://your-server.com" });
await client.connect();

const call = await client.call("+14155552671");
call.on("connected", () => console.log("Call answered"));
call.on("ended", (reason) => console.log("Call ended:", reason));
await call.hangup();
```

Features:
- Full TypeScript types for all Janus plugin APIs
- Auto-reconnect with exponential backoff
- Event emitter pattern
- React hooks package (`@momentum-voip/react`)
- Vue 3 composables package (`@momentum-voip/vue`)

### 3.4 Automated Test Suite

- Integration tests against a real Janus instance (Docker-based)
- SIP call flow tests using a test SIP server (Kamailio or Asterisk in Docker)
- Load tests (k6) for concurrent call capacity
- GitHub Actions CI pipeline


---

## 4. Phase 2 — Production-Grade Platform (Q4 2026)

**Goal:** Everything needed to run this in production for real users — monitoring, billing, multi-tenancy, and a polished admin panel.

### 4.1 Admin Dashboard (Web UI)

A full-featured operator dashboard built with **React + Tailwind CSS + shadcn/ui**.

**Pages:**

| Page | Features |
|------|----------|
| **Overview** | Live call count, active sessions, server health, RTP port usage, fraud alerts |
| **Active Calls** | Real-time table of all calls: user, destination, duration, codec, packet loss, jitter |
| **Call History (CDR)** | Searchable/filterable call detail records, export to CSV |
| **Users** | Create/edit/suspend users, view balance, call history per user |
| **Billing** | Top-up wallet, view transactions, rate configuration per destination |
| **Fraud Monitor** | Anomaly alerts, blocked calls, IRSF risk score per user |
| **Destinations** | Manage country whitelist/blacklist, per-prefix rates |
| **Janus Monitor** | Raw Janus Admin API view: sessions, handles, ICE state, media stats |
| **Configuration** | Edit Janus config, SIP plugin settings, transport settings via UI |
| **Logs** | Structured log viewer with filtering (JSON log from `janus_jsonlog`) |

**Design principles:**
- Dark mode by default (operators work in NOC environments)
- Real-time updates via WebSocket (no page refresh needed)
- Mobile-responsive for on-call monitoring
- Keyboard shortcuts for power users

### 4.2 Call Detail Records (CDR) System

Every call generates a structured CDR stored in PostgreSQL:

```typescript
interface CallDetailRecord {
  id: string;                    // UUID
  userId: string;
  direction: "inbound" | "outbound";
  callerNumber: string;
  destinationNumber: string;
  destinationCountry: string;
  startTime: Date;
  answerTime: Date | null;       // null if unanswered
  endTime: Date;
  durationSeconds: number;
  billableSeconds: number;
  ratePerMinuteCents: number;
  totalCostCents: number;
  terminationReason: string;     // "normal" | "no_answer" | "busy" | "fraud_block" | "balance_exhausted"
  janusSessionId: string;
  sipCallId: string;
  codec: string;                 // "opus" | "pcmu" | "pcma"
  mosScore: number | null;       // Mean Opinion Score if available
  packetLossPercent: number;
  jitterMs: number;
  recordingPath: string | null;
}
```

### 4.3 Billing & Payments

- **Stripe integration** for credit card top-ups
- **Pre-paid wallet** with configurable minimum balance alerts
- **Per-destination rate tables** (CSV import/export)
- **Invoice generation** (PDF) for business accounts
- **Webhook support** for balance low / balance exhausted events
- **Daily/monthly spend caps** per user and per account

### 4.4 Multi-Tenancy

Support multiple organizations on a single Janus deployment:

- **Tenant isolation** — Each tenant has their own SIP credentials, rate tables, user pool
- **Subdomain routing** — `tenant1.yourplatform.com`, `tenant2.yourplatform.com`
- **Per-tenant Janus API secrets** — Tenants cannot interfere with each other's sessions
- **Resource quotas** — Max concurrent calls, max users, max recording storage per tenant
- **White-label** — Custom logo, colors, domain per tenant

### 4.5 Monitoring & Alerting

**Metrics (Prometheus + Grafana):**
- Active WebRTC sessions
- Active SIP registrations
- Calls per minute (inbound/outbound)
- Call success rate (ASR — Answer Seizure Ratio)
- Average call duration (ACD)
- RTP packet loss per session
- ICE failure rate
- Janus CPU/memory/file descriptor usage
- Redis memory usage
- Backend API latency (p50/p95/p99)

**Alerts:**
- Janus process down
- ICE failure rate > 5%
- Fraud spike (calls/minute > threshold)
- Balance exhausted for high-volume user
- SIP registration failure for monitored accounts
- Disk space for recordings < 10GB

**Dashboards:**
- Pre-built Grafana dashboard JSON (importable)
- Janus-specific panels using event handler data


---

## 5. Phase 3 — Advanced Communication Features (Q1 2027)

**Goal:** Expand beyond basic SIP calling into a full unified communications platform, leveraging every Janus plugin capability.

### 5.1 Softphone Web App (Primary UI)

A production-quality browser softphone built with **React + TypeScript**, replacing the current Bootstrap demo pages. This is the flagship user-facing product.

Full design spec in [Section 8 — UI/UX Design System](#8-uiux-design-system).

Core features:
- Dial pad with E.164 number formatting
- Contact directory with search
- Call history with callback button
- Active call screen: mute, hold, transfer, DTMF, speaker
- Incoming call notification (browser push + in-app)
- Multiple concurrent calls with call switching
- Voicemail integration
- Presence indicators (available, busy, DND, away)
- Click-to-call from any web page (browser extension)

### 5.2 Video Conferencing (VideoRoom)

Leverage the existing `janus_videoroom` SFU plugin with a polished UI:

- **Room creation** — Instant rooms (no signup) and scheduled rooms
- **Up to 25 participants** with tile/spotlight/sidebar layouts
- **Screen sharing** — Full screen, window, or browser tab
- **Virtual backgrounds** — Blur, custom image (MediaPipe, already demoed)
- **Reactions** — Emoji reactions, raise hand queue
- **Chat** — In-meeting text chat (DataChannels via TextRoom plugin)
- **Recording** — Per-participant MJR → auto-converted to MP4 after meeting
- **Waiting room** — Host admits participants
- **Breakout rooms** — Split into sub-rooms (multiple VideoRoom instances)
- **Simulcast** — Automatic quality adaptation per viewer's bandwidth
- **E2EE** — End-to-end encryption via Insertable Streams (already supported in Janus)
- **Dial-in** — SIP plugin bridges PSTN callers into video rooms as audio-only participants

### 5.3 Audio Conferencing (AudioBridge)

Leverage `janus_audiobridge` for high-quality audio-only conferences:

- **Conference bridge** — Up to 100+ participants with server-side mixing
- **Spatial audio** — 3D positional audio for immersive meetings (already supported)
- **Noise suppression** — RNNoise per-participant denoising (already supported)
- **PSTN dial-in** — SIP plugin bridges phone callers into audio rooms
- **Recording** — Per-participant and mixed room recording
- **Moderator controls** — Mute all, kick, lock room, raise hand queue
- **Music on hold** — Play audio file when participant is on hold
- **IVR integration** — Automated attendant using Lua/Duktape scripting plugin

### 5.4 Live Streaming & Broadcast (Streaming Plugin)

Leverage `janus_streaming` for one-to-many broadcast:

- **WebRTC broadcast** — Stream from browser to unlimited viewers
- **RTSP re-streaming** — Pull IP camera feeds and broadcast to WebRTC viewers
- **FFmpeg/GStreamer ingest** — Accept RTP from external encoders
- **WHEP support** — Standard WebRTC-HTTP Egress Protocol for viewers
- **DVR / time-shift** — Buffer last N minutes for late joiners
- **Adaptive bitrate** — Simulcast-based quality switching per viewer
- **CDN integration** — Forward RTP to CDN edge nodes for global scale
- **Stream recording** — Auto-record all broadcasts to MJR → MP4

### 5.5 Call Transfer & Advanced SIP Features

Extend the SIP plugin capabilities exposed to the UI:

- **Blind transfer** (REFER) — Already supported in Sofia-SIP, expose in UI
- **Attended transfer** — Put call on hold, call third party, merge
- **Call parking** — Park a call, retrieve from any device
- **Call queuing** — ACD queue with position announcements (Lua scripting)
- **IVR / Auto-attendant** — DTMF-driven menu trees (Duktape scripting)
- **Voicemail** — Record to MJR, transcribe (Phase 4), email notification
- **Call recording consent** — Announce recording, store consent in CDR
- **SIP SUBSCRIBE/NOTIFY** — Presence and BLF (Busy Lamp Field) for shared lines

### 5.6 Mobile Apps

**React Native** apps sharing business logic with the web app:

- iOS (App Store) and Android (Play Store)
- Push notifications for incoming calls (APNs / FCM)
- Background call handling (CallKit on iOS, ConnectionService on Android)
- Same softphone UI as web, adapted for touch
- Biometric authentication
- Offline contact directory


---

## 6. Phase 4 — AI & Intelligence Layer (Q2 2027)

**Goal:** Add AI-powered features that make the platform genuinely smarter — for users, operators, and fraud prevention.

### 6.1 Real-Time Transcription

- **Live captions** during calls and video meetings (displayed in UI)
- **Post-call transcripts** — Auto-generated from MJR recordings via Whisper or cloud STT
- **Speaker diarization** — Label each speaker's turns in the transcript
- **Searchable transcripts** — Full-text search across all call recordings
- **Export formats** — TXT, SRT, VTT, JSON with timestamps

Implementation: Janus event handler streams RTP audio → Whisper.cpp (self-hosted) or OpenAI Whisper API → WebSocket push to UI.

### 6.2 Real-Time Translation

- **Live translation** of captions into any language (powered by DeepL or LibreTranslate)
- **Per-participant language preference** — Each viewer sees captions in their language
- **Multilingual meeting support** — Host speaks English, participants read in French/Spanish/etc.

### 6.3 AI Noise Suppression (Enhanced)

Janus already includes RNNoise per-participant in AudioBridge. Extend this:

- **Krisp-style noise suppression** in the browser (WebAssembly ONNX model)
- Applied before sending audio to Janus — works for all plugins, not just AudioBridge
- Toggle in the softphone UI (on/off, with visual indicator)
- Echo cancellation enhancement beyond browser AEC

### 6.4 AI-Powered Fraud Detection

Replace the current heuristic-based anomaly detection with an ML model:

- **Behavioral baseline** — Learn each user's normal calling patterns (time, destinations, duration)
- **Anomaly scoring** — Real-time risk score per call attempt (0–100)
- **Auto-block** — Calls above threshold blocked instantly, flagged for review
- **Feedback loop** — Operator marks false positives/negatives to retrain model
- **IRSF pattern library** — Pre-trained on known IRSF attack patterns
- **Velocity checks** — Detect account takeover (sudden change in calling behavior)

Tech: Python FastAPI microservice, scikit-learn or PyTorch, feature store in Redis.

### 6.5 Call Summarization

- **Post-call AI summary** — 3–5 bullet point summary of what was discussed
- **Action items extraction** — Identify commitments and next steps
- **Sentiment analysis** — Caller satisfaction score per call
- **CRM integration** — Push summary + action items to Salesforce, HubSpot, etc.

### 6.6 Virtual Assistant / IVR with LLM

Replace static DTMF IVR trees with a conversational AI assistant:

- **Natural language IVR** — "Press 1 for sales" → "How can I help you today?"
- **Intent routing** — LLM classifies intent, routes to correct queue/agent
- **FAQ answering** — Answers common questions without human agent
- **Handoff** — Seamlessly transfers to human agent with context summary
- **Voice synthesis** — TTS using ElevenLabs or Coqui TTS (self-hosted)

Implementation: Janus Lua/Duktape plugin → audio stream → STT → LLM → TTS → inject audio back into call.


---

## 7. Phase 5 — Enterprise & Scale (Q3 2027)

**Goal:** Horizontal scaling, enterprise integrations, and SLA-grade reliability.

### 7.1 Horizontal Scaling & Clustering

Janus is stateful per-session, so scaling requires careful design:

- **Load balancer** — Nginx/HAProxy routes new sessions to least-loaded Janus node
- **Session affinity** — WebSocket connections stick to the same Janus instance
- **SFU cascading** — VideoRoom remote publishers (already supported in Janus) for cross-node video rooms
- **Shared Redis** — Session metadata, wallet, rate limits shared across all nodes
- **Shared PostgreSQL** — CDR, users, billing data
- **Shared NFS/S3** — Recordings accessible from all nodes
- **Health checks** — Automatic removal of unhealthy nodes from load balancer pool
- **Auto-scaling** — Kubernetes HPA based on active session count

Target: 10,000+ concurrent WebRTC sessions across a cluster.

### 7.2 High Availability

- **Active-active Janus cluster** — No single point of failure
- **Redis Sentinel / Redis Cluster** — HA for session state
- **PostgreSQL streaming replication** — Read replicas + automatic failover
- **Multi-region deployment** — Janus nodes in multiple geographic regions
- **ICE candidate optimization** — Route media to nearest Janus node (GeoDNS)
- **TURN server cluster** — Coturn with shared Redis for HA TURN

### 7.3 Enterprise Integrations

- **Microsoft Teams** — Direct Routing integration (Janus as SBC)
- **Salesforce CTI** — Click-to-call from Salesforce, screen pop on inbound
- **HubSpot** — Call logging, contact lookup
- **Slack** — Incoming call notifications, call summary posts
- **Zapier / Make (Integromat)** — Webhook triggers for call events
- **LDAP / Active Directory** — Enterprise user directory sync
- **SAML 2.0 / OIDC** — SSO with corporate identity providers (Okta, Azure AD)
- **Webhook API** — Configurable webhooks for all call events (call.started, call.ended, etc.)

### 7.4 SIP Trunk Management

- **Multi-trunk support** — Configure multiple SIP trunk providers (Twilio, Telnyx, VoIP.ms, etc.)
- **Trunk failover** — Automatic failover to secondary trunk on primary failure
- **Least-cost routing (LCR)** — Route calls via cheapest trunk per destination
- **DID management** — Manage inbound phone numbers, routing rules
- **Number porting** — Track porting requests and status
- **Trunk health monitoring** — Registration status, call success rate per trunk

### 7.5 Compliance & Security

- **GDPR compliance** — Data retention policies, right-to-erasure for recordings/CDRs
- **HIPAA mode** — Encrypted recordings at rest, audit logs, BAA support
- **PCI DSS** — Pause recording during payment card entry (DTMF detection)
- **SOC 2 Type II** — Audit logging for all admin actions
- **End-to-end encryption** — Insertable Streams E2EE for video rooms (already in Janus)
- **Recording encryption** — AES-256 encryption of MJR files at rest
- **TLS everywhere** — WSS, HTTPS, SIPS, SRTP enforced in production mode
- **Penetration testing** — Annual third-party security audit

### 7.6 Developer Platform

- **Public API** — REST API with OpenAPI 3.0 spec, Swagger UI
- **Webhooks** — Reliable delivery with retry, signature verification
- **SDKs** — TypeScript, Python, Go, PHP client libraries
- **Marketplace** — Community plugins (Lua/Duktape scripts, event handlers)
- **Sandbox environment** — Free tier for developers to test integrations
- **CLI tool** — `voip-cli calls list`, `voip-cli calls hangup <id>`, etc.


---

## 8. UI/UX Design System

This section defines the complete design language for all user-facing interfaces.

### 8.1 Design Principles

1. **Clarity over cleverness** — Every control is immediately obvious. No hidden gestures, no mystery icons.
2. **Speed** — The UI responds in under 100ms to every interaction. Calls connect in under 2 seconds.
3. **Calm technology** — The UI stays out of the way during a call. Information appears when needed, disappears when not.
4. **Accessible by default** — WCAG 2.1 AA compliance. Full keyboard navigation. Screen reader support.
5. **Progressive disclosure** — Simple for new users, powerful for experts. Advanced settings are one level deeper.

### 8.2 Design Tokens

```css
/* Color palette */
--color-primary:        #6366F1;   /* Indigo — brand, CTAs */
--color-primary-hover:  #4F46E5;
--color-success:        #10B981;   /* Emerald — connected, online */
--color-warning:        #F59E0B;   /* Amber — hold, warning */
--color-danger:         #EF4444;   /* Red — end call, error */
--color-surface:        #0F172A;   /* Slate 900 — main background (dark) */
--color-surface-2:      #1E293B;   /* Slate 800 — cards, panels */
--color-surface-3:      #334155;   /* Slate 700 — hover states */
--color-border:         #475569;   /* Slate 600 — dividers */
--color-text-primary:   #F8FAFC;   /* Slate 50 — primary text */
--color-text-secondary: #94A3B8;   /* Slate 400 — secondary text */
--color-text-muted:     #64748B;   /* Slate 500 — timestamps, labels */

/* Typography */
--font-sans:   "Inter", system-ui, sans-serif;
--font-mono:   "JetBrains Mono", monospace;
--text-xs:     0.75rem;   /* 12px — timestamps, badges */
--text-sm:     0.875rem;  /* 14px — secondary labels */
--text-base:   1rem;      /* 16px — body text */
--text-lg:     1.125rem;  /* 18px — section headers */
--text-xl:     1.25rem;   /* 20px — page titles */
--text-2xl:    1.5rem;    /* 24px — call timer */
--text-4xl:    2.25rem;   /* 36px — dial pad digits */

/* Spacing (8px grid) */
--space-1: 0.25rem;  /* 4px */
--space-2: 0.5rem;   /* 8px */
--space-3: 0.75rem;  /* 12px */
--space-4: 1rem;     /* 16px */
--space-6: 1.5rem;   /* 24px */
--space-8: 2rem;     /* 32px */

/* Radius */
--radius-sm:   0.375rem;  /* 6px — buttons, inputs */
--radius-md:   0.5rem;    /* 8px — cards */
--radius-lg:   0.75rem;   /* 12px — modals */
--radius-full: 9999px;    /* pills, avatars */

/* Shadows */
--shadow-sm:  0 1px 2px rgba(0,0,0,0.4);
--shadow-md:  0 4px 6px rgba(0,0,0,0.4);
--shadow-lg:  0 10px 15px rgba(0,0,0,0.5);
--shadow-glow-green: 0 0 20px rgba(16,185,129,0.3);   /* active call */
--shadow-glow-red:   0 0 20px rgba(239,68,68,0.3);    /* end call hover */
```

### 8.3 Softphone UI — Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  SIDEBAR (240px)          │  MAIN CONTENT AREA                  │
│                           │                                     │
│  ┌─────────────────────┐  │  ┌─────────────────────────────┐   │
│  │  ● Avatar  Alice    │  │  │                             │   │
│  │    alice@company    │  │  │   [Active Call Screen]      │   │
│  │    ● Available  ▾   │  │  │   or                        │   │
│  └─────────────────────┘  │  │   [Dial Pad]                │   │
│                           │  │   or                        │   │
│  ┌─────────────────────┐  │  │   [Call History]            │   │
│  │  🔍 Search contacts │  │  │   or                        │   │
│  └─────────────────────┘  │  │   [Contacts]                │   │
│                           │  │   or                        │   │
│  RECENT                   │  │   [Video Room]              │   │
│  ┌─────────────────────┐  │  │                             │   │
│  │ 📞 Bob Smith        │  │  └─────────────────────────────┘   │
│  │    +1 415 555 0100  │  │                                     │
│  │    2 min ago        │  │                                     │
│  ├─────────────────────┤  │                                     │
│  │ 📞 Sales Queue      │  │                                     │
│  │    Missed · 1h ago  │  │                                     │
│  └─────────────────────┘  │                                     │
│                           │                                     │
│  ─────────────────────    │                                     │
│  [📞] [📹] [⚙️] [❓]      │                                     │
└─────────────────────────────────────────────────────────────────┘
```


### 8.4 Dial Pad Screen

```
┌──────────────────────────────────────┐
│                                      │
│   ┌──────────────────────────────┐   │
│   │  +1 (415) 555-0100        ✕  │   │  ← Number input, auto-formats E.164
│   └──────────────────────────────┘   │
│                                      │
│   ┌────┐  ┌────┐  ┌────┐            │
│   │ 1  │  │ 2  │  │ 3  │            │
│   │    │  │ABC │  │DEF │            │
│   └────┘  └────┘  └────┘            │
│   ┌────┐  ┌────┐  ┌────┐            │
│   │ 4  │  │ 5  │  │ 6  │            │
│   │GHI │  │JKL │  │MNO │            │
│   └────┘  └────┘  └────┘            │
│   ┌────┐  ┌────┐  ┌────┐            │
│   │ 7  │  │ 8  │  │ 9  │            │
│   │PQRS│  │TUV │  │WXYZ│            │
│   └────┘  └────┘  └────┘            │
│   ┌────┐  ┌────┐  ┌────┐            │
│   │ *  │  │ 0  │  │ #  │            │
│   │    │  │ +  │  │    │            │
│   └────┘  └────┘  └────┘            │
│                                      │
│         ┌──────────────┐             │
│         │  ●  Call     │             │  ← Large green call button
│         └──────────────┘             │
│                                      │
│   Balance: $12.40  ·  ~62 min to US  │  ← Real-time balance display
└──────────────────────────────────────┘
```

**Interaction details:**
- Number input auto-formats as user types: `14155550100` → `+1 (415) 555-0100`
- Paste a number from clipboard → auto-detected and formatted
- Long-press `0` → inserts `+` (international prefix)
- Backspace key works on keyboard; tap `✕` clears one digit
- Balance shows estimated minutes remaining for the dialed country
- Call button pulses with a subtle glow animation on hover
- Keyboard shortcut: `Enter` to call, `Escape` to clear

### 8.5 Active Call Screen

```
┌──────────────────────────────────────┐
│                                      │
│         ┌──────────────────┐         │
│         │                  │         │
│         │   👤  Bob Smith  │         │  ← Contact avatar (or initials)
│         │  +1 415 555 0100 │         │
│         │                  │         │
│         └──────────────────┘         │
│                                      │
│              02:47                   │  ← Call timer (large, monospace)
│           ● Connected                │  ← Status with green pulse dot
│                                      │
│   ┌──────────────────────────────┐   │
│   │  ████████████░░░░░░░░░░░░░░  │   │  ← Audio level visualizer (bars)
│   └──────────────────────────────┘   │
│                                      │
│   ┌──────┐ ┌──────┐ ┌──────┐        │
│   │  🔇  │ │  ⏸   │ │  ⌨️  │        │  ← Mute | Hold | Keypad
│   │ Mute │ │ Hold │ │ DTMF │        │
│   └──────┘ └──────┘ └──────┘        │
│   ┌──────┐ ┌──────┐ ┌──────┐        │
│   │  🔊  │ │  ↗️  │ │  ⏺   │        │  ← Speaker | Transfer | Record
│   │Spkr  │ │Xfer  │ │ Rec  │        │
│   └──────┘ └──────┘ └──────┘        │
│                                      │
│         ┌──────────────┐             │
│         │  ●  End Call │             │  ← Large red end call button
│         └──────────────┘             │
│                                      │
│  Quality: Excellent  ·  Opus 48kHz   │  ← Call quality indicator
└──────────────────────────────────────┘
```

**Interaction details:**
- Timer counts up in `MM:SS`, switches to `HH:MM:SS` after 1 hour
- Audio level bars animate in real-time (Web Audio API analyser)
- Mute button turns red when active, shows "Unmute" label
- Hold button shows "Resume" when on hold; remote party hears hold music
- DTMF keypad slides up from bottom as an overlay
- Transfer opens a search-as-you-type contact picker
- Record button shows red dot + elapsed recording time when active
- Quality indicator: Excellent / Good / Fair / Poor based on packet loss + jitter
- Keyboard shortcut: `Space` to mute/unmute, `H` to hold/resume, `Escape` to end

### 8.6 Incoming Call Screen

```
┌──────────────────────────────────────┐
│                                      │
│  ┌────────────────────────────────┐  │
│  │                                │  │
│  │   📞  Incoming Call            │  │
│  │                                │  │
│  │   ┌──────────────────────┐     │  │
│  │   │   👤  Unknown        │     │  │
│  │   │  +44 20 7946 0958    │     │  │
│  │   │  🇬🇧 United Kingdom  │     │  │
│  │   └──────────────────────┘     │  │
│  │                                │  │
│  │   ┌──────────┐  ┌──────────┐   │  │
│  │   │  ✕ Decline│  │ ✓ Answer │   │  │
│  │   └──────────┘  └──────────┘   │  │
│  │                                │  │
│  │   [ Send to voicemail ]        │  │
│  │                                │  │
│  └────────────────────────────────┘  │
│                                      │
└──────────────────────────────────────┘
```

**Interaction details:**
- Slides in from top with a subtle bounce animation
- Phone rings with browser notification sound (respects OS Do Not Disturb)
- Browser push notification if tab is not focused
- Flag emoji auto-detected from country code
- Decline → sends 486 Busy Here
- Answer → transitions directly to Active Call Screen
- Send to voicemail → sends 302 redirect to voicemail number
- Auto-dismiss after 30 seconds (missed call logged in history)

### 8.7 Video Room UI

```
┌─────────────────────────────────────────────────────────────────┐
│  Meeting: Team Standup  ·  00:12:34  ·  5 participants    [⋮]   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                                                          │   │
│  │                  [Active Speaker Video]                  │   │
│  │                                                          │   │
│  │                    Alice Johnson                         │   │
│  │                    ● Speaking                            │   │
│  │                                                          │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐             │
│  │ Bob  │  │Carol │  │Dave  │  │Eve   │  │  +2  │             │  ← Thumbnail strip
│  │  🎤  │  │  🔇  │  │  🎤  │  │  🎤  │  │      │             │
│  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘             │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  [🎤 Mute] [📹 Video] [🖥 Share] [✋ Raise] [💬 Chat] [⋮ More] [🔴 Leave] │
└─────────────────────────────────────────────────────────────────┘
```

**Layout modes:**
- **Speaker** (default) — Active speaker large, others in strip
- **Gallery** — Equal-size tiles, up to 9 visible (3×3 grid)
- **Sidebar** — Speaker large, participants in right sidebar
- **Presentation** — Screen share fills 80%, participants in strip

**Accessibility:**
- Keyboard navigation between participant tiles
- Screen reader announces when someone joins/leaves/speaks
- High contrast mode
- Closed captions toggle (Phase 4)


### 8.8 Admin Dashboard UI

**Color scheme:** Dark mode with indigo accents. Data-dense but not cluttered.

**Overview page:**

```
┌─────────────────────────────────────────────────────────────────┐
│  momentum VOIP  ·  Admin Dashboard                    👤 Admin  │
├──────────┬──────────────────────────────────────────────────────┤
│          │                                                       │
│ Overview │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐  │
│ Calls    │  │  Active  │ │  Today's │ │  ASR     │ │ Fraud  │  │
│ Users    │  │  Calls   │ │  Revenue │ │          │ │ Alerts │  │
│ Billing  │  │   247    │ │  $1,842  │ │  94.2%   │ │   3    │  │
│ Fraud    │  │  ▲ +12%  │ │  ▲ +8%  │ │  ▼ -1%  │ │  🔴    │  │
│ Trunks   │  └──────────┘ └──────────┘ └──────────┘ └────────┘  │
│ Config   │                                                       │
│ Logs     │  ┌─────────────────────────────────────────────────┐ │
│          │  │  Calls per hour (last 24h)                      │ │
│          │  │  ▁▂▃▄▅▆▇█▇▆▅▄▃▂▁▂▃▄▅▆▇█▇▆  (sparkline chart)  │ │
│          │  └─────────────────────────────────────────────────┘ │
│          │                                                       │
│          │  ┌──────────────────────┐ ┌──────────────────────┐   │
│          │  │  Active Calls        │ │  Recent Fraud Alerts  │   │
│          │  │  user123 → +1415...  │ │  🔴 user456: IRSF    │   │
│          │  │  user456 → +44207... │ │     +234 Nigeria     │   │
│          │  │  user789 → +49302... │ │     Blocked 2m ago   │   │
│          │  │  [View all 247]      │ │  [View all alerts]   │   │
│          │  └──────────────────────┘ └──────────────────────┘   │
│          │                                                       │
└──────────┴───────────────────────────────────────────────────────┘
```

### 8.9 Micro-interactions & Animations

These small details make the UI feel alive and responsive:

| Interaction | Animation |
|-------------|-----------|
| Call button press | Scale down 0.95 → scale up 1.0 (80ms) |
| Incoming call | Slide in from top + ring pulse on avatar |
| Call connected | Green glow fades in on call screen |
| Mute toggle | Icon cross-fade + subtle haptic (mobile) |
| Hold | Screen desaturates slightly, "On Hold" badge slides in |
| Call ended | Screen fades to summary card (duration, cost) |
| Participant joins video room | Tile slides in from edge |
| Speaking indicator | Subtle green border pulse on active speaker tile |
| Balance warning | Amber toast slides in from bottom-right |
| Network quality drop | Quality badge animates from green → yellow → red |
| Page transitions | Fade + slight upward slide (150ms ease-out) |

All animations respect `prefers-reduced-motion` — users who need it get instant transitions.

### 8.10 Component Library

Built on **shadcn/ui** (Radix UI primitives + Tailwind CSS), extended with VoIP-specific components:

| Component | Description |
|-----------|-------------|
| `<DialPad />` | Full numeric keypad with E.164 formatting |
| `<CallTimer />` | Animated MM:SS / HH:MM:SS counter |
| `<AudioLevelBar />` | Real-time audio level visualizer |
| `<CallQualityBadge />` | Excellent/Good/Fair/Poor with color coding |
| `<ContactAvatar />` | Avatar with presence indicator dot |
| `<PresenceIndicator />` | Available/Busy/DND/Away dot with tooltip |
| `<IncomingCallToast />` | Slide-in notification with accept/decline |
| `<ActiveCallCard />` | Compact call card for multi-call management |
| `<ParticipantTile />` | Video room participant tile with controls |
| `<BalanceDisplay />` | Wallet balance with estimated minutes |
| `<FraudAlertBanner />` | Dismissible fraud warning banner |
| `<CdrTable />` | Sortable/filterable call history table |


---

## 9. Feature Priority Matrix

| Feature | Impact | Effort | Phase | Priority |
|---------|--------|--------|-------|----------|
| Docker one-command setup | 🔴 Critical | Low | 1 | P0 |
| Backend service (JWT + wallet) | 🔴 Critical | Medium | 1 | P0 |
| Softphone web UI (dial pad + active call) | 🔴 Critical | High | 3 | P0 |
| Admin dashboard overview | 🔴 Critical | Medium | 2 | P0 |
| CDR system | 🔴 Critical | Medium | 2 | P0 |
| TypeScript SDK | High | Medium | 1 | P1 |
| Stripe billing integration | High | Medium | 2 | P1 |
| Prometheus + Grafana monitoring | High | Low | 2 | P1 |
| Video conferencing UI | High | High | 3 | P1 |
| Mobile apps (React Native) | High | Very High | 3 | P1 |
| Multi-tenancy | High | High | 2 | P1 |
| Real-time transcription | Medium | High | 4 | P2 |
| AI noise suppression (browser WASM) | Medium | Medium | 4 | P2 |
| AI fraud detection (ML model) | Medium | High | 4 | P2 |
| Audio conferencing UI | Medium | Medium | 3 | P2 |
| Live streaming UI | Medium | Medium | 3 | P2 |
| Call transfer / attended transfer UI | Medium | Medium | 3 | P2 |
| Horizontal scaling / clustering | High | Very High | 5 | P2 |
| Microsoft Teams Direct Routing | Medium | High | 5 | P3 |
| CRM integrations (Salesforce, HubSpot) | Medium | Medium | 5 | P3 |
| LLM-powered IVR | Low | High | 4 | P3 |
| Call summarization | Low | Medium | 4 | P3 |
| HIPAA / SOC 2 compliance | Medium | High | 5 | P3 |
| CLI tool | Low | Low | 5 | P3 |

---

## 10. Technical Debt & Infrastructure

Items that need addressing regardless of new features:

### 10.1 Immediate (before Phase 1)

- [ ] Add `meson`, `ninja-build`, `cmake` to the apt install block (done in SETUP-GUIDE.md)
- [ ] Add `libnice/`, `libsrtp-2.2.0/`, `libwebsockets/`, `v2.2.0.tar.gz` to `.gitignore` (done)
- [ ] Verify libnice builds and installs correctly (`pkg-config --modversion nice`)
- [ ] Confirm full Janus build succeeds end-to-end on a clean Ubuntu 22.04 machine
- [ ] Add `ldconfig` call after installing libsrtp and libwebsockets to update linker cache

### 10.2 Short-term (Phase 1)

- [ ] Replace `bower.json` (deprecated) with npm/CDN references
- [ ] Update HTML demos from CDN-loaded jQuery/Bootstrap to a proper build pipeline
- [ ] Add `package.json` at the repo root for the backend service
- [ ] Set up ESLint + Prettier for TypeScript code
- [ ] Add `.env.example` file documenting all required environment variables
- [ ] Write `CONTRIBUTING.md` with development setup instructions

### 10.3 Medium-term (Phase 2)

- [ ] Migrate Janus demo HTML from Bootstrap 5 CDN to self-hosted assets (offline support)
- [ ] Add Content Security Policy headers to all web assets
- [ ] Implement proper secret rotation for `api_secret` and `admin_secret`
- [ ] Add database migrations system (Prisma Migrate)
- [ ] Set up log aggregation (Loki or ELK) consuming `janus_jsonlog` output
- [ ] Document all Janus event handler payloads for the backend to consume

---

## Appendix A — Technology Stack Summary

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| WebRTC server | Janus Gateway (C) | Battle-tested, plugin architecture, all WebRTC features |
| SIP stack | Sofia-SIP (via Janus plugin) | Mature, RFC-compliant, handles all SIP edge cases |
| Browser SIP | SIP.js 0.21.2 (TypeScript) | Modern, well-maintained, full SIP stack in browser |
| Backend API | Node.js 22 + TypeScript + Express | Type safety, large ecosystem, async I/O for WebSocket |
| Database | PostgreSQL 16 | ACID, JSON support, excellent for CDR queries |
| Cache / session | Redis 7 | Sub-millisecond latency for wallet/rate-limit operations |
| Frontend | React 19 + TypeScript + Vite | Component model, TypeScript, fast HMR |
| Styling | Tailwind CSS + shadcn/ui | Utility-first, accessible primitives, dark mode |
| Mobile | React Native + Expo | Code sharing with web, native call handling |
| Payments | Stripe | Industry standard, PCI DSS compliant |
| Monitoring | Prometheus + Grafana | Open source, Janus metrics via event handler |
| Logging | JSON log (Janus) + Loki | Structured, queryable, Grafana integration |
| Container | Docker + docker-compose | Reproducible builds, easy deployment |
| Orchestration | Kubernetes (Phase 5) | Horizontal scaling, auto-healing |
| CI/CD | GitHub Actions | Already in repo, extend for full pipeline |

---

## Appendix B — Janus Plugin Capability Map

| Plugin | Current Capability | Roadmap Extension |
|--------|-------------------|-------------------|
| `janus_sip` | WebRTC↔SIP bridge, DTMF, REFER, SUBSCRIBE | Attended transfer UI, call parking, IVR via Lua |
| `janus_videoroom` | SFU, simulcast, SVC, E2EE, cascading | Polished UI, breakout rooms, PSTN dial-in |
| `janus_audiobridge` | MCU mixing, spatial audio, RNNoise | Conference UI, PSTN dial-in, IVR |
| `janus_streaming` | RTP/RTSP/file broadcast | Broadcast UI, DVR, CDN forwarding |
| `janus_recordplay` | MJR record + playback | Auto-convert to MP4, transcription (Phase 4) |
| `janus_textroom` | DataChannel text rooms | In-meeting chat UI for video rooms |
| `janus_videocall` | 1:1 video via Janus | Integrate into softphone as video call mode |
| `janus_lua` | Custom plugin logic in Lua | IVR, call queuing, voicemail |
| `janus_duktape` | Custom plugin logic in JS | Same as Lua, JS-familiar developers |
| `janus_nosip` | Raw SDP/RTP interop | Legacy PBX integration without SIP |

---

*Last updated: May 30, 2026 · Branch: momentum-VOIP-v1.0*

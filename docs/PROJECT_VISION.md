# Project Vision

**Version:** 3.3.0
**Last Updated:** 2026-07-19

## The Vision

StreamFlow exists to make enterprise-grade live streaming operationally simple. Broadcasting to multiple platforms simultaneously, monitoring stream health in real time, and recovering automatically from failures should require no specialized tooling or deep FFmpeg knowledge — just a web browser and a properly configured server.

---

## Core Philosophy

### 1. The Operator is the User
StreamFlow is built for the person watching the dashboard at 2am during a critical broadcast. Every design decision should reduce cognitive load for that person: clear health status, one-click log access, automatic reconnects, and meaningful alerts.

### 2. Simplicity of Deployment
No Kubernetes, no microservices, no container orchestration required. A single Go binary + PostgreSQL + Nginx is a full deployment. Anyone who can manage a Linux server can run StreamFlow.

### 3. FFmpeg is the Engine
We do not re-implement video encoding. FFmpeg is the industry standard; we orchestrate it intelligently. Our value is in the workflow, the monitoring, the scheduling, and the multi-destination routing — not in the codec logic.

### 4. Operational Visibility First
Before adding features, ensure operators can see what is happening. Real-time health metrics, FFmpeg log streaming, and a dedicated health dashboard are not nice-to-haves — they are table stakes for production use.

---

## v3.3.0 Milestone: Operational Completeness

With v3.3.0, StreamFlow reaches a state of operational completeness for its core use case:

- **Start:** Schedule a stream to begin automatically.
- **Broadcast:** Simulcast to multiple destinations, each with its own encode profile.
- **Monitor:** Watch real-time FPS, bitrate, dropped frames, and processing speed from the dashboard.
- **Debug:** View raw FFmpeg output directly in the browser without SSH.
- **Recover:** Automatic reconnect handles transient platform drops.

---

## Looking Forward

The next phase of StreamFlow's evolution focuses on:

1. **Alerting:** Proactive health webhooks so operators are notified before problems become outages.
2. **Hardware acceleration:** GPU transcoding support (NVENC, VAAPI) for high-density deployments.
3. **Richer analytics:** Time-series charting of viewer count, bitrate, and drop frame trends.
4. **Mobile reach:** Native mobile app or enhanced PWA for on-the-go stream management.

See `ROADMAP.md` for the full feature backlog and `IDEAS.md` for exploratory concepts.

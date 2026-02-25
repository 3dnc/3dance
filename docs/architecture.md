# 3Dance System Architecture

**What we offer is software:** ingestion (camera capture and data pipeline), data control (who sees what, retention, sharing, policies), and an **admin UI** to configure and operate it. Hardware (cameras, network) is supplied by the studio or third parties.

---

## 1. Product Scope (What We Ship)

| We offer | Description |
|----------|-------------|
| **Ingestion** | Capture from multiple camera sources, time sync, frame alignment, buffering, and delivery into storage or downstream processing. |
| **Data control** | Access control, retention rules, sharing and export policies, audit, and compliance—all configurable per studio. |
| **Admin UI** | Single place to configure ingestion (sources, sync), set data policies, manage users and roles, and monitor status. |

Hardware (cameras, NTP, local compute) is **customer-supplied**. Downstream processing (e.g. pose, 3D, analytics) may be part of the ecosystem but is not the core product; our software owns the pipeline and who can access the data.

---

## 2. Technology and Language Stance

The system is **AI-first**: ML (pose, 3D, analytics) is central. There are **no code-language constraints**—use whatever fits each component best.

- **Rust / C** are in scope wherever they add value: ingestion hot path (multi-stream capture, sync, encode), real-time pipelines, FFI to camera or encoder SDKs, low-latency or high-throughput services. Use them when performance, predictability, or integration with C libraries matters.
- **Python** remains the default for model inference and tooling (PyTorch, ONNX, pose/vision libs), orchestration, APIs, and admin/automation. Interop via subprocess, HTTP/gRPC, or FFI (e.g. PyO3, ctypes) is expected.
- **Other languages** (e.g. Go, TypeScript for UI) are fine for services or frontends where they are a better fit.

Choose by component: optimize the pipeline and AI stack per layer rather than standardizing on one language.

---

## 3. Logical Layers

```
┌─────────────────────────────────────────────────────────────────────────┐
│  CUSTOMER / EXTERNAL                                                     │
│  Cameras · NTP · Optional on-prem compute (customer-supplied)            │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  OUR SOFTWARE                                                            │
│  Ingestion · Data control · Admin UI                                    │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  DOWNSTREAM (optional)                                                   │
│  Processing · Analytics · Replay / export (us or third party)             │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Components

### 4.1 Customer environment (out of our scope)

- **Cameras**: Studio-supplied; typically three fixed wall cameras with overlapping FOV, consistent frame rate, and some form of time sync (NTP, PTP, or software).
- **Network / compute**: Studio network and optional on-prem box for running our ingestion or relay. We define requirements; we don’t supply the hardware.

### 4.2 Ingestion (our software)

- **Source configuration**: Define camera endpoints (RTSP, USB, SDK, etc.), credentials, and which streams to use. Configured via admin UI.
- **Capture**: Connect to sources, read frames, attach timestamps; handle reconnection and backpressure.
- **Sync**: Time-align frames across sources (NTP or software alignment) so downstream gets coherent multi-view frames.
- **Buffering / pipeline**: Frame buffers, optional encode or thumbnail, and output to our storage or to a defined downstream sink (e.g. object store, processing service).
- **Session boundary**: Optional concept of “session” (e.g. class start/stop) so data is grouped and retention can apply per session.

Ingestion is a strong candidate for **Rust or C** (or a Rust/C core with a thin API) when we need low-latency capture, hardware encode, or tight sync across many streams.

### 4.3 Data control (our software)

- **Access control**: Identity and roles (e.g. studio admin, instructor, viewer); which users or roles can view live, replay, or export; per-studio isolation.
- **Retention**: Configurable rules (e.g. keep raw 30 days, then delete or archive); automatic enforcement.
- **Sharing and export**: Policies for who can create share links or exports; time-limited links; revocable access; audit log of shares/exports.
- **Compliance and audit**: Log access to sensitive data, retention actions, and policy changes; support for minors and regional requirements.
- **Storage and egress**: Where data lives (our cloud or customer-designated); who can pull data out and under what conditions.

All of the above are configurable and operable via the **admin UI** (and backed by an API for automation).

### 4.4 Admin UI (our software)

- **Ingestion**: Add/edit/remove sources; set sync and pipeline options; view status (connected, errors, throughput).
- **Data control**: Configure retention, sharing and export policies, role permissions; view audit and compliance logs.
- **Users and org**: Manage studios, users, roles; optional SSO or directory integration.
- **Operational view**: Health of ingestion, storage usage, retention runs, and any alerts.

This is the primary interface for “running” the product; other UIs (e.g. instructor view, replay, family portal) can sit on top of the same data control and APIs.

### 4.5 Storage (our software or customer choice)

- **Where**: We may operate storage (e.g. per-tenant buckets) or support customer-owned storage; data control policies apply either way.
- **What**: Ingested streams (raw or encoded), session metadata, calibration if applicable; downstream results if we or a partner write them back.
- **Lifecycle**: Retention and deletion driven by data control rules; no access without passing access control.

### 4.6 Downstream (optional, not core product)

- **Processing**: Calibration, pose, triangulation, 3D, analytics—may be our own services or a partner; they consume data we ingest and are subject to our data control (e.g. who can trigger jobs, where results go). **AI-first**: typically Python (PyTorch, ONNX, OpenCV, etc.) or inference runtimes (Triton, TensorRT); Rust/C for latency-critical or embedded inference if needed.
- **Consumers**: Replay UIs, export tools, family-facing clips—all consume data through our APIs and respect access and retention.

---

## 5. Data Flow (simplified)

1. **Setup**: Admin configures camera sources and data-control policies in the admin UI.
2. **Ingestion**: Our software connects to sources, syncs frames, and writes to storage (or a downstream sink); session boundaries optional.
3. **Access**: Any read (live view, replay, export) goes through our APIs and enforces data control (roles, retention, sharing rules).
4. **Retention**: Background jobs or triggers delete or archive data per policy; audit logged.
5. **Downstream**: If processing or analytics exist, they consume from our pipeline/storage subject to the same data control.

---

## 6. Deployment

- **Our software** can run in our cloud (SaaS), in the customer’s environment (on-prem or their VPC), or hybrid (ingestion on-prem, control and admin in our cloud). Data control and admin UI are always part of the product.
- **Storage** can be ours or customer-designated; retention and access still enforced by our layer.

---

## 7. Security & Privacy

- **Minors**: Studios have minors; treat all ingested data as sensitive. Data control (access, retention, sharing) and audit are core.
- **TLS** for admin UI, API, and any streams we proxy or expose.
- **Encryption at rest** for stored data; keys and access under data control.
- **Retention and sharing** as in section 4.3; time-limited and revocable links; no public indexing.

---

## 8. Out of Scope (for this doc)

- Camera hardware specs and procurement.
- Detailed design of downstream processing (pose, 3D, analytics).
- Billing and subscriptions (separate product/backend doc).

---

## 9. Possible Next Steps

1. **Admin UI**: Wireframes and flows for ingestion config, data-control policies, and operational view.
2. **API**: Contract for source config, session lifecycle, access and retention policies, and audit.
3. **Ingestion**: Supported source types (RTSP, etc.), sync strategy, and output format/sink.
4. **Data control**: Schema for roles, retention rules, and share/export policies; enforcement points in API and storage layer.

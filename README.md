# 2Care.ai Assignment

# Real-Time Multilingual Voice AI Agent

## Clinical Appointment Booking System

---

## 1. Objective

This project implements a real-time multilingual voice AI agent for clinical appointment booking.

The agent can:

- Understand patient speech through browser speech recognition
- Detect English, Hindi, and Tamil
- Interpret appointment-related requests
- Book appointments
- Reschedule appointments
- Cancel appointments
- Check doctor availability
- Prevent invalid bookings
- Suggest alternate slots when conflicts occur
- Speak responses back to the patient
- Maintain session and persistent patient memory
- Support outbound reminder/follow-up campaign flows
- Measure and log latency for every agent turn

Target latency:

**Less than 450 ms from speech end to first audio response.**

In this demo, the browser handles STT/TTS and the backend measures the complete agent decision/tool orchestration latency.

---

## 2. Tech Stack

| Area | Technology |
| --- | --- |
| Backend | Python, FastAPI |
| Frontend | TypeScript, HTML, CSS |
| Voice Input | Browser Web Speech API |
| Voice Output | Browser Speech Synthesis API |
| Agent Orchestration | Python planner + explicit tool registry |
| Memory | JSON store demo, Redis-ready service boundary |
| Scheduling Data | JSON appointment store |
| Tests | Pytest |
| Docs | Markdown, Mermaid, SVG diagram |

---

## 3. System Architecture

The system follows a real-time conversational pipeline.

```text
Patient Voice
     ↓
Browser Speech-to-Text
     ↓
FastAPI /api/turn
     ↓
Language Detection
     ↓
Appointment Agent
     ↓
Tool Orchestration
     ↓
Scheduling / Memory / Campaign Services
     ↓
Localized Text Response
     ↓
Browser Text-to-Speech
     ↓
Audio Response
```

Architecture files:

- [Architecture SVG](docs/architecture-diagram.svg)
- [Mermaid Source](docs/architecture-diagram.mmd)

---

## 4. Project Structure

```text
.
├── backend
│   ├── app
│   │   ├── agent
│   │   │   └── orchestrator.py
│   │   ├── services
│   │   │   ├── campaigns.py
│   │   │   ├── json_store.py
│   │   │   ├── language.py
│   │   │   ├── memory.py
│   │   │   └── scheduler.py
│   │   ├── tools
│   │   │   └── registry.py
│   │   ├── main.py
│   │   └── models.py
│   ├── data
│   ├── tests
│   │   └── test_agent.py
│   └── requirements.txt
│
├── frontend
│   ├── src
│   │   ├── app.ts
│   │   └── app.js
│   ├── index.html
│   ├── styles.css
│   └── package.json
│
├── docs
│   ├── architecture-diagram.mmd
│   ├── architecture-diagram.svg
│   └── loom-walkthrough.md
│
├── pyproject.toml
└── README.md
```

---

## 5. Setup Instructions

Run these commands from the project root:

```powershell
cd "D:\2care.ai Assignment\New folder"
python -m pip install -r backend\requirements.txt
python -m uvicorn backend.app.main:app --host 127.0.0.1 --port 8000
```

Open:

```text
http://127.0.0.1:8000
```

If port `8000` is already busy:

```powershell
python -m uvicorn backend.app.main:app --host 127.0.0.1 --port 8001
```

Open:

```text
http://127.0.0.1:8001
```

---

## 6. How To Use The Demo

1. Open the web UI.
2. Keep `Patient ID` as `p001` or enter a new patient ID.
3. Click a demo prompt or type a request.
4. Press `Send`.
5. The agent response appears in the transcript and is spoken aloud by the browser.
6. The right-side trace panel shows language detection, intent detection, tool calls, and latency.

Example prompts:

```text
Book earliest cardiology appointment
Reschedule my appointment to second slot
Cancel my appointment
मुझे दिल के डॉक्टर की appointment चाहिए
Vanakkam, skin doctor appointment book pannunga
```

Voice input:

- Click the microphone button.
- Browser speech recognition works best in Chrome or Edge.
- If browser STT is unavailable, use typed input.

---

## 7. Core Features

### 7.1 Appointment Booking

The agent can book the earliest valid slot for a doctor.

Example:

```text
Book earliest cardiology appointment
```

Expected response:

```text
Done. I booked Dr. Meera Iyer for 22 May, 10:00 AM.
```

### 7.2 Rescheduling

The agent cancels the active appointment and books a new valid slot.

Example:

```text
Reschedule my appointment to second slot
```

### 7.3 Cancellation

The agent cancels the patient's next active appointment.

Example:

```text
Cancel my appointment
```

### 7.4 Conflict Detection

The scheduler rejects:

- Double bookings
- Past-time bookings
- Slots that do not belong to the doctor
- Invalid or unavailable doctors

When a conflict occurs, the agent offers alternate valid slots.

---

## 8. Multilingual Support

Supported languages:

| Language | Code | Voice Hint |
| --- | --- | --- |
| English | `en` | `en-IN` |
| Hindi | `hi` | `hi-IN` |
| Tamil | `ta` | `ta-IN` |

Language detection uses:

- Unicode script detection for Devanagari and Tamil
- Lightweight phrase hints for romanized Hindi/Tamil
- Patient/session language fallback

Examples:

```text
मुझे दिल के डॉक्टर की appointment चाहिए
```

```text
Vanakkam, skin doctor appointment book pannunga
```

The detected language is saved to patient memory, so returning patients continue in their preferred language.

---

## 9. AI Agent And Tool Orchestration

Agent file:

```text
backend/app/agent/orchestrator.py
```

The agent does not directly mutate appointments. It plans the turn and calls tools through `ToolRegistry`.

Available tools:

| Tool | Purpose |
| --- | --- |
| `find_doctor` | Finds a doctor by specialty, preference, and language |
| `next_available` | Returns valid future free slots |
| `book_slot` | Books a slot and rejects conflicts |
| `cancel_latest` | Cancels the patient's next active booking |

Each response includes a visible trace:

```json
[
  {"name": "language.detected"},
  {"name": "intent.detected"},
  {"name": "tool.find_doctor"},
  {"name": "tool.next_available"},
  {"name": "tool.book_slot"},
  {"name": "latency.total"}
]
```

This makes reasoning and tool orchestration demonstrable in the UI.

---

## 10. Memory Design

Memory is stored in:

```text
backend/data/memory.json
```

### 10.1 Session Memory

Session memory stores active conversation state.

Example:

```json
{
  "session_id": "sess-1234",
  "patient_id": "p001",
  "language": "hi",
  "current_intent": "book",
  "pending_confirmation": null,
  "transcript": []
}
```

Used for:

- Current language
- Current intent
- Follow-up context
- Conversation transcript

### 10.2 Persistent Patient Memory

Persistent memory stores cross-session information.

Example:

```json
{
  "patient_id": "p001",
  "name": "Asha",
  "preferred_language": "hi",
  "preferred_doctor_id": "cardio-1",
  "last_intent": "book",
  "notes": ["Prefers evening appointments"],
  "appointments": []
}
```

Used for:

- Returning patient language preference
- Preferred doctor
- Prior interaction context
- Future personalization

Production upgrade:

- Redis for session memory with TTL
- PostgreSQL for durable patient and appointment records

---

## 11. Scheduling And Database Design

Scheduling data is stored in:

```text
backend/data/schedule.json
```

Doctor schema:

```json
{
  "doctor_id": "cardio-1",
  "name": "Dr. Meera Iyer",
  "specialty": "cardiology",
  "languages": ["en", "hi", "ta"],
  "slots": ["2026-05-22T10:00:00+05:30"]
}
```

Appointment schema:

```json
{
  "appointment_id": "apt-123",
  "patient_id": "p001",
  "doctor_id": "cardio-1",
  "doctor_name": "Dr. Meera Iyer",
  "specialty": "cardiology",
  "starts_at": "2026-05-22T10:00:00+05:30",
  "status": "booked"
}
```

Validation rules:

- Slot must exist in the doctor's schedule
- Slot must be in the future
- Slot must not already be booked
- Doctor must support the detected language when possible

---

## 12. Outbound Campaign Mode

Campaign service file:

```text
backend/app/services/campaigns.py
```

Campaign data is stored in:

```text
backend/data/campaigns.json
```

Create a campaign:

```http
POST /api/campaigns?reason=appointment_reminder
["p001"]
```

In the UI:

1. Change `Channel` to `Outbound campaign`.
2. Send a message such as:

```text
Can we move it to second slot?
```

The agent uses the same booking/rescheduling/cancellation tools and logs the campaign interaction.

---

## 13. Latency Measurement

Latency is measured inside:

```text
backend/app/main.py
backend/app/agent/orchestrator.py
```

Every turn records:

- Session ID
- Patient ID
- Channel
- Language
- Intent
- Latency in milliseconds
- Whether latency was under 450 ms

Latency log:

```text
backend/data/latency.csv
```

Example row:

```csv
session_id,patient_id,channel,language,intent,latency_ms,under_450ms
sess-abc,p001,inbound,en,book,8.4,True
```

Expected local backend latency is usually under 20 ms because:

- STT is handled by browser
- TTS is handled by browser
- Memory and scheduling are local JSON operations
- Tool orchestration is lightweight

Production latency budget:

| Stage | Target |
| --- | --- |
| Speech-end detection | 40-80 ms |
| Streaming STT finalization | 80-140 ms |
| Agent planning and tool calls | 50-120 ms |
| First TTS audio chunk | 120-180 ms |
| Network and serialization | 20-60 ms |
| Total | < 450 ms |

---

## 14. Error Handling

Handled cases:

| Scenario | Behavior |
| --- | --- |
| Unknown request | Agent asks whether to book, reschedule, or cancel |
| No appointment to cancel | Agent responds gracefully |
| Slot conflict | Agent suggests alternatives |
| Past slot | Booking is rejected |
| Unsupported browser STT | Typed input still works |
| Existing patient language | Agent continues using stored language |

---

## 15. Testing

Run:

```powershell
python -m pytest
```

Tests cover:

| Test | Expected Result |
| --- | --- |
| Booking appointment | Appointment confirmed and `book_slot` tool is called |
| Hindi request | Language detected as Hindi |
| Cancel without appointment | Graceful recovery message |

---

## 16. Evaluation Mapping

| Evaluation Area | Implementation |
| --- | --- |
| Real-time voice architecture and latency | Browser STT/TTS, FastAPI turn endpoint, latency CSV |
| Agent reasoning and tool orchestration | `AppointmentAgent`, `ToolRegistry`, visible trace |
| Memory design | Session and persistent patient memory |
| Scheduling and conflict management | `SchedulingService` validation rules |
| Multilingual handling | English, Hindi, Tamil detection and responses |
| Performance optimization | Local tools, lightweight planner, measured latency |
| Code quality and structure | Separated agent, services, tools, models, tests |
| Documentation | README, diagram, Loom guide |

---

## 17. Bonus Features Included Or Ready

Implemented:

- Visible reasoning/tool traces
- Barge-in-friendly frontend behavior by cancelling active speech before speaking again
- Outbound campaign mode
- Latency logging

Ready extension points:

- Redis-backed memory with TTL
- Background campaign queue
- PostgreSQL appointment storage
- Cloud deployment with horizontally scaled FastAPI workers
- Streaming WebSocket audio transport

---

## 18. Known Limitations

- Browser STT/TTS is used for demo simplicity instead of a production telephony provider.
- The planner is deterministic and transparent rather than connected to a hosted LLM.
- JSON storage is inspectable but not suitable for concurrent production writes.
- The architecture supports WebSocket/Redis upgrades, but this demo uses HTTP turns and local JSON stores.
- Browser voice recognition quality depends on Chrome/Edge and OS language support.

---

## 19. Submission Checklist

Required deliverables:

- GitHub repository with complete code
- README with setup, architecture, memory, latency, tradeoffs, and limitations
- Architecture diagram in `docs/`
- Loom video up to 3 minutes

Suggested Loom flow:

1. Show UI and book an appointment in English.
2. Show Hindi or Tamil language detection.
3. Show cancellation or rescheduling.
4. Point to the trace panel and latency badge.
5. Briefly explain backend structure and memory design.


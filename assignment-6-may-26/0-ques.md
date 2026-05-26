# Assignment - MINI PROJECT - Project name decision

## init phase-0
ProductSync mini project:

### Gchat Message:
i need to discuss a project idea with you:
@Pratham Kumar Taheem @Vaibhavi Upreti I’m planning to start our next activity in parallel with the product; we’re going to build a small hands-on project.The goal is to design and build something that uses three important capabilities of our framework.Here’s a quick summary of what they do:
Service Jobs: Used to run background tasks automatically at scheduled times or intervals.
System Messages: Used to send and receive real-time data payloads (such as JSON or XML) reliably with external systems.
Data Manager: Used to download and import bulk data files in a single atomic step, including mapping, record creation, and clean error tracking.
I’d like you to brainstorm a few project ideas. Think about a simple workflow we can build that combines all three capabilities together in a meaningful way.We’ll discuss these around 7:45 - 8:00 PM.

### My inferences
this discussion will follow an activity to do.. based on all above mentioned things, yesterday we discussed a project which was on NASA asteroids tracking.. with the help of all 3, there is a remote system we need to talk to, the data coming in bulk is not directly saved to the database rather saved to data manager, and a service runs that polls that data from the external system at some predefined time intervals, if it is a real time feed then the intervals to fetch data will be minimal and seems to be happening in real time, this will cost a lot talking to an external system as it has kept its data somewhere which is non reachable easily and on the terms of the external system we are using, so to poll the data and to save that to the database there are dedicated services and system jobs that runs in the background, and if they fail to do their tasks then they retry upto a certain limit and in the end gives a failure status, the moqui framework which is used in hotwax follows the same.. it transforms the data internally that it brings and save to temporary space called data manager and handles it with the help of another service then.. this is how the flow works..
on the basis of this we can implement these useful features to several systems, we can diversify their usage and the use cases the applications, for developing robust systems that talk to some external systems in nearly real time which can be used in different domains, can be agriculture, ecommerce, defense, etc. my company hotwax focusess on erp, ecommerce, crm, mainly order management system, so this mini project will be reflecting my capabilities in how i work at an architect level to define the requirements with all points in mind and to develope robust system that can handle severities, without any extra load, it will be highly optimized, scalable, flexible, modular, can handle failures, and manage space, memory and CPU cycles in the best way possible, so i have to think like a system architect for this project module.. did you get the context in the way i wanted to deliver

### Core Pattern 
Integration architecture:
- Service Job — polls an external system at scheduled intervals (mimicking near-real-time without hammering the external API), with retry logic and failure status tracking
- System Message — handles the actual communication with the external system, receiving the raw data payload (JSON/XML) reliably
- Data Manager — acts as a staging/temporary space where the bulk data lands first, gets transformed/mapped, and then gets committed to the database atomically — not a direct write
- Case: The NASA asteroid tracking project

### Where am i heading?
I want to take this same architectural pattern and apply it to a domain closer to HotWax's world — ERP, eCommerce, OMS, CRM — to demonstrate the ability to think at an architect level, specifically:
Defining requirements with failure handling, retries, and status tracking in mind
Designing for scalability, modularity, and near-real-time responsiveness
Optimizing for CPU, memory, and space — no waste
Building something that reflects real production-grade thinking, not just a demo
---

## PHASE 1 - ARCHITECTURE AND TEST CASE SCENARIOS
NASA OBJECT LOCATION TRACKING  
The system will allow users to select an object/star and define the time period for tracking. At scheduled intervals, the system will automatically collect the latest location information from external space data providers. The collected information will be safely stored, even when large amounts of data are received. If data is temporarily unavailable, the system will automatically try again to ensure reliable tracking. Historical movement details will also be maintained so users can view past positions and tracking history anytime.

## Expected output
- Requirement Analysis.
- Implementation details: What comes where? [MDM, SysJobs,etc..].
- UML - Activity diagram, flow chart, and a sequence diagram.


---
# Solution

## Requirement analysis:
It is like a Product 
A Business Case.
---

## Sync: Business Requirements Analysis
### NASA Object Location Tracking System

**Domain context:** 
- A near-real-time space object tracking system.
- It reliably fetches, stages, and persists positional data from an external NASA data provider.
- Architectural discipline: similar to integration architecture with edge cases.

---

## REQ 01 — User needs and goals

The system must allow a user — todo 3 things:
- select a space object they want to track, 
- define how frequently they want fresh location data, 
- and trust that the system is doing its job reliably in the background. 
[They should never have to manually trigger a data fetch or worry that data was silently missed.]

---

## REQ 02 — Data collection requirements

**Normal operation.** At whatever frequency the user has configured (e.g. every 15 minutes, every hour), the system must automatically reach out to the NASA data provider and collect the latest positional information for the selected object. This must happen without any human initiation.

**Delayed or unavailable external data.** The external NASA system may sometimes be slow, temporarily offline, or returning empty results. The system must not treat this as a permanent failure immediately. It must wait a defined amount of time and try again. The number of retry attempts and the gap between retries must be configurable — not hardcoded.

**Rate limiting from the external provider.** NASA's API may restrict how many requests can be made per unit of time. The system must respect these limits. If the external system signals a rate-limit response, the system must pause and retry at an appropriate interval — not hammer the endpoint.

**Partial or malformed data received.** If the external system sends back a response that is incomplete, corrupted, or in an unexpected format, the system must not crash, must not save bad data, and must log exactly what was received and why it was rejected. The raw payload must be preserved for investigation.

**No data change since last fetch.** If the external system returns the same position as the last known record (no movement detected), the system must still record the fetch attempt and its timestamp, but must not create a duplicate positional record. Deduplication must be handled before anything is written to the database.

---

## REQ 03 — Staging and data handling requirements

**Bulk data safety.** When a large batch of positional records arrives in a single response, the system must not attempt to write them all to the database in one shot. Data must first land in an intermediate staging area (the data manager). From there, records are validated, mapped to the internal format, and only then committed. If the commit fails midway, no partial data must survive — it either all goes in or none of it does.

**Transformation rules.** NASA's data format may use coordinates, timestamps, or measurement units different from what the system's database expects. The mapping rules between the external format and internal format must be defined, versioned, and auditable — so that when the external provider changes their format, the impact is isolated and traceable.

**Space and storage hygiene.** The staging area is not permanent storage. After a successful commit, the staged raw data must be cleared. After a failed commit that has exhausted retries, the raw data must be archived for investigation — not silently deleted. Storage consumption must be bounded; old archives must be eligible for cleanup after a defined retention window.

---

## REQ 04 — Failure handling and retry requirements

**Retry limit.** Every fetch attempt is allowed a maximum number of retries before the system gives up and marks it as a permanent failure. This limit must be configurable per object or globally. The default should be conservative — three to five attempts — to prevent indefinite resource consumption.

**Retry interval and backoff.** Retries must not happen immediately one after another. There must be a wait period between attempts. Ideally this wait grows with each retry (exponential backoff), so the system does not pile pressure on an already-struggling external provider. The backoff strategy must be configurable.

**Failure visibility.** When a fetch permanently fails after exhausting retries, the following must happen: the failure must be logged with a clear status, the time of failure, the object being tracked, and the error reason. An alert or notification must be raised so a human knows something needs attention. The system must not silently swallow failures.

**Cascading failures.** If the external NASA system goes fully offline for an extended period, the retry queue must not grow unbounded. There must be a ceiling on how many pending retries can exist at any one time. Beyond that ceiling, new fetch attempts must be deferred — not created — until the backlog clears.

**Recovery after failure.** When the external system comes back online after an outage, the tracking system must automatically resume without needing manual intervention. It must also backfill the gap — recognising that positional data was missed during the outage window — and flag those gap periods in the history log as "data unavailable" rather than leaving them blank.

---

## REQ 05 — Scheduling and interval requirements

**Configurable interval per object.** A fast-moving near-Earth asteroid may need data fetched every minute. A distant star may only need updates daily. The poll interval must be configurable per tracked object, not a single global setting.

**Minimum interval guardrail.** The system must enforce a minimum interval between fetches for any single object to prevent accidental misconfiguration from generating thousands of requests per hour. A system-level floor (e.g. no faster than once every 5 minutes) must exist and must be enforceable by an administrator.

**Interval change in flight.** If a user changes the poll interval for an object that is currently being tracked, the change must take effect at the next scheduled run — not mid-run. The currently running fetch must complete normally.

**Pause and resume.** A user must be able to pause tracking for an object without deleting its history. When tracking resumes, the system must note the gap period in the history log.

---

## REQ 06 — History and auditability requirements

**Every fetch attempt is recorded.** Whether it succeeded, failed, was skipped (no change), or was retried — every attempt against the external system must produce a log entry. This log is the source of truth for "did the system do its job?"

**Positional history is immutable.** Once a positional record is committed to the database, it must never be overwritten or deleted during normal operation. Corrections must be additive — a new record flags the old one as superseded, never replaces it.

**Viewable timeline.** A user must be able to look at any tracked object and see a chronological history of its recorded positions, the timestamps those positions were fetched, and any gaps or failures in the timeline.

---

## REQ 07 — Edge cases and boundary scenarios

| Scenario | Expected system behaviour |
|---|---|
| External API changes its response format | System rejects the payload, logs the schema mismatch, raises an alert. Old data is unaffected. |
| Two fetch jobs for the same object overlap (race condition) | Only one job processes at a time. The second must wait or be discarded — no duplicate writes. |
| Object is deleted by the user mid-fetch | The in-progress fetch completes, its result is discarded, and all future schedules are cancelled. |
| Database runs out of storage mid-commit | The commit rolls back completely. The staged data is preserved. An alert is raised. |
| Retry queue fills up completely | New fetch triggers are held — not queued — until space is available. A threshold alert fires before the queue is full. |
| NASA returns coordinates in a new unit system | Transformation layer rejects unmapped fields. Payload is archived. Human review is flagged. |
| System restarts mid-fetch | On startup, the system must detect and resume or clean up any orphaned in-progress jobs. No data loss, no phantom runs. |
| Clock drift between internal scheduler and external API | Timestamps must always use UTC. Any timezone discrepancy in the external payload must be normalised before staging. |

---

## REQ 08 — Non-functional requirements (business framing)

**Reliability.** The system must guarantee that if data was available from NASA during a fetch window, it will eventually be recorded — even if it takes several retry cycles. Best-effort is not acceptable for a tracking system.

**Observability.** At any point in time, an operator must be able to answer: "What is the current status of tracking for object X?" The answer must be one of: Active, Retrying, Paused, or Failed — with a reason.

**Scalability.** Adding a second or tenth tracked object must not degrade performance for objects already being tracked. Each object's tracking lifecycle must be isolated.

**Graceful degradation.** If the system is under high load, it must slow down its fetch rate — not crash. A backpressure mechanism must exist so the scheduler does not create more work than the system can currently handle.

**No vendor lock-in on the external source.** The integration with NASA's API must be behind an abstraction layer. Switching to a different space data provider in the future must require changing only the connector — not the scheduling, staging, or persistence layers.

---



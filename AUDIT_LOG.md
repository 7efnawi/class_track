# ClassTrack — System Audit & Progress Log

This log tracks all architectural decisions, project setup, team distribution, and audit trails for the **ClassTrack** project. Every human engineer and AI coding agent must consult and update this document as work progresses.

---

## 1. Project Metadata

* **Project Name:** ClassTrack — Smart Mobile Attendance and Quiz System
* **Course:** Mobile Programming – Level 2 (Dr. Amany)
* **Team Size:** 5 Students
* **Creation / Initiation Date:** 2026-10-04
* **Target Delivery Window:** 4 Weeks
* **Current Status:** Planning & Task Allocation Phase (Zero code written - Clean Baseline)

---

## 2. Active Governance & Planning Documents

| Document | Purpose | Target Audience | Status |
| :--- | :--- | :--- | :--- |
| **`TEAM_RESPONSIBILITIES.md`** | Comprehensive division of responsibilities across the 5 students, files owned, Git branches, and deliverables. | Human Students & Doctor | ✅ Active & Complete |
| **`AGENT_SYSTEM_RULES.md`** | Strict architectural guidelines, boundaries, and inviolable rules (e.g. location visibility, backend validation, 2-minute limit). | AI Agents & Developers | ✅ Active & Complete |
| **`AUDIT_LOG.md`** | Real-time audit trail and system tracker. | All | ✅ Active & Complete |

---

## 3. Subsystem Readiness (Clean Slate)

| Subsystem | Owner | Primary Files | Status | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Team Division & Specs** | All Members | `TEAM_RESPONSIBILITIES.md` | ✅ Done | 5-member allocation ready |
| **Dependencies & Pubspec** | Member 1 / All | `pubspec.yaml` | ⏳ Default State | Clean initial template |
| **Platform Permissions** | Member 3 / All | `AndroidManifest.xml` | ⏳ Default State | Clean initial template |
| **Data Models** | All Members | `lib/models/*.dart` | ⏳ Not Started | To be implemented per track |
| **Backend & Services** | Members 1, 3, 5 | `lib/services/*.dart` | ⏳ Not Started | To be implemented per track |
| **Auth & Profiles (Track 1)** | Member 1 | `lib/screens/auth/*` | ⏳ Not Started | Ready for Member 1 |
| **Teacher Classes (Track 2)** | Member 2 | `lib/screens/teacher/*` | ⏳ Not Started | Ready for Member 2 |
| **Student Punch-In (Track 3)** | Member 3 | `lib/screens/student/*` | ⏳ Not Started | Ready for Member 3 |
| **Quiz Mode 2 (Track 4)** | Member 4 | `lib/screens/quiz/*` | ⏳ Not Started | Ready for Member 4 |
| **Teacher Live Monitor & Rules (Track 5)**| Member 5 | `live_session_screen.dart`, `firestore.rules` | ⏳ Not Started | Ready for Member 5 |

---

## 4. Chronological Audit Trail

### [2026-10-04 00:20] — Project Discovery & Specification Analysis
* Analyzed official project brief `ClassTrack_Project_Assignment.pdf`.
* Verified 5-member team structure and requirements.

### [2026-10-04 00:27] — System Governance Documents Established
* Created `TEAM_RESPONSIBILITIES.md`, `AGENT_SYSTEM_RULES.md`, and `AUDIT_LOG.md`.

### [2026-10-11 23:07] — Codebase Reset to Default Clean State
* **Action:** Per user instruction, reverted all codebase changes back to the original default Flutter template.
* **Result:** Codebase is completely clean (`lib/main.dart` default counter app, default `pubspec.yaml`, default `AndroidManifest.xml`).
* **Preserved:** Only the planning and team division documents (`TEAM_RESPONSIBILITIES.md`, `AGENT_SYSTEM_RULES.md`, `AUDIT_LOG.md`).

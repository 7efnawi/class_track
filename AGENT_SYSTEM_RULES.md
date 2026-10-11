# ClassTrack — AI Agent System Rules & Project Constraints

> **CRITICAL DIRECTIVE FOR ALL AI AGENTS & CONTRIBUTORS**  
> This file establishes the inviolable architecture, behavioral boundaries, and implementation rules for **ClassTrack**. Any AI Agent (Antigravity, Claude, Copilot, etc.) working on this repository **MUST** strictly adhere to every rule listed below. Do NOT deviate without explicit authorization from the development team.

---

## 1. System Philosophy & Level Scope
* **Target Audience:** University course **Mobile Programming – Level 2** (Supervised by Dr. Amany).
* **Stack:** Flutter (Dart) + Firebase (Authentication, Cloud Firestore, Storage).
* **Design Principle:** Keep the code readable, modular, well-commented, and straightforward.
* **Prohibition:** **DO NOT** introduce overly complex patterns, unrequested third-party state managers, or bloated microservices. Use `Provider` or standard Flutter state management.

---

## 2. Inviolable System Rules (NEVER BREAK THESE)

### Rule 1: The Location Visibility Rule (CRITICAL)
* **Statement:** **All enrolled students MUST be able to see an active attendance/quiz session, regardless of their physical location.**
* **What MUST NOT be done:** NEVER filter active session visibility based on GPS distance. NEVER hide the session card or notification from students who are outside the radius.
* **What MUST be done:** Any student enrolled in the class receives the real-time active session banner. Location is evaluated **ONLY** when the student attempts to punch in or submit the quiz.

### Rule 2: Backend Validation Rule (No Client-Only Decisions)
* **Statement:** The client device can be spoofed or modified. **The acceptance decision must NOT be made solely by a client-side `if (distance <= radius)` statement.**
* **What MUST NOT be done:** Never mark a student as `present` directly on the device without validating against the backend records, server timestamps, and session status.
* **What MUST be done:**
  1. Verify session `status == "active"`.
  2. Verify `request.time < session.expiryTime`.
  3. Verify student is in `enrollments` for that `classID`.
  4. Verify no previous record exists for `sessionID + studentID`.
  5. Distance is evaluated and saved; if valid $\rightarrow$ `status: "present"`, if outside $\rightarrow$ `status: "rejected"`.

### Rule 3: Session Duration Rule (Strictly 2 Minutes)
* **Statement:** Every attendance session (Punch-In or Quiz) is valid for **exactly 2 minutes** (120 seconds).
* **Implementation:** `expiryTime = startTime + Duration(minutes: 2)`.
* **Client UI:** Displays a 2-minute countdown timer (`mm:ss`). If the timer hits `00:00`, the UI automatically locks/submits and rejects further attempts.

### Rule 4: Quiz Source & Format Rule
* **Statement:** The quiz in Mode 2 **MUST be built entirely inside the Flutter application**.
* **What MUST NOT be done:** DO NOT use Google Forms or external webviews as the primary quiz mechanism.
* **What MUST be done:**
  - Exactly **5 Multiple-Choice Questions (MCQ)**.
  - Exactly 1 correct answer per question.
  - Interactive radio-button selection in Flutter.
  - In-app score calculation (out of 5).
  - Both score and attendance status (`present` or `rejected`) recorded in Firestore.

### Rule 5: Duplicate Attendance Prevention
* **Statement:** A student can have at most **ONE** attendance attempt/record per session.
* **Implementation:** The document ID in `attendance_records` MUST follow the deterministic compound key:
  ```text
  attendance_records/{sessionID}_{studentID}
  ```
  Attempting to create a second document with the same key will be blocked or detected as duplicate.

### Rule 6: Dual Role in a Single Application
* **Statement:** The application contains both Teacher and Student experiences in one codebase.
* **Routing:** Role is determined upon login by reading the user's profile from Firestore `users/{uid}`.
  - If `role == "teacher"` $\rightarrow$ route to `TeacherHomeScreen`.
  - If `role == "student"` $\rightarrow$ route to `StudentHomeScreen`.

---

## 3. Explicitly Out-of-Scope (DO NOT IMPLEMENT)

To prevent scope creep and project failure, the following features are **strictly prohibited** unless explicitly requested by the user:
- ❌ Facial recognition / Biometric attendance
- ❌ Continuous background GPS tracking (battery/privacy violations)
- ❌ Bluetooth BLE beacons or Wi-Fi fingerprinting
- ❌ Complex AI cheating detection / eye tracking
- ❌ Google Forms integration (until all primary requirements are 100% done and tested)
- ❌ Multi-university / Multi-tenant architecture

---

## 4. Required Firestore Schema Baseline

Every AI Agent modifying database services **MUST preserve** the following field names:

| Collection | Required Fields | Notes |
| :--- | :--- | :--- |
| `users` | `name`, `email`, `role`, `studentID` | `role` is "teacher" or "student". `studentID` is student-only. |
| `classes` | `name`, `teacherID`, `semester`, `joinCode` | Owned by teacher. Join code is unique 6-char string. |
| `enrollments` | `classID`, `studentID`, `enrolledAt` | Doc ID: `{classID}_{studentID}` |
| `attendance_sessions` | `classID`, `teacherID`, `mode`, `latitude`, `longitude`, `radius`, `startTime`, `expiryTime`, `status` | `mode`: "punch_in" \| "quiz". `status`: "active" \| "closed". |
| `attendance_records` | `sessionID`, `studentID`, `timestamp`, `latitude`, `longitude`, `distance`, `status`, `quizScore` | Doc ID: `{sessionID}_{studentID}`. `status`: "present" \| "rejected" \| "absent". |

---

## 5. Development Guidelines for Future AI Agents

1. **Incremental Work Only:** Never dump 20 files at once without explaining what is being built and verifying dependencies.
2. **Preserve Working Code:** Do not refactor or wipe working screens unless a clear flaw is identified and communicated.
3. **Update the Audit Log:** Whenever you create, modify, or complete a major subsystem, update `AUDIT_LOG.md` immediately with the date, action, files touched, and verification status.
4. **Error Handling:** Every async operation (Firebase, GPS) must have clear user-facing error dialogs or SnackBar feedback (e.g., GPS disabled, permission denied, session expired). Never fail silently.

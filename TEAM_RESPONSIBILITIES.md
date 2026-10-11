# ClassTrack — Team Member Responsibilities & Work Division (5 Members)

**Course:** Mobile Programming – Level 2  
**Instructor:** Dr. Amany  
**Project:** ClassTrack — Smart Mobile Attendance and Quiz System  
**Team Size:** 5 Students  
**Stack:** Flutter (Dart), Firebase (Auth, Firestore, Storage)  

---

## 1. Overview & Work Division Philosophy

To ensure equal contribution, clean code separation, and zero merge conflicts, the project is divided into **5 dedicated feature tracks**. Each team member owns specific data models, services, screens, and a Git branch.

```
                         ClassTrack (develop)
          ┌─────────────┬─────────────┼─────────────┬─────────────┐
          │             │             │             │             │
    feature/auth   feature/      feature/      feature/      feature/
      (Member 1)   teacher-      student-      quiz-mode    dashboard-
                   classes       punchin       (Member 4)      rules
                  (Member 2)    (Member 3)                  (Member 5)
```

---

## 2. Member Responsibilities Matrix

| Member | Primary Track | Core Feature | Owned Models & Services | Owned Screens / Files | Git Branch |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Member 1** | Auth & Identity | Login, Registration, Role Routing (Teacher/Student) | `UserModel`, `AuthService` | `lib/screens/auth/*`, `lib/core/routes.dart` | `feature/auth` |
| **Member 2** | Teacher & Classes | Class Creation, Join Codes, Teacher Home | `ClassModel`, Class Firestore logic | `lib/screens/teacher/teacher_home_screen.dart`, `create_class_screen.dart`, `class_details_screen.dart` | `feature/teacher-classes` |
| **Member 3** | Student & Mode 1 | Join Class, Location Service, Mode 1 Punch-In | `EnrollmentModel`, `LocationService` | `lib/screens/student/*`, `punch_in_screen.dart` | `feature/student-punchin` |
| **Member 4** | Mode 2: Quiz | 5 MCQ Questions, In-app Quiz UI, 2-Min Timer, Scoring | `QuizModel`, `QuestionModel`, Quiz logic | `lib/screens/quiz/quiz_screen.dart`, `quiz_result_screen.dart` | `feature/quiz-mode` |
| **Member 5** | Live Monitor & Rules | Teacher Live Attendance, Session History, Firestore Security Rules | `AttendanceRecordModel`, `SessionModel`, Security Rules | `lib/screens/teacher/live_session_screen.dart`, `session_history_screen.dart`, `firestore.rules` | `feature/dashboard-rules` |

---

## 3. Detailed Member Breakdown

### Member 1: Authentication & User Profiles
* **Objective:** Manage user authentication and assign roles (`teacher` vs `student`).
* **Detailed Tasks:**
  1. Set up Firebase Authentication (Email and Password).
  2. Implement `UserModel` with fields: `uid`, `name`, `email`, `role` ("teacher" or "student"), `studentID` (students only).
  3. Create `AuthService` with methods: `register()`, `login()`, `logout()`, `getCurrentUser()`.
  4. Build UI:
     - `LoginScreen`: email/password input with validation and error alerts.
     - `RegisterScreen`: name, email, password, role selector (Radio/Segmented button), and conditional `studentID` field for students.
  5. Implement auth-state routing in `lib/core/routes.dart` so teachers land on Teacher Home and students land on Student Home automatically.
* **Deliverable Checklist:**
  - [ ] Working Registration with role stored in Firestore `users/{uid}`.
  - [ ] Working Login and session persistence.
  - [ ] Appropriate error handling (weak password, user not found, invalid email).

---

### Member 2: Teacher Class Management
* **Objective:** Enable teachers to create and manage classes and generate join codes.
* **Detailed Tasks:**
  1. Implement `ClassModel` with fields: `id`, `name`, `teacherID`, `semester`, `joinCode`, `createdAt`.
  2. Build `TeacherHomeScreen`:
     - Displays list of classes owned by the logged-in teacher (stream from Firestore).
     - Floating Action Button to create a new class.
  3. Build `CreateClassScreen`:
     - Input for class name, semester.
     - Auto-generates a unique 6-character uppercase alphanumeric `joinCode` (e.g., `MP2026`).
     - Saves the class to Firestore `classes` collection.
  4. Build `TeacherClassDetailsScreen`:
     - Displays class info, join code (with copy button).
     - Displays list of enrolled students.
     - Button to "Start Attendance Session" (navigating to session setup).
* **Deliverable Checklist:**
  - [ ] Teacher can create a class with a unique Join Code.
  - [ ] Teacher sees only their created classes.
  - [ ] Class details shows enrolled students count and list.

---

### Member 3: Student Flow & Mode 1 (Location Punch-In)
* **Objective:** Implement student class enrollment and Mode 1 location-based attendance punch-in.
* **Detailed Tasks:**
  1. Implement `EnrollmentModel` (`classID`, `studentID`, `enrolledAt`).
  2. Build `JoinClassScreen`:
     - Student inputs the 6-character `joinCode`.
     - Validates code exists in `classes` collection, prevents duplicate enrollment, and creates record in `enrollments/{classID_studentID}`.
  3. Build `StudentHomeScreen` & `StudentClassDetailsScreen`:
     - Lists all enrolled classes.
     - Real-time listener for active sessions in enrolled classes.
     - Displays **Active Session Banner** with live countdown timer when a teacher opens a session (unrestricted visibility).
  4. Implement `LocationService`:
     - Checks GPS enabled & requests permissions using `geolocator`.
     - Obtains high-accuracy device coordinates (`latitude`, `longitude`).
     - Calculates Haversine distance in meters to teacher's session coordinates.
  5. Build `PunchInScreen`:
     - Single-tap "PUNCH IN" button.
     - Executes validation: checks session active, checks expiry, checks distance <= allowed radius.
     - Records attendance in Firestore `attendance_records`.
* **Deliverable Checklist:**
  - [ ] Student can join class via code.
  - [ ] Student sees active session regardless of distance.
  - [ ] GPS permission handled smoothly with error dialogs if disabled.
  - [ ] Attendance rejected if distance > radius; accepted if distance <= radius.

---

### Member 4: Mode 2 (Location-Restricted Quiz System)
* **Objective:** Implement the in-app 5-question multiple choice quiz with countdown timer and scoring.
* **Detailed Tasks:**
  1. Implement `QuizModel` and `QuestionModel`:
     - 5 multiple-choice questions per quiz session.
     - Each question has `questionText`, 4 `options`, and `correctAnswerIndex`.
  2. Preload/store quiz questions in Firestore or provide teacher quiz question selection.
  3. Build `QuizScreen`:
     - In-app Flutter UI (NOT an external Google Form).
     - Renders all 5 questions with Radio buttons for single-option selection.
     - Displays a prominent real-time 2-minute countdown timer (`02:00` -> `00:00`).
     - Auto-submits or locks the quiz when timer reaches 0.
  4. Implement Quiz Submission & Score Calculation:
     - Verifies all 5 questions are answered.
     - Obtains student GPS coordinates on submission.
     - Calculates score out of 5 (e.g., 4/5, 5/5).
     - If within radius: status = `present`, records `quizScore`.
     - If outside radius: status = `rejected`, records `quizScore`.
  5. Build `QuizResultScreen`:
     - Displays final score, time taken, distance from teacher, and attendance status.
* **Deliverable Checklist:**
  - [ ] 5 questions displayed with single-selection radio buttons.
  - [ ] 2-minute timer strictly counts down and prevents late submission.
  - [ ] Score calculated accurately (0 to 5).
  - [ ] Attendance record written with quiz score and location status.

---

### Member 5: Real-Time Dashboard, History & Security Rules
* **Objective:** Real-time teacher monitoring of attendance/quizzes, session history, and backend data integrity.
* **Detailed Tasks:**
  1. Implement `SessionModel` (`id`, `classID`, `teacherID`, `mode`, `latitude`, `longitude`, `radius`, `startTime`, `expiryTime`, `status`).
  2. Build `StartSessionScreen`:
     - Teacher selects Mode (Mode 1: Punch-In, Mode 2: Quiz).
     - Teacher sets radius in meters (e.g., 30m, 50m, 100m).
     - Captures teacher's current GPS location.
     - Creates active session with `expiryTime = startTime + 2 minutes`.
  3. Build `LiveSessionScreen` (Teacher Dashboard):
     - Displays session countdown timer.
     - Live Firestore Stream of attendees as they punch in or submit quiz.
     - Shows student name, distance, submission status, quiz score (Mode 2), and attendance status (`present` / `rejected`).
     - Button to manually close session early if needed.
  4. Build `SessionHistoryScreen`:
     - View previous sessions, total attendance count, and student breakdown.
  5. Security Rules (`firestore.rules`):
     - Prevent duplicate attendance using compound ID: `sessionID_studentID`.
     - Enforce `request.time < resource.data.expiryTime`.
     - Enforce authenticated read/write rules.
  6. Documentation & Polish:
     - Write the final `README.md` for project submission with member contributions, setup guide, and screenshots.
* **Deliverable Checklist:**
  - [ ] Teacher live session view updates instantly without manual refresh.
  - [ ] Session history displays past attendance records.
  - [ ] Firestore rules block duplicate records and late writes.
  - [ ] Polished error messages and README.

---

## 4. Git Branching & Collaboration Guidelines

1. **Never commit directly to `main`:**
   - `main`: Production-ready, fully tested code for submission.
   - `develop`: Integration branch where all features merge.
2. **Feature branches:**
   - Always branch off `develop`:
     ```bash
     git checkout develop
     git pull origin develop
     git checkout -b feature/<your-feature-name>
     ```
3. **Commit Messages Convention:**
   - `feat: add student join class by code`
   - `fix: prevent duplicate attendance record creation`
   - `style: improve quiz radio button layout`
   - `docs: update setup instructions in README`
4. **Pull Requests (PR):**
   - When your feature is done, push your branch and open a PR into `develop`.
   - At least one teammate must test and approve before merging.

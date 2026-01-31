# Exhaustive Analysis and Comparison: WorkflowS-project vs Wrk_Api

## 1. Project Overview

### WorkflowS-project (Node.js + Express)
A full-stack project management application. The backend is built with Node.js and Express, using Prisma as an ORM with support for both SQLite and PostgreSQL. It includes comprehensive features such as authentication, task management, sprint planning, evaluation rubrics, real-time-like chat, and file versioning.

### Wrk_Api (Go)
Currently, this directory is empty, containing only a `go.mod` file with no implementation. This means 100% of the functionality from `WorkflowS-project` is missing and needs to be implemented.

---

## 2. Data Models (Prisma Schema Analysis)

The following models are defined in `WorkflowS-project/prisma/schema.prisma` and need to be ported to Go structs (using an ORM like GORM):

| Model | Description | Key Fields |
|-------|-------------|------------|
| **User** | System users | `email`, `password` (hashed), `name`, `role`, `avatar`, `active` |
| **Project** | Main entity | `name`, `description`, `status`, `ownerId`, `startDate`, `endDate` |
| **ProjectMember** | Join table | `projectId`, `userId`, `role` |
| **Sprint** | Time-boxed work | `name`, `projectId`, `startDate`, `endDate`, `status` |
| **UserStory** | Agile requirements| `title`, `description`, `priority`, `storyPoints`, `status`, `assigneeId` |
| **Task** | Granular work | `title`, `description`, `priority`, `status`, `assigneeId`, `deadline` |
| **Document** | File management | `name`, `url`, `type`, `size`, `version`, `parentId` (for history) |
| **Rubric** | Grading template | `name`, `description`, `projectId` (can be global or project-specific) |
| **Criteria** | Rubric details | `rubricId`, `name`, `maxScore`, `weight` |
| **Evaluation** | Grading record | `score`, `feedback`, `evaluatorId`, `taskId/sprintId/projectId` |
| **EvaluationCriteria**| Individual scores | `evaluationId`, `criteriaId`, `score`, `comment` |
| **Chat** | Conversations | `projectId`, `type` (PROJECT, DIRECT) |
| **Message** | Chat content | `chatId`, `userId`, `content` |
| **Notification** | Alerts | `userId`, `title`, `message`, `type`, `read` |
| **RetrospectiveItem** | Sprint feedback | `sprintId`, `userId`, `type` (GOOD, BAD, ACTION), `content` |

---

## 3. API Routes and Functionality

The following routes are implemented in Express and are missing in the Go API:

### Authentication (`/api/auth`)
- `POST /register`: User registration with `bcryptjs` hashing.
- `POST /login`: Credential verification and JWT generation.
- `POST /logout`: Session termination.

### Projects & Members (`/api/projects`)
- CRUD operations for projects.
- Member assignment (`POST /:id/members`) with role management and notifications.

### Sprints & User Stories (`/api/sprints`, `/api/user-stories`)
- CRUD for Sprints and Stories.
- Story assignment to Sprints (`POST /:id/add-story`).
- Status-based completion tracking (updates `completedAt` automatically).

### Tasks & Evaluations (`/api/tasks`, `/api/evaluations`)
- Task CRUD and assignment.
- **Complex Evaluation System**: Support for grading at Project, Sprint, and Task levels using Rubrics and Criteria.
- Atomic transactions (Prisma `$transaction`) to save evaluation records and criteria scores simultaneously.

### Metrics (`/api/metrics`)
- **Burndown Chart**: Daily calculation of remaining points vs. ideal decrement.
- **Contribution**: Task completion count per user.
- **Velocity**: Committed vs. Completed points across sprints.
- **Export**: Generation of project reports in CSV format.

### Documents (`/api/documents`)
- File uploads using `multer`.
- **Versioning**: When a new version is uploaded (`POST /:id/versions`), it maintains a link to the parent document and increments the version number.

### Real-time Communication (`/api/chat`)
- Project-based group chats.
- Direct Messages (DMs) between users.
- Automatic creation of chats if they don't exist.

### Notifications & Retrospectives (`/api/notifications`, `/api/retrospectives`)
- Pull-based notifications (fetching last 20).
- Retrospective board for sprints with team-wide notifications.

---

## 4. Technical Stack Comparison & Recommendations

To match the Node.js implementation, the Go API should adopt the following equivalents:

| Feature | Node.js (WorkflowS-project) | Recommended Go Equivalent |
|---------|-----------------------------|---------------------------|
| **Web Framework** | Express | Gin, Fiber, or Echo |
| **ORM** | Prisma | GORM or Ent |
| **Auth (JWT)** | `jsonwebtoken` | `golang-jwt/jwt` |
| **Password Hashing** | `bcryptjs` | `golang.org/x/crypto/bcrypt` |
| **File Uploads** | `multer` | Gin's `c.FormFile` or `io.Copy` |
| **Database** | SQLite / PostgreSQL | `modernc.org/sqlite` or `pgx` |
| **CORS** | `cors` middleware | Gin's CORS middleware |
| **Environment** | `dotenv` | `joho/godotenv` |

---

## 5. Summary of What's Missing in `Wrk_Api`

1.  **Project Structure**: No folders for `cmd`, `internal`, `models`, `handlers`, `routes`, or `repository`.
2.  **Database Layer**: No connection logic or migration system.
3.  **Middlewares**: Missing Auth (JWT), Logging, and Recovery middlewares.
4.  **All Business Logic**: None of the 13 modules described above are implemented.
5.  **Utilities**: No logic for Metrics calculation or CSV generation.

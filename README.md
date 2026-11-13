# Canvas AI Assistant

> A full-stack, human-in-the-loop AI course management assistant that streamlines instructor workflows in the Canvas LMS through natural-language commands, dry-run previews, and auditable execution.

*Note: To protect proprietary implementation details, this public repository serves as an architectural and engineering showcase. Live walkthroughs and code samples are available upon request.*

---

## 🎯 The Problem & Solution

**The Problem:** Instructors and teaching assistants spend hours navigating repetitive menus in Canvas LMS to check missing submissions, update assignment dates, and message students—pulling focus away from teaching and course design. However, giving an LLM unrestricted write access to a live Learning Management System introduces severe data-integrity risks.

**The Solution:** Canvas AI Assistant pairs a conversational workspace with a **strict human-in-the-loop safety layer**. Instructors can query course analytics, draft communications, and stage assignment modifications using natural language, while every state-mutating action requires an explicit **dry-run preview and confirmation** before touching the live Canvas API.

### Key Capabilities
* **Natural-Language Course Operations:** Query missing/late submissions, draft targeted student messages, and create or update assignments and quizzes via chat.
* **Editable Dry-Run Previews:** Before executing any write operation, the backend stages a preview of the exact Canvas mutation so instructors can edit parameters, approve, or abort.
* **Persistent Action & Audit History:** Tracks every assistant request, proposed action, approval status, and execution outcome in an immutable audit log.
* **Administrative Oversight & Kill-Switches:** Role-based access control (Instructor, TA, Admin) with real-time service health monitoring and granular AI component toggles.

---

## 🏗️ System Architecture

The platform follows a **client-server architecture decoupled with a dedicated Model Context Protocol (MCP) tool server** to enforce strict trust boundaries between the LLM, application state, and external Canvas APIs.

<img width="812" height="458" alt="Screenshot 2026-11-13 at 2 16 27 PM" src="https://github.com/user-attachments/assets/fd253062-ee5f-4b6d-8e0d-215a07d8d06c" />


### Tech Stack & Component Breakdown

* **Frontend (`React`, `TypeScript`, `Vite`):** Presentation layer featuring the course dashboard, conversational AI workspace, interactive action-preview modals, and audit history views. Never stores Canvas credentials or calls Canvas directly.
* **Backend Orchestrator (`Python`, `FastAPI`):** Core control plane handling user authentication, session management, RBAC, action validation, pre-execution database previews, and audit logging.
* **Tool Layer (`Python`, `FastMCP`, `Docker`):** Isolated Canvas MCP Server that binds the LLM to strictly typed schemas and safely interfaces with the external Canvas LMS REST API.
* **Database (`PostgreSQL`):** Relational storage for user accounts, encrypted-at-rest Canvas API tokens, staged action previews, and audit logs.
* **CI/CD (`GitHub Actions`):** Automated pipeline enforcing linting and full test suite execution on every pull request.

### Core Technical Challenge: Distributed AI Action Reliability
Connecting non-deterministic LLM outputs to external LMS write operations required solving two critical engineering problems:
1. **Credential & Trust Boundaries:** Per-user Canvas API tokens are encrypted at rest, validated server-side, and never exposed to the client browser or the LLM context.
2. **State Integrity & Rollback Safety:** Write intents are intercepted by the backend orchestrator to generate a deterministic staging record, requiring explicit user confirmation before the MCP server dispatches the mutation to Canvas.

---

## 🧪 Testing Strategy & Coverage

Quality assurance was integrated in parallel with feature development across three isolated test zones, achieving **1,137 total automated test cases** and **99%+ code coverage** across the stack:

<img width="812" height="452" alt="Screenshot 2026-11-13 at 2 21 19 PM" src="https://github.com/user-attachments/assets/149ddc8e-a926-4cff-bacb-928bdeec44c0" />

| Layer / Service | Framework | Automated Tests | Code Coverage | Focus Areas |
| :--- | :--- | :---: | :---: | :--- |
| **Backend** | `pytest` | 672 | **99.00%** | REST APIs, auth/RBAC, action validation, preview staging, DB state |
| **Frontend** | `Vitest` + React Testing Library | 389 | **99.75%** | Component rendering, chat workflows, preview modals, UI regressions |
| **MCP Server** | `pytest` | 76 | **99.00%** | Canvas tool execution, schema argument validation, sanitized error handling |
| **Total** | **GitHub Actions CI** | **1,137** | **~99.2%** | Deterministic runs using mocked Canvas, LLM, and auth dependencies |

---

## 📊 Usability Testing & Iteration

We conducted two structured rounds of usability testing with **45 total participants** using the System Usability Scale (SUS) and task-completion benchmarks, iteratively refining onboarding and navigation between rounds:

<img width="810" height="457" alt="Screenshot 2026-11-13 at 2 23 02 PM" src="https://github.com/user-attachments/assets/4a4da3fd-6935-4e96-8cba-3af2b7064caf" />

| Metric | Round 1 (n = 28) | Round 2 (n = 45) | Improvement |
| :--- | :---: | :---: | :---: |
| **Average SUS Score** | 71.0 | **73.7** | **+2.7 pts** (Above 68 industry benchmark) |
| **Scores ≥ 68 (Above Average)** | 61% | **71%** | **+10%** |
| **Scores < 50 (Usability Friction)** | 14% | **7%** | **Cut in half** |
| **Core Functions Integrated** | 20 / 28 | **36 / 45** | Expanded workflow coverage |

**Key UX Improvements Shipped Between Rounds:**
* Added guided onboarding for first-time Canvas URL and API token setup.
* Unified course selection into a single global navigation state to eliminate duplicate selectors.
* Standardized time-zone and due-date formatting across natural-language previews.

---

## 👥 My Role & Engineering Contributions

Built by a 6-person agile engineering team over 12 weeks (**1,215+ total logged engineering hours**). As **Project Manager & Full-Stack Contributor**, my primary ownership included:
* **Architecture & DevOps Governance:** Designed the dual-zone testing structure, enforced feature-branch PR workflows (requiring 2 peer reviews per merge), and maintained GitHub Actions CI checks.
* **User & Account Management:** Engineered secure instructor/admin account lifecycle management and role-based access enforcement.
* **Student Messaging Workflows:** Built natural-language Canvas message drafting with human-in-the-loop recipient and content review prior to dispatch.
* **Usability & Release Coordination:** Led sprint planning, user requirement validation, and two rounds of SUS usability evaluations.

---

## 🎥 Demo & Documentation

* **Video Walkthrough:** `[Add Loom / YouTube Demo Link Here]`
* **Deep-Dive Architecture:** See [`ARCHITECTURE.md`](./ARCHITECTURE.md) for Entity-Relationship (ER) diagrams and Level 1 Data Flow Diagrams (DFDs).

# Canvas AI Assistant — System Architecture & Design

This document details the system architecture, component boundaries, security models, and data flows for the Canvas AI Assistant. 

---

## 1. High-Level Architecture Pattern

The system follows a **decoupled Client-Server Architecture paired with an isolated Model Context Protocol (MCP) tool layer**. This separation ensures that the frontend presentation layer never handles sensitive credentials, the core backend enforces strict business logic and safety checks, and the LLM operates inside a tightly sandboxed tool boundary.

<img width="813" height="459" alt="Screenshot 2026-11-13 at 2 30 54 PM" src="https://github.com/user-attachments/assets/6224421d-9008-4a10-9256-1f873e0765d1" />

---

## 2. Component Breakdown & Responsibilities

### Presentation Layer (`Client`)
* **Tech Stack:** React, TypeScript, Vite.
* **Responsibility:** Renders the responsive web interface, including the course dashboard, conversational chat workspace, interactive action-preview confirmation modals, and audit history logs.
* **Design Constraint:** Operates strictly as a presentation layer. It **never** stores Canvas API tokens locally or executes direct calls to the Canvas LMS API.

### Orchestration & Safety Layer (`Backend`)
* **Tech Stack:** Python, FastAPI.
* **Responsibility:** Acts as the primary control plane handling authentication, user session state, Role-Based Access Control (Instructor vs. TA vs. Admin), action validation, and immutable audit logging.
* **Key Mechanism:** Intercepts LLM-driven requests, coordinates pre-execution database staging, and serves dry-run preview diffs to the client.

### Isolated Tool Layer (`MCP Server`)
* **Tech Stack:** Python, FastMCP, Docker.
* **Responsibility:** Exposes strictly typed tool schemas to the LLM agent. It translates validated natural-language commands into structured payloads for the external Canvas LMS API.
* **Design Constraint:** Prevents arbitrary text generation from mapping to unvalidated network calls.

### Relational Persistence (`Database`)
* **Tech Stack:** PostgreSQL.
* **Responsibility:** Securely stores user profiles, encrypted-at-rest Canvas API access tokens, pending action states, and append-only system audit logs.

---

## 3. Core Safety Mechanism: Human-in-the-Loop Dry-Run Workflow

To eliminate the risk of autonomous hallucinations or unintended mass modifications to live university courses, the backend enforces a mandatory multi-step confirmation lifecycle for any state-mutating operation:

1. **Intent Interpretation:** The instructor inputs a natural-language command (e.g., *"Draft a reminder for students missing Lab 3"* or *"Update assignment due date"*).
2. **Parameter Validation & Staging:** The backend and MCP server validate required parameters. If parameters are ambiguous, execution pauses, and the system prompts the user for clarification.
3. **Dry-Run Preview Generation:** Before touching Canvas, the backend generates a pre-execution preview payload outlining the exact delta (e.g., target users, modified database records, or Canvas API mutations).
4. **Explicit Approval & Execution:** The client renders an interactive preview card. The mutation is only dispatched to the live Canvas REST API after the instructor explicitly clicks **Confirm**.
5. **Audit Logging:** An immutable record of the original command, interpreted action, user ID, timestamp, and final status is written to the audit log.

---

## 4. Security & Token Management

* **Token Encryption:** User-provided Canvas API access tokens are encrypted at rest inside PostgreSQL.
* **Privilege Isolation:** Operations are strictly executed using the authenticated instructor’s stored token, preventing cross-user token access or privilege escalation by TAs.
* **Credential Secrecy:** Stored tokens are never exposed back to the browser interface after initial submission.

---

## 5. Entity-Relationship (ER) Overview & Data Flow

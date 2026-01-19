# ProjectPath 2.0: Extracts and assigns tasks from PDFs for student teams

--- 

## Features:

* Extracts tasks from PDFs
* Classifies tasks into by type (Exam, presentation, demo, lab)
* Generates subtasks for classified task with deadline and required skills (improvement from v1)
* Assesses and takes into account skills present in the team (improvement from v1)
* Allocates subtasks to team members on the basis of skill and schedule (improvement from v1)

---

## Core Architecture for v2:

### System overview:

User is directed to a webapp, where they have the option to either create a team or log in solo. Following this, they can upload their calendar 
to give visibility of their schedule, input their own skill rating as well as take skill assessments to determine their skill level in fields like
writing or coding, and upload their syllabus for the course. Using this information, the program will parse the PDF to extract text, formulate coherent
tasks from the extracted text with relevant information like weightage and deadline, classify these tasks into their relevant types, generate subtasks
for these classified tasks based on type and deadline, and assign it across the team using its knowledge of the teams schedule and skills. It will also 
track the task completion to keep the team on track.

---
### Component breakdown:

- **Frontend:** Next.js App Router with TypeScript
- **Backend:** FastAPI as an orchestrator implementing a job/state machine. Long-running work is handled asynchronously using Redis + Celery, with intermediate results persisted so the UI can update progressively.
- **ML/AI:** Custom extraction/classification/generation models form the primary pipeline. External APIs/LLMs are used sparingly as optional refinement or fallback when custom model outputs fail validation or fall below confidence thresholds.
- **Key Architectural Change:** Replace the v1 synchronous pipeline with an asynchronous staged workflow (extract → task generation → classification → subtask/assignment), ensuring the user never waits for model inference and avoiding timeout-related deployment failures.

---
## Weekly Feature Plan and roll out:

### **Week 1 — System foundations**

**Goal:** lock architecture + types

#### Backend

- FastAPI project structure:
    
    ```
    app/
      api/
      core/
    schemas/
      services/
      users/
    	  skills/
    	  skillevidence/
    ```
    
- Define **state machine**:
    - `uploaded`
    - `extracting`
    - `extracted`
    - `task_generating`
    - `task_generated`
    - `classifying`
    - `classified`
    - `subtask_generating`
    - `completed`
    - `failed`
- Set up Redis + Celery
- Define skill formula: confidence = 0.3 * self_rating + 0.7 * evidence_score

#### Frontend

- Next.js App Router setup
- Strict TypeScript
- Auth pages
- Skill self-rating UI during onboarding
- Skill profile page (shows “Low / Medium / High confidence”)
- Project creation flow

**Deliverable:**

➡️ App runs, jobs can be queued, states update in DB.

---

### **Week 2 — PDF upload + text extraction**

**Goal:** deterministic, fast, reliable input pipeline.

#### Backend

- PDF upload endpoint
- PyMuPDF and tesseract extraction
- Text cleaning + segmentation
- Store:
    - raw text
    - cleaned text
    - extracted sections

#### Frontend

- Upload UI
- “Processing” state
- Display extracted syllabus sections

**Deliverable:**

➡️ Upload PDF → see extracted syllabus content.

---

### **Week 3 — Async orchestration + progress UI**

**Goal:** fix the v1 mistake.

#### Backend

- Celery task chaining
- Stage-based execution
- Retry + failure handling
- Partial result persistence

#### Frontend

- Polling or SSE for job state
- Progressive UI:
    - “Extraction complete”
    - “Task generation pending”

**Deliverable:**

➡️ User never waits.

---

### **Week 4 — Task generation model integration**

**Goal:** bring custom ML back safely.

#### Backend

- Wrap **task generation model** as a service
- Run inference inside Celery worker
- Schema-validated output only
- Confidence score stored

#### Frontend

- “Generate tasks” button
- Task list appears asynchronously
- Inline editing of tasks

**Deliverable:**

➡️ ML working without blocking.

---

### **Week 5 — Task classification**

**Goal:** structured intelligence

#### Backend

- Batch classification using RoBERTa model
- Category + metadata per task
- Cache results (task text hash)
- Extend task schema:
    - `required_skills: SkillType[]`
- Classification model outputs:
    - task category
    - required skills (coding / writing / scrum)

#### Frontend

- Task categories visible
- Manual override supported
- Regenerate classification for a single task
- Display required skills on each task

**Deliverable:**

➡️ Tasks feel *organized*, not dumped.

---

### **Week 6 — Subtask generation**

**Goal:** break work down realistically.

#### Backend

- Subtask generation model
- Difficulty estimation
- Per-task regeneration supported
- Subtasks inherit required skills from parent task
- Allow override per subtask

#### Frontend

- Expandable subtasks
- Edit / delete / regenerate per task
- Clear hierarchy (Task → Subtasks)
- Subtask editor shows required skills
- Clear visibility of “what skill this work trains”

**Deliverable:**

➡️ The product now mirrors initial promise and is usable.

---

### **Week 7 — Coding skill validation**

**Goal:** objective coding signal without external APIs.

#### Backend

- Coding assessment engine:
    - Question bank (start with 3–5 problems)
    - Time-boxed execution
    - Unit-test based grading
- Store:
    - score
    - time taken
    - pass/fail
- Convert result → `SkillEvidence`

#### Frontend

- Embedded code editor
- “Take coding assessment” CTA
- Show result + confidence boost

**Deliverable:**

➡️ Coding confidence is now evidence-backed

---

### **Week 8 — Writing skill validation**

**Goal:** measure clarity, not grammar perfection.

#### Writing assessment format

- Prompt (chosen from pool):
    - explain a concept
    - summarize a tradeoff
    - justify a technical decision
- Constraint:
    - 3 paragraphs max
    - word count bounds

#### Scoring (v2-safe)

- Structural rubric:
    - paragraph structure
    - clarity markers
    - conciseness
- Optional:
    - LLM feedback **only**, not scoring

#### Output

- Store rubric score as evidence
- Confidence updated

**Deliverable:**

➡️ Writing skill has real backing without heavy AI reliance.

---

### **Week 9 — Scrum / PM skill validation**

**Goal:** validate decision-making, not memorization.

#### Assessment type

- Scenario-based MCQs:
    - sprint planning
    - backlog grooming
    - blockers & retros
- 10–15 questions, auto-scored

#### Backend

- Store score as skill evidence

#### Frontend

- Lightweight quiz UI
- Feedback per question

**Deliverable:**

➡️ Scrum skill is now evidence-based.

---

### **Week 10 — Skill-aware allocation engine**

#### Allocation logic

For each subtask:

- required skills
- user confidence scores
- workload caps
- fairness constraints

Allow:

- **best-fit mode**
- **growth mode** (stretch tasks)

#### UX

- Show *why* a task was assigned:
    
    > “Assigned due to strong Coding + moderate Scrum skill”
    > 

**Deliverable:**

➡️ Allocation feels intelligent and fair.

---

### **Week 11 — Performance + reliability**

- Assessment timeouts
- Retry logic
- Evidence decay (older evidence weighs less)
- Fallback rules

---

### **Week 12 — Polish + ship**

- Skill dashboard
- Exportable skill profile
- Demo script
- Documentation
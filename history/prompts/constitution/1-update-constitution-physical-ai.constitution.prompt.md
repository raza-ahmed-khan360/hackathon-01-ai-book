---
id: 1
title: Update constitution for Physical AI book
stage: constitution
date: 2025-12-25
surface: agent
model: Claude Haiku 4.5
feature: none
branch: master
user: User
command: /sp.constitution
labels: [constitution, spec-driven-development, project-setup]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
 - .specify/memory/constitution.md
tests:
 - none
---

## Prompt

```text
Project: Physical AI & Humanoid Robotics

Audience:
Advanced learners, AI engineers, robotics developers, and educators.

Core principles:
- Specification-first, AI-assisted authorship
- Technical accuracy and verifiability
- Reproducible systems and architectures
- Clear, structured instructional writing
- Production-ready deployment mindset

Scope & focus:
- Authoring a complete technical book on Physical AI & Humanoid Robotics
- Embodied intelligence and AI systems operating in the physical world
- Spec-driven content creation using Claude Code and Spec-Kit Plus
- Seamless integration of an in-book Retrieval-Augmented Generation (RAG) chatbot

Book standards:
- Format: Docusaurus documentation site
- Deployment: GitHub Pages
- Content authored and maintained via Spec-Kit Plus
- Writing and refinement via Claude Code
- British English throughout

Technical standards:
- All factual and architectural claims must be verifiable
- Prefer official documentation and primary technical sources
- Clear separation of content, specs, and infrastructure
- Reproducible code and configuration examples

Integrated RAG chatbot requirements:
- Embedded directly within the published book
- Built using OpenAI Agents / ChatKit SDKs
- Backend: FastAPI
- Vector store: Qdrant Cloud (Free Tier)
- Metadata/state: Neon Serverless Postgres
- Must support:
  - Book-wide questions
  - Section-specific queries
  - Queries limited strictly to user-selected text
- Responses must not exceed indexed book content

Constraints:
- Modular, spec-compatible structure
- Strong alignment between content and retrieval indexing
- Maintainable, automated deployment pipeline

Success criteria:
- Fully deployed book titled "Physical AI & Humanoid Robotics"
- Accurate, non-hallucinated RAG responses
- Clean spec-to-content traceability
- End-to-end technical coherence
```

## Response snapshot

The project constitution has been successfully updated for the Physical AI & Humanoid Robotics book project. The new constitution includes six core principles:

1. Specification-First, AI-Assisted Authorship
2. Technical Accuracy and Verifiability
3. Reproducible Systems and Architectures
4. Clear, Structured Instructional Writing
5. Production-Ready Deployment Mindset
6. Integrated RAG Chatbot Excellence

The constitution also includes sections on Technical Standards and Constraints, Development Workflow and Quality Gates, and Governance. The file is now version 1.0.0 with ratification date 2025-12-25.

## Outcome

- ✅ Impact: Updated project constitution to guide development of Physical AI & Humanoid Robotics book
- 🧪 Tests: none
- 📁 Files: Updated .specify/memory/constitution.md with new principles and governance
- 🔁 Next prompts: Continue with spec-driven development using new constitution
- 🧠 Reflection: Constitution now properly reflects the project's specific requirements and principles

## Evaluation notes (flywheel)

- Failure modes observed: none
- Graders run and results (PASS/FAIL): N/A
- Prompt variant (if applicable): N/A
- Next experiment (smallest change to try): Begin creating feature specifications based on the new constitution
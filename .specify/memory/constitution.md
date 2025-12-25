<!-- SYNC IMPACT REPORT
Version change: N/A → 1.0.0
Added sections: All principles and sections for Physical AI & Humanoid Robotics Book project
Removed sections: None (new constitution)
Modified principles: N/A (new constitution)
Templates requiring updates: ⚠ pending - .specify/templates/plan-template.md, .specify/templates/spec-template.md, .specify/templates/tasks-template.md
Follow-up TODOs: None
-->

# Physical AI & Humanoid Robotics Book Constitution

## Core Principles

### Specification-First, AI-Assisted Authorship
All content creation follows specification-driven approach with AI assistance; Content must be planned via specs before writing; Use Claude Code and Spec-Kit Plus for content generation and refinement

### Technical Accuracy and Verifiability
All factual and architectural claims must be verifiable from official documentation and primary technical sources; Prefer authoritative sources over secondary interpretations; All technical examples must be reproducible

### Reproducible Systems and Architectures
All code examples and system architectures must be reproducible with clear, step-by-step instructions; Configuration examples must work in clean environments; Clear separation of content, specs, and infrastructure

### Clear, Structured Instructional Writing
Content must follow clear pedagogical structure with learning objectives, examples, and practical applications; Use consistent terminology throughout; Maintain British English spelling and grammar standards

### Production-Ready Deployment Mindset
All examples and implementations must consider production deployment scenarios; Include operational concerns like monitoring, error handling, and maintainability; Focus on GitHub Pages deployment with automated pipelines

### Integrated RAG Chatbot Excellence
The embedded RAG chatbot must provide accurate responses strictly limited to indexed book content; Built with FastAPI backend, Qdrant Cloud vector store, and Neon Serverless Postgres for metadata; Must support book-wide, section-specific, and user-selected text queries

## Technical Standards and Constraints

Format: Docusaurus documentation site; Deployment: GitHub Pages; Content authored via Spec-Kit Plus; Writing refined via Claude Code; Prefer official documentation and primary technical sources; Modular, spec-compatible structure with strong alignment between content and retrieval indexing

## Development Workflow and Quality Gates

Content creation follows spec-driven approach using Spec-Kit Plus tools; All changes must maintain traceability from specs to content; Automated deployment pipeline required; Regular verification of RAG response accuracy against source content

## Governance

All content changes must verify compliance with specification requirements; Complexity must be justified with clear learning objectives; Use Docusaurus documentation guidelines for content structure; All PRs must verify content-index alignment for RAG functionality

**Version**: 1.0.0 | **Ratified**: 2025-12-25 | **Last Amended**: 2025-12-25
# 1. Philosophy and Architectural Principles  
1.1 Platform Vision and Operating Modes  
1.2 Aspect-Oriented Architecture Model  
1.3 Contracts-First Doctrine  
1.4 Functional Core / Imperative Shell  
1.5 Idempotency and State-Based Design  
1.6 Eventual Consistency as a First-Class Assumption  
1.7 Immutable Artifacts and Release Philosophy  

# 2. Operating Profiles and Deployment Philosophy  
2.1 Infrastructure-Automation-First Strategy (Ansible-first)  
2.2 Deployment Profiles (POC, Self-Host, SaaS, Enterprise-Isolated, Hybrid)  
2.3 Immutable Container Images and Version Locking  
2.4 Service Configuration and Environment Modeling  
2.5 Secrets and Configuration Management (placeholder)  
2.6 Upgrade, Rollback, and Migration Strategy  

# 3. Cross-Cutting Aspects  
3.1 Observability Aspect  
 3.1.1 Correlation Model  
 3.1.2 Structured Logging Contract  
 3.1.3 Metrics Contract  
 3.1.4 Audit Event Model  
3.2 Provisioning Aspect  
 3.2.1 Tenant Provisioning  
 3.2.2 App Provisioning  
3.3 Identity Aspect  
 3.3.1 Tenant-Scoped Users  
 3.3.2 Role Model  
 3.3.3 Bridge Entity Model  
3.4 Entitlements Aspect  
 3.4.1 Canonical Entitlement Schema  
 3.4.2 Runtime Evaluation Rules  
 3.4.3 Gating Conventions (UI + BFF)  

# 4. Product Layer  
4.1 Billing Product  
 4.1.1 Ledger and Ticket Model  
 4.1.2 Redemption and Entitlement Binding  
 4.1.3 Payment Processor Adapters  

# 5. Subsystems (Black-Boxable Components)  
5.1 Ergo Execution Subsystem  
 5.1.1 Message Lifecycle (Kafka SOR, RabbitMQ routing)  
 5.1.2 HTTP Gateway Patterns (Fire-and-Forget, Sync, Async 202)  
 5.1.3 Write Convergence and DB Writer Model  
 5.1.4 Substrate Variants (Remote, Local RMQ, Stdio, Single-Process Loop)  
5.2 AI Subsystem  
 5.2.1 Model Broker (Cloud + Local)  
 5.2.2 Vector Store Abstraction  
 5.2.3 STT / TTS Services  
5.3 Object Storage Abstraction  
 5.3.1 S3-Compatible Contract  
 5.3.2 Local/LAN Adapters  

# 6. Data Architecture  
6.1 System of Record (Kafka)  
6.2 Projection and Read Model Strategy  
6.3 Postgres as Primary Query Store  
6.4 JSONB as Semi-Structured Store  
6.5 SQLite and Lightweight Modes  
6.6 Event Streams as Tape  
6.7 Large Document Strategy (Metadata + Object Store)  
6.8 No ORM Policy and Data Accessor Pattern  

# 7. BFF / Experience Layer  
7.1 BFF Responsibility Model  
7.2 Specialized REST Dialect  
7.3 Request/Response Envelope Specification  
7.4 Error Envelope and Taxonomy  
7.5 Sync vs Async vs Fire-and-Forget Contracts  
7.6 Projection-Backed Reads  
7.7 Command Forwarding to Ergo  
7.8 Extension Model (Stock vs Plugin BFF Behavior)  

# 8. Frontend Platform  
8.1 SSR-First Architecture  
8.2 Microfrontend Model  
8.3 Plugin Runtime Evolution Path  
8.4 SDK Contract (Frontend ↔ BFF)  
8.5 Shell Runtime Responsibilities  
8.6 Immutable Build and Composition Strategy  
8.7 Mobile (React Native) and Desktop (Tauri) Targets  

# 9. Synchronization Model  
9.1 Real-Time Channel (Speed)  
9.2 Delta Channel (Corrective Overlay)  
9.3 Reconciliation Channel (Full ETL Overlay)  
9.4 Eventual Consistency UX Contract  

# 10. Coding Standards and Development Strategy  
10.1 Object-Oriented Core with Injected Pure Functions  
10.2 Runtime-Wraps-Behavior Pattern (Ergo Inversion)  
10.3 Composition Over Inheritance  
10.4 Controlled Reflection Usage  
10.5 Canonical Idioms and Contractions  
10.6 Helper Function Policy  
10.7 Guard Clauses and Branching Discipline  
10.8 Async-First Runtime Rule  
10.9 Strong Typing from Day One (TypeScript + mypy)  
10.10 Error Handling Strategy  
 10.10.1 Exception Specificity  
 10.10.2 Boundary Translation Rules  
 10.10.3 Root Logging Policy  
10.11 Module Structure and File Organization (One Public Class per File)  

# Addenda  
AA_ADDENDUM_01 — Homelab, Amendments, Implementation Maturity  
AA_ADDENDUM_02 — Ergo Formal Spec (Tenets, Principles, Philosophy)  
AA_ADDENDUM_03 — Hot Potato Formal Spec (Tenets, Principles, Philosophy)  
AA_ADDENDUM_04 — Homelab Ref Impl Notes (Consistency Only; Does Not Amend Plan)  
AA_ADDENDUM_05 — Ergo Additional Lineage (Begroupd, Bento; Reference Only; Does Not Amend Ergo Spec)  
AA_ADDENDUM_06 — Development Standards (Project-Independent; Philosophy, Principles, Implementation Norms)  

# 11. Contracts and Versioning  
11.1 Schema Versioning Strategy  
11.2 Backward Compatibility Rules  
11.3 SDK Versioning and Microfrontend Compatibility  
11.4 Envelope Versioning  

# 12. Testing Philosophy  
12.1 Pure Function Testing  
12.2 Runtime Integration Tests  
12.3 Projection Consistency Tests  
12.4 Contract Tests Across Boundaries  
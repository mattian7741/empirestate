# 8. Frontend Platform

This section defines the architectural model, composition strategy, and development constraints for the frontend layer across web, mobile, and desktop targets.

The frontend is not a web app. It is a portable experience runtime with multiple render targets.

---

## 8.1 Architectural Model

The frontend platform consists of:

- Shell Runtime
- Platform SDK
- Microfrontend Modules
- Build and Composition System

The platform must support:

- SSR-first web
- CSR hydration
- Mobile (React Native)
- Desktop (Tauri or equivalent)
- Immutable build artifacts
- Version-locked releases

The frontend must remain portable across deployment profiles without requiring cloud-specific infrastructure.

---

## 8.2 Shell Runtime

The Shell Runtime is the host environment responsible for:

- SSR rendering and hydration
- Routing orchestration
- Module loading
- Global context management
- Theme injection
- Entitlement gating
- Error boundary handling
- Instrumentation hooks

The Shell owns:

- Principal context (authenticated user)
- Scope context (tenant + app domain)
- Entitlement resolution snapshot
- Navigation state
- Capability interfaces (network, storage, telemetry, crypto)

The Shell must not contain domain-specific logic.

---

## 8.3 Microfrontend Model

Microfrontends are leaf modules.

They define:

- Routes
- Navigation entries
- UI components
- Asset bundles
- Domain-specific workflows

They do not:

- Access backend services directly
- Bypass the SDK
- Access shell internals directly

Microfrontends compile against a single version of the Platform SDK.

Version skew is resolved by shipping a new immutable build artifact.

---

## 8.4 Composition Strategy

Composition occurs at build time by default.

A single deployable artifact contains:

- Shell Runtime
- Platform SDK
- Microfrontend bundles
- Static assets

Runtime dynamic loading may be introduced later but is not the default.

Microfrontends must declare a manifest including:

- Identity (name, version)
- Compatible SDK version
- Routes
- Navigation metadata
- Required capabilities
- Required entitlements
- Optional server extension declarations

The Shell validates manifest compatibility at startup.

---

## 8.5 SSR-First Strategy

Server-side rendering is the default web rendering strategy.

The Shell must:

- Render page frame server-side
- Support optional SSR for microfrontend route content
- Hydrate client state cleanly
- Preserve compatibility with CSR-only modules

SEO is treated as an overlay concern and must not alter core architecture.

SSR must not assume public unauthenticated access.

---

## 8.6 SDK Contract

The Platform SDK defines the only contract between microfrontends and the platform.

The SDK provides:

- BFF client abstraction
- Typed request/response envelopes
- Entitlement evaluation helpers
- Navigation registration APIs
- Instrumentation hooks
- Capability interfaces (storage, network, crypto)

The SDK must:

- Be versioned
- Be typed
- Avoid exposing shell internals
- Remain stable across minor platform evolution

Shared code does not replace contracts.

---

## 8.7 BFF Interaction Model

Microfrontends communicate exclusively through the SDK.

The SDK communicates with the BFF.

Microfrontends must not:

- Call backend services directly
- Access projection databases
- Depend on transport details

BFF interaction supports:

- Synchronous operations
- Async job-based operations
- Fire-and-forget commands

All responses follow the canonical envelope specification.

---

## 8.8 Synchronization Model

The frontend supports three synchronization strategies:

1. Real-time channel
   - Speed-focused
   - Non-authoritative
   - May be corrected later

2. Delta synchronization
   - Cursor-based corrective overlay
   - Confirms and reconciles real-time events

3. Reconciliation
   - Authoritative full refresh
   - Used for gap correction and recovery

Microfrontends must tolerate eventual consistency.

The SDK must provide consistent primitives for:

- Subscribing to updates
- Fetching deltas
- Triggering reconciliation

---

## 8.9 Theming and Design System

The design system is token-based and shared logically across microfrontends.

Rules:

- Structural styles are shared.
- App-specific styling is isolated.
- Tokens define brand and theme.
- Microfrontends may override tokens but not structural contracts.

The design system must be:

- Skinnable
- Portable across web, mobile, and desktop
- Independent from domain logic

---

## 8.10 Cross-Platform Strategy

The frontend must target:

- Web (SSR + CSR)
- Mobile (React Native)
- Desktop (Tauri)

Shared elements:

- Domain logic
- SDK contract
- Type definitions
- Envelope handling
- Synchronization logic

Platform-specific layers implement:

- Rendering primitives
- Storage adapters
- Transport adapters

No business logic should depend on browser-only features.

---

## 8.11 Immutable Build and Versioning

Every build produces a version-locked artifact.

Rules:

- No runtime mutation of module code.
- No mixing incompatible SDK versions within a single build.
- Rollback via artifact replacement only.

Microfrontends target exactly one SDK version per release.

Compatibility shims may exist in the Shell but are explicit and versioned.

---

## 8.12 Extension Evolution Path

The initial model is microfrontend composition.

As patterns stabilize:

- Common workflows migrate into shared modules.
- Reusable behaviors may become plugins.
- Contracts become more rigid over time.

Reflection and dynamic registration may be used during convergence phases but should be replaced with stable declarative contracts as the system matures.

---

## 8.13 Frontend Error Discipline

All frontend errors must:

- Follow canonical error envelope
- Preserve correlation identifiers
- Distinguish retryable vs non-retryable
- Avoid leaking internal stack traces

Unhandled exceptions must be caught at the Shell boundary and logged with full context.

---

## 8.14 Async-First Execution

All frontend IO must be async.

Domain logic may remain synchronous.

The SDK must abstract async transport details from microfrontends where possible.

Async must not leak into pure domain logic.

---

This section defines the frontend as a composable runtime platform rather than a monolithic application. All implementation choices must remain consistent with these constraints.
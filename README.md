# openwisp-wifi-pages-demo

## Proposal: WiFi Login Pages Modernization
**Google Summer of Code 2026** | **OpenWISP**

**Proposal by Avidu Witharana**  
avidu@ieee.com | +94 71 017 1111 | Sri Lanka (UTC+5:30)  
Project size: 175 hours | Difficulty: Medium | Mentors: Federico Capoano, Sankalp, Gagan Deep

---

### 1. My Details
| Field | Details |
| :--- | :--- |
| **Full Name** | Avidu Witharana |
| **Date of Birth** | 15th January 2003 |
| **Country / Region** | North Western Province, Sri Lanka |
| **Email** | avidu@ieee.com |
| **GitHub** | [github.com/avidzcheetah](https://github.com/avidzcheetah) |
| **Phone** | +94 71 017 1111 |
| **UTC Availability** | 04:30 – 19:30 UTC (I'm UTC+5:30, Sri Lanka) |

### 2. About Myself
#### Background
I'm a third-year Computer Engineering student at the Department of Computer Engineering, University of Jaffna, Sri Lanka. I'm also pursuing a BSc in Information Technology and Cybersecurity at PSB University Cambodia (Sep 2023 – Present). Networking and security aren't just coursework for me — they run through most of my practical work.

Last year I designed and simulated a full network topology for the Faculty of Engineering at University of Jaffna. That involved subnetting, routing protocol configuration, and testing everything in simulation before proposing it for actual deployment. That project gave me a concrete feel for how network infrastructure works end-to-end — not just the software layer, but protocols and physical layout too. It's the kind of context that makes OpenWISP's scope feel very real to me.

On the frontend side I've shipped a handful of React applications. **JamHub** is a real-time collaborative music platform built with WebSockets and WebAudio — low-latency audio sync across multiple browsers was the hard part. **Mew** is a real-time AI chatbot integrating both ChatGPT and Gemini with message streaming and Markdown rendering. My personal portfolio uses React and TypeScript. The **Lab Rescheduling System** I built for my faculty uses React, Next.js, and PHP with role-based dashboards and automated email notifications. These aren't demos — they're deployed and used.

I'm currently an intern software developer at Enigma Solutions (Pvt) Ltd (July 2025 – present), and I chair the IEEE RAS Student Branch Chapter at University of Jaffna, having previously served as Vice Chairman. That combination of technical work and community coordination is something I take seriously.

#### Open Source Contributions
I haven't made contributions to OpenWISP yet, but I've been studying the `openwisp-wifi-login-pages` codebase since early 2026. I've published several open-source tools of my own: **RepoSpector** (GitHub API analysis tool), **Crawler** (web vulnerability checker using Bash), and a real-time ML intrusion detection system combining Random Forest and Autoencoder models for zero-day detection. I'm working on my first contribution to OpenWISP before this proposal is finalized.

OpenWISP contributions: [links to be added before submission]

#### OpenWrt Experience
I don't have hands-on OpenWrt experience, but I understand its role in OpenWISP's architecture — it's the router-side agent. This specific project, WiFi Login Pages, is purely a Node.js / React frontend that talks to OpenWISP RADIUS. OpenWrt knowledge isn't a hard requirement here. I can run the full development environment locally using Docker.

#### Motivation
I came across OpenWISP while researching captive portal systems during a university network project. What kept me reading was the problem space — public WiFi management is real infrastructure, not a toy project. When I dug into issue #918 (status component complexity) and issue #272 (redirect logic in OrganizationWrapper), I recognized these immediately as the kind of architectural problems I'd seen in my own React projects: a component that started focused and grew outward until nobody wanted to touch it. I want to fix this properly, not patch over it.

---

### 3. My GSoC Project
| Title | WiFi Login Pages Modernization |
| :--- | :--- |
| **Mentors** | Federico Capoano, Sankalp, Gagan Deep |
| **Project Size** | 175 hours |
| **Difficulty** | Medium |

#### Measurable Outcomes
1. **Status component refactored**: auth, verification, payment, session, captive portal, and UI split into focused sub-components. Test coverage maintained.
2. **OrganizationWrapper cleaned up**: redirect logic moved to individual components. Wrapper reduced to a route list only.
3. **Unified header**: one HTML structure replaces two separate desktop / mobile blocks. Responsive CSS handles layout.
4. **RFC 8908 support**: Captive Portal API implemented as an opt-in, per-org YAML-configurable feature with 2-second default timeout.
5. **React 19 upgrade**: Enzyme replaced by React Testing Library, all deps updated, full test suite passing.
6. **Documentation**: React migration guide, captive portal API usage docs, and a short demo video for YouTube.

#### Understanding the Problem
The app started focused. Then it grew. Over time, every new feature — payment flows, SMS verification, social login — landed in the same two files: `status.js` and `OrganizationWrapper`. Nobody planned it that way. It just happened.

**Issue #918 — The Status Component**  
Open `client/components/status/status.js` and you'll find a component doing at least six distinct jobs:
- Validating the user's auth token
- Checking whether email or phone verification is complete
- Managing payment state and paid plans
- Tracking the active RADIUS session
- Running captive portal detection
- Rendering the entire status UI

That's not a component — that's a module. When you need to fix a bug in the payment flow, you read session logic first. When you write a test, you mock the universe before testing one thing. The cognitive load is high, and test coverage suffers for it.

**Issue #272 — Redirect Logic in OrganizationWrapper**  
`OrganizationWrapper` re-renders every time `setLoading()` is called. That's by design. The problem is that redirect logic lives in the wrapper, so every loading state change triggers the redirect checks, which trigger HTTP requests. When a user is being redirected to a payment gateway, this causes a race condition. The fix is simple in principle: move the redirects into the components that own them.

**Header Duplication — Issue #314**  
There are two separate header HTML blocks: one for desktop, one for mobile. Every change to the header has to be made twice. This is pure maintenance tax. One HTML structure with responsive CSS should handle both.

**React Version and Testing Framework**  
The project is on React 17.x. Enzyme, used for tests, hasn't kept up with modern React. It's officially deprecated. Staying here means falling further behind as the ecosystem moves on. React 19 brings better async handling, improved hydration, and the modern patterns we'd want to use in the refactored components.

**RFC 8908 — Captive Portal API**  
Modern devices — iOS, Android, macOS — query a well-known URL when they connect to a network to determine if they're behind a captive portal. This is RFC 8908. Right now, `openwisp-wifi-login-pages` doesn't support it. Supporting it as an opt-in feature means devices that implement the standard can get instant status detection instead of relying on the existing fallback mechanism.

---

### Proposed Solution
#### Task 1 — Refactor the Status Component (#918)
I'll split `status.js` into six focused pieces. Each handles exactly one concern:

| Component / Hook | Responsibility |
| :--- | :--- |
| AuthLayer | Calls `/api/v1/{org}/account/token/validate/`, manages auth state |
| VerificationLayer | Checks phone/email verification status, routes to correct verify component |
| PaymentLayer | Reads paid plan state, handles payment redirects |
| SessionLayer | Tracks active RADIUS session data |
| CaptivePortalLayer | Runs portal detection + RFC 8908 check (see Task 4) |
| StatusView | Pure rendering — reads from context/props, no side effects |

The `StatusView` component becomes a pure function — it takes state as props and renders. No API calls, no redirects. This makes it trivially testable.

#### Task 2 — OrganizationWrapper Redirect Logic (#272)
`OrganizationWrapper` becomes a thin route list. Each component owns its own redirect logic.

#### Task 3 — Header Unification (#314)
Single HTML structure, responsive CSS handles the rest. No new dependencies. Customization now requires one change instead of two.

#### Task 4 — RFC 8908 Captive Portal API (#947)
RFC 8908 defines a JSON endpoint that captive portals expose at a known URL. Devices query it to determine their internet access state.

#### Task 5 — React 19 Upgrade (#870)
The upgrade happens in stages to minimize risk:
1. React 18.3 first — This surfaces all deprecation warnings.
2. Run `react-codemods` — Automated fixes for deprecated lifecycle methods, string refs, etc.
3. Migrate Enzyme to RTL incrementally — One PR per refactoring task migrates the corresponding tests.
4. Upgrade to React 19 — With deprecations already resolved and tests migrated.
5. Full test suite pass — Fix any remaining regressions.

---

### Technology Choices — Why These Tools
| Choice | Reason |
| :--- | :--- |
| **React 19** | The app is already React. Upgrading avoid a rewrite and brings better async handling. |
| **React Testing Library** | Enzyme is deprecated. RTL tests interact with components as users do, making them resilient to implementation changes. |
| **CSS Flexbox/Grid** | No new dependencies needed. Pure structural refactor. |
| **AbortController** | Standard browser API for cancellable fetch. No extra dependency needed. |
| **Incremental RTL migration** | Big-bang migrations fail. Migrating tests alongside refactoring keeps the codebase consistent. |

### Prototype
I've built a working prototype of the refactored component architecture and the RFC 8908 detection module.
**Demo**: [openwisp-wifi-pages-demo.vercel.app](https://openwisp-wifi-pages-demo.vercel.app)  
**Repository**: [github.com/avidzcheetah/openwisp-wifi-pages-demo](https://github.com/avidzcheetah/openwisp-wifi-pages-demo)

---

### Project Schedule
| Period | Tasks | Deliverables | Hours |
| :--- | :--- | :--- | :--- |
| **Pre-GSoC** (Now – May 26) | Codebase study, first contributions, prototype | Merged PRs, prototype repo | — |
| **Wk 1–2** (May 27 – Jun 9) | Dev env setup, architecture design | Architecture doc, mentor sign-off | 20 |
| **Wk 3–4** (Jun 10 – Jun 23) | Status refactor: Auth + Verification Layer | PR #1 + RTL tests | 25 |
| **Wk 5–6** (Jun 24 – Jul 7) | Status refactor: Payment, Session, Captive Portal Layers, StatusView | PR #2 + RTL tests | 25 |
| **Wk 7** (Jul 8 – Jul 14) | OrganizationWrapper redirect logic | PR #3 + tests | 15 |
| **MID-TERM** | Expected: status component refactored, OrganizationWrapper cleaned up | | |
| **Wk 8** (Jul 15 – Jul 21) | Header unification (responsive CSS) | PR #4 + cross-browser test | 15 |
| **Wk 9–10** (Jul 22 – Aug 4) | RFC 8908 Captive Portal API feature | PR #5, YAML config, tests, docs | 25 |
| **Wk 11–12** (Aug 5 – Aug 18) | React 19 Upgrade (staged) | PR #6, all tests passing | 25 |
| **Wk 13** (Aug 19 – Aug 25) | Documentation & Demo Video | Merged docs PR, YouTube video | 10 |
| **Wk 14** (Aug 26 – Sep 1) | Final submission & Buffer | Final submission | 15 |
| **TOTAL** | | | **175** |

---

### 4. After GSoC
#### Continuing to Collaborate
Yes — and not just for maintenance. The networking and captive portal space genuinely interests me. OpenWISP is one of the few open-source tools that takes network management seriously at the community level.

#### Maintaining the Implementation
I'll stay reachable for bug reports and follow-up PRs. The refactoring work especially needs someone who knows the design decisions.

#### Freelance Work
Yes. I'm already working as a contract developer, so professional delivery timelines are familiar. GSoC experience combined with active maintainership would make me genuinely useful for OpenWISP-related client work.

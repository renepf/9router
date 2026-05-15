# 9Router iOS App - Full Lifecycle Strategy

## Executive Summary

Port 9Router (AI API proxy/router) to an iOS companion + cloud relay architecture.
The iOS app manages providers, monitors usage, and controls routing config.
A lightweight cloud relay handles actual HTTP proxying for desktop CLI tools.

---

## Phase Map

| Phase | Name | Duration | Milestone |
|-------|------|----------|-----------|
| 0 | Foundation & Planning | Week 1-2 | GitHub Project board live, repo scaffolded |
| 1 | Core Swift SDK | Week 3-6 | Provider models, API translation, SQLite layer |
| 2 | Local Proxy Server | Week 5-8 | Vapor-based /v1/* endpoints working on-device |
| 3 | Dashboard UI | Week 7-10 | SwiftUI dashboard with Charts, provider management |
| 4 | OAuth & Auth | Week 9-11 | ASWebAuthenticationSession flows for all providers |
| 5 | Cloud Relay | Week 10-13 | Lightweight relay server, iOS push config |
| 6 | Integration Testing | Week 13-14 | End-to-end with Claude Code, Codex, Cursor |
| 7 | Beta & TestFlight | Week 14-16 | TestFlight distribution, feedback loop |
| 8 | Marketing & Launch | Week 16-18 | App Store submission, landing page, socials |
| 9 | Post-Launch & Exit | Week 18+ | Iteration, monetization, exit strategy |

---

## Architecture Decision

### Why NOT a standalone iOS proxy:
- iOS kills background HTTP servers when app is suspended
- Desktop CLI tools can't reach iPhone's localhost
- Network Extension (VPN entitlement) adds massive complexity + Apple review risk

### Chosen: Hybrid Architecture (Option C)

```
Desktop CLI Tools (Claude Code, Codex, Cursor...)
       |
       v
[Cloud Relay] <--- config sync ---> [iOS App]
       |                                |
       v                                v
[AI Providers]                   [Dashboard + Config]
(OpenAI, Claude, etc.)           (SwiftUI, Keychain, Push)
```

- **iOS App**: Management plane (SwiftUI dashboard, OAuth, config, push notifications)
- **Cloud Relay**: Data plane (lightweight proxy, receives routing config from iOS)
- **Desktop**: Points to relay URL instead of localhost

### Alternative: Local Network Mode
When iOS + Mac on same WiFi, the iOS app CAN serve as direct proxy via Bonjour/mDNS.
Falls back to cloud relay when not co-located.

---

## Tool & Workflow Strategy

### Repository Structure

```
9router/                    # Main repo (existing Next.js)
worktree: ios/              # iOS app (Swift Package + Xcode)
worktree: relay/            # Cloud relay (Node.js, lightweight)
worktree: docs/             # Marketing site, App Store assets
```

### GitHub Infrastructure

| GitHub Feature | Purpose |
|---|---|
| **Issues** | Task tracking, bug reports, feature requests |
| **Discussions** | Architecture decisions, community feedback, Q&A |
| **Projects (Board)** | Kanban: Backlog -> In Progress -> Review -> Done |
| **Milestones** | One per phase (0-9) with due dates |
| **Labels** | `ios`, `relay`, `core`, `ui`, `auth`, `infra`, `marketing`, `p0-critical`, `p1-high`, `p2-medium` |
| **Releases** | Semantic versioning, TestFlight builds |
| **Wiki** | API docs, architecture diagrams |

### Zellij Layout

```
Tab 1: "ios-dev"        - iOS app development (Xcode build, swift test)
Tab 2: "relay-dev"      - Cloud relay development (node, docker)
Tab 3: "9router-main"   - Main repo reference, git operations
Tab 4: "testing"        - Integration tests, simulators
Tab 5: "ops"            - GitHub CLI, project management, deployments
```

### Claude Code Workflow

| Tool | Usage |
|---|---|
| **Worktrees** | Isolate ios/, relay/, docs/ work without branch switching |
| **Subagents (Explore)** | Codebase research across main repo for porting logic |
| **Subagents (Plan)** | Architecture decisions for each phase |
| **Subagents (general)** | Parallel task execution (e.g., write tests while coding) |
| **Background agents** | Long-running builds, test suites |
| **gh CLI** | Issue/PR creation, project board updates, releases |

### Execution Pattern per Phase

```
1. /gsd:plan-phase         -> Create PLAN.md
2. gh issue create          -> Break into GitHub Issues
3. Worktree + Zellij tab    -> Isolated dev environment
4. Subagents (parallel)     -> Code + test simultaneously
5. /gsd:verify-work         -> UAT checklist
6. gh pr create             -> PR for review
7. /gsd:progress            -> Update project board
```

---

## Phase 0: Foundation & Planning (Week 1-2)

### GitHub Setup
- [ ] Create GitHub Project board (Kanban)
- [ ] Create Milestones for Phase 0-9
- [ ] Create label taxonomy
- [ ] Pin architecture discussion
- [ ] Create issue templates (bug, feature, task)

### Repo Setup
- [ ] Create `ios` branch + worktree
- [ ] Initialize Swift Package (9RouterSDK)
- [ ] Initialize Xcode project (9Router iOS)
- [ ] Create `relay` branch + worktree
- [ ] Initialize relay project (Node.js/Bun)

### Architecture Docs
- [ ] ADR-001: Hybrid vs Standalone architecture
- [ ] ADR-002: Cloud relay technology choice
- [ ] ADR-003: Data sync protocol (iOS <-> Relay)
- [ ] ADR-004: Auth token storage (Keychain strategy)

---

## Phase 1: Core Swift SDK (Week 3-6)

### Port from 9Router
- [ ] Provider definitions (src/shared/constants/providers.js -> Swift enums)
- [ ] Model definitions + aliases
- [ ] API format translation (OpenAI <-> Claude <-> Gemini)
- [ ] SQLite storage layer (Swift Data or GRDB)
- [ ] Token refresh logic
- [ ] Combo/fallback routing engine
- [ ] Usage tracking models

### Key Files to Port
| Source (JS) | Target (Swift) |
|---|---|
| `src/shared/constants/providers.js` | `Sources/9RouterSDK/Providers/` |
| `src/sse/handlers/chat.js` | `Sources/9RouterSDK/Routing/ChatRouter.swift` |
| `src/sse/services/tokenRefresh.js` | `Sources/9RouterSDK/Auth/TokenRefresh.swift` |
| `src/lib/db/repos/*` | `Sources/9RouterSDK/Storage/` |
| `src/shared/constants/models.js` | `Sources/9RouterSDK/Models/` |

---

## Phase 2: Local Proxy Server (Week 5-8)

### Vapor HTTP Server
- [ ] /v1/chat/completions endpoint
- [ ] /v1/embeddings endpoint
- [ ] /v1/models endpoint
- [ ] SSE streaming via AsyncSequence
- [ ] Request logging middleware
- [ ] API key validation middleware

### Local Network Mode
- [ ] Bonjour/mDNS service advertisement
- [ ] Auto-discovery from Mac companion tool
- [ ] TLS with self-signed cert (Security framework)

---

## Phase 3: Dashboard UI (Week 7-10)

### SwiftUI Views
- [ ] TabView: Dashboard | Providers | Logs | Settings
- [ ] Dashboard: Usage charts (Swift Charts), active providers, quota gauges
- [ ] Providers: List, add (OAuth), test, delete, priority reorder (drag)
- [ ] Logs: Real-time request log with filtering
- [ ] Settings: Combo strategy, auth, relay config
- [ ] Widgets: WidgetKit for home screen quota display
- [ ] Live Activity: Active routing session on lock screen

---

## Phase 4: OAuth & Auth (Week 9-11)

### ASWebAuthenticationSession Flows
- [ ] Claude OAuth (PKCE)
- [ ] GitHub/Copilot OAuth
- [ ] Codex OAuth
- [ ] Cursor OAuth
- [ ] Gemini OAuth
- [ ] API key entry for non-OAuth providers

### Security
- [ ] Keychain storage for all tokens
- [ ] Biometric lock (FaceID/TouchID) for app access
- [ ] Certificate pinning for provider API calls

---

## Phase 5: Cloud Relay (Week 10-13)

### Relay Server
- [ ] Lightweight Node.js/Bun proxy
- [ ] Receives routing config from iOS app via WebSocket
- [ ] OpenAI-compatible /v1/* endpoints
- [ ] Deploy options: Fly.io, Railway, Cloudflare Workers
- [ ] Auth: iOS app generates relay access tokens

### Sync Protocol
- [ ] WebSocket connection (iOS -> Relay)
- [ ] Config push: provider list, priorities, combos, aliases
- [ ] Usage pull: relay reports usage stats back to iOS
- [ ] Push notifications: quota warnings, provider failures

---

## Phase 6: Integration Testing (Week 13-14)

### Test Matrix
| CLI Tool | Direct (Local Net) | Via Relay |
|---|---|---|
| Claude Code | [ ] | [ ] |
| Codex | [ ] | [ ] |
| Cursor | [ ] | [ ] |
| Cline | [ ] | [ ] |
| OpenCode | [ ] | [ ] |

### Automated Tests
- [ ] XCTest suite for SDK
- [ ] Integration tests with mock providers
- [ ] Performance benchmarks (latency overhead)
- [ ] Relay load testing

---

## Phase 7: Beta & TestFlight (Week 14-16)

- [ ] TestFlight internal build
- [ ] Beta tester recruitment (GitHub Discussions + Discord)
- [ ] Feedback collection via GitHub Issues (label: `beta-feedback`)
- [ ] Crash reporting (Sentry/Firebase Crashlytics)
- [ ] Iterate on top 5 feedback items

---

## Phase 8: Marketing & Launch (Week 16-18)

### Pre-Launch
- [ ] App Store listing (screenshots, description, keywords)
- [ ] Landing page update (9router.com/ios)
- [ ] Blog post: "9Router Goes Mobile"
- [ ] Product Hunt launch
- [ ] Hacker News Show HN
- [ ] Twitter/X announcement
- [ ] Reddit (r/programming, r/MachineLearning, r/iOSProgramming)
- [ ] GitHub release with changelog

### App Store Optimization
- Keywords: AI router, token saver, API proxy, Claude, GPT, coding assistant
- Category: Developer Tools
- Screenshots: Dashboard, provider management, usage charts

### Pricing Strategy
| Tier | Price | Features |
|---|---|---|
| Free | $0 | Local network mode, 3 providers, basic dashboard |
| Pro | $4.99/mo | Cloud relay, unlimited providers, widgets, push alerts |
| Lifetime | $29.99 | Everything, forever |

---

## Phase 9: Post-Launch & Exit Strategy (Week 18+)

### Growth
- [ ] Community contributions (open-source SDK)
- [ ] Plugin system for custom providers
- [ ] Mac Catalyst / macOS native app
- [ ] Apple Watch complication (quota at a glance)
- [ ] Shortcuts integration (Siri: "switch to free tier")

### Exit Scenarios

| Scenario | Strategy |
|---|---|
| **Acqui-hire** | AI tooling company (Cursor, Anthropic, etc.) buys team + product |
| **Asset sale** | Sell codebase + user base to competitor |
| **Open source** | Release under MIT, community maintains |
| **Sunset** | 6-month notice, export tools, relay shutdown plan |
| **Scale** | Raise seed round, expand to Android + desktop native |

### Monetization Diversification
- Relay hosting fees (usage-based)
- Enterprise tier (team management, SSO, audit logs)
- Affiliate commissions from AI providers
- Premium provider integrations

---

## GitHub Project Board Structure

### Columns
1. **Backlog** - All future work
2. **This Sprint** - Current 2-week sprint
3. **In Progress** - Actively being worked on
4. **In Review** - PR open, awaiting review
5. **Done** - Merged + verified

### Automation
- Issue created -> Backlog
- PR opened -> In Review
- PR merged -> Done
- Issue assigned -> In Progress

---

## Risk Register

| Risk | Impact | Mitigation |
|---|---|---|
| App Store rejection (proxy concerns) | HIGH | Position as "AI dashboard", not "proxy". No VPN entitlement needed. |
| Background execution limits | HIGH | Cloud relay handles persistence. iOS is management plane only. |
| OAuth provider changes | MEDIUM | Abstract OAuth behind protocol. Quick iteration via TestFlight. |
| Relay hosting costs | MEDIUM | Usage-based pricing covers relay. Self-host option available. |
| Competition (similar apps) | LOW | First-mover advantage. Integration with 9Router ecosystem. |
| Apple API deprecation | LOW | Use only stable APIs (URLSession, SwiftUI, Keychain). |

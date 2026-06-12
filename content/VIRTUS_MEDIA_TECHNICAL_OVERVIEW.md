# Virtus Media Platform — Technical Overview

**Audience:** Senior developer evaluating project takeover
**Scope:** Product context, architecture, workflows, risks, and first-steps checklist
**Not included:** Credentials, secrets, private access details, admin passwords, or clone instructions

---

## 1. Purpose

This document gives a senior developer enough product, architecture, workflow, and operational context to evaluate whether they can take over development and begin shipping features quickly if engaged.

It does not include credentials, private system access details, or sensitive deployment secrets.

---

## 2. Product Summary

Virtus Media is an **investor relations (IR) campaign management platform**. It supports managed media campaigns between:

- **Clients:** Public companies / IR teams
- **Distribution partners:** Creators (influencers), publishers, and UGC/media agencies

Virtus Media Group is the **operational middle layer** — closing deals, creating campaigns, assigning work, reviewing submitted performance data, generating reports, and delivering those reports to clients.

The current product is **~50% software, ~50% professional service**. Admins are the operational center.

---

## 3. Operating Model

### What This Platform Is

- Managed-service campaign operations software
- Internal admin tooling for Virtus Media Group
- Campaign coordination system for companies, creators, and agencies
- Reporting and performance-management layer
- Purpose-built for high-touch IR media campaigns sold and managed by Virtus

### What This Platform Is Not (Current Phase)

- Not a self-serve affiliate marketplace
- Not a system where companies post campaigns for creators to browse
- Not a system where creators freely claim available campaigns
- Not designed to remove Virtus from the transaction
- Not currently dependent on automated social API reporting
- Not intended to expose direct company-to-creator relationships that bypass Virtus

---

## 4. Current Managed-Service Rationale

Campaigns are typically **$50,000+**, often six figures or larger. These are high-value, relationship-driven engagements.

Virtus must remain the managed-service layer between IR clients and the creator/publisher/agency supply side. The platform should prevent companies from using it to discover and directly contact creators or publishers — this is **by design**, protecting Virtus's business model from client leakage.

---

## 5. Future Marketplace / Affiliate-Style Artifacts

> **Important:** Codebase artifacts for a possible future marketplace phase exist. They do not represent the current operating model.

The codebase contains routes, components, and API endpoints that suggest a future state where the platform behaves more like an affiliate marketplace:

| Artifact | Location | Notes |
|---|---|---|
| `campaigns/available` page | `/dashboard/[tenant]/(virtus)/campaigns/available/` | Browse/apply UI for influencers |
| `POST /api/campaigns/[id]/apply` | `/app/api/campaigns/[id]/apply/` | Influencer self-application to campaigns |
| `approve-application` / `reject-application` | `/app/api/campaigns/[id]/` | Admin approval flow for applications |
| Connections system | `/collections/connections/` | Bilateral influencer-company relationship records |
| Register page | `/app/(app)/register/` | Self-service registration route |

These features may be partially built, present in the route tree, linked in parts of the UI, accessible in staging, or reachable under some authenticated role conditions. Runtime access should be verified. They should be **audited to confirm they do not expose company-to-creator discovery or self-serve workflows** that contradict the current managed-service model.

---

## 6. Future Marketplace Business Hypothesis

The affiliate-style marketplace use case is **not yet proven** for this market.

If it becomes applicable, it would likely target:

- Low-budget campaigns (~$5,000)
- Smaller influencers/publishers willing to accept lower-touch work
- Campaigns that don't justify Virtus admin time at the same level as core managed campaigns

A developer taking over should distinguish between:

1. **Current managed-service workflows** — must work reliably now
2. **Hidden/partial marketplace artifacts** — should not leak into current UX
3. **Future-phase experiments** — may need to be removed, completed, or feature-flagged

---

## 7. Tenant / User Types

| Type | Role Value | Summary |
|---|---|---|
| Virtus Super Admin | `super-admin` (global) | Full platform access; creates tenants, manages all data |
| Tenant Admin | `tenant-admin` | Admins within a tenant (e.g., Virtus operator for a client workspace) |
| Campaign Manager | `campaign-manager` | Creates/manages campaigns within their tenant |
| Company / IR Client | `client` | Views timelines and receives campaign reports |
| Individual Creator | `influencer` | Accepts assigned campaigns, submits metrics, receives direct payment |
| UGC / Media Agency | `tenant` with `isAgency: true` | Manages creator accounts, accepts campaigns, receives agency-level payout |
| Agency-Managed Creator | `influencer` under agency | Executes work; paid by agency outside the platform |

**Tenant flags:** Each Tenant record has boolean flags — `isCompany`, `isInfluencer`, `isAgency` — to identify its type. Agencies can `managedBy` another tenant (their managing agency).

---

## 8. Core Workflow

1. Virtus sales closes a deal with a company **outside the platform**
2. Deal defines budget, scope, deliverables, and timeline
3. **Virtus admin creates the campaign** in the platform (`status: pending_payment`)
4. Company payment to Virtus is confirmed outside the current platform flow; an admin/system update moves the campaign toward `status: available`
5. Admin assigns or invites individual creators and/or agencies (`Collaboration` record created)
6. Creators/agencies accept the collaboration (`status: accepted → in-progress`)
7. Work is executed externally (social posts, newsletters, publisher placements, etc.)
8. Creators/agencies manually submit `CampaignDeliverable` records with URLs and metrics
9. System auto-advances collaboration status based on deliverable completion
10. Admin reviews and verifies metrics
11. Admin generates a client-facing performance report
12. Company views or receives the report through the platform
13. Funds are released to creators/agencies (dual approval: influencer + admin)

---

## 9. Current Phase Constraints

- Companies do **not** create campaigns directly
- Companies do **not** interact directly with creators or agencies
- Creators do **not** browse a marketplace of available company campaigns
- Agencies manage creators within their own account
- Performance metrics are **manually entered** — no social API automation
- Company billing (Virtus's invoice to the IR client) is handled **outside the platform**
- All campaign creation, assignment, review, and reporting is **admin-controlled**
- Future marketplace features should remain disabled, hidden, or clearly separated

---

## 10. Architecture Overview

| Layer | Technology |
|---|---|
| **Frontend** | Next.js 15 (App Router), React 19, Tailwind CSS v4, shadcn/ui (Radix UI) |
| **Backend / CMS** | Payload CMS 3.67 (headless, Payload admin panel **disabled**) |
| **Database** | Vercel Postgres (Drizzle ORM, UUID PKs) — previously MongoDB (commented out) |
| **Auth** | Payload built-in (cookie-based JWT, 48h token expiration, 5 login attempt lockout) |
| **Multi-tenancy** | `@payloadcms/plugin-multi-tenant` — tenant-scoped collections |
| **Storage** | Vercel Blob Storage (private; custom adapter for media uploads) |
| **Email** | SendGrid (via nodemailer-sendgrid); Brevo SDK also present |
| **Payments** | Stripe Connect (creator/agency payouts); Stripe Checkout code paths exist and should be verified against the current external company-billing model |
| **KYC / Tax Docs** | Azure AI Document Intelligence (W-9 / W-8BEN parsing) |
| **AI** | OpenAI SDK present (likely analytics/report generation — scope TBD) |
| **Testing** | Vitest (unit); analytics/calculator test scripts; no apparent e2e suite |
| **Deployment** | Vercel (inferred from all Vercel adapters and Cron job support) |
| **Package manager** | pnpm |
| **Build tool** | Turbopack (dev), Next.js build (production) |

**Payload admin panel is disabled.** All admin UI is a custom Next.js dashboard under `/dashboard/[tenant]/(virtus)/`.

**Marketplace artifact isolation strategy:** Currently appears to rely on route structure and role-based access checks. No explicit feature-flag system was identified. Future marketplace routes are present in the routing tree and linked in parts of the UI; actual runtime access by role should be verified and hardened.

---

## 11. Data Model Summary

| Collection | Purpose |
|---|---|
| `tenants` | Account/workspace entities; flags: `isCompany`, `isInfluencer`, `isAgency`; `managedBy` for agency hierarchy |
| `users` | Auth users with per-tenant role assignments; KYC status and document fields |
| `campaigns` | Core campaign unit — ticker, title, budget, dates, target URL, status, payment status |
| `collaborations` | Creator-campaign assignment record; tracks work status and payment lifecycle |
| `campaign-deliverables` | Individual content deliverables with URL, metrics (views, likes, comments), status |
| `campaign-tasks` | Task-level work items within a collaboration |
| `task-submissions` | Submissions against campaign tasks |
| `campaign-contacts` | Contact list for a campaign |
| `campaign-messages` | Messaging within campaigns |
| `channels` | Creator social/media properties (pending → approved/rejected) |
| `platforms` | Social platform reference data (seeded) |
| `influencer-packages` | Creator service packages (pricing, deliverable types) |
| `collaboration-packages` | Packages selected within a collaboration |
| `proposals` | Collaboration proposals / applications |
| `connections` | Bilateral influencer-company relationship records (statusInfluencer, statusCompany) |
| `user-agreements` | Legal agreement acceptance records |
| `onboarding-forms` | Onboarding form submission data |
| `media` | Uploaded files (private Vercel Blob storage) |
| `geo-targets` | Geographic targeting taxonomy |
| `industries` | Industry taxonomy |
| `product-categories` | Product category taxonomy |

**Globals:**
- `campaign-settings` — platform-wide campaign configuration

---

## 12. Role-Specific Workflows

### Virtus Super Admin / Campaign Manager

- Create and configure campaigns
- Set campaign payment status (unpaid → paid → waived)
- Assign creators or agencies to campaigns (creates Collaboration records)
- Approve/reject creator channel submissions
- Review and verify submitted deliverables and metrics
- Release collaboration payments (admin approval side of dual-approval)
- Review KYC documents (W-9, W-8BEN)
- Manage tenants and user accounts
- Generate client-facing reports
- Maintain separation between company users and creator/agency users

### Company / IR Client (`client` role)

- View campaign calendar and timeline
- View live, upcoming, and completed campaigns
- View performance reports
- **Does not** create campaigns
- **Does not** browse or contact creators, publishers, or agencies

### Individual Creator (`influencer` role)

- Complete onboarding (profile, channels, KYC/tax documents, Stripe Connect setup)
- Accept assigned campaigns (Collaborations in `pending` state)
- Submit deliverables (URL, content type, metrics)
- Track collaboration status through the work lifecycle
- Receive direct payment via Stripe (after dual-approval release)
- **Does not** browse a general marketplace of available company campaigns (current phase)

### UGC / Media Agency

- Tenant with `isAgency: true`
- Complete Stripe Connect onboarding at agency level
- Create and manage creator accounts under the agency (`managedBy` relationship)
- Accept campaigns on behalf of creator network
- Submit deliverables and metrics
- Receive agency-level payment; pays creators outside the platform

---

## 13. Reporting

- Reports are generated by Virtus admins based on verified campaign metrics
- Performance data is **manually entered** by creators/agencies via `CampaignDeliverable` records
- Metrics tracked: views, likes, comments per deliverable; campaign-level budget progress
- Reports are delivered to companies through the platform

**Current status:** `[TBD — confirm whether report generation UI is fully built, partially built, or pending]`

OpenAI SDK is present in dependencies — may be used for report generation or analytics summaries. Scope should be confirmed.

---

## 14. Payments

| Scenario | Payment Flow |
|---|---|
| Independent creator | Paid directly via Stripe Connect (platform-managed) |
| UGC agency | Agency paid via Stripe Connect; agency pays creators externally |
| Agency-managed creator | Paid by agency outside the platform |
| Company → Virtus | Company pays Virtus outside platform during the current phase; internal payment-status fields may be updated after confirmation |

### Payment Lifecycle (Collaboration)

```
pending → paid_held → released
           ↓              ↓
        refunded      completed
```

- `paid_held`: Stripe-related payment state used by collaboration payout flows; verify whether this is true escrow, delayed capture, or an internal hold abstraction before relying on it operationally
- `released`: Dual-approval required — `influencerApproval` + `companyApproval` (or admin force-release via `/api/admin/contracts/[id]/force-release`)
- Stripe webhook handler implemented at `/api/stripe/webhooks` handling: `checkout.session.completed`, `async_payment_succeeded`, `async_payment_failed`, `payment_intent.*` events

### Campaign Payment

- Current business understanding: company billing is handled outside the platform during this phase
- Code paths for Stripe Checkout and campaign payment handling exist and should be reconciled with the current external billing process
- Campaign `campaignPaymentStatus`: `unpaid` → `paid` (or `waived` by admin)
- Campaign `status`: `pending_payment` → `available` → `completed`

### KYC / Tax Onboarding

- Creators/agencies must submit W-9 (US) or W-8BEN (non-US) before payout
- Azure AI Document Intelligence used for document parsing
- KYC states: `not_submitted` → `pending_review` → `approved` / `rejected`
- Admin reviews via KYC queue in the admin dashboard

**Production readiness:** `[TBD — confirm webhook completeness, payout flows, and Stripe Connect onboarding for all user types]`

---

## 15. Current Implementation Status

**Legend:** “Appears built” means relevant code paths exist, but the workflow has not been verified end-to-end from this document alone. `[TBD]` means implementation or production readiness needs confirmation.

| Area | Status | Notes |
|---|---|---|
| Admin dashboard | `[TBD]` | Custom Next.js dashboard; Payload admin panel disabled |
| Campaign creation (admin) | Appears built | Campaign CRUD with full status lifecycle |
| Campaign assignment | Appears built | Collaboration records created and managed |
| Creator/agency acceptance flow | Appears built | Status transitions, proposal/apply routes present |
| Deliverable submission | Appears built | `CampaignDeliverable` with URL + metrics; auto-status sync |
| Metrics review (admin) | `[TBD]` | Verify admin review/approval UI completeness |
| Report generation | `[TBD]` | Confirm whether client-facing report UI exists |
| Company calendar / report view | `[TBD]` | Confirm company-scoped views are working |
| Creator channel approval | Appears built | Pending/approved/rejected channel queues |
| KYC review | Appears built | Admin KYC review queue in admin-view |
| Payments / Stripe Checkout | Appears built | Webhook handler, checkout routes present; reconcile with current external company-billing model |
| Stripe Connect (creator onboarding) | Appears built | `/stripe-setup/[tenantId]` flow, connect API routes |
| Agency management | Appears built | `managedBy` tenant relationship, agency flag |
| Connections system | Appears built | Bilateral influencer-company relationship records |
| Deployment | `[TBD]` | Vercel deployment assumed; confirm CI/CD pipeline |
| Automated tests | Partial | Vitest configured; analytics scripts present; no apparent e2e |
| Future marketplace isolation | Risk | Available-campaigns browse + apply routes are present in the route tree and linked in parts of the UI |

---

## 16. Known Risks / Open Questions

### Permissions / Access Control

- `campaignReadAccess` for `influencer` role has an incomplete implementation: the `in: []` clause (empty array) in the query constraint may fail closed at the collection-access layer, while page/API-level queries could still expose data if they bypass or override that access path. Needs verification.
- The `available` campaigns page is present in the live route tree and linked in parts of the UI — confirm whether runtime access should be gated or removed in the current phase.
- Confirm company users (`client` role) cannot see creator/agency data or campaign financials.
- Confirm creators cannot see campaigns they are not assigned to.
- Confirm agencies can only manage their own child creator tenants.

### Marketplace Artifacts

- `campaigns/available` browse page and `POST /api/campaigns/[id]/apply` are present in the application. If the intent is to keep these hidden from current users, access controls or routing guards need to be verified.
- Connections system allows influencer-company bilateral connections — confirm this does not enable direct discovery or communication outside the admin-managed flow.

### Payments

- Confirm Stripe webhook completeness and error handling for all payment states.
- Confirm the dual-approval fund release flow works end-to-end.
- Confirm KYC/tax onboarding is gating payouts correctly.
- Azure Document Intelligence integration scope and reliability is unknown.

### Reporting

- Confirm whether client-facing report generation UI is built and delivering to company users.
- OpenAI SDK usage scope is unclear.

### Data / Schema

- Only one migration exists (`20260509_002126_init`) — migration history is shallow/early, and production schema drift should be verified.
- MongoDB adapter is commented out; Postgres is active.

### Testing

- No apparent end-to-end test suite. Regression risk is high for complex payment and permission flows.

### Deployment

- Vercel deployment assumed but not confirmed from code alone.
- `regenerate` npm script wipes and reseeds the database — confirm this is dev-only and cannot affect production.

---

## 17. Technical Commands and Local Setup

```bash
# Install dependencies
pnpm install

# Development (Turbopack + HTTPS experimental)
pnpm dev

# Production build
pnpm build

# Start production server
pnpm start

# Payload CLI
pnpm payload [command]

# Generate Payload types, import map, GraphQL schema
pnpm generate

# Run database migrations
pnpm payload migrate

# Seed reference/test data
pnpm seed

# Lint
# (ESLint configured via eslint.config.js)

# Tests
pnpm test
pnpm test:watch
pnpm test:coverage

# Analytics-specific test scripts
pnpm test:analytics
pnpm test:calculators
```

**Runtime requirements:**
- Node.js `^18.20.2` or `>=20.9.0`
- pnpm (package manager)
- Postgres database (Vercel Postgres in production; local Postgres for dev)
- Environment variables (see Section 18)

**Known setup assumptions:**
- HTTPS experimental flag is used in dev (`--experimental-https`) — local SSL cert may be required
- Turbopack used in dev; standard Next.js webpack in production build

---

## 18. Environment Variable Categories

| Category | Purpose | Examples |
|---|---|---|
| Database | Postgres connection | `POSTGRES_URL` |
| Auth / Session | Payload JWT secret | `PAYLOAD_SECRET` |
| Payments | Stripe keys, webhook secret | `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `STRIPE_PUBLISHABLE_KEY` |
| Email | SendGrid / Brevo API keys and sender config | `SENDGRID_API_KEY`, `SENDGRID_FROM_ADDRESS`, `BREVO_SENDER_EMAIL` |
| Storage | Vercel Blob token | `BLOB_READ_WRITE_TOKEN` |
| App URLs | Local, staging, production base URLs | `NEXT_PUBLIC_SERVER_URL` |
| Public client config | Browser-safe configuration | `NEXT_PUBLIC_*` vars |
| AI / Document services | OpenAI, Azure AI keys | `OPENAI_API_KEY`, Azure credentials |
| Jobs / Cron | Cron job authentication | `CRON_SECRET` |
| Platform metadata | Company name, branding | `COMPANY_NAME` |

No real values are included in this document. Environment variables are managed outside the codebase (Vercel environment dashboard or equivalent).

---

## 19. Deployment Overview

| Aspect | Detail |
|---|---|
| **Hosting** | Vercel (inferred from `@payloadcms/db-vercel-postgres`, `@payloadcms/storage-vercel-blob`, Cron job pattern) |
| **Build** | `pnpm build` → standard Next.js build via `withPayload` wrapper |
| **Database migrations** | Payload `migrate` command (Drizzle under the hood); single `init` migration currently |
| **Preview environments** | Vercel preview deployments expected (per-branch) |
| **Production releases** | `[TBD — confirm deployment pipeline: manual release process, automated provider deploys, or another workflow]` |
| **Rollback** | `[TBD — confirm rollback strategy]` |
| **Marketplace artifact control** | Currently route-tree and role-access based; no explicit feature-flag system identified |
| **Dev database reset** | `pnpm regenerate` wipes + reseeds; confirm this cannot reach production |

---

## 20. Developer Evaluation Checklist

A senior developer taking this on should be able to answer:

- [ ] Do I understand why Virtus must remain between companies and creators/agencies?
- [ ] Do I understand the role boundaries (`super-admin`, `campaign-manager`, `client`, `influencer`)?
- [ ] Do I understand the campaign lifecycle (`pending_payment → available → completed`)?
- [ ] Do I understand the collaboration lifecycle and payment hold/release model?
- [ ] Do I understand what is admin-driven versus self-serve?
- [ ] Do I understand which features are current phase versus future marketplace artifacts?
- [ ] Do I know which areas are built, partial, or missing?
- [ ] Do I understand the Stripe Connect payout architecture and any Stripe Checkout code paths?
- [ ] Do I understand the KYC gating requirements before payout?
- [ ] Have I identified the access-control risk in `campaignReadAccess` for influencers and checked whether any page/API queries bypass it?
- [ ] Have I identified the marketplace route exposure risk?
- [ ] Can I estimate how quickly I could ship the next feature?

---

## 21. Suggested First Areas to Inspect If Engaged

Listed in priority order:

1. **Role and tenant permissions** — verify `client`, `influencer`, and `campaign-manager` access boundaries end-to-end; confirm the incomplete `campaignReadAccess` influencer clause
2. **Marketplace artifact audit** — trace `campaigns/available`, `apply`, and Connections routes; confirm whether they expose unintended company-creator discovery
3. **Campaign lifecycle** — admin campaign creation → payment → assignment → deliverable submission → completion
4. **Collaboration payment flow** — payment event handling → held/internal payment state → dual-approval release
5. **KYC and Stripe Connect onboarding** — verify gating and completeness for creators and agencies
6. **Company-facing views** — confirm `client` role sees only campaign calendar and reports, nothing else
7. **Report generation** — confirm whether client-facing report delivery is built or pending
8. **Automated tests** — assess coverage; identify highest-risk untested flows
9. **Deployment pipeline** — confirm Vercel setup, environment variable management, and migration process
10. **OpenAI and Azure AI usage** — identify scope and whether these are in production paths

---

## 22. Immediate Product Priorities

| Priority | Area | Reason |
|---|---|---|
| P0 | Role permission audit | Protects Virtus's managed-service position; influencer access constraint is incomplete |
| P0 | Marketplace artifact gating | Prevents users from encountering workflows outside current operating model |
| P0 | Reporting workflow | Core company-facing deliverable; status unclear |
| P1 | Metrics review / admin approval UI | Required before reports are reliable |
| P1 | Payment flow end-to-end verification | Needed before campaign payouts can run in production |
| P1 | Agency-managed creator flow | Important for UGC agency model |
| P2 | Automated test coverage | High-risk flows (payments, permissions) have no apparent e2e coverage |
| P2 | Future social API integrations | Later automation phase |
| P3 | Future affiliate-style marketplace | Possible later phase; not proven for this market |

---

## 23. Information Needed Before Taking Over

This section covers what a new developer would need to understand the project — not credentials or access.

| Item | Status |
|---|---|
| Production URL | `[TBD — confirm if shareable]` |
| Staging / preview URL | `[TBD]` |
| Frontend framework | Next.js 15, React 19 |
| Backend / CMS | Payload CMS 3.67 |
| Database | Vercel Postgres |
| Hosting | Vercel |
| Payment provider | Stripe Connect; Stripe Checkout code paths present and require business-flow confirmation |
| Email provider | SendGrid (primary); Brevo (dependency present) |
| Storage provider | Vercel Blob |
| Document AI | Azure AI Document Intelligence |
| AI / LLM | OpenAI |
| Current blockers | Reporting completeness, permissions audit, marketplace artifact gating |
| Product decision owner | Virtus Media Group stakeholders |
| Business rule clarification | Virtus admin team |

---

## 24. Security Notes

- **Secrets:** Never commit `.env` files. All secrets are managed via hosting environment variables.
- **Role enforcement:** `super-admin` is a global role set on the `User.roles` array; tenant-specific roles (`tenant-admin`, `campaign-manager`, `influencer`, `client`) are set per-tenant in `User.tenants[].roles`. Both layers must be checked.
- **Company isolation:** `client` users must only see campaigns and reports scoped to their tenant. Verify multi-tenant plugin enforces this.
- **Creator isolation:** `influencer` users should only see campaigns they are assigned to. The current `campaignReadAccess` implementation for influencers has an incomplete query clause (`in: []`) — **this is a known risk**, especially if route/page/API queries do not consistently rely on the same access layer.
- **Agency isolation:** Agency tenants should only manage creator tenants with `managedBy` pointing to them. Verify this constraint is enforced in all relevant queries.
- **Marketplace routes:** `campaigns/available` and `POST /api/campaigns/[id]/apply` are present in the application. Confirm they do not expose company-to-creator discovery or unintended self-serve behavior.
- **Payment data:** Stripe handles all PCI-sensitive data. Platform stores only Stripe IDs (session IDs, payment intent IDs). Do not log or expose full payment objects.
- **KYC documents:** W-9/W-8BEN files are stored in private Vercel Blob storage. Confirm access tokens are not publicly accessible.
- **Force-release endpoint:** `/api/admin/contracts/[id]/force-release` bypasses dual-approval. Confirm this is restricted to `super-admin` only.
- **Cron jobs:** Protected by `CRON_SECRET` in Authorization header. Confirm secret rotation process.
- **Database reset script:** `pnpm regenerate` drops and reseeds the database. Confirm this cannot target production.

---

## 25. Suggested First-Day Developer Checklist

1. Read Sections 2–9 of this document (product model, operating model, constraints)
2. Confirm the managed-service role boundaries by reading `src/access/campaign-access.ts` and `src/collections/users/`
3. Map all user types to their tenant flags (`isCompany`, `isInfluencer`, `isAgency`) in `src/collections/tenants/`
4. Trace the campaign lifecycle: creation → payment → assignment → deliverable → report
5. Trace the collaboration payment lifecycle: payment event handling → held/internal payment state → dual-approval release
6. Audit the `campaigns/available` route and `apply` API endpoint for current-phase exposure risk
7. Audit the `Connections` collection for company-creator discovery risk
8. Confirm `client` role cannot reach creator/admin views
9. Confirm `influencer` role cannot reach campaigns they are not assigned to, including any routes or APIs that bypass collection-level access
10. Review Stripe webhook handler completeness (`src/app/api/stripe/webhooks/route.ts`)
11. Confirm KYC/Stripe Connect onboarding is gating payouts before any live campaign payout
12. Run `pnpm test` to assess current test pass rate
13. Set up local environment and confirm `pnpm dev` runs cleanly
14. Identify the single highest-priority unfinished feature and scope the first shipping task

---

## 26. Glossary

| Term | Meaning |
|---|---|
| **Virtus Admin** | Internal Virtus Media Group operator; manages all campaigns, assignments, reviews, and reports |
| **Company / IR Client** | Public company or investor relations team; the paying client |
| **Creator / Influencer** | Individual content creator, publisher, or social media account holder who executes campaign work |
| **UGC Agency** | Agency managing multiple creators; accepts campaigns at agency level; pays creators externally |
| **Agency-Managed Creator** | Creator operating under an agency's account; paid by agency outside the platform |
| **Campaign** | Core unit of work — a paid IR media campaign managed through Virtus |
| **Collaboration** | A creator or agency's assignment to a campaign; tracks work status and payment lifecycle |
| **Deliverable** | An individual required piece of content (post, article, newsletter, video) with a URL and metrics |
| **Metric** | Manually entered performance data (views, likes, comments) for a deliverable |
| **Report** | Company-facing campaign performance summary generated by Virtus admin |
| **Tenant** | Account/workspace entity in the platform; may represent a Virtus workspace, a company, a creator, or an agency |
| **Super Admin** | Global platform administrator; full access across all tenants |
| **Managed Service** | Current operating model — Virtus controls all campaign operations between IR clients and creators |
| **Stripe Connect** | Stripe's sub-account system used to onboard creators/agencies and route payouts |
| **KYC** | Know Your Customer — tax and identity verification (W-9 for US persons, W-8BEN for non-US) |
| **Marketplace / Affiliate-Style Flow** | Possible future model where companies post campaigns and creators browse/apply independently |
| **Hidden Artifact** | Code or UI for future-phase functionality that is not currently intended for production users |
| **Connection** | A bilateral relationship record between an influencer tenant and a company tenant |
| **Ticker** | Stock ticker symbol associated with a campaign (e.g., `AAPL`) |

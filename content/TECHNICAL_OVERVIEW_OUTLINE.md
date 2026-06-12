# Virtus Media Platform Technical Overview Outline

## 1. Purpose

This document gives a senior developer enough product, architecture, workflow, and operational context to evaluate whether they can take over development and begin shipping features quickly if engaged.

It is not intended to include credentials, private access details, repository access details, or sensitive deployment secrets.

## 2. Product Summary

Virtus Media is an investor relations campaign management platform that supports managed media campaigns between public companies / IR teams and publishers, creators, influencers, and UGC agencies.

Virtus Media Group remains the operational middle layer. The platform supports Virtus admins in creating campaigns, assigning campaign work, reviewing submitted performance metrics, generating reports, and delivering those reports back to company clients.

The current business model is primarily managed service, not self-serve marketplace. This matters because the platform should preserve Virtus Media Group's role as the central operator between IR clients and the creator / publisher / agency network.

## 3. Operating Model

### What This Platform Is

- Managed-service campaign operations software.
- Internal admin tooling for Virtus Media Group.
- A campaign coordination system for companies, creators, publishers, and UGC agencies.
- A reporting and performance-management layer.
- A platform that supports large, high-touch investor relations media campaigns that are sold and managed by Virtus.

### What This Platform Is Not In The Current Phase

- Not a self-serve affiliate marketplace.
- Not a system where companies directly post campaigns for creators to browse.
- Not a system where creators freely browse and claim all available campaigns.
- Not designed to remove Virtus from the transaction.
- Not currently dependent on automated social API reporting.
- Not intended to expose direct company-to-creator relationships that would let IR clients bypass Virtus.

## 4. Current Managed-Service Rationale

The current real-world Virtus business is centered on managed campaigns that are typically much larger than small affiliate-style jobs. Campaigns can be upward of $50,000 and may reach much higher budgets, including six-figure or larger engagements.

Because these campaigns are high-value and relationship-driven, Virtus Media Group needs to remain the managed-service layer between IR clients and creators, publishers, influencers, or agencies.

The platform should preserve a separation between companies and the creator / publisher / agency supply side. Companies should not be able to use the platform to discover and directly contact creators or publishers in a way that bypasses Virtus.

This separation is intentional. The current product should help Virtus operate campaigns more efficiently without turning the business into a fully open peer-to-peer marketplace.

## 5. Future Marketplace / Affiliate-Style Artifacts

There may be artifacts in the codebase for future phases where the platform behaves more like an affiliate marketing marketplace.

These artifacts may include, or eventually include:

- Company self-service account creation.
- Company-created campaign posting.
- Creator self-service account creation.
- Creator browsing of available campaigns.
- Creator opt-in / self-selection for available campaigns.
- Marketplace-style campaign discovery.
- Public or semi-public campaign availability pages.
- Workflows where smaller creators or publishers accept small campaign offers without heavy Virtus admin involvement.

These artifacts should not be interpreted as the current business model unless explicitly enabled for the current phase.

Some of these features may be commented out, hidden from users, behind feature flags, unfinished, or scheduled to be hidden. They represent possible future-phase work, not necessarily production-ready functionality.

## 6. Future Marketplace Business Hypothesis

The affiliate-style marketplace use case is not yet proven for this market.

If it becomes useful, it would likely apply only to a small percentage of the overall user base, especially:

- Smaller companies running low-budget campaigns.
- Campaigns around a few thousand dollars, for example roughly $5,000.
- Smaller influencers, publishers, or agencies willing to accept lower-touch jobs.
- Campaigns that do not justify the same level of Virtus admin time and oversight as larger managed-service campaigns.

The future marketplace concept may or may not come to fruition based on market feedback. A developer taking over the project should distinguish between:

- Current managed-service workflows that need to work now.
- Hidden or commented future-phase marketplace workflows that should not leak into the current user experience.
- Product experiments that may need to be removed, completed, or feature-flagged later.

## 7. Tenant / User Types

| Type | Role | Summary |
|---|---|---|
| Virtus Admin | Internal operator | Creates campaigns, assigns work, verifies metrics, generates reports |
| Company / IR Client | Client | Views timelines and receives campaign reports |
| Individual Creator | Campaign executor | Accepts assigned campaigns, executes externally, submits metrics |
| UGC / Media Agency | Manager of creators | Accepts campaigns, manages creator accounts, submits metrics, receives payment |
| Agency-Managed Creator | Creator under agency | Executes work under agency management |

## 8. Core Workflow

1. Virtus sales closes a deal with a company outside the platform.
2. The deal defines budget, campaign scope, deliverables, and timeline.
3. Virtus admin creates the campaign in the platform.
4. Admin assigns or invites individual creators and/or agencies.
5. Creators/agencies accept campaign work.
6. Work is executed externally on social channels, websites, newsletters, or publisher properties.
7. Creators/agencies manually submit performance metrics.
8. Admin reviews and verifies metrics.
9. Admin generates a client-facing performance report.
10. Company views or receives the report through the platform.
11. Payments are released to individual creators or agencies depending on who accepted the campaign.

## 9. Current Phase Constraints

- Companies do not create campaigns directly.
- Companies do not directly interact with creators/agencies.
- Creators do not browse a marketplace of all available company offers.
- Agencies may manage creators under their own account.
- Performance metrics are manually entered.
- Company billing is handled outside the platform.
- Virtus admins control campaign creation, assignment, review, and reporting.
- Future marketplace features should remain hidden, disabled, or clearly separated unless intentionally activated.

## 10. Architecture Overview

Include high-level architecture only:

- Frontend framework.
- Backend/CMS framework.
- Database model.
- Auth/session approach.
- Tenant/role model.
- Payments model.
- Reporting model.
- Hosting/deployment model.
- External services used.
- Any feature-flag, hidden-route, commented-code, or role-gating strategy used to keep future marketplace artifacts out of the current production experience.

Avoid specific private access details.

## 11. Data Model Summary

| Entity | Purpose |
|---|---|
| User | Authenticated person with role/profile |
| Tenant | Account/workspace entity |
| Company | IR/public company client |
| Campaign | Core unit of work sold and managed by Virtus |
| Creator | Individual executing campaign work |
| Agency | Entity managing creators |
| Channel | Social, publisher, newsletter, or media property |
| Deliverable | Required post, article, newsletter, video, etc. |
| Metric | Submitted performance result |
| Report | Client-facing campaign performance summary |
| Payment/Payout | Compensation record for creators/agencies |

## 12. Role-Specific Workflows

### Virtus Admin

- Create campaigns.
- Assign creators/agencies.
- Review metrics.
- Generate reports.
- Manage tenants/users.
- Manage campaign statuses.
- Maintain the separation between company users and creator / publisher / agency users.

### Company / IR Client

- View campaign calendar.
- See live/upcoming/completed campaigns.
- View reports.
- Limited managed-service view.
- Does not create campaigns in the current phase.
- Does not directly browse or contact creators, publishers, or agencies through the platform in the current phase.

### Individual Creator

- Onboard/apply.
- Add payment/tax/bank information.
- Accept assigned campaigns.
- Submit performance metrics.
- Receive direct payment.
- Does not browse a general marketplace of company campaigns in the current phase.

### UGC / Media Agency

- Add agency payment/tax/bank information.
- Create/import/manage creators.
- Accept campaigns on behalf of creator network.
- Submit metrics.
- Receive agency-level payment.
- Pay creators outside the platform.
- View campaign timelines for creators under the agency account.

## 13. Reporting

- Reports are created by Virtus admins.
- Reports are based on submitted and verified campaign metrics.
- Reports are delivered to companies.
- Reporting is a core product feature and should be treated as high priority.
- Current state should be documented as:
  - Built.
  - Partially built.
  - Not built.
  - Known issues.
  - Expected near-term changes.

## 14. Payments

| Scenario | Payment Flow |
|---|---|
| Independent creator | Paid directly through platform |
| UGC agency | Agency paid through platform |
| Agency-managed creator | Paid by agency outside platform |
| Company/client | Pays Virtus outside platform during current phase |

Payment-related notes to document:

- Whether Stripe Connect is used.
- Whether QuickBooks or another external billing process is involved for company payments.
- Whether KYC, tax, and bank onboarding are required before payout.
- Whether payout flows are production-ready.
- Whether webhook handling is complete.

## 15. Current Implementation Status

| Area | Status | Notes |
|---|---|---|
| Admin dashboard |  |  |
| Company calendar |  |  |
| Campaign creation |  |  |
| Campaign assignment |  |  |
| Creator onboarding |  |  |
| Agency management |  |  |
| Metrics submission |  |  |
| Metrics review |  |  |
| Report generation |  |  |
| Company report access |  |  |
| Payments/KYC |  |  |
| Deployment |  |  |
| Hidden/commented future marketplace features |  |  |

## 16. Known Risks / Open Questions

- Does the current permissions model fully enforce the managed-service structure?
- Can company users see only company-facing information?
- Can creators see only assigned campaigns?
- Can agencies manage only their own creators?
- Are any future marketplace features visible to users unintentionally?
- Are any commented, hidden, or unfinished marketplace artifacts still connected to live navigation, routing, permissions, or APIs?
- Is reporting already implemented or still pending?
- Are payments production-ready?
- Are KYC/tax/bank flows complete?
- Are campaign statuses clearly modeled?
- Are there automated tests?
- Is deployment stable?

## 17. Technical Commands and Local Setup

This section can include commands if appropriate, but keep it neutral.

Example:

```bash
pnpm install
pnpm dev
pnpm build
pnpm lint
```

Include:

- Runtime requirements.
- Package manager.
- Local database requirements.
- Environment variable categories.
- Known setup assumptions.

Do not include:

- Clone instructions.
- Repository URL.
- Access instructions.
- Credentials.

## 18. Environment Variable Categories

Do not list real values.

| Category | Purpose |
|---|---|
| Database | App data and CMS persistence |
| Auth/session | Login, sessions, secure cookies |
| Payments | Creator/agency onboarding and payouts |
| Email | Invitations, notifications, report delivery |
| Storage | Media/file uploads |
| App URLs | Local, staging, production routing |
| Public client config | Browser-safe configuration only |
| Feature flags | Hiding or enabling future-phase functionality |

## 19. Deployment Overview

High-level only:

- Hosting provider.
- Build process.
- Migration process.
- Environment variable categories.
- Preview/staging availability.
- Production release process.
- Rollback approach.
- Current deployment confidence.
- Whether future marketplace features are controlled through feature flags, hidden routes, permissions, or commented code.

Avoid:

- Account names.
- Credentials.
- Private URLs if not meant to be shared.
- Repository/provider access instructions.

## 20. Developer Evaluation Checklist

The senior developer should be able to answer:

- Do I understand the business model?
- Do I understand why Virtus must remain between companies and creators/agencies?
- Do I understand the role boundaries?
- Do I understand the campaign lifecycle?
- Do I understand the payment flow?
- Do I understand what is admin-driven versus self-serve?
- Do I understand what is already built?
- Do I understand what is unfinished?
- Do I understand which artifacts are future-phase marketplace work and not current product behavior?
- Do I understand the likely technical risks?
- Do I know what I would inspect first if given access?
- Can I estimate how quickly I could ship the next feature?

## 21. Suggested First Areas to Inspect If Engaged

Keep this conceptual, not access-oriented.

- Tenant and role permissions.
- Campaign lifecycle implementation.
- Admin campaign creation flow.
- Creator/agency acceptance flow.
- Metrics submission and review.
- Report generation.
- Company-facing calendar/report views.
- Payment/KYC flow.
- Deployment/build configuration.
- Automated tests or lack thereof.
- Hidden, commented, or feature-flagged marketplace code.
- Any routes, components, or APIs that suggest company-created campaigns or creator campaign browsing.

## 22. Immediate Product Priorities

| Priority | Area | Reason |
|---|---|---|
| P0 | Reporting workflow | Core company-facing deliverable |
| P0 | Role permissions | Protects Virtus's managed-service position |
| P0 | Hide or isolate future marketplace artifacts | Prevents users from seeing workflows that are not part of the current model |
| P1 | Agency-managed creator flow | Important for UGC agency model |
| P1 | Metrics review/approval | Required before reports are reliable |
| P1 | Payment readiness | Needed before campaign payouts |
| P2 | Future social API integrations | Later automation phase |
| P2 | Future affiliate-style marketplace | Possible later phase, not yet proven for this market |

## 23. Information Needed Before Taking Over

The new developer may eventually need access to systems, but this document should focus only on what they need to understand the project.

Include:

- Production URL, if shareable.
- Staging/demo URL, if shareable.
- Current tech stack.
- Hosting provider name, if known.
- Database provider name, if known.
- Payment provider name, if known.
- Email provider name, if known.
- Storage provider name, if known.
- Third-party services used.
- Current deployment process at a high level.
- Known operational dependencies.
- Current blockers or unfinished features.
- Who owns product decisions.
- Who can clarify business rules.

Avoid:

- Repository access.
- Repository permissions.
- Credentials.
- Secrets.
- Admin passwords.
- Production database credentials.
- Anything suggesting covert access to the current developer's workflow.

## 24. Security Notes

Include:

- Do not commit secrets.
- Document where credentials are stored only at a high level.
- Explain role boundaries clearly.
- Confirm company users cannot access creator/admin workflows.
- Confirm creators cannot see unrelated campaigns.
- Confirm agencies can only manage their own child creators.
- Confirm payment data and tax/KYC data are protected.
- Confirm future marketplace code cannot expose company-to-creator discovery or contact paths in the current phase.

## 25. Suggested First-Day Developer Checklist

This is the practical "hit the ground running" section.

1. Review the product summary and operating model.
2. Confirm the managed-service role boundaries.
3. Confirm which user types exist and what each can do.
4. Identify current-phase workflows versus future marketplace artifacts.
5. Review the campaign lifecycle.
6. Review admin campaign creation.
7. Review creator/agency campaign acceptance.
8. Review metrics submission and approval.
9. Review reporting implementation status.
10. Review payment/KYC implementation status.
11. Review how hidden/commented future features are isolated.
12. Run available quality checks once access is available.
13. Identify blockers to shipping the next feature.

## 26. Glossary

| Term | Meaning |
|---|---|
| Virtus Admin | Internal Virtus operator managing campaigns |
| Company | Public company or IR client |
| Creator | Individual influencer/publisher/content distributor |
| UGC Agency | Agency managing multiple creators |
| Agency-Managed Creator | Creator operating under an agency relationship |
| Campaign | Paid media/IR campaign managed through Virtus |
| Deliverable | Required post, article, newsletter, video, etc. |
| Metric | Manually entered performance data |
| Report | Company-facing performance summary |
| Tenant | Account/workspace entity in the platform |
| Managed Service | Current operating model where Virtus controls campaign operations |
| Marketplace / Affiliate-Style Flow | Possible future model where companies post campaigns and creators browse/accept them |
| Hidden Artifact | Code or UI for future-phase functionality that is disabled, commented out, feature-flagged, or otherwise not exposed to users |

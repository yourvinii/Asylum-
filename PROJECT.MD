# HAVEN — Master Project Specification and Magic Door Build Plan

> **Source of truth:** `59_RealEstate.pdf` supplies product screens, workflows, search filters, and core data relationships. Use it for product requirements, not as a mandate to copy its AWS/Cognito/Next.js architecture.
>
> **Build rule:** Implement one approved chunk at a time. Start with Chunk 0 only; inspect and report. Stop after every chunk and wait for explicit approval.

## 1. Vision

Build **HAVEN**, an India-first rental marketplace for tenants, property owners, and admins. Initial focus: cities such as Bhopal and Shahdol, Madhya Pradesh, with the ability to expand to other Indian cities. It should be a responsive, maintainable, portfolio-quality full-stack app—not a one-shot code dump.

Principles:
- India-first: ₹ INR, Indian addresses, localities, cities, states, PIN codes, rooms/PG/shared accommodation, flats, and houses.
- Clearly separate monthly rent, security deposit, maintenance, brokerage, and application/platform fees.
- Never invent property facts, reviews, availability, payment outcomes, or verification status.
- Owners can manage only their own properties and related records.
- Enforce access control on the backend, not only in the UI.
- Preserve existing workspace/user work; inspect before editing.
- Build/test/report sequentially; do not continue automatically.

## 2. Product requirements derived from `59_RealEstate.pdf`

Adapt the PDF's “Renter” to **Tenant** and “Manager” to **Owner**.

### Public experience
- Landing/home page with property discovery.
- Search/listing results page and individual property detail page.
- Search criteria encoded in URL query parameters for shareable/bookmarkable searches.
- Property gallery, key facts, and Mapbox location.
- Filters: location, min/max price, beds, baths, property type, min/max area, amenities, availability date, and geographic proximity.

### Tenant experience
- Favorites: list, add, remove.
- Applications list/detail/status.
- Current residences and individual residence detail.
- Lease details and payment records/history.
- Inquiry/contact workflow; real-time messaging later.

### Owner experience
- My properties list and individual property management page.
- Create, edit, publish/unpublish, archive/remove own listings.
- Manage property photos.
- View applications for owned properties; approve/reject valid applications.
- Approved applications can lead to leases.
- View leases and payment records related to owned properties.

### Core relationships
- Owner has many properties.
- Tenant has many favorites and applications.
- Property has one owner, location data, applications, and potentially leases.
- Application connects a tenant and property; approval can lead to a lease.
- Lease connects tenant, owner, property, and payment records.
- Favorites connect tenant and property.
- PDF diagrams include Manager/Owner, Tenant, Property, Location, Application, Lease, Payment.
- PDF's AWS architecture (Cognito, Amplify/Next.js, S3, API Gateway, EC2, relational DB) is **not** the chosen architecture for this build.

## 3. Confirmed stack

### Frontend
- React + Vite + TypeScript
- React Router
- Tailwind CSS
- shadcn/ui where useful
- Redux Toolkit for genuinely shared state, not every local input
- React Hook Form + Zod
- Axios
- Lucide icons
- Mapbox
- Responsive mobile/tablet/desktop UI

### Backend
- Node.js + Express + TypeScript
- REST API under `/api`
- MongoDB + Mongoose
- Zod request/query validation
- JWT authentication and role-based authorization
- Password hashing with a maintained library (prefer Argon2id where compatible; otherwise bcrypt)
- Centralized errors, consistent responses, safe server logging
- Rate limits for auth, AI, uploads, and payment endpoints

### Integrations
- Email OTP/transactional email via configured SMTP/provider (Nodemailer is acceptable).
- S3-compatible object storage, behind an abstraction; local adapter for development.
- AI provider called from backend only.
- Mapbox.
- Real payment gateway in v1. Plan **Razorpay** as the initial India-focused provider, subject to merchant account, credentials, current provider support, and confirmed settlement model. Isolate it behind a provider adapter.
- Socket.IO messaging after core rental flows work.
- Git/GitHub from the start; GitHub Actions CI after foundation stabilizes.
- Deployment only after local workflows/security checks pass.
- Docker optional/deferred; n8n optional/deferred and must not be a core dependency.

Inspect existing files and compatibility before installing dependencies. Do not install anything before Chunk 0 is reviewed and approved.

## 4. Roles and permissions

Roles: `tenant`, `owner`, `admin`.

### Tenant
- Register, verify email, sign in/out, manage own profile.
- Search/filter published properties and view details/maps.
- Manage favorites.
- Submit applications and view own statuses.
- View approved residence, lease, and payment history.
- Pay through server-created gateway flows when configured.
- Use AI search, recommendations, rental assistant, inquiry/chat when implemented.

### Owner
- Register, verify email, manage own profile.
- Create/edit/publish/unpublish/archive own listings and manage photos.
- Generate AI listing drafts, review them, and explicitly save/publish.
- View applications for own listings and approve/reject valid transitions.
- View related leases/payment records.
- Communicate with tenants through inquiry/chat when implemented.

### Admin
- Protected dashboard and real database-backed metrics.
- Search/view users; suspend/reactivate according to policy.
- Review property reports and moderate listings.
- Review owner/property verification if enabled.
- View complaints and administrative audit logs.
- See only safe operational metadata for AI/gateway integrations.

### Mandatory security
- Public registration must never allow choosing `admin`; provision admins securely on the server.
- Enforce all role permissions and ownership checks in backend middleware/services.
- Never trust client-provided role, owner ID, tenant ID, payment status, amount, approval status, or privileged fields.
- Owners may only manage their own listings and related applications/leases/payments.
- Tenants may only access their own profile, favorites, applications, leases, payments, and conversations.
- Avoid leaking private records through IDs, populated references, logs, or admin views.
- Record consequential admin actions in an audit log without logging secrets.

## 5. India-first product behavior

- Currency: INR (₹), formatted for India.
- Address: address line, locality/area, city, state, PIN code, country, optional landmark, coordinates.
- Accommodation types should include configurable India-relevant choices such as room, PG, shared room, studio, 1BHK, 2BHK, 3BHK+, independent house, and apartment.
- Furnishing: furnished, semi-furnished, unfurnished.
- Amenities are selected/stored values; never infer them from prose.
- Cost breakdown explicitly separates rent, deposit, maintenance, brokerage, and any fee.
- Support availability date and minimum stay if the owner supplies them.
- Listing states should distinguish draft, pending review, published, rejected, rented/unavailable, archived.
- English-first with architecture ready for Hindi later; do not claim bilingual support before translated strings exist.
- Do not expose tenant phone numbers or application details publicly.
- Do not label an owner/property “verified” unless checks are implemented and passed.

## 6. AI features (five separate capabilities)

Build only after core property APIs and permissions are reliable.

### 6.1 Rental assistant
- Answer questions about real property details, amenities, costs, and HAVEN workflows.
- Retrieve current property records before answering property-specific questions.
- If facts are missing, say so; never invent availability, price, amenities, location, or policy.
- Treat user/listing text as untrusted input.
- Rate-limit and cap request size; handle timeout/provider failure.

### 6.2 Natural-language search
- Example: “Find a furnished 1BHK in Bhopal under ₹8,000.”
- Convert the request into a strict structured filter object.
- Validate/normalize the object server-side and call the normal search service.
- Never execute model-generated SQL, MongoDB operators, code, or raw queries.

### 6.3 Recommendations
- Begin with explainable rule-based ranking using budget, city/locality, property type, furnishing, amenities, and availability.
- Recommend only real published properties.
- Explain with actual listing fields; don't claim to learn preferences unless they are stored and used.

### 6.4 AI listing-description generator
- Draft only from owner-provided facts.
- Never invent facilities, measurements, amenities, location, safety claims, or availability.
- Keep text editable; owner must explicitly save/publish.

### 6.5 Listing improvement
- Suggest clearer wording and missing fields.
- Keep changes optional/editable; never auto-publish.

### AI safety/engineering
- Browser calls HAVEN backend; backend calls AI provider.
- Never expose AI keys in `VITE_*`.
- Validate input/output schemas, rate-limit, set timeouts, and gracefully handle errors.
- Minimize personal data sent to providers.
- Log usage/error metadata without unnecessary sensitive content.
- Core app must work when AI credentials are absent.

## 7. Real payment gateway

Real gateway integration is required in v1, but do not invent the commercial settlement model. Before enabling live money movement, explicitly document whether HAVEN collects application/platform fees, rent/deposit for owners, or both, and how funds reach the intended recipient. Marketplace/split settlement can require a specific provider product, onboarding, compliance, and approval. Do not simulate split settlements.

### Required safeguards
- Backend payment service + provider adapter.
- Server calculates amounts from trusted records; never trust a browser-supplied amount.
- Server creates gateway order/session and a local pending payment record.
- Verify signatures server-side according to current provider documentation.
- Verify signed webhooks; preserve raw request body if required by provider.
- Treat webhook events as authoritative state signals; safely handle retries and out-of-order delivery.
- Idempotent processing using gateway order/payment/event IDs and unique constraints.
- Never mark paid based only on frontend callback.
- Never store card numbers, CVV, UPI PIN, bank credentials, or payment instrument secrets.
- Persist gateway IDs, amount in minor units, currency, purpose, status, safe metadata, and timestamps.
- Avoid floating-point arithmetic for money.
- Keep sandbox separate from live mode; never present sandbox payments as real.
- Support provider-appropriate pending, captured/paid, failed, cancelled, refunded, disputed states.
- Test invalid signature, duplicate/out-of-order webhooks, amount/currency mismatch, retries, and unknown order IDs.
- Only verified payment status can trigger payment-dependent business rules.
- If credentials/merchant approval are missing, report sandbox-ready/unconfigured—not “live payments working.”

### Decisions to finalize in Chunk 0
- Enabled payment purposes: application fee, rent, deposit, maintenance, platform fee.
- Whether HAVEN collects for itself or on behalf of owners; whether split settlement is required.
- Refund/cancellation policy and authorized refund initiator.
- Gateway sandbox/live credentials are supplied only through environment variables.

## 8. MongoDB/Mongoose data model

Use schema validation, references, timestamps, indexes, explicit enums, and documented seed/migration guidance. Adjust names only with a documented reason.

### User
- `name`, normalized unique `email`, `passwordHash`, optional private `phoneNumber`
- `role: tenant | owner | admin`
- `isEmailVerified`, `accountStatus`, optional owner-verification state
- timestamps

### EmailVerification / OTP
- user/email reference, hashed OTP, expiry, attempt count, consumed state, timestamps
- TTL index where appropriate
- never log OTP or retain plaintext beyond the send operation

### Property
- owner reference
- title/name, description, property type, furnishing
- bedrooms, bathrooms, area and unit
- monthly rent, security deposit, maintenance amount/frequency, brokerage, configured fee
- amenities, highlights, image object keys/URLs and order
- location: address, locality, city, state, PIN, country, optional landmark, GeoJSON coordinates
- availability date/minimum stay if used
- publication/moderation/availability status and optional verification metadata
- timestamps
- Embed location initially unless a separate collection is justified. Use consistent GeoJSON and a `2dsphere` index for proximity searches; do not mix coordinate order.

### Favorite
- tenant reference, property reference, created timestamp
- unique compound index on tenant + property

### Inquiry
- property, tenant, owner references; message/contact preference; status; timestamps
- enforce private access

### Application
- property, tenant, owner references; application date; status (`pending`, `approved`, `rejected`, `withdrawn`, `cancelled` as appropriate)
- tenant-supplied application details, optional message, status history, optional lease reference, timestamps
- prevent duplicate active applications according to defined rules

### Lease
- property, tenant, owner, originating application references
- start/end dates; agreed rent/deposit terms copied at creation; optional payment schedule
- status such as draft, pending-signature, active, completed, terminated/cancelled
- timestamps; prevent unintended duplicate active leases

### Payment
- lease reference when lease-related; optional application/platform purpose reference
- payer and intended recipient references as applicable
- purpose, amount in minor units, currency, local/gateway status
- gateway order/payment/event IDs, due/paid dates, refund metadata, idempotency keys, safe metadata
- timestamps and unique constraints for provider IDs/processed webhook events

### Conversation / Message (later)
- participants, optional property/application reference, sender, content, timestamp
- read state only if implemented
- check conversation membership for every read/send/subscribe action

### PropertyReport / AuditLog
- reports: reporter, target, reason, status, reviewer, timestamps
- audit: actor, action, target type/ID, timestamp, non-sensitive metadata

## 9. API conventions

- All routes under `/api`.
- Consistent success/error response shapes, e.g. `{ success, data, message }`.
- Validate body, params, and query strings.
- Paginate search/admin lists; use safe sort allowlists.
- Centralized error handling; no production stack traces/secrets.
- Health/readiness endpoints must not reveal secrets.
- Keep route handlers thin; business rules in services.
- Use public DTOs so password hashes/private fields never leak.
- Add indexes for unique email, favorites, common filters, references, geospatial lookup, gateway IDs, and audit queries as justified.
- Status changes use explicit transition rules.

Route families (implement only in assigned chunks):
- `/api/health`
- `/api/auth` — register, OTP verify/resend, login/logout/session, current user
- `/api/profile` or `/api/users` — own profile
- `/api/properties` — public list/detail/search; owner CRUD/archive
- `/api/favorites`
- `/api/inquiries`
- `/api/applications`
- `/api/leases`
- `/api/payments` and `/api/webhooks/<provider>`
- `/api/admin`
- `/api/ai`
- `/api/conversations` plus Socket.IO events later

## 10. Page inventory

### Public
Home, search results, property detail, about/help/policies as needed, signup, email verification, signin, unauthorized/not-found/error.

### Tenant
Dashboard, profile/settings, favorites, applications list/detail/status, residences list/detail, lease detail, payment history/checkout/result, inquiry/chat inbox later, AI assistant and natural-language search.

### Owner
Dashboard, profile/settings, my properties, create/edit property, property detail/manage, image manager, applications inbox/detail and approve/reject, leases/payment records, AI listing draft/improvement, inquiry/chat inbox later.

### Admin
Dashboard, users list/detail, property moderation, reports/complaints queue, owner/property verification review if enabled, audit logs.

### UX requirements
- Consistent navigation, loading/empty/error/success/disabled states.
- Responsive, keyboard-accessible controls, visible focus, labels.
- Actionable form validation; confirmation for destructive actions.
- No fake dashboard numbers once API data exists.
- Seed/demo data must be development-only and labelled as demo.
- Cohesive UI; avoid unnecessary animation and overengineering.

## 11. Suggested repository structure

```text
HAVEN/
  project.md
  README.md
  .gitignore
  .env.example
  client/
    package.json
    vite.config.ts
    src/
      app/ components/ features/ layouts/ pages/ routes/
      services/ store/ types/ lib/ styles/ assets/
  server/
    package.json
    src/
      app.ts server.ts config/ middleware/ models/ services/
      validators/ utils/ jobs/ tests/
      modules/
        auth/ users/ properties/ favorites/ inquiries/
        applications/ leases/ payments/ admin/ ai/ messaging/
  docs/
    architecture.md
    api.md
    payment-flow.md
```

This is guidance, not permission to restructure blindly. Inspect the workspace in Chunk 0 and adapt if a working repository already exists.

## 12. Environment and secrets

Create `.env.example` with placeholders only; never commit real secrets.

Possible server variables:
```env
NODE_ENV=development
PORT=8080
CLIENT_URL=http://localhost:5173
MONGODB_URI=
JWT_SECRET=
JWT_ACCESS_TTL=
EMAIL_HOST=
EMAIL_PORT=
EMAIL_USER=
EMAIL_PASSWORD=
EMAIL_FROM=
STORAGE_PROVIDER=local
S3_ENDPOINT=
S3_REGION=
S3_BUCKET=
S3_ACCESS_KEY_ID=
S3_SECRET_ACCESS_KEY=
S3_PUBLIC_BASE_URL=
MAPBOX_TOKEN=
AI_PROVIDER=
AI_PROVIDER_API_KEY=
AI_MODEL=
PAYMENT_PROVIDER=razorpay
PAYMENT_KEY_ID=
PAYMENT_KEY_SECRET=
PAYMENT_WEBHOOK_SECRET=
```

Possible client variables:
```env
VITE_API_BASE_URL=http://localhost:8080/api
VITE_MAPBOX_TOKEN=
VITE_PAYMENT_PUBLIC_KEY=
```

Anything prefixed `VITE_` is public. Never place database credentials, JWT secrets, SMTP passwords, AI keys, gateway secret keys, or webhook secrets in frontend variables. Only expose a provider public/key ID if its integration requires it. Document required/optional variables and behavior when integrations are not configured. Use separate sandbox/live credentials.

## 13. Testing and quality gates

### Backend
- Unit tests for business rules/validators.
- API integration tests and MongoDB-backed workflow tests.
- Role and resource ownership tests for every protected resource.
- OTP expiry/reuse/attempt limit, login failures, malformed IDs, and validation tests.
- Property filter combinations, pagination, favorite uniqueness, application transitions, lease creation rules, payment calculation tests.
- Payment tests: invalid signatures, duplicate/out-of-order webhooks, wrong amount/currency, unknown order, failure/cancel/refund states.
- AI tests with mocked provider responses, malformed output, timeouts, and provider failures.

### Frontend
- TypeScript check and production build.
- Tests for important forms/components where test setup exists.
- Manual public/tenant/owner/admin journey walkthrough.
- Responsive, loading, empty, and error-state checks.

### Every chunk
- Run relevant tests plus available type-check/lint/build commands.
- Report actual commands and outcomes; never claim unrun tests passed.
- If a service or secret is unavailable, say exactly what couldn't be verified and how to verify it next.

## 14. Magic Door chunk plan

**Only one chunk per approval cycle. Complete it, test it, report, then stop and wait for explicit approval.**

### Chunk 0 — Workspace inspection and plan (START HERE)
- Inspect repository structure, package manager, dependencies, env files, Git status, tests, runtime versions, MongoDB availability, and the PDF/spec.
- Do not delete, overwrite, refactor, or install dependencies during inspection.
- Map existing files to architecture; identify conflicts and missing pieces.
- Present proposed structure, dependencies, scripts, environment variables, risks, implementation order.
- Identify decisions still needed: payment purposes/settlement, gateway account readiness, owner/property approval policy, accommodation scope, contact workflow, language scope.
- Wait for approval before Chunk 1.

### Chunk 1 — Foundation
- Create/adapt TypeScript client/server foundations and scripts for dev/build/lint/type-check/test.
- Express JSON/CORS for known client origin, centralized errors, safe request logging, health route.
- Vite/React entry, router, global styles, Tailwind and UI primitives.
- `.gitignore`, `.env.example`, README setup.
- Verify client/server start and client reaches health endpoint. No business features yet.

### Chunk 2 — Database and shared API utilities
- MongoDB connection, lifecycle/error handling, Mongoose setup.
- Response/error helpers, validation strategy, not-found handler, request IDs/safe logging as practical.
- Test setup and health/database test.
- Document local MongoDB setup.

### Chunk 3 — Auth and email OTP
- User schema, normalized email, password hashing, registration, OTP send/verify/resend with expiry, attempt limit, cooldown and one-time use.
- Login, JWT, current-user endpoint, logout/session strategy.
- Auth/role middleware, safe admin provisioning.
- Email adapter with development-safe behavior.
- Tests for duplicate email, invalid/expired/reused OTP, login failures, token failures, role denial.
- Choose/document secure cookie or Bearer-token strategy and its tradeoffs; do not store tokens carelessly.

### Chunk 4 — Profiles and role application shell
- Tenant/owner own-profile read/update.
- Protected route layouts and role-aware navigation.
- Shared header/footer/sidebar/dashboard shell and common loading/error/empty states.
- Test cross-user access and profile validation.

### Chunk 5 — Property model and owner CRUD
- Property schema, address/location, media metadata, indexes/enums.
- Owner create/read/update/publish/unpublish/archive.
- Public list/detail returns only listings permitted by publication policy.
- India-specific types, furnishing, beds/baths/area, itemized costs, amenities, availability and address form.
- Ownership validation, pagination, tests. No AI required yet.

### Chunk 6 — Images and storage
- Storage adapter, local dev adapter and S3-compatible adapter.
- Safe upload flow, validate size/MIME/content/count, ownership, ordering and removal.
- Store object keys/safe URLs; handle orphaned uploads where possible.
- Test invalid uploads and owner-only modifications; never expose storage secrets.

### Chunk 7 — Discovery, filters, sort, maps
- Listing cards, search page, property details.
- URL filters for city/locality, rent bounds, beds/baths, type, area bounds, amenities, furnishing, availability, geographic radius.
- Pagination and safe sort allowlist.
- Mapbox location search and markers, GeoJSON/geospatial index, correct coordinate order.
- Empty/no-results/loading/error states and combined-filter tests.
- Never leak unpublished listings.

### Chunk 8 — Favorites
- Tenant add/remove/list favorites.
- Unique tenant-property pair and idempotent behavior.
- Favorite page and property-card toggle.
- Test unauthenticated access, duplicates, and cross-tenant access.
- Saved searches are optional and must not block core favorites.

### Chunk 9 — Inquiries and applications
- Private inquiry/contact form.
- Tenant submits applications; tenant sees own applications/status.
- Owner sees applications only for owned listings.
- Valid status transitions, status history, approve/reject.
- Prevent duplicate active applications as policy dictates.
- Do not create lease until approved/business conditions are met.
- Tests for role/ownership and invalid transitions.

### Chunk 10 — Lease and residences
- Create lease from approved application with correct tenant/owner/property references.
- Store agreed rent/deposit terms and dates.
- Tenant residences and lease detail; owner lease list/detail.
- Valid lease transitions and no accidental duplicate active leases.
- Contract/signature flow out of scope unless separately approved.
- Tests for access, dates, amounts, and application relationship.

### Chunk 11 — Real payment gateway
- Document gateway account, payment purpose, recipient/settlement model, fees/refunds, sandbox/live setup before live processing.
- Provider adapter, payment model/service, server-created order, checkout/result UI.
- Backend-calculated amount, signature verification, signed webhook, idempotency, event persistence, retries and safe status transitions.
- Payment history/detail; supported pending/success/failure/cancel/refund states.
- Test invalid signatures, duplicate/out-of-order webhooks, amount/currency mismatch, retries.
- Start in sandbox. Live mode needs merchant credentials/approval and confirmed settlement model. If unavailable, say sandbox-ready, not live.

### Chunk 12 — Admin, reports, moderation, audit
- Admin-only routes and UI.
- Real DB-backed metrics.
- User search and safe suspend/reactivate.
- Property moderation, report queue, complaints.
- Optional owner/property verification based on approved policy.
- Audit consequential actions; confirmations for destructive operations.
- Tests proving tenant/owner cannot use admin endpoints.

### Chunk 13 — AI assistant and natural search
- Backend provider abstraction, schema validation, rate limits, timeouts and errors.
- Retrieve real property records for property-specific assistant answers.
- Natural-language request to validated filters then ordinary search service.
- Unsupported/no-result behavior and non-AI fallback.
- No exposed keys or model-generated database queries.
- Mock-provider tests and example “furnished 1BHK in Bhopal under ₹8,000.”

### Chunk 14 — Recommendations and AI owner tools
- Explainable ranking based on actual user preferences and listing data.
- AI description drafts and listing improvement suggestions.
- Use only owner-supplied facts; flag missing facts.
- Owner reviews and explicitly saves generated text.
- Test malformed output, fabricated fields, provider failure, and authorization.

### Chunk 15 — Real-time messaging
- Conversation/message models and legitimate tenant-owner context.
- Authenticated Socket.IO connections; membership checks on every read/send/join.
- Persist messages and derive sender identity server-side.
- Handle reconnects, ordering, validation, rate limits and errors.
- Test unauthorized conversation access and sender spoofing.

### Chunk 16 — Integration, accessibility, security, E2E
- Walk through tenant, owner and admin journeys.
- Review every sensitive route for role and ownership checks.
- Review validation, rate limits, CORS, secrets, log redaction, uploads and payment webhooks.
- Fix mobile/responsive/accessibility issues.
- Remove debug logs, dead code, fake values, misleading placeholders.
- Verify AI-disabled and payment-sandbox modes.
- Add labelled demo seed data and safe reset instructions.
- Run full test/type-check/lint/build and report actual results.

### Chunk 17 — Documentation, CI, deployment preparation
- README setup/scripts/env/architecture/testing/seed/troubleshooting.
- API and status-transition documentation.
- Payment sandbox/webhook guide and live-mode prerequisites.
- GitHub Actions for install, lint, type-check, tests and build.
- Review deployment configuration and secrets.
- Deployment only after approval and successful local workflows.
- Docker optional; no unnecessary infrastructure.
- Do not claim production readiness without deployment, security and payment compliance verification.

## 15. Definition of done

The complete local version should provide:
- Public discovery, filters, property detail and maps.
- Email verification and secure tenant/owner login; securely provisioned admin.
- Owner listing CRUD and protected image upload.
- Tenant favorites, inquiries, applications and status tracking.
- Owner application decisions and lease creation.
- Tenant residences, lease details and payment history.
- Real gateway integration with server-verified states and sandbox tests; live money movement only after merchant setup and settlement decisions.
- Admin moderation, reports, metrics and audit logs.
- Five AI capabilities with hallucination safeguards.
- Optional real-time messaging.
- Authorization, validation, errors, tests, reproducible setup and CI.
- Honest notes for integrations that remain unconfigured.

## 16. Antigravity operating rules

1. Read `project.md` and inspect workspace before changes.
2. Start with **Chunk 0 only**.
3. Do not implement multiple chunks per approval cycle.
4. Preserve existing work; inspect Git status and relevant files first.
5. Explain dependencies and compatibility before installing.
6. Prefer simple, readable, maintainable code.
7. Validate on frontend and backend; frontend validation is not security.
8. Add tests with features.
9. Report actual command/test outcomes only.
10. If blocked, report exact error, what was tried, and smallest next step.
11. Never silently omit requirements or fake test/payment success.
12. At chunk end, report: implementation; changed files; dependencies; commands/outcomes; test/build results; limitations/decisions; manual verification; next chunk objective.
13. Stop and wait for explicit approval before continuing.
14. Do not deploy, create paid resources, use live credentials, or perform destructive data actions without explicit approval.
15. Do not change this specification unless requested.

## 17. Start instruction

**Start Chunk 0 only.** Inspect the workspace and report the proposed implementation plan, dependency list, existing-work risks, environment requirements, and unresolved product/payment decisions. Do not install dependencies or implement Chunk 1 until the user explicitly approves.

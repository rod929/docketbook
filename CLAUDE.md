# Docketbook — project guide for Claude Code

## What this is
Docketbook ("Day Labour Book") is a mobile-first web app for recording, signing,
and approving construction day-labour dockets. It is a SINGLE self-contained file:
`index.html` (~6,700 lines, HTML + CSS + JS in one file). Deployed to GitHub Pages
at https://rod929.github.io/docketbook/.

The owner (Rod) is non-technical. Explain changes in plain language. Keep edits
surgical — do not refactor or restructure the file unless asked.

## CRITICAL: how to work in this file
- After EVERY change to index.html, extract the main <script> block and run
  `node --check` on it to confirm there are no syntax errors before finishing.
- Build ONE feature at a time. After each, summarise what changed and where.
- Do not introduce build tools, frameworks, or npm packages. It must stay a
  single static HTML file that runs by opening it in a browser.

## Deploying
- Deploy = commit and push to the GitHub repo; GitHub Pages serves it.
- Always remind: test on the LIVE https URL with a hard refresh (Cmd/Ctrl+Shift+R)
  or an incognito window — never the local file:// (localStorage is blocked there,
  so logins/saves won't persist and caching hides updates).

## Roles & access (five tiers)
1. Main Administrator (Owner) — full access; the ONLY one who can grant the PM/CA
   level. Allocates access for everyone.
2. Administrator — PM / Contracts Admin — final docket sign-off; can allocate
   workers and standard admins (but NOT other PM/CAs).
3. Administrator (standard) — view, create own, settings; allocate workers; NO
   final sign-off; NO Users tab.
4. Supervisor (hc_supervisor) — signs dockets on site; sees all dockets on
   allocated projects; NO money values.
5. Worker — fills out own dockets + drafts; NO money values.
Implemented via `adminTier` ('owner' | 'pmca' | 'std') on admin users, with
helpers: adminTierOf(), isOwner(), canApproveDockets(), canAllocateAccess().

## Money rules (non-negotiable)
- ONLY company administrators see dollar values. Workers, supervisors, the signing
  page, and the emailed/contracts-admin copy show hours/crew/work/materials
  DESCRIPTIONS/signatures only — NO rates, NO totals, NO material costs.
- The money gate is strictly `CU.role==='admin'` (never `!=='worker'`).
- 10% materials purchasing charge: per-docket tick box, admin-only, shown ONLY
  when the docket has materials.
- All workers on a docket share ONE hourly rate.
- Up-chain (mirrored) dockets are charged at the CLIENT rate, not the sub pay rate.

## Docket lifecycle
Created → Signed (supervisor) → Billing assigned (chargeable / backcharge / own
cost) → Awaiting final approval (PM/CA) → Approved (firm committed cost) or
Queried (sent back with a note) → Invoiced (future).

## Cross-company flow
Signing chain: Head Contractor (Builder, no app) signs the Main Sub supervisor's
docket; the Main Sub supervisor (has app) signs the lower sub's worker docket.
A lower sub on one project may be a main sub on another — so no fixed tier is
stored; the mirror always goes signed docket → signer's own company docket for
the Builder above. When a supervisor signs a DIFFERENT company's docket it lands
as Pending in both the supervisor's and admin's "Pending billing" folder, with a
committed-cost total + ageing badges (cashflow early-warning).

## Signing & distribution (the cashflow heart)
How a docket reaches the supervisor to sign — two paths, both valid:
  1. In person — sub worker shows the docket on their phone; supervisor signs there.
  2. Remote — sub worker texts/emails a signing link; supervisor opens it and signs.
The signing method is independent of cashflow visibility.

THE SIGNATURE IS THE CASHFLOW TRIGGER. Visibility hangs off Event A (signed, today),
not Event B (sub invoices, weeks later). The instant a docket is signed it appears
on the company admin's dashboard (Pending billing) with an estimated value + a
"signed today" ageing badge — so the admin knows the cost is coming before the sub
invoices. End-to-end (post-backend): sub fills out → supervisor signs (trigger) →
appears instantly on admin dashboard → tagged chargeable/backcharge/own cost →
PM/CA approves (firm committed cost) → month-end the sub invoice reconciles against
the already-known docket.

Copies on signing — configurable in Settings → Company setup ("Who receives a copy
when a docket is signed?"), NOT hard-coded:
  - The signer — on/off toggle. Money-free record copy (no rates) to whoever signed.
    Signer email captured at signing (profile if logged in, or entered on remote page).
    Default: on.
  - Company administrators — a recipient list the admin builds (Main Admin, PM/CA,
    or specific people). An alert that a new day-labour cost exists; MAY include the
    cost estimate (admins see money). Default: Main Administrator on the list.
The two payloads differ: signer = money-free record; admin = alert with estimate.

## Known limitation (don't "fix" by faking it)
All data is in each browser's localStorage — there is NO shared server. So true
cross-device / cross-company sync does NOT work yet; the same-device versions do.
Truly automatic server-sent emails/alerts on signing also need the backend; today
they can only be one-tap pre-filled emails. The real fix is a backend
(Supabase/Firebase). All cross-company logic is built to slot in once that exists.
Be honest about this rather than implying it works.

## Style preferences
- Bigger touch targets; orange (#EF9F27) rounded buttons; mobile-thumb-friendly.
- Palette: #2D2D2D dark, #EF9F27 orange, #185FA5 blue, #1D9E75 green.
- Iterative single-feature requests; build-then-test-on-phone loop.
- Honest caveats about the backend/localStorage limits are wanted, not glossed over.

## Current state & outstanding work (handover into Claude Code)
The app (index.html) is a working prototype. Recently completed and verified:
- Multiple workers per docket; enlarged Review & Sign; money hidden from all
  non-admin views; per-docket 10% materials fee; per-worker admin costing.
- Five-tier role model (owner/pmca/std admin + supervisor + worker) with
  Owner-controlled access allocation; PM/CA approval gate (Approve / Query /
  override-billing); committed-cost total + ageing badges (cashflow warning).
- Free docket (admin, no project); cross-company pending-billing flow with
  client-rate up-chain mirror + "from sub #" reference badge.
- Copies on signing (Settings → Company setup): "email a copy to the signer"
  toggle + configurable "company administrators to alert" list. On signing,
  distributeSignedCopies() builds the recipient list, stamps d.copiesQueued,
  and offers a one-tap mailto. Auto-send is a backend feature (see below).

Verify-on-deploy note: the Administrator role option + PM/CA tier selector ARE
in the code. If they don't show live, it's a stale upload/cache — re-deploy this
exact index.html and hard-refresh / open incognito.

OUTSTANDING WORK (in priority order):
1. Two PDF docket layouts — worker no-cost / admin full-cost. Mockups approved
   and saved as docket_worker_copy.html and docket_admin_copy.html. Wire into
   printD()/sendDocketEmails(), gated by role (admin sees costs, others don't).
   Buildable now; likely needs html2canvas via CDN, so test on the live site.
2. BACKEND (Supabase) — the big unlock. Replaces the localStorage data layer so
   data syncs across devices and companies. Makes real: cross-device signing,
   the instant admin alert, auto-sent copies-on-signing emails, and persistence.
   Must be built MULTI-TENANT (each company's data isolated via row-level
   security) because the business model is multi-company SaaS. Add docket
   metering for billing. All cross-company/approval logic already anticipates
   this — switch the data layer, keep the logic.
3. Before production: remove demo logins; add proper auth/security.

Business context (for reference, not for the app code): pricing is $80/month per
subcontractor for 50 dockets; builders get free access initially then a smaller
fee; targets are laddered (~0.25–0.5% of AU construction businesses at 2yr, ~1%
growth, ~5% vision ceiling). See Docketbook_Business_Plan.docx.

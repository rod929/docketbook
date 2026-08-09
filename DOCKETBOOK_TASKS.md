# Docketbook — task list (Rod's punch list)

Work through these ONE AT A TIME in Claude Code. After each: run `node --check`,
show the diff, push to the deploy branch (or hand Rod the file to upload to main),
and test on the LIVE https site with a hard refresh. Keep the money rules and role
tiers from CLAUDE.md intact on every change.

Legend:  [ ] todo   [~] in progress   [x] done   ⚠ = needs backend to fully work

---

## 1. OVERALL
- [x] **1. Languages regression.** Spanish + Mandarin/Chinese language options must
  appear and work on EVERY part of the app (worker, supervisor, admin, new docket,
  dashboards, etc.). This was working before and has regressed — restore it
  everywhere, and make sure dynamically-generated text also translates.
  DONE: switcher added to admin + site-user/supervisor/manager banners and the
  New Docket form for all roles; nav/lists/status badges re-translate on switch.

## 2. NEW DOCKET
- [x] **2. Company name hard-coded.** The "your company" field should be fixed to
  the registered company name (appSettings.name), not free-typed.
  DONE: setCompanyField() locks the field per role — worker→parent sub, sub→own
  company, admin/supervisor/site user→appSettings.name; never free-typed.
- [x] **3. Prefill staff member.** "Requested by" / staff member should be
  pre-filled (even for the trial data).
  DONE: seed trial staff (Site Manager / Project Manager / Foreman) and
  prefillReqBy() auto-selects a default (logged-in user if on the list, else the
  first staff member) whenever the field is blank.
- [x] **4. Per-worker times.** When adding additional workers, allow each to have
  their own start/finish time, OR a "copy same hours as main worker" tick box.
  (A "same hours for all" tick already exists — extend to per-worker start/finish.)
  DONE: each additional worker row has Start/Finish inputs that auto-calc their
  hours; "Same hours for all" disables those and copies the main hours. Times are
  saved per worker (workersOnSite[].start/finish).

## 3. DOCKET PDF  (everything must fit on ONE A4 page)
- [x] **5. PDF setout order:** DONE — Job Details (adds Client, drops start/finish)
  → Labour on Site (renamed, with per-worker Time column) → Description → Billing
  & Client → Sign-off; PDF now scales to fit a single A4 page.
  1. Job details — keep, PLUS add Client, MINUS start/finish time.
  2. Workers on site section (rename it to something better) — include start/finish
     time here.
  3. Description of works.
  4. Billing and Client.
  5. Signoff.
  - All of the above on a single A4 page.

## 4. DASHBOARD (admin)
- [~] **6. Restyle admin dashboard** to match the other dashboards (same fonts,
  boxes, appearance). It's currently messy with too many sections — declutter.
  PARTIAL: decluttered the mobile quick-nav (7 buttons → 4, dropping the ones
  already in the bottom nav) and restyled them with the shared .ndock-tile look;
  added a New docket button. A fuller visual pass needs Rod's eyes on a phone.
- [x] **7. Subcontractor pending dockets** section — dockets signed by our
  supervisors but not yet sent by the subcontractor.
  DONE: this is the existing "Pending billing" folder (signed by our supervisor,
  pending); retitled "Subcontractor pending dockets" with a clarifying subtitle.
- [x] **8. Admin final sign-off** section — all dockets signed by a supervisor but
  awaiting final admin sign-off; on approval, alert whoever submitted the docket so
  they can invoice.
  DONE: the "Awaiting final approval" folder already lists these; approveDocket now
  posts an in-app notice to the submitter ("approved — ready to invoice") shown as
  a banner on their dashboard. (Cross-device alert needs the backend.)
- [x] **9. Admin can create a docket** (same options as workers/supervisors).
  DONE: added a prominent "New docket" button to the admin dashboard. The form
  already supports admins (company locked to appSettings.name; billing chooser is
  hidden for admin/worker/supervisor alike, consistent with the others).

## 5. SUPERVISOR SIGNOFF
- [x] **10. Per-line-item notes.** Supervisor can add notes to each line item /
  description he's signing (worker may not have given enough info).
  DONE: a "Supervisor note" field on the Review & Sign page, saved on the docket
  (d.supervisorNote) and shown in the docket view + PDF. NOTE: a docket has one
  description (not multi-line items); true per-line notes would need the
  description restructured into line items — flagged for Rod.
- [x] **11. ⚠ Auto-email includes PDF attachment.** When signed by supervisor and
  submitted, the auto email should also carry the PDF. NOTE: mailto: cannot attach
  files — true attachment needs the backend. Build the logic/UI now; it attaches
  once the backend lands.
  DONE (interim): the signed-copy email now generates + downloads the money-free
  PDF and the body tells the sender to attach it, noting auto-attach arrives with
  the backend. Same PDF the backend will attach automatically.
- [x] **12. Sign-later prompt.** If the supervisor didn't sign on site and signs at
  a later date, auto-ask if it's chargeable / backcharge / etc. (logic exists for
  the pending-billing flow — wire it to the later-signing path).
  DONE: remote/later signing (submitRemoteSig) now flags d.signedLater, keeps the
  docket pending-billing, notifies the submitter, and surfaces it in the admin
  pending-billing folder's chargeable/backcharge/own-cost chooser.

## 6. PASSWORD
- [x] **13. ⚠ Forgot-password / recovery.** A complete password recovery setup.
  NOTE: real recovery (email reset links) needs the backend. Build the UI now; full
  function lands with the backend.
  DONE: the working security-question recovery already exists (Forgot password →
  verify username → answer → set new password, offline). Added the backend-deferred
  "Email me a reset link" option (looks up the account email, masks it, explains it
  activates with the backend). Admin one-click reset also already exists.

---

## Suggested order
1 (languages regression — quick, it broke) → 2, 3, 4 (new docket) → 5 (PDF setout)
→ 6, 7, 8, 9 (admin dashboard — biggest chunk) → 10, 12 (supervisor signoff) →
defer 11 and 13 until the backend.

## Backend-dependent (don't expect full function pre-Supabase)
- 11 (email PDF attachment)
- 13 (real password recovery)
Both: build UI/logic now, fully works once the backend exists.

## Post-list work (done after the numbered punch list)
- [x] **Clean demo docket set + My-dockets status filter.** Versioned one-time
  reset (DEMO_DATA_VERSION) wipes dockets and seeds 4 purposeful dockets
  (#0001–#0004), one per workflow state. My dockets gained a "Filter by status"
  dropdown and a workflow Status badge column (docketStatus()/statusBadge()).
- [x] **Worker sign/draft restriction.** In Review & Sign, workers now only see
  "Sign on site" plus a "Save as draft" button — the remote email/SMS send tabs
  are hidden and blocked for workers (that path belongs to the supervisor/admin).
- [x] **Ready to sign list.** Admin/supervisor/sub can "Prepare for a worker" from
  Review & Sign — fills a docket on the worker's behalf into that worker's new
  "Ready to sign" tab (top nav + mobile bottom nav, with a count badge). The
  worker opens it straight into Review & Sign to capture the signature; it
  clears on submit. Stored in dlb_ready. ⚠ Cross-device routing to the correct
  worker's phone needs the backend (localStorage is per-device) — flagged in-app.

- [x] **Query / reject with reasons + resolve & re-submit.** When approving,
  the "⚠ Query" button now opens a modal: choose Query (needs info, comes back)
  or Reject (formal), tick reason chips (more photos / clarify hours / not in
  scope / wrong rate / missing details / duplicate) and add a note. The docket
  view shows the reasons + note prominently to our side (submitter, signing
  supervisor, admin) with a "↩ Resolve & re-submit" button → answer the query,
  attach photos, re-submit (status back to Awaiting approval). A running
  query/response history is kept on the docket (queryThread). In-app notices are
  colour-coded (red query/reject, blue re-submitted, green approved) and click
  through to the docket. ⚠ Cross-company/device delivery of these alerts needs
  the backend.

- [x] **Signature pads lock until tapped.** Every signature box (Review & Sign,
  new-docket form, docket-view sign-on-screen, and the remote signing page) now
  shows a "✍️ Tap here to sign" overlay and only accepts a signature once tapped
  — so a stray touch or a scroll over the pad can't leave an accidental mark.
  Clearing re-locks it.

- [x] **Subcontractor dockets received (supervisor dashboard).** A new section
  listing every sub docket on the supervisor's projects, with a status badge on
  each. Ones awaiting the supervisor's signature sort to the top with a
  "Review & sign" button (opens the sign-on-screen pad); already-handled ones
  show their status with a "View" button. No money shown. Sits above Pending
  billing (received = before signing; pending billing = after signing).

## MUST-FIX BEFORE GO-LIVE (needs backend)
- **Notifications to close out open items (needs-billing / needs-forwarding /
  awaiting-signature).** Optimal strategy, all backend-dependent:
  1. PUSH (primary) — app installed to home screen (PWA groundwork added: inline
     manifest + icon, single-file). Backend + service worker sends free,
     instant, phone-native push that deep-links to the item.
  2. SMS (escalation only) — for the time-sensitive cashflow event (signed →
     needs billing) or if a push goes unread; costs per message, use sparingly.
  3. EMAIL (digest) — daily "outstanding items" summary, mainly for the office.
  Rule: batch/digest, don't fire one-per-event (notification fatigue). Today the
  app can only alert IN-app when opened; proactive reach-out needs the backend.
- **Cross-device sync — the cashflow trigger (CORE).** Admin and supervisor are
  on DIFFERENT devices. The instant "our supervisor signs a docket → it appears
  on the admin's dashboard (Pending billing) with the cost estimate" must work
  across phones, not just on one device. Today all the logic is built
  (signedBySupervisor + bill:pending routing, both dashboards, distributeSignedCopies
  admin alert), but delivery is per-device localStorage. The backend (shared
  server storage) makes it real: sign on phone A → visible on phone B instantly.
  This is the product's core value; MUST work at go-live. (Owner confirmed
  different devices, 2026-08-09.)
- **Docket-number uniqueness per company.** Each company runs its own docket
  book (Company A #0001 and Company B #0001 is fine), but WITHIN a company a
  number must be used once — no matter who creates it (worker / supervisor /
  admin) or on which device. Today the counter is per-device (v4_ctr), so it's
  unique on a single device but two phones in the same company can both make
  #0001. The fix: the backend keeps one docket counter PER COMPANY and hands out
  the next number centrally. Acceptable to leave during dev; MUST be fixed before
  production. (Owner flagged 2026-08-09.)

# Docketbook — task list (Rod's punch list)

Work through these ONE AT A TIME in Claude Code. After each: run `node --check`,
show the diff, push to the deploy branch (or hand Rod the file to upload to main),
and test on the LIVE https site with a hard refresh. Keep the money rules and role
tiers from CLAUDE.md intact on every change.

Legend:  [ ] todo   [~] in progress   [x] done   ⚠ = needs backend to fully work

---

## 1. OVERALL
- [ ] **1. Languages regression.** Spanish + Mandarin/Chinese language options must
  appear and work on EVERY part of the app (worker, supervisor, admin, new docket,
  dashboards, etc.). This was working before and has regressed — restore it
  everywhere, and make sure dynamically-generated text also translates.

## 2. NEW DOCKET
- [ ] **2. Company name hard-coded.** The "your company" field should be fixed to
  the registered company name (appSettings.name), not free-typed.
- [ ] **3. Prefill staff member.** "Requested by" / staff member should be
  pre-filled (even for the trial data).
- [ ] **4. Per-worker times.** When adding additional workers, allow each to have
  their own start/finish time, OR a "copy same hours as main worker" tick box.
  (A "same hours for all" tick already exists — extend to per-worker start/finish.)

## 3. DOCKET PDF  (everything must fit on ONE A4 page)
- [ ] **5. PDF setout order:**
  1. Job details — keep, PLUS add Client, MINUS start/finish time.
  2. Workers on site section (rename it to something better) — include start/finish
     time here.
  3. Description of works.
  4. Billing and Client.
  5. Signoff.
  - All of the above on a single A4 page.

## 4. DASHBOARD (admin)
- [ ] **6. Restyle admin dashboard** to match the other dashboards (same fonts,
  boxes, appearance). It's currently messy with too many sections — declutter.
- [ ] **7. Subcontractor pending dockets** section — dockets signed by our
  supervisors but not yet sent by the subcontractor.
- [ ] **8. Admin final sign-off** section — all dockets signed by a supervisor but
  awaiting final admin sign-off; on approval, alert whoever submitted the docket so
  they can invoice.
- [ ] **9. Admin can create a docket** (same options as workers/supervisors).

## 5. SUPERVISOR SIGNOFF
- [ ] **10. Per-line-item notes.** Supervisor can add notes to each line item /
  description he's signing (worker may not have given enough info).
- [ ] **11. ⚠ Auto-email includes PDF attachment.** When signed by supervisor and
  submitted, the auto email should also carry the PDF. NOTE: mailto: cannot attach
  files — true attachment needs the backend. Build the logic/UI now; it attaches
  once the backend lands.
- [ ] **12. Sign-later prompt.** If the supervisor didn't sign on site and signs at
  a later date, auto-ask if it's chargeable / backcharge / etc. (logic exists for
  the pending-billing flow — wire it to the later-signing path).

## 6. PASSWORD
- [ ] **13. ⚠ Forgot-password / recovery.** A complete password recovery setup.
  NOTE: real recovery (email reset links) needs the backend. Build the UI now; full
  function lands with the backend.

---

## Suggested order
1 (languages regression — quick, it broke) → 2, 3, 4 (new docket) → 5 (PDF setout)
→ 6, 7, 8, 9 (admin dashboard — biggest chunk) → 10, 12 (supervisor signoff) →
defer 11 and 13 until the backend.

## Backend-dependent (don't expect full function pre-Supabase)
- 11 (email PDF attachment)
- 13 (real password recovery)
Both: build UI/logic now, fully works once the backend exists.

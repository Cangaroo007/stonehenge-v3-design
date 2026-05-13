# Assumptions register — for review with Jay Henderson (Northcoast Stone)

**Purpose:** Every place the Manufacturing design track had to guess at NCS workflow rather than reading it from the spec or the FMS-21 screenshots. Jay's the domain authority. His answers either confirm an assumption (we lock it in the §34 amendment) or invalidate it (we redesign).

**How to use this with Jay:** This is a checklist for a focused conversation, not a document to send him to read. The questions are the conversation — the assumptions are the context that frames each question.

**Source labelling:**
- `[FMS-21]` = inferred from the screenshots Sean shared
- `[SPEC]` = read directly from spec, not really an assumption, included for completeness
- `[GUESS]` = pure speculation, the highest-priority items to validate
- `[SEAN]` = direction from Sean, taken as given

**Status field:**
- `PENDING` = not yet discussed with Jay
- `CONFIRMED` = Jay has confirmed
- `INVALIDATED` = Jay has pushed back; redesign needed
- `MODIFIED` = Jay has refined; minor adjustment

**Version:** v0.1 · 11/05/2026 · seeded from M01 build · grows as Batch 1–4 progress

---

## Category A — Workflow stages

### A01 — Ten-stage workflow matches NCS today
**Source:** `[FMS-21]` Production Menu screenshot (Image 4 of the FMS-21 set)
**Status:** PENDING
**Assumption:** NCS currently runs a 10-stage lifecycle: Pending → Measure → Office Review → CAD → Production → Quality → Pickup → Installation → Maintenance → Office Completion.
**Risk if wrong:** Columns mismatch real workflow; jobs accumulate in wrong stages; foreman doesn't recognise the board.
**Question for Jay:** Does this 10-stage list exactly match how NCS thinks about a job moving through the shop? Are there any stages that are actually two distinct steps in your head (e.g. "CAD" might really be "Drawing prep" + "Programming")? Are there any that you'd merge?

### A02 — Stage ownership
**Source:** `[FMS-21]` Image 6 (Job Production Summary) shows assigned names per stage: Tyson Brunner (Measure), Melanie Gray (Office Review), Thomas Logan (CAD)
**Status:** PENDING
**Assumption:** Each stage has a designated owner role, not just any-staff-can-progress. Office Review is owned by office admin (Melanie), CAD by Michael Ottaway / Thomas Logan, Production by floor staff.
**Risk if wrong:** Permissions model is wrong; the wrong people see the wrong queues.
**Question for Jay:** Which roles own which stages? Could you give me the rough hierarchy — who's authorised to advance a job from each stage to the next?

### A03 — Stages are sequential, no skips
**Source:** `[FMS-21]` UI suggests linear progression
**Status:** PENDING
**Assumption:** Jobs progress strictly forward through stages. No skipping (e.g. Production directly to Installation without Quality). Mockup enforces drag-to-adjacent-column-only.
**Risk if wrong:** If skips happen in reality (urgent jobs, simple repairs), the UI blocks legitimate work.
**Question for Jay:** Are there cases where a job legitimately skips a stage? E.g. a maintenance ticket that doesn't need CAD? Or a "supply only — no install" job that skips Installation?

### A04 — Backward transitions
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** Jobs can move backward (e.g. Quality back to Production if a defect is found, Office Review back to Measure if measurements are wrong). Backward moves trigger an audit-log entry and possibly a notification.
**Risk if wrong:** Either we over-restrict (no backward moves, real recovery is awkward) or under-restrict (any user can shuffle jobs anywhere, causing chaos).
**Question for Jay:** When something needs rework, does the job physically move back to a previous stage in your system, or does it stay where it is with a remake/issue flag, or do you create a new job?

---

## Category B — Per-piece station routing

### B01 — Per-piece station routing exists
**Source:** `[FMS-21]` Image 14 (Production Matrix) — shows per-piece station list (Main Saw, Polisher, Genny Lin, CNC, Water Jet, Hand Finish, Mitres)
**Status:** PENDING
**Assumption:** Within the "Production" stage, each piece has its own route through fabrication stations. Different pieces in the same job follow different routes.
**Risk if wrong:** Either we build a complex routing UI that NCS doesn't use, or we miss the fact that NCS already routes pieces individually and Production is more granular than V3 spec captures.
**Question for Jay:** When pieces are on the floor, do different pieces in the same job follow different routes through your machines? Is the route decided per-piece, or is there a standard route based on piece type?

### B02 — Station catalogue
**Source:** `[FMS-21]` Image 14 shows seven stations
**Status:** PENDING
**Assumption:** The seven stations are: Main Saw, Polisher, Genny Lin (assume this is a person? or a machine name?), CNC, Water Jet, Hand Finish, Mitres. Each station has an operator.
**Risk if wrong:** Station list is incomplete or includes things that aren't real stations.
**Question for Jay:** Is this the complete list of stations on your floor? What's "Genny Lin" — a machine, a person, or a workstation type? Are there others not shown in this screenshot?

### B03 — Routing template per piece role
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** Each piece role (benchtop, splashback, fascia, strip, etc.) has a default station route. The CAD operator can override per piece.
**Risk if wrong:** If routing is purely ad-hoc per piece, our "template" model is overengineered. If routing is fully automatic with no overrides, our "CAD operator decides" model is too flexible.
**Question for Jay:** When CAD generates the cut card, is the route automatic based on piece type, or does the CAD operator decide each piece's route?

### B04 — Re-routing when station is down
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** When a station is bottlenecked or down (e.g. CNC backed up), work re-routes manually — hand-finish around it, wait, or outsource.
**Risk if wrong:** We may design a more or less sophisticated re-routing UI than is needed.
**Question for Jay:** When the CNC is backed up (or any other station), what actually happens? Does someone manually decide to hand-finish a piece that would normally go on CNC, or does it wait?

---

## Category C — Remakes, rework, defects

### C01 — Remake declaration moment
**Source:** `[FMS-21]` Image 3 — red/orange/pink dot column on the left implies issue tracking
**Status:** PENDING
**Assumption:** A remake is declared at the moment a piece is found defective (during fab, QC, or install). The original piece is marked as defective and a new piece is added to the job with a "remake" flag.
**Risk if wrong:** Workflow for remakes is wrong — either too rigid or too loose.
**Question for Jay:** When a piece chips during polish, what actually happens? Does the operator stop the line, mark the piece as a remake in the system, and start a new piece? Or is it handled informally and only logged later?

### C02 — Cost attribution for remakes
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** Remake cost is attributed to one of: the operator who broke it, the company (general overhead), or the customer (if scope changed). Default is "the company."
**Risk if wrong:** Cost reporting is wrong; operator-level accountability is wrong; pricing implications are wrong.
**Question for Jay:** When a piece is remade, how do you track the cost? Does it come out of someone's pay, eat into job margin, or get logged separately?

### C03 — Remake reason codes
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** Remake reasons fall into categories: operator error, material defect (vein/crack found late), spec change from customer, dimensional measurement error, equipment failure, transport damage.
**Risk if wrong:** Reason code list doesn't match how Jay actually thinks about why remakes happen.
**Question for Jay:** What are the top 3-5 reasons pieces get remade in your shop? Roughly how often does each happen?

### C04 — Current remake rate
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** Rough order of magnitude: 5% of pieces need remaking, with maybe 20% of jobs having at least one remake.
**Risk if wrong:** If remake rate is much higher (e.g. 15-20%), the UI needs to make remakes a primary workflow, not an edge case.
**Question for Jay:** Roughly what percentage of pieces need re-cutting? What percentage of jobs have at least one remake?

---

## Category D — Materials and slab handling

### D01 — Slab reservation timing
**Source:** `[FMS-21]` Image 8 — slabs S-21882, S-21883, S-21884 are "Reserved" against a job pre-fab
**Status:** PENDING
**Assumption:** Slabs are reserved against a specific job between Office Review and Production. The reservation is held until the job is cut.
**Risk if wrong:** V3 spec §10.24 currently defers reservation to Phase 2; if NCS uses reservations daily this is a Day-1 must-have.
**Question for Jay:** When do you reserve specific slabs against a specific job? Is it always done, or only for grain-match jobs, or only for premium materials?

### D02 — Reservation override
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** Operators can pull a different slab than the reserved one if circumstances require (e.g. reserved slab found to have a defect on closer inspection). System tracks the actual slab used.
**Risk if wrong:** Either we prevent legitimate overrides (operator can't substitute) or we don't track substitutions (inventory drift).
**Question for Jay:** If you reserve slab S-21882 for a job and then on the day of cutting it has a vein you don't like, what happens? Do you swap to another slab, and how is that tracked?

### D03 — Grain-match workflow
**Source:** `[FMS-21]` Image 7 (Grain Match Jobs) is its own daily worklist
**Status:** PENDING
**Assumption:** Grain-match jobs sit in a dedicated daily queue between Office Review and Production. Someone (Beau? Jay?) photographs the slabs side by side, decides on cut layout, then advances the job to Production.
**Risk if wrong:** Either we miss the grain-match step entirely or we over-emphasise it.
**Question for Jay:** Walk me through the grain-match step. Who decides? What's the artefact? How do you decide which slabs to pair?

### D04 — Batch tracking
**Source:** `[FMS-21]` Image 9 — slab S-17522 shows batch "2442-012"
**Status:** PENDING
**Assumption:** Every slab has a batch number from the supplier. Tracking batch numbers matters for grain-match (same batch = better colour consistency) and warranty (defect claims back to supplier).
**Risk if wrong:** Either we over-emphasise batch tracking or we miss it.
**Question for Jay:** How important is batch number in practice? Do you specifically grain-match within batch, or across batches if needed?

---

## Category E — Scheduling

### E01 — Foreman dispatch
**Source:** `[SEAN]` Sean's earlier description: "they have a foreman who comes in and he schedules what gets cut when based on the urgency of the project and when it's due"
**Status:** PENDING
**Assumption:** The foreman opens the production queue each morning (or as needed during the day) and verbally tells the floor what to cut next. Decision factors include install due date, material thickness batching, customer priority, who's on shift.
**Risk if wrong:** AI scheduling will replace something simpler or more complex than we think.
**Question for Jay:** Walk me through your foreman's morning routine. What's the first thing he looks at? What's the second? What does he ignore? What decisions does he make in his head versus on paper?

### E02 — Re-plan frequency
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** The foreman re-plans the day's cut schedule once at the start of the day, then adjusts continuously as jobs come in or things go wrong on the floor.
**Risk if wrong:** Either AI scheduling becomes a one-shot morning thing (missing intra-day reality) or constantly running (too noisy).
**Question for Jay:** How often does the day's plan actually change? End-of-shift review? When a new urgent job arrives? When something breaks?

### E03 — AI trust threshold
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** The foreman would trust an AI recommendation if it (a) shows its reasoning ("cut J-14690 first because install Tuesday and 20mm batches with J-14767"), (b) lets him override with one click, and (c) doesn't override his decisions silently.
**Risk if wrong:** AI scheduling gets adopted differently or not at all.
**Question for Jay:** If we built a system that recommended what to cut next and explained why, what would have to be true for your foreman to actually use it? Be honest — even if the answer is "he never would."

---

## Category F — Customer-facing surfaces

### F01 — Install-day notification
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** Customers want to know (a) when their install is scheduled, (b) day-of progress, (c) photos of completed install, (d) sign-off form, (e) warranty info.
**Risk if wrong:** Customer portal scope is wrong.
**Question for Jay:** What do customers actually ask you about between accepting the quote and seeing the installed kitchen? What's the most common phone call or email you get during fab?

### F02 — Photo sharing
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** Customers like progress photos but not too many — one or two during fab is enough to reassure them. Daily photos would be overkill.
**Risk if wrong:** Photo flow is too noisy or too sparse.
**Question for Jay:** Do you currently send progress photos to customers? How often? Has anyone ever asked for more or fewer?

---

## Category G — Regulatory / SWMS

### G01 — SWMS coverage
**Source:** `[FMS-21]` Image 15 — SWMS form with Hazard Identification, Safe Work Methods, Material Safety Data, Quality Checklist, Acceptance
**Status:** PENDING
**Assumption:** Every commercial install job requires a SWMS, signed by stonemason + assistants + truck. Regulatory requirement under AU WHS Act.
**Risk if wrong:** Either we over-engineer for SWMS (every job, including small residential repairs) or under-engineer (missing the regulatory requirement entirely).
**Question for Jay:** Which jobs actually need a SWMS — every install, just commercial, just over a certain size? Has WorkSafe ever audited or reviewed your SWMS records? Anything they pinged you on?

### G02 — Sign-off chain
**Source:** `[FMS-21]` Image 15 shows signature fields for Stonemason, Assistant Stonemasons (×3), Truck
**Status:** PENDING
**Assumption:** Lead stonemason signs first (with date/time), assistants sign in agreement, truck assignment is logged.
**Risk if wrong:** Sign-off workflow doesn't match real on-site practice.
**Question for Jay:** Who actually signs the SWMS on site? In what order? What happens if an assistant refuses to sign because they think a hazard hasn't been properly addressed?

---

## Category H — Mobile / shop-floor

### H01 — Tablet vs phone
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** Foreman and CAD operator use desktop. Floor masons use tablets (cut card reference, mark complete, photo upload). Install crew uses phones (SWMS sign-off, install photos).
**Risk if wrong:** We design mobile UIs for the wrong contexts.
**Question for Jay:** What devices do different people actually use? Where do they use them — at a desk, on the floor, in the truck, on site?

### H02 — Dark mode for shop floor
**Source:** `[SPEC]` §28.6 mentions dark mode for shop floor
**Status:** PENDING
**Assumption:** Workshop lighting and dust make dark-mode displays easier to read on tablets.
**Risk if wrong:** Dark mode is a vanity feature, not a practical one.
**Question for Jay:** Have your masons ever complained about screen glare or readability on the tablet?

---

## Category I — Customisation (post-NCS)

### I01 — Other masons won't follow the same process
**Source:** `[SEAN]` Sean's explicit direction: "this is the way Northcoast does it, not every stone mason will follow the same process"
**Status:** CONFIRMED (Sean has confirmed; Jay not relevant)
**Assumption:** The NCS 10-stage workflow ships as the default template. Other tenants can add, remove, or reorder stages via Settings.
**Risk if wrong:** None — direction from Sean.
**Question for Jay:** N/A — this isn't a Jay question.

### I02 — Northcoast doesn't need customisation themselves
**Source:** `[GUESS]`
**Status:** PENDING
**Assumption:** Once NCS is on V3, they won't routinely modify their workflow. They'll use the default template. Customisation is mostly a multi-tenant concern, not an NCS concern.
**Risk if wrong:** If NCS expects to tweak their own workflow frequently, the customisation UI needs to be more prominent.
**Question for Jay:** Once you're on the new system, do you imagine wanting to adjust your workflow as things change in your shop? Or set it once and forget it?

---

## Maintenance notes

- Add new assumptions as new screens are designed. Format: source, status, assumption, risk, question.
- When Jay confirms an assumption, mark `CONFIRMED` and date the confirmation.
- When Jay invalidates an assumption, mark `INVALIDATED` and link to the redesign decision.
- When Jay modifies, mark `MODIFIED` and capture the refinement.
- At the end of Jay's review, export the confirmed/modified assumptions into the §34 spec amendment.

## Companion documents

- `FINDINGS-FOR-MAIN-PROJECT.md` — assumptions that became gap-register entries
- `COORDINATION-BRIEF-FOR-QUOTE-BUILDER.md` — joint decisions with the other design track
- `mfg-job-board.html` — the screen these assumptions were captured from (M01)

# Edge Painting — Mobile Bid Generator

A zero-dependency, mobile-first HTML estimating tool that lets a painting contractor build a client-ready PDF quote on their phone at the job site in under 10 minutes.

---

## What It Does

1. **Captures client info** — name, address, phone, email, job type, and projected start/completion dates in a single form
2. **Adds surfaces one by one** — each surface gets its own texture, product, coat count, labor hours, and specialty flag; the tool recalculates automatically on every input
3. **Auto-calculates paint quantity** — `ceil((sqft / coverage_rate) × coats × 1.10)` with 8 texture presets and a 10% waste buffer
4. **Enforces CA law** — deposit is hard-capped at `min($1,000, 10% of total)`, with the CA B&P Code notice embedded in the client estimate
5. **Generates an internal breakdown** — owner margin, overhead, and profit are visible to the contractor but hidden from the client-facing view
6. **Exports a print-to-PDF estimate** — formatted to match Sherwin-Williams estimate style; prints clean with a single tap

---

## Architecture

```
User input (touch/keyboard)
        ↓
  4-step wizard UI
        ↓
  calcSurf() → live cost per surface
        ↓
  addSurf() → surfaces[] state array
        ↓
  jobTotals() → subtotal + overhead (15%) + profit (15%) = total
        ↓
  renderEstimate()
    ├── client-facing view (Sherwin-Williams estimate format)
    ├── internal margin breakdown (not shown on print)
    └── payment schedule with CA deposit cap enforced
        ↓
  window.print() → PDF   |   localStorage.setItem() → saved quote
```

---

## Stack

| Layer | Technology |
|-------|------------|
| Language | Vanilla JavaScript (ES6+) |
| UI | HTML5 + CSS3, no framework |
| Persistence | Browser localStorage API |
| PDF Export | `window.print()` + print CSS media query |
| Hosting | Static file — open in any browser, no server |
| Dependencies | **Zero** — no CDN, no npm, no build step |

---

## Features

| Feature | Description | Skill Demonstrated |
|---------|-------------|-------------------|
| Coverage rate engine | 8 texture presets (smooth → stucco heavy) auto-calculate gallons with waste buffer | Domain math + constants mapping |
| Paint product defaults | 8 SW products auto-select by surface type with per-gallon price | UX decision logic |
| Specialty labor premium | Faux finish and wood repair apply 35% uplift automatically | Business rule encoding |
| Owner vs. crew labor | Differentiates Tyler ($65/hr) vs. W-2 ($35/hr) vs. 1099 (custom rate) | Multi-rate calculation |
| CA deposit cap | Hard-coded `Math.min(1000, total * 0.10)` — cannot be overridden | Compliance enforcement |
| Internal/client split | Step 3 shows owner margin; Step 4 hides it; print CSS removes it from PDF | Role-aware UI |
| Payment schedule | Auto-generates 4-installment schedule: deposit → 40% at start → substantial → 10% final | Cash flow planning |
| localStorage save | Quotes persist by client name + timestamp | Offline-first data layer |

---

## Project Structure

```
tools/
├── bid-generator.html    — Complete estimating tool (single file, 654 lines)
├── bid-generator-spec.md — Data model, wizard flow, and calculation formulas
└── README.md             — This file
```

---

## Getting Started

**Prerequisites:** A browser. That's it.

```bash
# Option 1 — open directly
open tools/bid-generator.html

# Option 2 — serve locally if testing webhooks later
python -m http.server 8080
# then open http://localhost:8080/tools/bid-generator.html
```

**Tyler's on-site use:** AirDrop the file to his iPhone → open in Safari → "Add to Home Screen" for app-like access.

---

## Skills Demonstrated

| Skill | Where It's Used | Level |
|-------|----------------|-------|
| State management (vanilla JS) | `surfaces[]` array + `job{}` object, no framework | Intermediate |
| Dynamic DOM rendering | `renderSurfList()`, `renderSummary()`, `renderEstimate()` — no innerHTML libraries | Intermediate |
| Business logic encoding | Coverage rates, CA deposit cap, specialty premium, overhead + profit formula | Applied |
| Compliance-driven design | CA B&P Code §7159 deposit limit hard-enforced in calculation layer, not just UI | Applied |
| Mobile UX | 44px+ touch targets, 16px font (no iOS zoom), sticky header + fixed bottom bar | Intermediate |
| Print CSS | `@media print` hides nav/internal views; Step 4 forced visible for clean PDF | Intermediate |
| Offline-first | `localStorage` persistence, zero network requests, no API keys needed | Foundational |

---

## Impact

| Metric | Before | After |
|--------|--------|-------|
| Time to produce a client estimate | 30–45 min (pencil + calculator + email template) | 8–12 min on-site |
| Risk of CA deposit law violation | Present (manual calculation) | Eliminated (hard-capped) |
| Estimating software cost | $49–$99/month (Jobber, Estimate Rocket) | $0 — static HTML |
| Estimate format consistency | Variable (hand-written or ad hoc) | Consistent SW-style PDF every time |
| Works without cell service | No | Yes — fully offline |

---

## Phase 2 Roadmap

### Google Sheets Auto-Log
**Trigger:** Contractor taps "Save Quote to Device" or "Print PDF"
**Action:** `fetch()` posts quote JSON to an n8n webhook → n8n appends a row to the Edge Painting Quotes Google Sheet
**Time saved:** ~5 min/quote in manual data entry + eliminates lost paper quotes

### Twilio SMS Notification
**Trigger:** New quote saved
**Action:** n8n sends Tyler an SMS: "New quote saved — $X,XXX for [Client Name] at [Address]. Check Sheets."
**Time saved:** Eliminates end-of-day "what did I quote today?" review

### Claude API Follow-Up Email
**Trigger:** 48 hours after quote date, if no signed contract recorded
**Action:** n8n calls Claude API → generates a personalized follow-up email referencing the client's address and quote total → sends via SendGrid
**Time saved:** 3–5 follow-up emails/week × 5 min each = 15–25 min/week; conversion rate lift expected

### Airtable Quote Database
**Trigger:** Every saved quote
**Action:** n8n posts to Airtable base with status field (Draft → Sent → Signed → Completed) + revenue tracking
**Time saved:** Replaces manual CRM entry; enables monthly revenue forecasting from Day 1

### PWA Manifest + Home Screen Install
**Trigger:** First visit on mobile
**Action:** Add `manifest.json` + service worker → Tyler installs it like a native app; works offline with cached assets
**Time saved:** Eliminates "where's the file?" friction; always one tap away on job site

---

## Part of the Edge Ecosystem

| Project | Description |
|---------|-------------|
| [Edge Painting](../) | Full ops layer — compliance docs, pricing guide, brand guide, marketing strategy |
| Edge Agency | The automation services arm behind all Edge builds |

---

## Contact

**Japheth Gordon** — AI Automation Builder
- Email: japgordon@gmail.com
- Building in public under the Edge brand

---

## License

MIT — free to fork and adapt for any service business.

# Bid Generator — Full Specification

**Last updated:** 2026-06-05
**Source templates:** SW Bid Guide + SW Painting Estimate + Edge Painting pricing formula
**Status:** Spec complete — ready to build

---

## Overview

Two tools, one data model:

| Tool | Who uses it | Where | Output |
|------|------------|-------|--------|
| **Tyler's Estimator** | Tyler, on-site during walkthrough | Mobile web app | Internal cost breakdown + client PDF |
| **Website Lead Capture** | Prospective client | Website | Rough estimate range + Tyler notification + AI follow-up |

---

## Data Model (mirrors SW Bid Guide structure)

Each job = one or more **surfaces**. Each surface has materials + labor.

```
Job
├── client_name
├── client_address
├── client_phone
├── client_email
├── job_type: exterior | interior | both
├── date
├── estimated_start
├── estimated_completion
├── payment_terms
├── notes
└── surfaces[]
    ├── surface_name         (e.g. "Living Room Walls", "Front Exterior")
    ├── surface_type         (walls | ceiling | trim | exterior | faux | wood_repair | pressure_wash)
    ├── num_units            (number of rooms or sections)
    ├── total_sq_ft          (calculated or entered)
    ├── wall_texture         (smooth | orange_peel | knockdown | heavy_texture)
    ├── num_coats            (1 | 2)
    ├── primer_needed        (yes | no)
    ├── primer_product       (product name)
    ├── primer_qty           (gallons)
    ├── paint_product        (product name — SW default)
    ├── paint_qty            (gallons — auto-calculated)
    ├── paint_color          (name + SW color number)
    ├── paint_sheen          (flat | eggshell | satin | semi-gloss | gloss)
    ├── paint_base           (white-base | light-base | medium-base | deep-base)
    ├── specialty_type       (none | faux_finish | wood_repair) 
    ├── owner_hours          (Tyler's hours on this surface)
    ├── employee_hours       (crew hours on this surface)
    ├── labor_type           (w2 | 1099)
    ├── materials_cost       (auto-calculated)
    ├── labor_cost           (auto-calculated)
    └── surface_total        (materials + labor)
```

---

## Tyler's Estimator — Step-by-Step Mobile Flow

### Step 1: Client Info
- Client name
- Property address
- Phone
- Email
- Job type: Exterior / Interior / Both

### Step 2: Add Surface (repeat per surface)
**Surface setup:**
- Surface name (free text — e.g. "Master Bedroom Walls")
- Surface type (select): Walls | Ceiling | Trim | Exterior Siding | Faux Finish | Wood Repair | Pressure Wash

**Measurements:**
- Number of units (rooms or sections)
- Total square footage (or input dimensions → auto-calculate)

**Paint spec:**
- Primer needed? Yes / No
  - If yes: product name + quantity
- Paint product (SW product name — dropdown of common products)
- Number of coats: 1 / 2
- Color name + SW color number
- Sheen: Flat / Eggshell / Satin / Semi-gloss / Gloss
- Base: White / Light / Medium / Deep

**Auto-calculated:**
- Gallons needed = (sq ft ÷ coverage rate) × coats × 1.10 buffer
- Materials cost = gallons × price per gallon (SW 35% discount applied)

**Labor:**
- Labor type: W-2 / 1099
- Owner hours (Tyler)
- Employee hours (if applicable)
- Auto-calculated labor cost:
  - Owner: hours × $65
  - Employee (W-2): hours × $35
  - Employee (1099): hours × [entered rate]

**Specialty premium:**
- If surface type = Faux Finish or Wood Repair → apply 35% premium to labor

**Surface total:** Materials cost + Labor cost

### Step 3: Add Another Surface?
- Yes → repeat Step 2
- No → proceed to totals

### Step 4: Job Summary & Totals
```
Subtotal (all surfaces)        = sum of surface totals
Overhead markup (15%)          = subtotal × 0.15
Profit margin (15%)            = (subtotal + overhead) × 0.15
──────────────────────────────────────────────────────
TOTAL QUOTE                    = subtotal + overhead + profit

Max deposit (CA law)           = MIN($1,000, total × 0.10)
Recommended payment schedule:
  Deposit (signing):           = deposit amount
  At job start (40%):          = total × 0.40
  Substantial completion (40%):= total × 0.40
  Final walkthrough (10%):     = total × 0.10
```

### Step 5: Notes + Crew
- Crew members + deadline (mirrors SW Bid Guide field)
- Extra prep notes
- Special client requests

### Step 6: Generate Outputs
- **Internal view:** Full cost breakdown with margin details (Tyler sees this, client does not)
- **Client PDF:** SW Painting Estimate format (see below)
- **Save to:** Google Sheets log + Airtable record

---

## Client-Facing PDF (SW Painting Estimate Format)

```
EDGE PAINTING — PAINTING ESTIMATE

Customer: [Name]              Estimate Total: $[total]
Address: [address]
City/State/ZIP: [city, CA, zip]
Phone: [phone]
Email: [email]

PROJECT SUMMARY
┌─────────────────┬──────────────┬─────────────────────────────┬──────────┐
│ Room/Area       │ Surface(s)   │ Description of Work         │ Cost     │
├─────────────────┼──────────────┼─────────────────────────────┼──────────┤
│ [surface_name]  │ [type]       │ [auto-generated description]│ $[cost]  │
│ ...             │ ...          │ ...                         │ ...      │
└─────────────────┴──────────────┴─────────────────────────────┴──────────┘

Notes: [prep notes, special requests]

Payment Terms:                        Total Job Cost:
[payment schedule]                    Total Cost:        $[subtotal+overhead+profit]
                                      Discounts:         $[0 or discount amount]
Estimated Completion Date:            ─────────────────────────────
[estimated_completion]                NET COST:          $[final total]

CONTRACTOR INFORMATION
Edge Painting                         Phone: [Tyler's phone]
Tyler [Last Name]                     Email: [Tyler's email]
CSLB License No.: [license number]
```

---

## Coverage Rate Lookup (used for auto-gallon calculation)

| Surface Texture | Sq Ft per Gallon |
|----------------|-----------------|
| Smooth interior drywall | 375 |
| Eggshell / light texture | 325 |
| Orange peel | 300 |
| Knockdown | 275 |
| Smooth exterior siding | 325 |
| Rough / weathered exterior | 250 |
| Stucco (smooth) | 275 |
| Stucco (heavy) | 200 |
| Trim (LF, brush) | 425 LF/gal |

*Add 10% buffer to all calculations. Always round up to nearest gallon.*

---

## SW Product Defaults (auto-populate in estimator)

| Surface | Default Product | Default Sheen |
|---------|----------------|--------------|
| Exterior siding (SD coastal) | SW Duration Exterior | Satin |
| Exterior trim | SW Duration Exterior | Semi-gloss |
| Interior walls | SW Cashmere Interior | Eggshell |
| Interior ceiling | SW Ceiling Paint | Flat |
| Interior trim | SW Emerald Urethane Trim | Semi-gloss |
| Faux finish base | SW Emerald Interior | Eggshell |
| Primer (bare wood) | SW Premium Wall & Wood Primer | — |
| Primer (stain block) | SW Extreme Bond Primer | — |

---

## Website Lead Capture — Simplified Form

**Client inputs:**
- Name
- Email
- Phone
- Property address
- Job type: Exterior / Interior / Both / Power Washing
- Approximate home size: Under 1,500 / 1,500–2,500 / 2,500+ sq ft
- Surfaces: Walls / Ceiling / Trim (multi-select)
- Timeline: ASAP / Within 1 month / 1–3 months / Just exploring
- How did you hear about us?
- Message (optional)

**Instant auto-response to client:**
- Acknowledgment + rough estimate range based on job type + size
- "Tyler will be in touch within [X hours]"
- Edge Painting contact info

**Tyler notification (SMS + email):**
- Full lead details
- Rough job size
- Client timeline urgency flag

**AI follow-up (if no Tyler response in 2 hours):**
- Claude API generates personalized follow-up email using client's name, job type, and address
- Tone: Tyler's voice (direct, professional, owner-operated)
- Includes link to schedule a free estimate

---

## Quote Estimate Ranges (for website auto-response)

| Job Type | Size | Estimate Range |
|----------|------|---------------|
| Exterior | <1,500 sq ft | $3,000–$4,500 |
| Exterior | 1,500–2,500 sq ft | $4,500–$8,000 |
| Exterior | 2,500+ sq ft | $8,000–$13,000 |
| Interior | Single room | $400–$1,000 |
| Interior | Full home (3/2) | $5,000–$9,000 |
| Power washing | Any | $150–$450 |
| Faux finish | Accent wall | $700–$1,500 |

*These are ranges for the website form only. Tyler's estimator produces exact quotes from on-site measurements.*

---

## Save & Log Flow

Every completed Tyler estimate:
1. Generates client PDF (downloadable / emailable)
2. Logs row to Google Sheet: Date | Client | Address | Job Type | Total Quote | Deposit | Status
3. Creates Airtable record with full surface breakdown
4. Tyler emails PDF to client directly

Every website lead:
1. Logs to same Airtable (marked as "Lead — not yet quoted")
2. Triggers n8n workflow: auto-response + Tyler notification + AI follow-up timer

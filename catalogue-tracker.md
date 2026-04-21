# OrchardGuard V2 — Catalogue Ingestion Tracker

> **Status.** Scaffold based on V1 seed catalogue (from V1 audit Section 8). OMAFRA expansion (pests, diseases, herbicides, foliar nutrients, amendments) is open work.
> **Companion to.** Planning doc Section 5 (Label Ingestion Workflow).
> **Purpose.** Track every product through V2's six-stage ingestion pipeline: Discovery → Scrape → Label Attachment → Manual Entry → Verification → Publication.
> **Updating.** This is a living document. Append rows as products are discovered; update stage as they move through the pipeline.

---

## Ingestion Stage Legend

| Stage | Meaning | DB state |
|---|---|---|
| 🔍 Discovery | Identified as in-scope, shell row exists | `ontario_registered=false, label_ingested_at=null` |
| 🕸️ Scrape | OMAFRA data populated into staging | same as above |
| 📄 Label Attached | Label PDF URL captured; PDF cached | same as above |
| ✍️ Manual Entry | Author filling remaining fields from label | `label_ingested_at` set but `ontario_registered=false` |
| ✅ Verified | Second-pass verification complete | same as above |
| 🚀 Published | Recommendation-eligible | `ontario_registered=true, label_ingested_at` set |

A product at 🚀 Published is advisor-eligible. Earlier stages are not.

---

## Bootstrap Scope Reminder

Per planning doc Section 5.2, bootstrap is **hybrid** — ~100–150 products covering:

1. V1's seed catalogue (~30 products) — itemized below
2. Every product OMAFRA lists in its recommended programs for: apple scab, fire blight, powdery mildew, cedar rust, summer diseases, codling moth, oriental fruit moth, apple maggot, plum curculio, aphids, mites
3. Every apple-registered herbicide in OMAFRA's weed management guidance
4. Every foliar nutrient and soil amendment commonly used in Ontario apple production (OMAFRA Pub 360 nutrition sections)

Products beyond the bootstrap scope are added via the grower request flow (Section 5.9) as invited growers need them.

---

## Part 1 — V1 Carry-Forward (~30 products)

These are the products seeded into V1's database per the V1 audit. They migrate into V2 at the Manual Entry stage (V1 data is treated as scraped-quality, not verified). Each runs through the full pipeline before Publication.

### Fungicides (13)

| # | Trade Name | Active Ingredient | FRAC/IRAC | Stage | PCP # | Label URL | Last Verified | Notes |
|---|---|---|---|---|---|---|---|---|
| 1 | Captan 80 WDG | captan | FRAC M4 | ✍️ Manual Entry | — | — | — | V1 workhorse protectant |
| 2 | Dithane M-45 | mancozeb | FRAC M3 | ✍️ Manual Entry | — | — | — | V1 workhorse protectant |
| 3 | Nova 40W | myclobutanil | FRAC 3 | ✍️ Manual Entry | — | — | — | Scab + rust |
| 4 | Merivon | fluxapyroxad + pyraclostrobin | FRAC 7+11 | ✍️ Manual Entry | — | — | — | Combo; scab + summer |
| 5 | Inspire Super | difenoconazole + cyprodinil | FRAC 3+9 | ✍️ Manual Entry | — | — | — | Combo |
| 6 | Syllit 400 | dodine | FRAC U12 | ✍️ Manual Entry | — | — | — | Scab kickback |
| 7 | Flint | trifloxystrobin | FRAC 11 | ✍️ Manual Entry | — | — | — | QoI |
| 8 | Copper 53W | copper hydroxide | FRAC M1 | ✍️ Manual Entry | — | — | — | Dormant/delayed-dormant; fire blight |
| 9 | Streptomycin | streptomycin sulfate | antibiotic | ✍️ Manual Entry | — | — | — | Fire blight; 3 applications/season max |
| 10 | Kasumin 2L | kasugamycin | antibiotic | ✍️ Manual Entry | — | — | — | Fire blight |
| 11 | Blossom Protect | *Aureobasidium pullulans* | biological | ✍️ Manual Entry | — | — | — | Fire blight biocontrol |
| 12 | Serenade OPTI | *Bacillus subtilis* | biological | ✍️ Manual Entry | — | — | — | Broad-spectrum biocontrol |
| 13 | Sulfur | sulfur | FRAC M2 | ✍️ Manual Entry | — | — | — | Scab + powdery mildew |

### Insecticides (8)

| # | Trade Name | Active Ingredient | FRAC/IRAC | Stage | PCP # | Label URL | Last Verified | Notes |
|---|---|---|---|---|---|---|---|---|
| 14 | Admire 240F | imidacloprid | IRAC 4A | ✍️ Manual Entry | — | — | — | Neonicotinoid; aphids, scale |
| 15 | Assail 70WP | acetamiprid | IRAC 4A | ✍️ Manual Entry | — | — | — | Neonicotinoid |
| 16 | Altacor | chlorantraniliprole | IRAC 28 | ✍️ Manual Entry | — | — | — | Codling moth, OBLR |
| 17 | Imidan 70WP | phosmet | IRAC 1B | ✍️ Manual Entry | — | — | — | Organophosphate; plum curculio, apple maggot |
| 18 | Sevin XLR | carbaryl | IRAC 1A | ✍️ Manual Entry | — | — | — | Carbamate; thinning use noted |
| 19 | Entrust SC | spinosad | IRAC 5 | ✍️ Manual Entry | — | — | — | OMRI-listed |
| 20 | Surround WP | kaolin clay | barrier | ✍️ Manual Entry | — | — | — | Sunburn + insect deterrent |
| 21 | Superior Oil | mineral oil | — | ✍️ Manual Entry | — | — | — | Dormant / delayed-dormant; mite eggs, scale |

### Growth Regulators (6)

Growth regulators are a separate product class in V1; V2 places them in `products` with category `pesticide` unless a better classification emerges. Flagging for review during ingestion.

| # | Trade Name | Active Ingredient | FRAC/IRAC | Stage | PCP # | Label URL | Last Verified | Notes |
|---|---|---|---|---|---|---|---|---|
| 22 | Apogee | prohexadione-calcium | — | ✍️ Manual Entry | — | — | — | Vegetative growth suppressor; fire blight shoot management |
| 23 | Fruitone-N | NAA (naphthaleneacetic acid) | — | ✍️ Manual Entry | — | — | — | Thinner, stop-drop |
| 24 | MaxCel | 6-BA (benzyladenine) | — | ✍️ Manual Entry | — | — | — | Thinner |
| 25 | Ethrel | ethephon | — | ✍️ Manual Entry | — | — | — | Ripening, color development |
| 26 | ReTain | AVG (aminoethoxyvinylglycine) | — | ✍️ Manual Entry | — | — | — | Pre-harvest; delays drop and maturity |
| 27 | Harvista | 1-MCP (1-methylcyclopropene) | — | ✍️ Manual Entry | — | — | — | Pre-harvest spray; ethylene action inhibitor |

**V1 carry-forward subtotal: 27 products.**

Note: V1 models also reference "Pristine" (a FRAC 7+11 fungicide by BASF) which is NOT in V1's seed catalogue. Pristine was a hardcoded string in model recommendations without a corresponding database row — one of the V1 rough edges called out in the audit. For V2, Pristine either needs to be added to the catalogue or removed from model recommendation strings. Placeholder row:

| # | Trade Name | Active Ingredient | FRAC/IRAC | Stage | PCP # | Label URL | Last Verified | Notes |
|---|---|---|---|---|---|---|---|---|
| 28 | Pristine | boscalid + pyraclostrobin | FRAC 7+11 | 🔍 Discovery | — | — | — | Referenced in V1 models but not in seed catalogue; reconcile during V2 ingestion |

---

## Part 2 — OMAFRA-Recommended Pesticides (TBD)

This section is a placeholder for products beyond V1's seed that are called out in OMAFRA's recommended programs for the target categories listed in the bootstrap scope. Populating this requires going through OMAFRA's Crop Protection Hub by target.

### Target: Apple Scab — Additional Products

| # | Trade Name | Active Ingredient | FRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA* | | | 🔍 | |

### Target: Fire Blight — Additional Products

| # | Trade Name | Active Ingredient | FRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA* | | | 🔍 | |

### Target: Powdery Mildew — Additional Products

| # | Trade Name | Active Ingredient | FRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA* | | | 🔍 | |

### Target: Cedar Apple Rust — Additional Products

| # | Trade Name | Active Ingredient | FRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA* | | | 🔍 | |

### Target: Summer Diseases (sooty blotch/flyspeck, bitter rot, black rot, white rot, Brooks spot, bull's-eye rot) — Additional Products

| # | Trade Name | Active Ingredient | FRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA* | | | 🔍 | |

### Target: Codling Moth — Additional Products

| # | Trade Name | Active Ingredient | IRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA* | | | 🔍 | |

### Target: Oriental Fruit Moth — Additional Products

| # | Trade Name | Active Ingredient | IRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA* | | | 🔍 | |

### Target: Apple Maggot — Additional Products

| # | Trade Name | Active Ingredient | IRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA* | | | 🔍 | |

### Target: Plum Curculio — Additional Products

| # | Trade Name | Active Ingredient | IRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA* | | | 🔍 | |

### Target: Aphids (rosy, green, woolly) — Additional Products

| # | Trade Name | Active Ingredient | IRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA* | | | 🔍 | |

### Target: Mites (European red, two-spotted) — Additional Products

| # | Trade Name | Active Ingredient | IRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA* | | | 🔍 | |

---

## Part 3 — Herbicides (TBD)

Ontario-registered herbicides for apple orchards per OMAFRA weed management guidance. HRAC group required per product.

| # | Trade Name | Active Ingredient | HRAC | Stage | Notes |
|---|---|---|---|---|---|
| | *to be populated from OMAFRA weed management* | | | 🔍 | |

Common Ontario apple orchard herbicides likely to appear here (candidate list, not verified):

- Glyphosate products (Roundup WeatherMAX, Credit, etc.) — HRAC 9
- Glufosinate (Ignite) — HRAC 10
- Simazine (Simadex) — HRAC 5
- Dichlobenil (Casoron) — HRAC 29
- 2,4-D products — HRAC 4
- Diuron — HRAC 5

Each requires research for Ontario apple registration, PCP number, and current label.

---

## Part 4 — Foliar Nutrients (TBD)

Common Ontario apple foliar nutrients per OMAFRA Publication 360 nutrition sections. Generally not PMRA-registered (nutrition products aren't pesticides); source of truth is manufacturer technical data sheets plus OMAFRA sufficiency ranges.

| # | Trade Name | Primary Nutrient | Stage | Notes |
|---|---|---|---|---|
| | *to be populated* | | 🔍 | |

Candidate foliars (not verified):

- Calcium chloride (several brands) — for bitter pit management
- Calcium nitrate — foliar Ca + N
- Boron (Solubor, sodium borate) — pink-stage boron
- Zinc (ZnSO₄, zinc chelate) — pre-bloom zinc
- Manganese (MnSO₄) — foliar Mn
- Urea (foliar-grade) — foliar N, post-harvest scab sanitation
- Magnesium (Epsom salt, MgSO₄) — foliar Mg
- Multi-nutrient products (various brands)

---

## Part 5 — Soil Amendments (TBD)

Liming materials, gypsum, granular fertilizers, sulfur. Per OMAFRA Pub 360.

| # | Trade Name | Amendment Type | Stage | Notes |
|---|---|---|---|---|
| | *to be populated* | | 🔍 | |

Candidate amendments:

- Agricultural lime (calcitic) — pH raise, Ca source
- Dolomitic lime — pH raise, Ca + Mg source
- Gypsum (calcium sulfate) — Ca source without pH impact
- Elemental sulfur — pH reduction
- Granular N (ammonium sulfate, urea, calcium ammonium nitrate)
- Granular K (potassium sulfate, potassium chloride)
- Blended complete fertilizers (various formulations)

---

## Part 6 — Discovery Queue

Products identified as needed but not yet staged. Move to appropriate section above when entering ingestion pipeline.

| # | Trade Name | Source of Request | Notes |
|---|---|---|---|
| | | | |

---

## Completion Metrics

Updated as ingestion progresses. Targets per Section 5.2.

| Category | Target Count | Published | Verified (not yet published) | In Pipeline | Discovered |
|---|---|---|---|---|---|
| V1 carry-forward | ~27 | 0 | 0 | 27 | 1 |
| OMAFRA fungicides beyond V1 | ~10–25 | 0 | 0 | 0 | 0 |
| OMAFRA insecticides beyond V1 | ~10–20 | 0 | 0 | 0 | 0 |
| Herbicides | ~15–25 | 0 | 0 | 0 | 0 |
| Foliar nutrients | ~15–30 | 0 | 0 | 0 | 0 |
| Soil amendments | ~5–10 | 0 | 0 | 0 | 0 |
| **Total** | **~100–150** | **0** | **0** | **27** | **1** |

---

## Ingestion Workflow Reminder

Per Section 5.3, each product moves through these stages. Stop before Publication unless strict gate is met.

1. **🔍 Discovery.** Create shell row. `ontario_registered=false`.
2. **🕸️ Scrape.** Run OMAFRA scraper; populate staging fields.
3. **📄 Label Attached.** Capture label URL; download PDF.
4. **✍️ Manual Entry.** Fill remaining fields from the label directly.
5. **✅ Verified.** Second-pass check against label, field by field.
6. **🚀 Published.** Flip `ontario_registered=true`, set `label_ingested_at`. Advisor-eligible.

Fields that are scraping-from-OMAFRA acceptable: trade name, active ingredients, FRAC/IRAC/HRAC, target pests/weeds, efficacy ratings, rate (advisory check only).

Fields that are label-only (never scraped): PCP number, seasonal max applications, seasonal max rate, min days between applications, REI hours, REI-by-work-type matrix, min tree age, growth stage min/max, kickback hours, rainfast hours, resistance risk, tank-mix known compat/incompat, hard nutrition caps.

---

## Next Actions

Immediate, in priority order:

1. **Populate PCP numbers and label URLs for the 27 V1 products.** These are needed for the Label Attached stage. Source: PMRA Public Registry at [pr-rp.hc-sc.gc.ca](https://pr-rp.hc-sc.gc.ca/).
2. **Resolve Pristine reconciliation.** Either add to catalogue or remove from V2 model recommendation strings.
3. **OMAFRA Crop Protection Hub survey pass.** Go through the Hub target-by-target, populating candidate products in Part 2. Target: 1–2 hours per target category.
4. **OMAFRA Pub 360 nutrition section survey pass.** Populate Parts 4 and 5.
5. **HRAC research pass for herbicides.** Populate Part 3.

Phase 3 of V2 is where this tracker's completion matters. Bootstrap before first invited grower per Section 5.15 #7.

---

> **End of catalogue tracker.** Update as ingestion progresses. Every product that reaches 🚀 Published becomes advisor-eligible.

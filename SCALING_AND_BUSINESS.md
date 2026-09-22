# Scaling Up & Business Potential

This document covers two things:

1. **How to scale** this project from a Colab demo into a production system.
2. **Who would pay for it**: target industries, Indian and international
   customers, business models, and how to reach them.

---

## Where the project stands today

Be clear about this when pitching; reviewers respect honesty about maturity.

| Area | Current state |
|---|---|
| Detection | Random forest on 9 texture features per 256x256 SAR patch |
| Training data | 14 Sentinel-1 scenes, Gulf of Mexico only (2018-2020) |
| Attribution | Hand-weighted AIS suspicion score (distance, bearing, AIS gap) |
| Escalation | 3 tiers: `AUTO-ESCALATE`, `HUMAN REVIEW`, `LOG ONLY` |
| Deployment | FastAPI inside a Colab notebook, exposed via ngrok; static HTML frontend |
| Data input | Pre-downloaded dataset; no live satellite or AIS feed yet |

**What's already strong:** it goes beyond *detecting* a spill to *naming the
likely polluter*. Most free or academic tools stop at detection. Attribution is
what enforcement agencies, insurers, and courts actually need.

---

## Part 1: Scaling roadmap

### Phase 1: Make it production-ready

- Move the FastAPI backend out of the notebook into `app.py`.
- Package with Docker; add automated tests and CI (GitHub Actions).
- Replace ngrok with a real deployment (a cloud VM or container service).
- Add logging, error handling, and an API key for access.

### Phase 2: Better detection

- **More data:** add all 23 scenes from the current dataset, plus other open
  Sentinel-1 oil spill datasets (e.g. the Trujillo-Acatitla et al. Zenodo
  series), and label Indian Ocean / Arabian Sea scenes.
- **Look-alikes:** the biggest source of false alarms in SAR oil detection is
  "look-alikes" (low-wind areas, algae/biogenic films, rain cells, ship wakes).
  Train on these explicitly as negatives.
- **Deep learning:** move from patch classification to pixel-level
  segmentation (U-Net or similar) so the output is the spill's actual shape
  and area.
- **Context data:** add wind speed (e.g. ERA5 or scatterometer data), since
  oil is only visible in SAR within a certain wind range.

### Phase 3: Live data pipelines

- **Satellite imagery:** auto-download new Sentinel-1 passes over chosen areas
  from the Copernicus Data Space Ecosystem (free). Pre-process (calibration,
  speckle filtering), tile, and run detection automatically.
- **More satellites for faster revisits:** Sentinel-1 alone revisits an area
  only every few days. Add commercial SAR (ICEYE, Capella, Umbra) and, for
  India, ISRO's EOS-04 and the NASA-ISRO NISAR mission.
- **Live AIS:** plug into AIS providers (terrestrial and satellite AIS) instead
  of sample fleets. Free/open sources exist for research (e.g. national
  maritime authorities, Global Fishing Watch); commercial feeds cover open
  ocean.
- **Drift back-tracking:** oil moves with wind and currents. Use an open-source
  drift model (e.g. OpenDrift) to trace a slick *backwards* in time, so it's
  matched against where ships were *when the oil was released*, not where
  they are now.

### Phase 4: Smarter attribution

- **Dark vessels:** detect ships directly in the SAR image and flag ones with
  no matching AIS signal (AIS switched off or spoofed); these are prime
  suspects.
- **Learned scoring:** once confirmed cases exist (from enforcement outcomes),
  replace the hand-weighted formula with a trained model.
- **Vessel history:** add risk factors like past detentions, flag state, age,
  and prior incidents.

### Phase 5: Platform

- Cloud architecture: job queue → detection workers → PostGIS database →
  web dashboard + map.
- Real-time alerts (email, SMS, webhook) when a spill is `AUTO-ESCALATE`d.
- **Evidence reports:** auto-generated PDF with satellite image, spill outline,
  AIS tracks, drift model, and timestamps: a court-ready audit trail.
- Multi-customer support: each customer watches their own areas.

---

## Part 2: Business potential

### The problem it solves

- Deliberate illegal discharges (dumping oily bilge water or tank-washing
  residue at sea) are banned under **MARPOL Annex I**, but are hard to prove.
- Accidental spills from ships, pipelines, and offshore rigs need to be found
  fast to limit damage and to assign liability for clean-up costs.
- Planes and patrol boats are expensive and cover little area. Satellite SAR
  covers huge areas, day and night, through cloud.
- **Detection alone isn't enough.** Authorities need to know *who did it* to
  fine or prosecute. That's this project's edge.

### Target customers: India 🇮🇳

| Customer | Why they'd care |
|---|---|
| **Indian Coast Guard** | Nodal agency for oil spill response in Indian waters (National Oil Spill Disaster Contingency Plan). Needs detection + polluter identification. |
| **DG Shipping** (Ministry of Ports, Shipping & Waterways) | Enforces MARPOL and the Merchant Shipping Act; needs evidence against violators. |
| **INCOIS** (Hyderabad) | Runs ocean information services including oil spill trajectory advisories; natural data/research partner. |
| **Indian Navy / IFC-IOR** (Information Fusion Centre – Indian Ocean Region) | Maritime domain awareness; dark-vessel detection is directly relevant. |
| **Major & private ports**: JNPA, Mumbai, Chennai, Deendayal (Kandla), Paradip, Visakhapatnam, Adani Ports (Mundra), JSW | Monitor port waters and anchorages; hold polluters liable. |
| **Offshore oil & gas**: ONGC (Mumbai High), Oil India, Cairn/Vedanta, Reliance (KG basin) | Monitor rigs and pipelines; regulatory compliance; early leak detection. |
| **Refineries & SPM terminals**: Reliance Jamnagar, Nayara Vadinar, IOCL Paradip | Single-point moorings are high-risk transfer points. |
| **Pollution regulators**: MoEFCC, CPCB, coastal State Pollution Control Boards | Environmental enforcement and reporting. |
| **Marine insurers**: New India Assurance, GIC Re, and others | Claim verification and risk assessment. |

Past Indian incidents that show the need: the **MSC Chitra** collision off
Mumbai (2010) and the **Ennore** tanker collision near Chennai (2017).

### Target customers: International 🌍

| Customer type | Examples |
|---|---|
| **Coast guards & maritime agencies** | US Coast Guard / NOAA, UK MCA, Norwegian Coastal Administration, Australian AMSA, Singapore MPA, Indonesia & Malaysia (Strait of Malacca), Gulf states |
| **Regional bodies** | EMSA (EU; runs the CleanSeaNet satellite service), REMPEC (Mediterranean), MEMAC (Gulf region) |
| **Oil & gas majors** | Shell, BP, ExxonMobil, TotalEnergies, Equinor, Petrobras, Saudi Aramco, ADNOC |
| **Oil spill responders** | Oil Spill Response Ltd (OSRL), ITOPF (technical adviser to industry) |
| **Insurance & compensation** | P&I Clubs, IOPC Funds; they pay spill claims and want liability assigned correctly |
| **High-need regions** | West Africa / Gulf of Guinea (Nigeria), Brazil (the 2019 north-east coast spill's source was very hard to trace), Southeast Asia, Persian Gulf, Red Sea |
| **Maritime intelligence platforms** | Could license or integrate the attribution engine as a feature |
| **ESG investors & NGOs** | Track environmental behaviour of shipping and energy companies |

### Other industries that benefit

- **Shipping companies & charterers:** vet vessels before hiring them; prove
  their own fleet is clean.
- **Fisheries & aquaculture:** early warning to protect fish farms and fishing
  grounds.
- **Coastal tourism & local governments:** early warning before oil reaches
  beaches.
- **Desalination & power plants** with seawater intakes: shut intakes before
  oil arrives (major concern in the Gulf states).
- **Legal firms:** evidence for spill liability cases.
- **Researchers & universities:** long-term pollution statistics.

### Competitive landscape

Existing players to know (and position against):

- **EMSA CleanSeaNet**: EU government service; Europe only.
- **KSAT (Kongsberg)**: commercial satellite oil spill detection.
- **SkyTruth Cerulean**: open, non-profit slick detection with vessel
  matching.
- **Orbital EOS**, and satellite operators like **ICEYE**: commercial
  monitoring.
- **Maritime intelligence firms** (e.g. Windward, Kpler/MarineTraffic, Spire):
  vessel tracking and risk, not focused on spills.

**How this project differentiates:**

- **Attribution + escalation built in**, not just a slick map.
- **India / Indian Ocean focus**: an underserved region compared to Europe
  and the US.
- **Low cost**: built on free Sentinel-1 data and open-source tools.
- **Explainable scoring**: each suspicion score shows *why* (distance,
  bearing, AIS gap), which matters for legal use.

### Business models

| Model | How it works | Best for |
|---|---|---|
| **SaaS subscription** | Monthly/annual fee per monitored area (port, oil field, coastline) | Ports, oil companies |
| **Government contract / tender** | Custom deployment for a national agency | Coast guards, regulators |
| **Pay-per-report** | Fee per incident investigation / evidence report | Insurers, legal firms, P&I clubs |
| **API licensing** | Other platforms call our detection/attribution API | Maritime intelligence companies |
| **White-label** | Partner sells it under their own brand | Satellite operators, consultancies |
| **Freemium / open core** | Free public spill map; paid alerts, reports, and private areas | Building visibility and trust |

### Go-to-market plan

**India first:**

1. **Competitions & challenges**: Smart India Hackathon, iDEX (Innovations
   for Defence Excellence) challenges for Coast Guard / Navy problem
   statements, ISRO and IN-SPACe startup programmes.
2. **Pilot with a research partner**: INCOIS, NIO (National Institute of
   Oceanography, Goa), or an IIT ocean engineering department; validate on
   Indian waters.
3. **Funding**: Startup India Seed Fund, DST NIDHI programmes, college
   incubators.
4. **First paying customer**: a port or offshore operator (faster sales cycle
   than government), then use that case study for Coast Guard / DG Shipping.

**Then international:**

5. Target regions with weak existing coverage: Southeast Asia, Middle East,
   West Africa.
6. Partner with a satellite data provider or an established maritime
   intelligence company rather than selling alone.
7. Publish results (paper or open benchmark) to build credibility.

---

## Risks & limitations to address

| Risk | Mitigation |
|---|---|
| False positives from look-alikes | Train on look-alikes; add wind data; keep `HUMAN REVIEW` tier |
| Trained only on Gulf of Mexico | Collect and label Indian Ocean and other regional data |
| Satellite revisit gaps (hours to days) | Combine multiple SAR satellites; prioritise high-risk areas |
| AIS turned off or spoofed | Dark-vessel detection from SAR itself |
| Evidence must stand up in court | Audit trail, timestamps, drift modelling, human sign-off |
| Commercial AIS & SAR data costs | Start with free Sentinel-1 + open AIS; pass data costs into pricing |
| Long government sales cycles | Start with ports/industry; use pilots and challenges to get in |

---

## Summary

- **Now:** working research prototype with a real differentiator (attribution).
- **Next:** production-ready backend, more and more local training data,
  live satellite + AIS feeds, drift back-tracking.
- **Market:** coast guards, ports, oil & gas, insurers, and regulators, with a
  strong India / Indian Ocean angle and clear international expansion paths.

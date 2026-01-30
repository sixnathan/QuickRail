# QuickRail Logistics & Operations Research

**Date:** January 2026
**Focus:** Pre-seed EU startup, aggressive market entry

---

## 1. Train Provider Partnership Requirements

### Deutsche Bahn (Germany)
- **Portal:** [DB API Marketplace](https://developers.deutschebahn.com/db-api-marketplace/apis/frontpage)
- **Access:** Self-service registration for basic APIs; enterprise partnerships require negotiation
- **Data:** Open data under CC BY 4.0 license (limited to long-distance trains in test operation)
- **Note:** DB does not allow direct integrations for booking; must connect via authorized partners
- **Source:** [DB Developer Docs](https://developer-docs.deutschebahn.com/apis)

### SNCF (France)
- **Portal:** [SNCF Numerique API](https://numerique.sncf.com/startup/api/)
- **Access:** Free developer registration; 90,000 ops/month, 3,000/day limit
- **Supplier Requirements:** ATOUT France registration, financial examination, EUR 10,000 deposit
- **Affiliate:** Managed via Effiliation platform
- **Source:** [SNCF Connect Partnership](https://www.sncf-connect.com/en-en/tools/partnership-affiliation)

### Trenitalia (Italy)
- **Official API:** None publicly available
- **Options:**
  - Unofficial ViaggiaTreno API (information only, no booking)
  - Third-party aggregators (Lyko, Rail Europe)
- **NTV Italo:** No public API; requires direct business contact
- **Source:** [GitHub Trenitalia-API](https://github.com/SimoDax/Trenitalia-API)

### Renfe (Spain)
- **Portal:** [datos.gob.es](https://datos.gob.es/en/noticia/renfe-data-you-can-find-datosgobes)
- **Access:** 63 datasets available (stations, schedules, notices)
- **Booking API:** Limited official access; Lyko provides workaround
- **Source:** [Lyko Renfe API](https://lyko.tech/en/portfolio/train-api/renfe-api/)

### OBB (Austria)
- **Portal:** [apiportal.oebb.at](https://apiportal.oebb.at/portal/)
- **Access:** Registration-based; specific requirements not publicly disclosed
- **Reality:** Most developers use unofficial HAFAS endpoints
- **Source:** [OBB API Docs (unofficial)](https://github.com/hoenic07/oebb-api-docs)

### SBB (Switzerland)
- **Portal:** [developer.sbb.ch](https://developer.sbb.ch/apis?sortType=all)
- **Process:**
  1. Register at developer-int.sbb.ch (integration environment)
  2. Sign standard distribution/system contract
  3. Implement APIs with SBB acceptance testing
  4. Mandatory updates within 9 weeks of release
- **Source:** [SBB Swiss Mobility APIs](https://company.sbb.ch/en/sbb-as-business-partner/services/sbb-swiss-mobility-solutions/sbb-swiss-mobility-apis.html)

### NS (Netherlands)
- **Portal:** [apiportal.ns.nl](https://apiportal.ns.nl/)
- **Access:** Free subscription provides primary/secondary API keys immediately
- **Limitation:** Passenger information only; no booking API
- **Source:** [NS API Info](https://www.ns.nl/en/travel-information/ns-api)

---

## 2. API Access Processes

### Fastest Path: Aggregators (Recommended for Pre-Seed)

| Aggregator | Carriers | Integration Time | Notes |
|------------|----------|------------------|-------|
| **Trainline Partner Solutions** | 280+ rail/coach | Weeks | RESTful API, 10 currencies, commercial account manager assigned |
| **Rail Europe** | 200+ operators | Weeks | Official agent for 24 countries, commission-based |
| **Lyko** | Major EU operators | Weeks | No volume restrictions, French/English docs |
| **Travelfusion tfRail** | EU/NA/China | Weeks | Location-based search, highest booking success rates |

**Sources:**
- [Trainline Partner Solutions](https://tps.thetrainline.com/)
- [Rail Europe Agent Portal](https://agent.raileurope.com/)
- [Lyko Train API](https://lyko.tech/en/portfolio/train-api/)

### GDS Route (Longer Timeline)

| GDS | Carriers | Requirements | Timeline |
|-----|----------|--------------|----------|
| **Amadeus** | 90+ railways | Self-service: immediate; Enterprise: IATA/ARC cert, weeks-months negotiation | 1-6 months |
| **Sabre** | 9 major EU networks | Enterprise agreement | Months |

**Integration Cost:** USD 1,000-5,000 base + per-transaction fees
**Source:** [Amadeus API Integration Guide](https://www.altexsoft.com/blog/amadeus-api-integration/)

### Direct Operator Access (Longest)
- Requires business development, legal agreements, technical integration
- Timeline: 3-12 months typical
- Often restricted to established travel agencies

---

## 3. Disruption Data Sources

### GTFS-RT Feeds (Real-Time)

| Country | Provider | Coverage | Access |
|---------|----------|----------|--------|
| Switzerland | [opentransportdata.swiss](https://opentransportdata.swiss/en/cookbook/realtime-prediction-cookbook/gtfs-rt/) | All PT with real-time data, 3hr preview | API key required |
| Netherlands | [OVapi](https://groups.google.com/g/transit-developers/c/MbGRNM9keJ8) | Bus/metro/train/tram + international | CC0 license |
| Hungary | BKK OpenData | Vehicle positions, alerts, trip updates | Public |

**Feed Types:**
- Trip Updates: delays, cancellations, route changes
- Service Alerts: station closures, incidents
- Vehicle Positions: real-time location

**Source:** [GTFS Realtime Reference](https://gtfs.org/documentation/realtime/reference/)

### Aggregated Disruption Data
- **Trainline API:** Real-time delays, disruptions, platform changes included
- **HAFAS Endpoints:** Used by DB, OBB, SBB; can generate GTFS-RT via [hafas-gtfs-rt-feed](https://github.com/derhuerst/hafas-gtfs-rt-feed)

### Feed Discovery
- [OpenMobilityData](https://transitfeeds.com/): 1,327 providers, 677 locations
- [MobilityData Awesome Transit](https://github.com/MobilityData/awesome-transit): Curated list

---

## 4. Payment Infrastructure (Multi-Country EU)

### Recommended Providers

| Provider | Strengths | EU Presence |
|----------|-----------|-------------|
| **Stripe** | 1% of global GDP processed, EPC member, instant setup | Strong |
| **Adyen** | EUR 23.82B volume, 150+ currencies, SEPA/iDEAL/Bancontact | HQ Amsterdam |
| **Mollie** | EU-native, simpler integration | Growing |

**Source:** [Stripe Cross-Border Payments](https://stripe.com/resources/more/cross-border-payment-platforms)

### PSD2/SCA Compliance (Mandatory)

**Requirements:**
- 2-factor authentication for transactions over EUR 30 in EEA
- Implement 3D Secure 2.0
- Categories: Knowledge (PIN) + Possession (device) + Inherence (biometric)

**Travel-Specific Exemptions:**
- Virtual Credit Cards (VCC): exempt from SCA
- Mail Order/Telephone Order (MOTO): exempt if manually entered
- Merchant-Initiated Transactions: out of scope (no-shows, subscriptions)

**Consequence of Non-Compliance:** Increased decline rates, lost volume, potential fines

**Note:** PSD3 expected 2026 with updated requirements

**Source:** [Stripe SCA Guide](https://stripe.com/guides/strong-customer-authentication)

### Multi-Currency Considerations
- Local payment methods essential (iDEAL NL, Bancontact BE, SEPA EU-wide)
- Currency display: EUR primary, support local (CHF, GBP, SEK, etc.)
- Transaction fees: 1.5-3% typical cross-border

---

## 5. Multi-Country Operations Setup

### Entity Structure Options

| Structure | Requirements | Best For |
|-----------|--------------|----------|
| **Single EU Entity** | Register in one country, use OSS for VAT | Pre-seed, testing market |
| **Societas Europaea (SE)** | Active in 2+ EU countries, capital requirements | Scaling phase |
| **Subsidiaries** | Separate entity per country | Mature operations |

**Recommended Pre-Seed:** Single entity (Ireland, Netherlands, or Estonia popular) + OSS scheme

**Source:** [Your Europe - Starting Business](https://europa.eu/youreurope/business/running-business/start-ups/starting-business/index_en.htm)

### VAT Requirements

**One-Stop-Shop (OSS) Scheme:**
- Single registration covers B2C sales across EU
- Threshold: EUR 10,000 combined EU cross-border sales
- File periodic returns for all EU sales

**Triggers for Local Registration:**
- Fixed establishment (office, warehouse)
- Stock in fulfillment centers (e.g., Amazon FBA)
- OSS suspension

**Rates:** 17-27% across EU countries

**Source:** [Taxually VAT Triggers](https://www.taxually.com/blog/9-common-triggers-for-vat-registration-in-the-eu)

### Regulatory Framework

**Rail Market Access:**
- Directive 2012/34/EU: Single European Railway Area
- Fourth Railway Package: Full market liberalization (franchises for regional, open access for long-distance)
- No operator license needed for booking/aggregation platform

**Data Standards:**
- ERA + railML.org collaboration for interoperability
- GTFS/GTFS-RT becoming standard

**Source:** [ALLRAIL Policy](https://www.allrail.eu/policy/)

### Key Organizations

| Organization | Role |
|--------------|------|
| [RailNetEurope (RNE)](https://rne.eu/) | Infrastructure coordination, real-time train data |
| [Europe's Rail](https://rail-research.europa.eu/) | EUR 1.2B R&D program, potential partnership |
| [ERA](https://www.era.europa.eu/) | Regulatory standards, data harmonization |

---

## Aggressive Entry Strategy Recommendations

1. **Week 1-2:** Sign with Trainline Partner Solutions or Lyko for immediate 200+ carrier access
2. **Week 2-4:** Integrate Stripe/Adyen with PSD2 compliance
3. **Month 1:** Register single EU entity (Estonia e-Residency fastest)
4. **Month 1-2:** Register for OSS VAT scheme
5. **Month 2-3:** Add GTFS-RT feeds for disruption layer
6. **Month 3-6:** Negotiate direct operator partnerships for better margins

**Key Insight:** Aggregators provide 80% of functionality for 20% of integration effort. Direct operator APIs only make sense at scale for margin optimization.

---

*Research compiled January 2026*

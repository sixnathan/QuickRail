# EU Rail Schedule/Timetable APIs Research

## Executive Summary

For a rail booking MVP, **GTFS static feeds are the recommended approach** for schedule data. They are standardized, widely available across the EU, and free for commercial use under permissive licenses. HAFAS APIs provide real-time capabilities but have rate limits and uncertain futures. The EU regulatory environment (Delegated Regulation 2017/1926) mandates open transport data via National Access Points (NAPs).

**Key Findings:**
- Germany (GTFS.de) offers one of the world's largest GTFS datasets with daily updates
- DB HAFAS API has been permanently shut off - use db-vendo-client with lower rate limits
- MERITS (UIC) provides pan-European integrated timetables covering ~600,000 train services (paid license)
- EU PSI Directive (2019/1024) establishes transport data as "high-value" requiring free reuse

---

## 1. GTFS Static Feeds

### Coverage by Country

| Country | Provider | GTFS URL | Update Frequency | License |
|---------|----------|----------|------------------|---------|
| **Germany** | DELFI/GTFS.de | https://gtfs.de/en/feeds/de_full/ | Daily | CC-BY 4.0 |
| **France** | SNCF/transport.data.gouv.fr | https://eu.ftp.opendatasoft.com/sncf/plandata/export-opendata-sncf-gtfs.zip | Regular | ODbL |
| **Switzerland** | SBB/opentransportdata.swiss | https://data.opentransportdata.swiss/de/dataset/timetable-2025-gtfs2020 | Daily | Open |
| **Netherlands** | ND-OV loket | Via GTFS-NL | Regular | CC0 |
| **Spain** | Renfe | https://data.renfe.com/dataset?res_format=GTFS | Varies | CC-BY 4.0 |
| **UK** | Rail Delivery Group | ATOC-CIF format (requires conversion) | Weekly | CC-BY 2.0 |
| **Austria** | ÖBB | https://data.oebb.at/de/datensaetze~soll-fahrplan-gtfs~ | Yearly (Dec-Dec) | Open |
| **Italy** | Trenitalia | Via Transitland (regional only) | Varies | Check source |

### Key GTFS Resources

- **Mobility Database**: https://mobilitydatabase.org/feeds - 4,000+ feeds from 70+ countries
- **European Transport Feeds**: https://eu.data.public-transport.earth/ - Centralized EU collection
- **Transitland**: https://transit.land - Feed registry with version history
- **TransitFeeds**: https://transitfeeds.com - Deprecated Dec 2025, migrate to Mobility Database

### Germany (Best Coverage)

DELFI dataset is one of the world's largest:
- 20,000+ lines
- 500,000+ stop points
- 2 million vehicle journeys
- Covers DB + all regional operators

Download: https://gtfs.de/en/feeds/
- Free basic feeds: 7-day validity, daily updates
- Extended feeds: Full timetable period (registration required at https://www.opendata-oepnv.de/)

### France

SNCF provides comprehensive coverage:
- TGV, Intercit, TER lines
- 151 days forward schedule
- GTFS + NeTEx + SIRI Lite (real-time)

Direct downloads:
- GTFS: https://eu.ftp.opendatasoft.com/sncf/plandata/export-opendata-sncf-gtfs.zip
- NeTEx: https://eu.ftp.opendatasoft.com/sncf/plandata/export-opendata-sncf-netex.zip
- Real-time: https://proxy.transport.data.gouv.fr/resource/sncf-siri-lite-estimated-timetable

NAP: https://transport.data.gouv.fr

### Switzerland

SBB/opentransportdata.swiss:
- Full year timetable (365 days from mid-December)
- API access requires token registration
- geOps provides daily GTFS conversions: https://gtfs.geops.ch/

API Manager: https://api-manager.opentransportdata.swiss/

### Netherlands

Most permissive licensing (CC0):
- All bus/metro/train/tram/ferry
- Includes international connections (Belgium, France, Germany, UK)
- GTFS-RT available

NS API Portal: https://apiportal.ns.nl/ (requires registration)

---

## 2. HAFAS Endpoints

### Current Status (CRITICAL)

**DB HAFAS API has been shut off permanently.** Replacement is `db-vendo-client` with significantly lower rate limits.

### Available Endpoints

| Operator | Status | Rate Limit | Notes |
|----------|--------|------------|-------|
| Deutsche Bahn | Deprecated | N/A | Use db-vendo-client |
| ÖBB (Austria) | Active | Unknown | Via hafas-client |
| SBB (Switzerland) | Active | Unknown | Via hafas-client |
| VBB (Berlin) | Active | Unknown | Regional |
| RMV (Frankfurt) | Active | Unknown | Regional |

### REST API Wrappers

- **v6.db.transport.rest**: https://v6.db.transport.rest/
  - Rate limit: 100 requests/minute
  - CORS enabled
  - No authentication required
  - Now uses db-vendo-client backend

### hafas-client (npm)

JavaScript client supporting multiple profiles:

```javascript
import {createClient} from 'hafas-client'
import {profile as oebbProfile} from 'hafas-client/p/oebb/index.js'

const client = createClient(oebbProfile, 'your-project-name')
```

Profiles available: DB, ÖBB, SBB, VBB, BVG, RMV, NAH.SH, and 20+ more

GitHub: https://github.com/public-transport/hafas-client

### Recommendation

Use HAFAS sparingly for real-time only. For static schedules, prefer GTFS. HAFAS endpoints are:
- Unofficial/unsupported
- Subject to shutdown without notice
- Rate limited
- CORS blocked (need proxy server)

---

## 3. Public Timetable Data Sources

### EU-Level Portals

| Portal | URL | Description |
|--------|-----|-------------|
| data.europa.eu | https://data.europa.eu/data/datasets/rail-timetables | EU Open Data Portal |
| MERITS (UIC) | https://uic.org/passenger/passenger-services-group/merits | 67,680 stations, 4x/week updates |

### National Access Points (NAPs)

EU Delegated Regulation 2017/1926 requires member states to publish transport data:

| Country | NAP URL |
|---------|---------|
| Germany | https://www.opendata-oepnv.de/ |
| France | https://transport.data.gouv.fr |
| Switzerland | https://opentransportdata.swiss |
| Netherlands | https://ndovloket.nl/ |
| Spain | https://datos.gob.es/ + https://data.renfe.com |
| UK | https://www.nationalrail.co.uk/developers/ |

### MERITS Database (UIC)

Pan-European integrated timetable - the most comprehensive source:
- ~600,000 train services across Europe (including Russia, Turkey)
- 67,680 stations with geocoordinates
- Updated 3-4x/week
- Available in GTFS and EDIFACT formats
- Minimum connection times and pedestrian links included

**Commercial Access:**
- Third-party access approved April 2017, available since March 2019
- Requires paid license via UIC Shop: https://shop.uic.org/en/460-merits
- License tiers:
  - MERITS Gold (full Europe, 3x/week updates)
  - MERITS Gold Limited (1-3 countries, lower cost)
- Contact: merits@uic.org for pricing
- Data provided "as is" with no warranties on completeness

---

## 4. Data Formats & Parsing

### GTFS (Recommended)

Standard files in ZIP archive:
- `stops.txt` - Station data
- `routes.txt` - Route definitions
- `trips.txt` - Individual trips
- `stop_times.txt` - Departure/arrival times
- `calendar.txt` / `calendar_dates.txt` - Service dates

Parsing libraries:

**Python (Recommended for MVP):**
- `gtfs-kit` - Full toolkit with validation, merging, analysis
- `pygtfs` - SQLite-backed interface for efficient queries
- `partridge` - Memory-efficient for large feeds
- `transitfeed` (Google) - Official validation library

**JavaScript/Node.js:**
- `node-gtfs` - SQLite storage, multiple feed support
- `gtfs-utils` - Utilities for calendar flattening, time computation
- `gtfs-to-geojson` - Convert stops/shapes to GeoJSON

**TypeScript:**
- `gtfs-types` - Type definitions for GTFS structures

**Go:**
- `gtfs` - Native Go parsing

**Rust:**
- `gtfs-structures` - Parsing and validation
- `transit_model` - GTFS <-> NeTEx conversion

### NeTEx

EU regulatory standard (more complex):
- XML-based
- Multiple profiles per country
- More complete than GTFS in some cases

Conversion tools:
- `transit_model` (Rust): GTFS <-> NeTEx
- netex-java
- DATA4PT tools

### UK-Specific (ATOC-CIF)

Requires conversion to GTFS:
- **UK2GTFS** (R): https://github.com/ITSLeeds/UK2GTFS
- **ATOCCIF2GTFS**: https://github.com/thomasforth/ATOCCIF2GTFS

---

## 5. Licensing & Commercial Use

### License Comparison

| License | Commercial OK | Attribution | Share-Alike | Best For |
|---------|--------------|-------------|-------------|----------|
| **CC0** | Yes | No | No | Most flexible |
| **CC-BY** | Yes | Yes | No | Standard |
| **ODbL** | Yes | Yes | Yes (derivatives) | SNCF data |

### Key Considerations

1. **CC0 (Netherlands)**: No restrictions whatsoever - most permissive
2. **CC-BY (Germany, Spain, UK)**: Must credit source in application
3. **ODbL (SNCF France)**: Must share derivative databases under same license
4. **Open Government License (UK)**: Similar to CC-BY

### EU Regulatory Framework

**Delegated Regulation (EU) 2017/1926:**
- Requires National Access Points (NAPs) for multimodal travel info
- Mandates NeTEx format (GTFS often provided as alternative)
- Covers static timetables, routes, stops, fares

**PSI/Open Data Directive (EU) 2019/1024:**
- Transport data classified as "high-value"
- Must be available free of charge
- Machine-readable formats required
- APIs and bulk downloads mandated
- Applies to public undertakings (rail operators included)

**Implementing Regulation (EU) 2023/138:**
- Lists transport as high-value dataset category
- CC-BY 4.0 recommended license

### Commercial API Alternatives

If open data proves insufficient:

| Provider | Coverage | Notes |
|----------|----------|-------|
| **Trainline Partner Solutions** | 280+ rail/coach carriers | B2B API, 20+ year track record |
| **Omio** | 3,000+ transport providers | API + white-label solutions |
| **Lyko** | Renfe, Trenitalia, DB, others | No affiliation requirements |
| **Travelfusion** | 360+ carriers | tfRail API for rail content |
| **Distribusion** | Ground transport globally | REST API |
| **Amadeus** | 90+ railways worldwide | SOAP APIs, GDS integration |
| **Sabre** | 9 major EU networks | REST APIs |

**Trainline:** https://tps.thetrainline.com/our-products/global-api/
**Omio B2B:** https://www.omio.com/b2b

---

## Recommendations for QuickRail MVP

### Phase 1: Static Schedules

1. **Start with Germany (GTFS.de)** - Best coverage, daily updates, clear licensing
2. **Add France (SNCF)** - Comprehensive, ODbL (attribution + share-alike)
3. **Add Switzerland (SBB)** - Excellent API, token required
4. **Add Netherlands** - CC0 license, most permissive

### Phase 2: Real-Time

1. Use GTFS-RT where available (SNCF, SBB, Netherlands)
2. Supplement with HAFAS for ÖBB/regional operators
3. Consider v6.db.transport.rest for German real-time (100 req/min)

### Architecture Notes

- **No LLM needed** for basic routing - use OpenTripPlanner or custom graph
- Cache GTFS feeds locally, refresh daily
- Build station matching layer (normalize names across countries)
- Handle timezone differences (esp. cross-border routes)
- Consider MERITS for cross-border journeys if free data proves insufficient

### Data Gaps to Note

- **Italy (Trenitalia)**: No comprehensive official GTFS for national network; ViaggiaTreno API is unreliable
- **Cross-border**: Individual country feeds may not include international trains - MERITS fills this gap
- **Real-time delays**: GTFS-RT coverage varies significantly by operator

---

## Sources

### Official Data Portals
- [GTFS.de - Germany](https://gtfs.de/en/)
- [SNCF Open Data](https://ressources.data.sncf.com/)
- [transport.data.gouv.fr - France NAP](https://transport.data.gouv.fr)
- [opentransportdata.swiss](https://opentransportdata.swiss/en/)
- [ÖBB Open Data](https://data.oebb.at/)
- [Mobilitydata Austria](https://www.mobilitydata.gv.at/)
- [NS API Portal](https://apiportal.ns.nl/)
- [Renfe Data](https://data.renfe.com/)
- [UK National Rail Developers](https://www.nationalrail.co.uk/developers/)

### Aggregators and Registries
- [Mobility Database](https://mobilitydatabase.org/feeds) - Successor to TransitFeeds
- [European Transport Feeds](https://eu.data.public-transport.earth/)
- [Transitland](https://transit.land/)
- [data.europa.eu Rail Timetables](https://data.europa.eu/data/datasets/rail-timetables)

### Technical Resources
- [hafas-client GitHub](https://github.com/public-transport/hafas-client)
- [v6.db.transport.rest](https://v6.db.transport.rest/)
- [GTFS Specification](https://gtfs.org/)
- [NeTEx Standard](https://www.netex-cen.eu/)
- [gtfs-kit (Python)](https://github.com/mrcagney/gtfs_kit)

### Pan-European Data
- [MERITS/UIC](https://uic.org/passenger/passenger-services-group/merits)
- [UIC Shop - MERITS Licenses](https://shop.uic.org/en/460-merits)

### Commercial APIs
- [Trainline Partner Solutions](https://tps.thetrainline.com/)
- [Omio B2B](https://www.omio.com/b2b)
- [Lyko Rail API](https://lyko.tech/en/portfolio/train-api/)

### Regulatory
- [EU Delegated Regulation 2017/1926](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32017R1926)
- [PSI/Open Data Directive 2019/1024](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32019L1024)
- [ODbL License](https://opendatacommons.org/licenses/odbl/)

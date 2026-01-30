# EU Rail Schedule/Timetable APIs Research

## Executive Summary

For a rail booking MVP, **GTFS static feeds are the recommended approach** for schedule data. They're standardized, widely available across EU, and free for commercial use under permissive licenses. HAFAS APIs provide real-time capabilities but have rate limits and uncertain futures. The EU regulatory environment (Delegated Regulation 2017/1926) mandates open transport data via National Access Points (NAPs).

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
| **Austria** | ÖBB | Via Transitland/Mobility Database | Varies | Check source |
| **Italy** | Trenitalia | Limited official; available via Transitland | Varies | Check source |

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

### MERITS Database

UIC's comprehensive European timetable:
- 67,680 European stations with geocoordinates
- Updated 4x/week
- EDIFACT format (European standard)
- 12-month timetables (mid-December to mid-December)

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
- **Python**: `gtfs-kit`, `partridge`, `gtfstk`
- **JavaScript**: `gtfs`, `gtfs-utils`
- **Rust**: `gtfs-structures`, `transit_model`
- **Go**: `gtfs`

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

1. **CC0 (Netherlands)**: No restrictions whatsoever
2. **CC-BY (Germany, Spain, UK)**: Must credit source
3. **ODbL (SNCF France)**: Must share derivative databases under same license

### Commercial API Alternatives

If open data proves insufficient:

| Provider | Coverage | Notes |
|----------|----------|-------|
| **Trainline Partner Solutions** | 87 operators, 24 countries | B2B API, contact required |
| **Lyko** | Renfe, Trenitalia, others | Commercial integration |
| **SilverRail** | Multi-operator | Enterprise |

Trainline API: https://tps.thetrainline.com/our-products/global-api/

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

---

## Sources

- [GTFS.de - Germany](https://gtfs.de/en/)
- [transport.data.gouv.fr - France NAP](https://transport.data.gouv.fr)
- [opentransportdata.swiss](https://opentransportdata.swiss/en/)
- [Mobility Database](https://mobilitydatabase.org/feeds)
- [European Transport Feeds](https://eu.data.public-transport.earth/)
- [hafas-client GitHub](https://github.com/public-transport/hafas-client)
- [v6.db.transport.rest](https://v6.db.transport.rest/)
- [MERITS/UIC](https://uic.org/passenger/passenger-services-group/merits)
- [NS API Portal](https://apiportal.ns.nl/)
- [Trainline Partner Solutions](https://tps.thetrainline.com/)
- [data.europa.eu Rail Timetables](https://data.europa.eu/data/datasets/rail-timetables)
- [UK National Rail Developers](https://www.nationalrail.co.uk/developers/)
- [Renfe Data](https://data.renfe.com/)
- [ODbL License](https://opendatacommons.org/licenses/odbl/)

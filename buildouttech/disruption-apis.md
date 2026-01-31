# EU Rail Disruption Data APIs - Research Findings

## Executive Summary

Real-time rail disruption data in the EU is fragmented across national operators. GTFS-RT is the emerging standard but adoption varies. For LLM-powered rebooking logic, combining multiple sources with appropriate polling strategies will be essential.

---

## 1. GTFS-RT Feeds - EU Coverage

### What is GTFS-RT?
GTFS Realtime extends static GTFS with three feed types:
- **Trip Updates**: Delays, cancellations, changed routes
- **Service Alerts**: Disruptions affecting stations/routes/network
- **Vehicle Positions**: Real-time location and congestion

Protocol: Protocol Buffers (smaller, faster than XML)

### EU Operators with GTFS-RT

| Country | Operator | GTFS-RT Available | Update Frequency |
|---------|----------|-------------------|------------------|
| France | SNCF (TGV/TER/IC) | Yes | 2 minutes |
| France | Eurostar | Yes | 30 seconds |
| Finland | HSL | Yes | Real-time |
| Italy | GTT (Turin) | Yes | Real-time |
| Spain | Renfe | Partial (static GTFS) | N/A |
| Germany | DB | Via unofficial APIs | ~30 seconds |

### Key Endpoints - France (transport.data.gouv.fr)

```
# SNCF Trip Updates (delays/cancellations)
https://proxy.transport.data.gouv.fr/resource/sncf-all-gtfs-rt-trip-updates

# SNCF Service Alerts
https://proxy.transport.data.gouv.fr/resource/sncf-gtfs-rt-service-alerts

# SNCF SIRI Estimated Timetable (Beta)
https://proxy.transport.data.gouv.fr/resource/sncf-siri-lite-estimated-timetable

# SNCF SIRI Situation Exchange
https://proxy.transport.data.gouv.fr/resource/sncf-siri-lite-situation-exchange

# Static GTFS
https://eu.ftp.opendatasoft.com/sncf/plandata/export-opendata-sncf-gtfs.zip

# Eurostar (updates every 30 seconds)
Available via transport.data.gouv.fr
```

---

## 2. Deutsche Bahn Real-Time APIs

### Official API (developers.deutschebahn.com)
- **Registration**: Free account required
- **Portal**: https://developers.deutschebahn.com
- **Limitation**: Only 18 hours ahead, primarily long-distance

**Setup Process:**
1. Create account at developers.deutschebahn.com
2. Create application, save client ID/secret
3. Subscribe to "Timetables" API (free tier)
4. Use credentials for authentication

**Endpoints:**
- `get_timetable()` - Scheduled departures (no delays)
- `get_timetable_changes()` - Delays, platform changes, cancellations

### Unofficial REST API (Recommended for Development)
```
Base URL: https://v6.db.transport.rest

# Departures with real-time delays
GET /stops/{id}/departures

# Arrivals
GET /stops/{id}/arrivals

# Journey planning with disruptions
GET /journeys

# Real-time vehicle radar
GET /radar
```

**Advantages:**
- No authentication required
- Rate limit: 100 requests/minute
- CORS enabled (browser-friendly)
- Returns DB Navigator app data including delays

**Data Freshness**: Mirrors DB Navigator app; removes real-time data after journey arrival

---

## 3. SNCF Disruption Feeds

### Navitia API (Powers SNCF)
**Base URL**: https://api.navitia.io/v1

**Key Disruption Endpoints:**
```
# Traffic reports (disruptions)
GET /coverage/{region}/traffic_reports

# Departures with disruption status
GET /coverage/{region}/stop_areas/{stop_id}/departures

# Journey planning with real-time
GET /coverage/{region}/journeys

# Stop schedules
GET /coverage/{region}/stop_schedules
```

**Journey Status Values:**
- `NO_SERVICE` - Stop deleted in real-time
- `MODIFIED_SERVICE` - Stop added in real-time
- `SIGNIFICANT_DELAYS` - Early or late
- Empty - On schedule

**Coverage**: TGV, TER, Transilien, Intercites

### Direct SNCF Resources
- Traffic info: https://www.sncf-connect.com/en-en/trafficInfo
- App push notifications available for journey subscriptions

---

## 4. transport.data.gouv.fr (France National Access Point)

**URL**: https://transport.data.gouv.fr

### Available Data Types
- GTFS (static schedules)
- GTFS-RT (real-time updates)
- NeTEx (European standard)
- SIRI Lite (real-time interface)

### Key Features
- Proxy service for GTFS-RT distribution
- Quality validation tools
- 151-day schedule horizon for SNCF
- Update frequency: 2 minutes for real-time

### All GTFS-RT Datasets
Browse: https://transport.data.gouv.fr/datasets?q=gtfs-rt

---

## 5. Third-Party Aggregators

### Transitland (transit.land)
- **Coverage**: 2,500+ operators, 55+ countries
- **Data Types**: GTFS, GTFS-RT, GBFS
- **API**: REST API with JSON/GeoJSON output
- **Features**: Daily feed validation, archived versions
- **EU Rail**: Includes UK Rail Delivery Group feeds

### Mobility Database (mobilitydatabase.org)
- **Successor to**: TransitFeeds/OpenMobilityData
- **Note**: TransitFeeds deprecated December 2025
- **Coverage**: 4,000+ feeds from 70+ countries
- **API**: New REST API (US FTA funded)

### transport.rest Network
Community-maintained APIs wrapping national operator data:

| API | Coverage | URL |
|-----|----------|-----|
| DB | Germany | https://v6.db.transport.rest |
| OBB | Austria | https://v6.oebb.transport.rest |
| SBB | Switzerland | https://transport.opendata.ch |

---

## 6. Other Major EU Operators

### NS - Netherlands (ns.nl/en/travel-information/ns-api)
- **Registration**: https://apiportal.ns.nl
- **Features**: Departures, arrivals, journey planning, disruptions
- **Data Source**: InfoPlus feed via NDOVloket

### OBB - Austria
- **Official**: ÖBB Scotty app/API
- **Unofficial**: https://v6.oebb.transport.rest
- **Features**: Real-time delays, train radar, platform changes

### SBB - Switzerland
- **Delay Data**: Previous day via API
- **Real-time**: Push notifications via SBB Mobile app
- **Thresholds**: <3min not shown, 3-20min exact, 21-90min rounded to 5min

### Renfe - Spain
- **GTFS**: https://data.renfe.com (static only)
- **Real-time Map**: New interactive Cercanias tracker
- **No public API**: Real-time limited to app/website

### Trenitalia - Italy
- **Smart Caring**: Real-time notifications service
- **Community APIs**: GitHub projects for unofficial access
- **RFI Data**: iechub.rfi.it for commuter data

---

## 7. Push vs Polling Strategies

### GTFS-RT Best Practices (MobilityData Standards)

| Data Type | Max Age | Recommended Poll |
|-----------|---------|------------------|
| Trip Updates | 90 seconds | 30 seconds |
| Vehicle Positions | 90 seconds | 30 seconds |
| Service Alerts | 10 minutes | 1-5 minutes |

### Polling Implementation
```
Typical agency refresh: 1 second to 1 minute
Recommended client polling: Every 30 seconds
HTTP optimization: Use If-Modified-Since header
Error budget: <1% invalid responses acceptable
```

### Push/Webhook Options (Limited)
- **UK National Rail Darwin**: True push feeds available
- **SBB**: Push notifications via mobile app
- **DB**: No native webhooks; polling required
- **SNCF**: Journey subscription alerts via app

### Recommendation for LLM Rebooking
1. **Primary**: Poll GTFS-RT Trip Updates every 30 seconds
2. **Service Alerts**: Poll every 2-5 minutes
3. **Cache**: Implement client-side caching with TTL
4. **Backoff**: Exponential backoff on failures

---

## 8. Data Freshness & Reliability by Provider

| Provider | Freshness | Reliability | Notes |
|----------|-----------|-------------|-------|
| SNCF (GTFS-RT) | 2 min | High | Official gov.fr proxy |
| DB (transport.rest) | ~30 sec | Medium | Unofficial, rate limited |
| DB (Official) | Real-time | High | Auth required, limited scope |
| NS | Real-time | High | Official API portal |
| OBB | Real-time | Medium | Unofficial API available |
| SBB | 1-2 min lag | High | Different path than station boards |
| Renfe | N/A | Low | No real-time API |
| Trenitalia | Varies | Medium | No official public API |

### Known Issues
- **DB**: App and platform displays often mismatch (2025 reports)
- **SBB**: Online 1-2 min behind station boards
- **Renfe**: Highest delay rate in Spanish rail history (2024)

---

## 9. EU National Access Points (NAPs)

Under EU Regulation 2017/1926 (MMTIS), each member state operates a NAP:

| Country | NAP URL | Rail Data |
|---------|---------|-----------|
| France | transport.data.gouv.fr | Yes (GTFS-RT) |
| Belgium | transportdata.be | Yes |
| Slovenia | nap.si | Yes |
| Greece | nap.imet.gr | Yes |

**NAPCORE**: EU coordination project (since 2022) harmonizing NAPs across Europe.

---

## 10. LLM Rebooking Integration Considerations

### Data Inputs for LLM
1. **Current journey status** (from Trip Updates)
2. **Delay magnitude** (minutes late)
3. **Cancellation status**
4. **Alternative routes** (from journey planning API)
5. **Service alerts** (text descriptions)

### API Call Pattern for Disruption Detection
```
1. Monitor GTFS-RT Trip Updates (30s polling)
2. On delay >15min or cancellation:
   a. Fetch Service Alerts for context
   b. Query journey planning API for alternatives
   c. Pass to LLM with passenger preferences
   d. Execute rebooking via operator API
```

### Challenges
- **No unified EU booking API**: Must integrate per-operator
- **Language**: Alerts often in local language only
- **Compensation**: Different rules per country/operator
- **Real-time booking availability**: Not always exposed

---

## Key API Endpoints Summary

```yaml
# France - SNCF
trip_updates: https://proxy.transport.data.gouv.fr/resource/sncf-all-gtfs-rt-trip-updates
service_alerts: https://proxy.transport.data.gouv.fr/resource/sncf-gtfs-rt-service-alerts
navitia: https://api.navitia.io/v1/coverage/fr-idf/

# Germany - DB (Unofficial)
departures: https://v6.db.transport.rest/stops/{id}/departures
journeys: https://v6.db.transport.rest/journeys

# Austria - OBB (Unofficial)
departures: https://v6.oebb.transport.rest/stops/{id}/departures
journeys: https://v6.oebb.transport.rest/journeys

# Netherlands - NS
portal: https://apiportal.ns.nl

# Switzerland - SBB
api: https://transport.opendata.ch

# Aggregators
transitland: https://transit.land
mobility_db: https://mobilitydatabase.org
```

---

## Sources

- [GTFS.org - Realtime Best Practices](https://gtfs.org/documentation/realtime/realtime-best-practices/)
- [transport.data.gouv.fr](https://transport.data.gouv.fr)
- [Navitia.io Documentation](https://doc.navitia.io/)
- [v6.db.transport.rest](https://v6.db.transport.rest/)
- [v6.oebb.transport.rest](https://v6.oebb.transport.rest/)
- [NS API Portal](https://www.ns.nl/en/travel-information/ns-api)
- [Transitland](https://www.transit.land/)
- [Mobility Database](https://mobilitydatabase.org/)
- [EU NAPs - European Commission](https://transport.ec.europa.eu/transport-themes/smart-mobility/road/its-directive-and-action-plan/national-access-points_en)
- [Deutsche Bahn Developer Portal](https://developers.deutschebahn.com)

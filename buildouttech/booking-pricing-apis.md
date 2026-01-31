# EU Rail Booking & Pricing APIs - Research Summary

## Executive Summary

For a pre-seed MVP focused on EU rail booking, **Lyko** offers the fastest path to market with commission-based pricing (no upfront costs), multi-carrier coverage, and startup-friendly terms. **All Aboard** is a strong alternative for simpler use cases with embedded widgets.

---

## 1. Aggregator APIs Comparison

### Trainline Partner Solutions (TPS)
| Aspect | Details |
|--------|---------|
| **Coverage** | 280+ rail/coach carriers, 40+ countries |
| **Access Model** | B2B partnership required |
| **Pricing** | Commission + transaction fees (negotiated) |
| **Affiliate Rate** | 3% per sale (affiliate program) |
| **Sandbox** | Available for partners |
| **Certification** | Required - partnership approval process |
| **Best For** | Enterprise clients, high-volume TMCs |
| **MVP Fit** | LOW - lengthy partnership process, enterprise focus |

### SilverRail (SilverCore)
| Aspect | Details |
|--------|---------|
| **Coverage** | 8+ countries, major EU carriers |
| **Access Model** | Enterprise API platform |
| **Pricing** | Custom enterprise pricing (not public) |
| **Features** | Calendar Price Search, Seat Maps, Split Ticketing |
| **Sandbox** | Available |
| **Certification** | Enterprise onboarding required |
| **Best For** | GDS integrations, large OTAs, corporate travel |
| **MVP Fit** | LOW - enterprise-focused, complex onboarding |

### Lyko
| Aspect | Details |
|--------|---------|
| **Coverage** | 8 EU carriers: SNCF, DB, Trenitalia, Renfe, Eurostar, SNCB, NS, CFL |
| **Access Model** | Open API - no volume requirements |
| **Pricing** | Commission-sharing model (% of ticket sales) |
| **Upfront Costs** | None stated |
| **Sandbox** | Available |
| **Certification** | Minimal - API key access |
| **Documentation** | English/French, detailed |
| **Best For** | Startups, travel apps, mobility platforms |
| **MVP Fit** | HIGH - no barriers to entry, commission-only model |

### All Aboard
| Aspect | Details |
|--------|---------|
| **Coverage** | Pan-European, hundreds of train operators |
| **Access Model** | API + Embeddable widgets |
| **Pricing** | Commission-based (details via contact) |
| **Features** | Dashboard, Embeds (deploy in minutes), API |
| **Languages** | EN, DE, SE, NO (more on request) |
| **Currencies** | Most world currencies supported |
| **Best For** | Small teams, quick launches, travel startups |
| **MVP Fit** | HIGH - embeds allow launch within days |

### Distribusion
| Aspect | Details |
|--------|---------|
| **Coverage** | 40+ rail carriers, EU + North America + Asia |
| **Access Model** | B2B API, OSDM standard |
| **Pricing** | Enterprise (negotiated) |
| **Notable** | Selected for DB's multi-carrier sales solution |
| **Best For** | Multi-modal (bus, rail, ferry), corporate travel |
| **MVP Fit** | MEDIUM - broader focus, enterprise orientation |

### Rail Europe
| Aspect | Details |
|--------|---------|
| **Coverage** | 200+ train providers, 50+ railroads |
| **Access Model** | API, trade website, affiliation |
| **Products** | Tickets + Rail Passes (Eurail, Swiss Travel) |
| **Best For** | Tourist-focused, passes/multi-ride products |
| **MVP Fit** | MEDIUM - good for tourism-oriented MVP |

---

## 2. Direct Carrier APIs

| Carrier | Booking API Available? | Access Notes |
|---------|----------------------|--------------|
| **Deutsche Bahn (DB)** | Limited | DB API Marketplace exists but booking access restricted; timetable/info APIs available; use aggregator for booking |
| **SNCF** | No public booking API | OpenData for timetables only; booking requires partnership or aggregator |
| **Trenitalia** | No public booking API | ViaggiaTreno for real-time info only; use aggregator for booking |
| **Renfe** | No public booking API | Open data portal limited; use aggregator for booking |
| **Eurostar** | Partner API only | Available through aggregators (Lyko, TPS, SilverRail) |
| **SNCB (Belgium)** | Partner API only | Available through Lyko, SilverRail |
| **NS (Netherlands)** | Partner API only | Available through Lyko |
| **OBB (Austria)** | Partner API only | Limited direct access |

**Key Finding**: Direct carrier booking APIs are not publicly accessible. All require either formal partnerships (lengthy process) or integration through aggregators.

---

## 3. Commission Structures

| Provider | Model | Estimated Rate |
|----------|-------|----------------|
| **Lyko** | Revenue share | % of ticket sale (exact % negotiated) |
| **All Aboard** | Commission | Not publicly disclosed |
| **Trainline Affiliate** | Per-sale commission | 3% |
| **Trainline TPS** | Commission + fees | Negotiated B2B rates |
| **SilverRail** | Transaction fees | Enterprise pricing |
| **Distribusion** | Transaction-based | Enterprise pricing |
| **Rail Europe** | Commission | Partner-specific rates |

**Note**: Most B2B rail APIs do not publicly disclose exact commission rates. Lyko and All Aboard appear most transparent about commission-only models without upfront fees.

---

## 4. Integration Requirements

### Authentication & Access

| Provider | Auth Method | Access Process |
|----------|-------------|----------------|
| **Lyko** | API Key | Sign up, get credentials |
| **All Aboard** | API Key | Contact for API access |
| **TPS** | OAuth/API Key | Partnership application |
| **SilverRail** | OAuth 2.0 | Enterprise onboarding |
| **Distribusion** | API Key | Partner application |

### Sandbox/Test Environments

| Provider | Sandbox Available | Notes |
|----------|-------------------|-------|
| **Lyko** | Yes | Test bookings supported |
| **All Aboard** | Yes | Full test environment |
| **TPS** | Yes (partners only) | Requires approval |
| **SilverRail** | Yes | Enterprise process |
| **Distribusion** | Yes | Partner access required |

### Certification Requirements

- **TPS**: Full partnership certification required
- **SilverRail**: Enterprise certification process
- **Lyko**: Minimal - API integration validation
- **All Aboard**: Minimal - straightforward onboarding
- **RDG (UK)**: Formal accreditation required for UK rail

---

## 5. MVP Recommendation

### Fastest Path: Lyko or All Aboard

**For Full API Control: Lyko**
- Commission-only model (no upfront costs)
- 8 major EU carriers in single integration
- No volume requirements
- English documentation
- Direct booking without redirects
- Timeline: 2-4 weeks to production

**For Rapid Launch: All Aboard Embeds**
- Deploy searchable widget in days
- No backend integration needed initially
- Upgrade to API later
- Timeline: 1 week to basic booking capability

### Recommended MVP Architecture

```
Phase 1 (Week 1-2):
- All Aboard Embeds for immediate booking capability
- Validate user demand

Phase 2 (Week 3-6):
- Lyko API integration for full control
- Custom booking flow
- Multi-carrier coverage

Phase 3 (Post-MVP):
- Add carriers via TPS or SilverRail as volume grows
- Consider direct carrier partnerships
```

### Decision Matrix

| Criteria | Lyko | All Aboard | TPS | SilverRail |
|----------|------|------------|-----|------------|
| Time to market | Fast | Fastest | Slow | Slow |
| Upfront cost | None | None | Negotiated | High |
| EU coverage | Good (8) | Excellent | Excellent | Good |
| Startup-friendly | Yes | Yes | No | No |
| Documentation | Good | Good | Enterprise | Enterprise |
| **MVP Score** | **9/10** | **8/10** | **4/10** | **3/10** |

---

## 6. Key Contacts & Resources

- **Lyko**: https://lyko.tech/en/portfolio/train-api/
- **All Aboard**: https://allaboard.eu/api-solution
- **Trainline TPS**: https://tps.thetrainline.com/
- **SilverRail**: https://silverrailtech.com/
- **Distribusion**: https://www.distribusion.com/rail-solutions
- **Rail Europe Partners**: https://agent.raileurope.com/

---

## Sources

- [Trainline Partner Solutions](https://tps.thetrainline.com/)
- [SilverRail Technologies](https://silverrailtech.com/)
- [Lyko Train API](https://lyko.tech/en/portfolio/train-api/)
- [All Aboard API](https://allaboard.eu/api-solution)
- [Distribusion Rail Solutions](https://www.distribusion.com/rail-solutions)
- [Rail Europe Partners](https://agent.raileurope.com/)
- [AltexSoft Rail Booking API Guide](https://www.altexsoft.com/blog/rail-booking-integration/)
- [DB API Marketplace](https://developers.deutschebahn.com/)

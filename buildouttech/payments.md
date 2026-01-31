# EU Multi-Country Rail Booking Payment Infrastructure

## Executive Summary

For a pre-seed MVP requiring fast integration, **Stripe is the clear recommendation**. Adyen targets mid-market/enterprise with 5-6 month onboarding timelines and minimum volume requirements unsuitable for early-stage startups.

---

## 1. Stripe vs Adyen: EU Coverage & Rail/Travel Features

### Stripe (Recommended for MVP)

| Factor | Details |
|--------|---------|
| **Target Market** | Startups, SMBs, fast-growing companies |
| **Onboarding** | Instant approval, self-service |
| **Integration Time** | Days to weeks |
| **EU Coverage** | Full EEA coverage, local acquiring |
| **Travel Features** | Standard payment processing, no travel-specific features |
| **Developer Experience** | Industry-leading documentation, SDKs in all major languages, CLI tools, AI/MCP integration |

### Adyen (Future Consideration)

| Factor | Details |
|--------|---------|
| **Target Market** | Mid-market to enterprise (EUR 2M+ ARR) |
| **Onboarding** | 5-6 month implementation timeline, application review required |
| **Integration Time** | Months |
| **EU Coverage** | Excellent - Dutch company with European banking license |
| **Travel Features** | Purpose-built for travel/hospitality: dynamic pricing, pre-authorizations |
| **Developer Experience** | Good but complex, enterprise-focused |

### Pricing Comparison (EU)

**Stripe:**
- Blended rate: ~1.5% + EUR 0.25 (EU cards)
- IC++ available for high volume (negotiated)
- No monthly minimums

**Adyen:**
- Interchange++ only: IC fee + scheme fee + 0.60% markup + EUR 0.10-0.15 per transaction
- Average EU consumer card transaction: ~1%
- Minimum monthly invoice requirements
- EUR 25 per chargeback

**EU Interchange Caps (Both Providers):**
- Consumer debit: 0.2%
- Consumer credit: 0.3%

### Recommendation

**Start with Stripe. Re-evaluate Adyen at EUR 2M+ annual processing volume.**

**Integration Complexity: LOW (1-2 weeks for basic checkout)**

---

## 2. PSD2/SCA Implementation: 3D Secure 2.0

### Requirements

All card-not-present payments within EEA require Strong Customer Authentication (SCA) using two of three factors:
- **Knowledge**: Password, PIN
- **Possession**: Phone, hardware token
- **Inherence**: Biometrics

### Implementation with Stripe

```javascript
// Stripe Payment Intent with 3D Secure
const paymentIntent = await stripe.paymentIntents.create({
  amount: 5000,
  currency: 'eur',
  payment_method: 'pm_card_visa',
  confirmation_method: 'manual',
  confirm: true,
  return_url: 'https://quickrail.com/payment/complete',
  // Stripe automatically handles 3DS when required
});
```

### SCA Exemptions for Travel

| Exemption Type | Threshold | Requirements |
|----------------|-----------|--------------|
| **Low Value** | < EUR 30 | Cumulative limit EUR 100 or 5 consecutive |
| **TRA (Transaction Risk Analysis)** | < EUR 100 | Fraud rate < 0.13% |
| **TRA** | EUR 100-250 | Fraud rate < 0.06% |
| **TRA** | EUR 250-500 | Fraud rate < 0.01% |
| **One Leg Out** | Any | Issuer OR acquirer outside EEA |

**Travel-Specific Note:** No special travel exemption exists. However, 40-50% of e-commerce transactions may qualify for TRA exemption if fraud rates are maintained below thresholds.

### PSD3 Considerations (2025-2026)

- Tighter SCA rules and exemption management
- Real-time fraud monitoring requirements
- Increased fintech supervision

### Implementation Strategy

1. Use Stripe's automatic SCA handling (default behavior)
2. Request TRA exemptions for low-risk transactions
3. Monitor fraud rates quarterly to maintain exemption eligibility
4. Implement frictionless 3DS2 for better conversion

**Integration Complexity: LOW (handled by Stripe SDK)**

---

## 3. Multi-Currency Handling

### Stripe Multi-Currency Support

- **135+ currencies** supported
- **Presentment currency**: Charge in customer's local currency
- **Settlement currency**: Receive in your bank's currency

```javascript
// Multi-currency payment
const paymentIntent = await stripe.paymentIntents.create({
  amount: 4500,
  currency: 'sek', // Swedish Krona
  automatic_payment_methods: { enabled: true },
});
```

### Dynamic Currency Conversion (DCC)

Not recommended for travel MVP - adds complexity and customer confusion. Instead:
- Display prices in local currency
- Charge in local currency
- Let Stripe handle FX at settlement

### Recommended Currency Strategy

| Market | Presentment Currency | Notes |
|--------|---------------------|-------|
| Eurozone | EUR | 20 countries |
| UK | GBP | Post-Brexit |
| Sweden | SEK | Non-Euro EU |
| Denmark | DKK | Non-Euro EU |
| Switzerland | CHF | Non-EU |
| Poland | PLN | Non-Euro EU |

### Settlement Options

- Single EUR account (simplest for MVP)
- Multiple currency accounts (reduce FX fees at scale)

**Integration Complexity: LOW (native Stripe feature)**

---

## 4. Local Payment Methods

### Priority Payment Methods by Country

| Country | Method | Stripe Support | Market Share |
|---------|--------|----------------|--------------|
| **Netherlands** | iDEAL | Yes | 60%+ online |
| **Belgium** | Bancontact | Yes | 50%+ online |
| **Germany** | SEPA Direct Debit, Klarna | Yes | High |
| **France** | Cartes Bancaires | Yes (via cards) | 90%+ cards |
| **Nordics** | Klarna, Swish (SE) | Yes | High |
| **Pan-EU** | SEPA Credit Transfer | Yes | B2B |

### Implementation

```javascript
// Stripe Payment Element with local methods
const elements = stripe.elements({
  mode: 'payment',
  amount: 4999,
  currency: 'eur',
  paymentMethodTypes: [
    'card',
    'ideal',
    'bancontact',
    'sepa_debit',
    'klarna'
  ],
});
```

### iDEAL (Netherlands)

- Bank redirect, instant confirmation
- SEPA mandate created for recurring (if needed)
- Settlement: 1-2 business days

### Bancontact (Belgium)

- Bank redirect, instant confirmation
- Supports recurring via SEPA mandate
- Settlement: 1-2 business days

### Klarna (Buy Now Pay Later)

- Pay Now, Pay Later, Pay in Installments
- Available in 22 countries
- Higher AOV potential for expensive routes
- Settlement: Next business day

### SEPA Direct Debit

- For recurring/subscription models
- 14-day chargeback window (vs 120 days for cards)
- Lower fees than cards
- Best for: loyalty programs, rail passes

### MVP Recommendation

**Phase 1:** Cards + iDEAL + Bancontact (cover 80%+ of EU transactions)
**Phase 2:** Add Klarna, SEPA for specific use cases

**Integration Complexity: MEDIUM (2-3 weeks with Payment Element)**

---

## 5. Refund & Chargeback Handling

### Rail Ticket Refund Best Practices

| Scenario | Recommended Policy | Implementation |
|----------|-------------------|----------------|
| **< 24h from booking** | Full refund | Automatic via API |
| **24h-7d before departure** | 80-90% refund | Admin approval |
| **< 24h before departure** | 50% or voucher | Admin approval |
| **No-show** | No refund | N/A |
| **Operator cancellation** | Full refund | Automatic |

### Stripe Refund Implementation

```javascript
// Full refund
const refund = await stripe.refunds.create({
  payment_intent: 'pi_xxx',
  reason: 'requested_by_customer',
});

// Partial refund (e.g., 80%)
const partialRefund = await stripe.refunds.create({
  payment_intent: 'pi_xxx',
  amount: 4000, // 80% of 5000
});
```

### Chargeback Prevention

**Documentation Requirements:**
- Clear refund policy displayed at checkout
- Order confirmation emails with policy
- Proof of ticket delivery (PDF, mobile pass)
- Customer service interaction logs

**Technical Measures:**
1. Address Verification (AVS)
2. CVV verification
3. 3D Secure authentication
4. Velocity checks (limit transactions per IP/card)
5. Device fingerprinting

### Chargeback Response

- Stripe chargeback fee: Variable by region
- Adyen chargeback fee: EUR 25
- Response window: 7-21 days
- Win rate with proper documentation: 40-60%

### Key Metrics to Monitor

| Metric | Target | Action if Exceeded |
|--------|--------|-------------------|
| Chargeback rate | < 0.5% | Review fraud rules |
| Refund rate | < 5% | Improve policies |
| Fraud rate | < 0.1% | Tighten controls |

**Integration Complexity: LOW (built into Stripe)**

---

## 6. Payment Latency Optimization

### Tokenization Strategy

**Network Tokenization Benefits:**
- 0.10% interchange reduction (Visa CNP)
- Higher authorization rates
- Auto-updating card credentials
- Sub-100ms token resolution

```javascript
// Stripe automatically uses network tokens when available
const customer = await stripe.customers.create({
  email: 'user@example.com',
});

const setupIntent = await stripe.setupIntents.create({
  customer: customer.id,
  payment_method_types: ['card'],
  usage: 'off_session', // For saved cards
});
```

### Saved Cards (Returning Customers)

```javascript
// Charge saved card
const paymentIntent = await stripe.paymentIntents.create({
  amount: 5000,
  currency: 'eur',
  customer: 'cus_xxx',
  payment_method: 'pm_xxx',
  off_session: true,
  confirm: true,
});
```

### Optimization Techniques

| Technique | Latency Impact | Implementation |
|-----------|---------------|----------------|
| **Payment Element** | Optimized | Use Stripe.js |
| **Network Tokens** | -50ms | Automatic |
| **Saved Cards** | -200ms | Customer accounts |
| **Pre-fetch Payment Intent** | -100ms | Create on page load |
| **Optimistic UI** | Perceived -500ms | Show success early |

### Pre-fetching Strategy

```javascript
// Create Payment Intent when user selects route (not at checkout)
const preCreatedIntent = await stripe.paymentIntents.create({
  amount: calculateRoutePrice(route),
  currency: 'eur',
  automatic_payment_methods: { enabled: true },
  capture_method: 'manual', // Capture after confirmation
});
```

### Target Metrics

| Metric | Target |
|--------|--------|
| Time to first byte | < 200ms |
| Payment form render | < 500ms |
| Payment processing | < 3s (incl. 3DS) |
| Total checkout time | < 10s |

**Integration Complexity: MEDIUM (requires frontend optimization)**

---

## 7. Fraud Prevention

### Stripe Radar

**Capabilities:**
- ML-based scoring on 1000+ signals
- 92% of cards seen before on network
- Device fingerprinting
- IP geolocation
- Behavioral analytics
- 40% fraud reduction vs manual review

**Pricing:**
- Radar for Fraud Teams: $0.02-0.07 per screened transaction
- Basic Radar: Included with Stripe

### Custom Fraud Rules

```javascript
// Stripe Radar rules (configured in Dashboard)
// Block if: risk_level = 'highest'
// Review if: risk_score > 65
// Allow if: customer_lifetime_value > 1000

// Manual rule example: Block disposable emails
// :email_domain: in ('tempmail.com', 'throwaway.com')
```

### Travel-Specific Fraud Patterns

| Risk Signal | Action |
|-------------|--------|
| First-time customer + high-value route | 3DS required |
| Multiple bookings same card, different names | Block or review |
| IP country != billing country | Review |
| Booking far-future travel | Lower risk |
| Last-minute booking + new card | Higher risk |
| Velocity: >3 bookings in 1 hour | Review |

### Recommended Radar Rules

```
# Block high-risk
Block if :risk_level: = 'highest'

# Review elevated risk
Review if :risk_level: = 'elevated'

# Block card testing attempts
Block if :card_funding: = 'prepaid' AND :is_new_card: = true AND :amount_in_eur: < 10

# Review first-time expensive bookings
Review if :is_new_customer: = true AND :amount_in_eur: > 200
```

### 3D Secure Strategy

```javascript
// Request 3DS for risky transactions
const paymentIntent = await stripe.paymentIntents.create({
  amount: 5000,
  currency: 'eur',
  payment_method: 'pm_xxx',
  payment_method_options: {
    card: {
      request_three_d_secure: 'automatic', // or 'any' for always
    },
  },
});
```

**Integration Complexity: LOW-MEDIUM (Radar included, custom rules optional)**

---

## Implementation Roadmap

### Phase 1: MVP (Weeks 1-2)

- [ ] Stripe account setup
- [ ] Payment Element integration
- [ ] Card payments (EUR)
- [ ] Basic refund handling
- [ ] Stripe Radar (default)

### Phase 2: EU Expansion (Weeks 3-4)

- [ ] iDEAL (Netherlands)
- [ ] Bancontact (Belgium)
- [ ] Multi-currency support (GBP, SEK, CHF)
- [ ] Saved cards / customer accounts

### Phase 3: Optimization (Month 2)

- [ ] Custom Radar rules
- [ ] Klarna BNPL
- [ ] Payment latency optimization
- [ ] Chargeback prevention workflow

### Phase 4: Scale (Month 3+)

- [ ] SEPA Direct Debit (subscriptions/passes)
- [ ] IC++ pricing negotiation
- [ ] Evaluate Adyen for enterprise features

---

## Cost Estimates (Monthly)

### Low Volume (1,000 transactions, EUR 50 avg)

| Item | Cost |
|------|------|
| Stripe processing (1.5% + 0.25) | EUR 1,000 |
| Radar (included) | EUR 0 |
| **Total** | **~EUR 1,000** |

### Medium Volume (10,000 transactions, EUR 50 avg)

| Item | Cost |
|------|------|
| Stripe processing | EUR 7,500 |
| Radar for Fraud Teams | EUR 200-700 |
| Chargebacks (~0.3%) | EUR 150 |
| **Total** | **~EUR 8,000-8,500** |

---

## Key Decisions Summary

| Decision | Recommendation | Rationale |
|----------|---------------|-----------|
| Payment Provider | Stripe | Fast integration, no minimums, great DX |
| 3D Secure | Automatic with exemptions | Balance security/conversion |
| Local Methods | iDEAL + Bancontact first | Highest impact for EU |
| Multi-currency | EUR primary, expand as needed | Simplicity |
| Fraud Prevention | Stripe Radar | Included, effective |
| Tokenization | Enabled by default | Better auth rates |

---

## Sources

- [Stripe vs Adyen Comparison (Airwallex)](https://www.airwallex.com/au/blog/comparison-stripe-vs-adyen)
- [Top Payment Gateways in Europe 2026](https://www.payfirmly.com/blogs/top-payment-gateways-in-europe)
- [PSD2 SCA Compliance (Mastercard)](https://na-gateway.mastercard.com/api/documentation/integrationGuidelines/supportedFeatures/psd2ScaCompliance.html)
- [SCA Transaction Optimization Guide (Ravelin)](https://www.ravelin.com/blog/sca-transaction-optimization-guide-exemptions)
- [Adyen Pricing](https://www.adyen.com/pricing)
- [Stripe Pricing](https://stripe.com/pricing)
- [Stripe Radar Documentation](https://docs.stripe.com/radar)
- [Adyen Risk Management](https://docs.adyen.com/risk-management)
- [TRA Exemptions (SEON)](https://seon.io/resources/tra-exemptions/)
- [Network Tokenization (Optimized Payments)](https://optimizedpayments.com/insights/card-fees/network-tokenization-a-strategic-advantage-in-modern-payments/)
- [Stripe for Startups](https://stripe.com/startups)
- [Adyen Onboarding](https://docs.adyen.com/platforms/quickstart-guide/onboarding-and-kyc)

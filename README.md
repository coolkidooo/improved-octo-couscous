# Car Rental — DDD Scaffold

This is a practice project for a car-rental website using **Domain-Driven Design (DDD)**.  
The MVP ships **without Sign-In and Stripe**, but the architecture already includes stubs so they can be enabled later with minimal refactor.

## Monorepo layout
```
apps/            # UI (Next.js later)
services/        # Bounded contexts
  booking/       # Booking aggregate, commands, events, ports, policies
  fleet/         # Vehicle specs, photos, status
  availability/  # Per-vehicle calendar rules & blocks
  pricing/       # Flat daily rate, taxes, add-ons, policies
  payments/      # Port/adapter (stub now, Stripe later)
  iam/           # Principal provider (Guest now, users later)
  notifications/ # Email templates & sender
libs/
  shared-kernel/ # Money, DateRange, IDs, Result, guards
  events/        # Domain/integration event contracts
  persistence/   # Repo base, outbox, migrations
ops/             # env/scripts/CI later
```

## Feature flags (MVP)
- `AUTH_ENABLED=false`  → everyone is Guest (IAM stub)  
- `PAYMENTS_ENABLED=false` → Stripe skipped (payments stub)  
- `DEPOSIT_ENABLED=false`  
- `CHECKIN_ENABLED=false`  

## MVP scope
- Search → Vehicle details → 3-step booking (confirm without Stripe)  
- Availability blocks dates on confirmation; cancel unblocks  
- Flat daily rate (global), taxes, optional add-ons  
- Email confirmation via dev sender  

## Future upgrades
- Enable Auth (NextAuth) → My Trips  
- Enable Stripe (Payment Intents + webhooks)  
- Admin backoffice (roles/permissions, damage reports)  
- Document generation (agreement PDF), photo check-in/out  

## Bounded contexts (overview)
- **Fleet** — Vehicle (specs, photos, status)  
- **Availability** — Calendar rules/blocks  
- **Pricing & Policy** — Flat rate, taxes, add-ons, cancellation  
- **Booking** — Requested → Confirmed → Completed/Cancelled  
- **Payments** — Port/adapter; stub now, Stripe later  
- **IAM** — Principal provider; stub now, users later  
- **Notifications** — Email templates & sender  

## API (public) — MVP (no auth)
- `GET /vehicles?location&from&to` → Search results  
- `GET /vehicles/{slug}` → Vehicle details  
- `POST /quote` `{vehicleId, from, to, addOns}` → Pricing breakdown  
- `POST /bookings` `{vehicleId, trip, driverInfo, addOns}` → `{bookingId, status:'confirmed'}`  
- `GET /bookings/{id}` → Booking view  

_Admin (temporary, gated at edge):_  
`/admin/vehicles/*`, `/admin/bookings/*`, `/admin/pricing/*`

## Domain events (stable names)
- `BookingRequested`, `BookingConfirmed`, `BookingCancelled`  
- *(Future)* `BookingPaymentAuthorized`, `PaymentCaptured`, `PaymentRefunded`, `DepositReleased`

## Data model (MVP → future-ready)
- **Vehicle** `{id, slug, specs..., status, photos[]}`  
- **PricingPlan** `{flat_daily_rate, tax_rate, min_days, addOns[], deposit_amount}`  
- **Booking** `{id, vehicle_id, start_at, end_at, pickup_mode, driver_*, extras_json, pricing_snapshot_json, status}`  
  - Future fields: `renter_user_id?`, `payment_intent_id?`, `deposit_hold?`  

## Notes
- Keep `AUTH_ENABLED=false`, `PAYMENTS_ENABLED=false` for MVP.  
- Use `.env.example` as a template; **do not commit real secrets**.  
- Empty folders are tracked with `.gitkeep`.

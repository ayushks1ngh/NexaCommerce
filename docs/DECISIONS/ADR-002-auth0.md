# ADR-002: Auth0 for customer and administrator identity

- **Status:** ACCEPTED; integration is PLANNED and NOT YET IMPLEMENTED
- **Date:** 2026-10-02

## Context

NexaCommerce needs registration, login, logout, account recovery, secure token issuance, and administrator authorization. Identity is security-critical but is not the commerce problem the project aims to reinvent. Application profile data and commerce policy must remain under NexaCommerce ownership.

## Problem

Should NexaCommerce build identity itself or use an identity provider, and where is the boundary between identity and application user data?

## Options considered

1. **Auth0:** managed OAuth 2.0/OpenID Connect provider with SDKs, hosted login, extensibility, and operational support.
2. **Another managed provider** such as Amazon Cognito, Clerk, Okta, or a cloud-native identity service.
3. **Self-hosted provider** such as Keycloak.
4. **Custom password/session system.**

## Decision

Use **Auth0 as the identity provider**. Prefer Authorization Code flow with PKCE/appropriate secure Next.js integration, hosted Universal Login, and short-lived audience-bound access tokens. Backend APIs validate tokens and permissions. Auth0 owns credentials, authentication factors, recovery, and identity-provider records. The User Service owns application profile, addresses, commerce status/preferences, and the mapping to the immutable Auth0 `sub` claim.

Use Auth0 roles/permissions or namespaced claims only to convey coarse application authorization. Each service still enforces resource ownership and business authorization. Detailed token storage/session architecture is decided during Phase 2 after reviewing the current Auth0/Next.js guidance and threat model.

## Rationale

- Avoids implementing password storage, reset, MFA, federation, and attack defenses from scratch.
- Teaches standards used by real systems: OAuth 2.0, OIDC, JWT validation, scopes, and claims.
- Provides a clear boundary between authentication and commerce data.
- Hosted login reduces credential exposure to NexaCommerce.

## Consequences

### Positive

- Faster and safer identity foundation than a custom implementation.
- Supports future MFA and social/enterprise connections without redesigning commerce records.
- Central identity lifecycle with API-local authorization.

### Negative and risks

- Vendor cost, quotas, outages, configuration complexity, and lock-in.
- Login UX and availability depend on an external service.
- Incorrect callback, audience, issuer, claim, cookie, or token handling remains dangerous.
- User deletion/profile synchronization spans Auth0 and the User Service.

### Required mitigations

- Use tenant separation/configuration appropriate to environments and infrastructure-as-code only in a later phase.
- Validate signature, issuer, audience, expiry, algorithm, and permissions in every API trust boundary.
- Keep tokens/secrets out of logs and browser storage where avoidable; narrowly allow callbacks/logout URLs.
- Use Auth0 `sub`, not email, as the identity link; design provisioning/synchronization idempotently.
- Document outage behavior and an export/migration strategy before public launch.

## Alternatives rejected

- **Custom identity:** rejected because its security and operations risk outweigh learning value; standards can be learned through integration.
- **Self-hosted Keycloak:** rejected initially because patching, availability, backups, and hardening create premature operational burden.
- **Other managed providers:** not inherently unsuitable; rejected as the default because Auth0 is an explicit project objective and fits the learning scope. Provider choice should be revisited if cost, region, features, or portability become unacceptable.

## Revisit when

Reconsider before production procurement and when pricing, data residency, required authentication methods, availability, tenant portability, or compliance needs are known. A provider change must preserve the internal User ID and avoid coupling business records to Auth0-specific profile fields.

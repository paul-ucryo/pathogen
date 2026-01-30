# LICENSE 42 — Folder Policy License (Construction Format)
Version: 0.1

This license is not a static EULA. It is a **policy construction format** for Folder.

Folder operates by routing intent through context and mediation. The allowed behavior of the system is defined by a **Policy Bundle** composed from modular capability sets.

## 1) Policy Bundle Authority
- The active ruleset is the currently selected Policy Bundle (YAML) located at:
  - `policy/bundles/license42.bundle.yaml`
- Bundles may be swapped, upgraded, or composed, subject to invariants.

## 2) Invariants (Non-Overridable)
The following invariants MUST be preserved by any bundle:
1. Intent does not bypass context evaluation.
2. Context does not bypass mediation.
3. Mediation does not bypass reversibility / rollback.
4. All nodes are mediatable within scope.
5. The system must remain learnable (no permanent lock-in without review).

## 3) Capabilities (Modular, Composable)
All permissions are expressed as capabilities. Capabilities are granted by bundles, constrained by context, and optionally gated by authentication.

## 4) Security Upgrades
Bundles may require stronger security over time:
- No auth → local auth → federated auth (SSO/OIDC)
- Plain transport → TLS transport
- Unsigned policies → signed policy bundles with verification

## 5) Enforcement Mode
Policy can be enforced by:
- software (policy router / correlator)
- process (human mediation)
- or hybrids
as specified by the active bundle.

## 6) Binding
Use of Folder under License 42 constitutes agreement to operate under the currently active Policy Bundle and its invariants.

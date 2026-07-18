# ADR-0003: signed user extensions and allowlisted built-in native messaging

- Status: Proposed
- Date: 2026-07-18
- Owners: mixi-bro maintainers

## Context

GeckoView supports persistent WebExtension install, enable, disable, update, uninstall, actions, and built-in extensions. User extensions can read or modify page data according to permissions. Built-in extensions are packaged by the app, do not require Mozilla signing, and can use native messaging, making them privileged application code.

## Decision

mixi-bro distinguishes:

1. one minimal built-in bridge extension packaged in the APK, with a stable ID, versioned typed message protocol, explicit native-app/channel allowlist, payload limits, and no Android-provided arbitrary JavaScript;
2. user extensions installed only from an approved Mozilla-signed XPI source in production, with a full permission prompt, private access disabled by default, visible source/version/status, and controlled enable/disable/update/uninstall flows.

Unsigned local extension installation is limited to clearly labeled non-release developer mode. User extensions receive no native messaging by default.

## Consequences

### Positive

- uses GeckoView's signature and permission mechanisms;
- limits native compromise surface;
- gives users transparent extension and private-mode control;
- supports a compatibility-tested extension catalog without claiming universal desktop support.

### Negative / tradeoffs

- fewer install sources and add-ons than an unrestricted browser;
- permission/UI/update plumbing is substantial;
- some desktop Firefox extensions will be incompatible;
- built-in bridge changes require app releases and security review.

## Alternatives considered

- Allow arbitrary unsigned XPI files in production: rejected due supply-chain and permission risk.
- Give all extensions native messaging: rejected as unnecessary native capability exposure.
- No user extensions: rejected because extension support is a stated product requirement.
- General userscript injection: deferred beyond version one.

## Validation and rollback

Every extension change receives `extension_security_reviewer` review and adversarial tests. A broken user extension can be disabled/uninstalled. A built-in bridge regression can disable dependent features safely; it must never crash normal browsing or fall back to unvalidated script injection.

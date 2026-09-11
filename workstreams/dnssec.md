# DNSSEC Automation

Status: Discovery and pilot design

Goal: Automate DNSSEC across independently operated registrars and DNS providers using existing standards.

## Current understanding

CDS/CDNSKEY and RFC 9615 provide standards-based child-to-parent signaling. Seamless may provide coordination and/or Parental Agent infrastructure where needed.

See the [Registrar Integration Checklist](../registrar-integration-checklist.md) and [DNS Provider Integration Checklist](../dns-provider-integration-checklist.md).

## Pilot

- Domain: `seamlessdns.org`
- Domain Owner: Linux Foundation
- Registrar: DNSimple
- DNS Provider: deSEC

## Open questions

- Should initiation be registrar-originated, DNS-provider-originated, or support both?

## How to participate

Share implementation experience and interest in the pilot.

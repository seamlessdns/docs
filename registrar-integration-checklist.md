# Registrar Integration Checklist

The initial Registrar integration is DNSSEC only.

- [ ] Let Seamless Connect act on an authorized request to enable DNSSEC for a domain.
- [ ] Support reading and updating DS state through an existing API or Parental Agent mechanism.
- [ ] Process standard CDS/CDNSKEY records according to Registrar policy.
- [ ] Let Seamless Connect verify that the DS record was published.

The working pilot starts at the Registrar. A DNS-provider-originated flow should also work where the Registrar already scans CDS/CDNSKEY records.

No registration, transfer, pricing, payment, renewal, or other Registrar integration is required for this pilot.

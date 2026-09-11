# DNS Provider Integration Checklist

For DNS updates:

- [ ] Allow Seamless Connect to use the existing DNS API.
- [ ] Enable OAuth with appropriately scoped DNS permissions for Seamless Connect.
- [ ] Optionally publish `_domainconnect` responses that point to Seamless Connect endpoints.

For providers without OAuth, Seamless Connect may initially support a token-based integration.

For DNSSEC:

- [ ] Allow Seamless Connect to enable or inspect DNSSEC through the existing API, where supported.
- [ ] Publish standard CDS/CDNSKEY records when the zone is ready.

Seamless Connect coordinates the registrar side.

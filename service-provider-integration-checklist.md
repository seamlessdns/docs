# Service Provider Integration Checklist

For the initial DNS configuration use case:

- [ ] Implement standard Domain Connect.
- [ ] Either route Domain Connect traffic through Seamless Connect, or use Seamless Connect as a fallback when normal Domain Connect discovery returns no `_domainconnect` endpoint.

That is all the Service Provider needs for the basic integration. Seamless Connect handles discovery and fallback, template handling, authorization routing, execution, and result reporting.

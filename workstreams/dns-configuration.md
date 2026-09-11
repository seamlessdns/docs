# DNS Configuration

Status: Discovery and pilot design

Goal: Support portable DNS configuration through standard Domain Connect and advanced zone operations.

## Current understanding

For standard use cases, Service Providers continue using Domain Connect requests routed through Seamless Connect.

For advanced use cases, Service Providers may submit explicit DNS record create, read, update, and delete requests. Seamless Connect handles discovery, authorization routing, execution, and result reporting across DNS Providers.

Both synchronous execution, the current default, and asynchronous execution should be supported. Asynchronous execution is better suited to agent-driven workflows.

See the [Service Provider Integration Checklist](../service-provider-integration-checklist.md) and [DNS Provider Integration Checklist](../dns-provider-integration-checklist.md).

## Pilot

The initial Service Provider, DNS Providers, configuration use case, and integration path are to be selected.

## Open questions

- Should the initial pilot focus on Domain Connect, advanced zone operations, or both?

## How to participate

Share implementation experience and interest in the pilot.

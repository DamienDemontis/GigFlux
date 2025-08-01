# GigFlux Plugins

Plugins are **out-of-process** jobs/connectors using **JSON-RPC over stdio**. They report capabilities and yield normalized Job objects.

## Manifest (`plugin.json`)
```json
{
  "name": "greenhouse",
  "version": "1.0.0",
  "runtime": "nodejs18",
  "domainsAllowlist": ["boards.greenhouse.io"],
  "requiredSecrets": [],
  "capabilities": ["listings"]
}

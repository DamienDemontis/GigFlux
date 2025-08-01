---
name: Connector (plugin) request
about: Request a new job source plugin
labels: plugin, help wanted
---

### Source
- Name:
- Type: (Greenhouse/Lever/Ashby/Workable/RSS/Other)
- Public API or feed? (links)

### Data needed
Fields required (title, company, location, description, salary, URL, posted_at, etc.)

### Access & Limits
Auth method, rate limits, ToS notes

### Acceptance criteria
- Plugin manifest
- `discover()` and `listings(since)` implemented
- Normalization to Job schema
- Contract tests with fixtures

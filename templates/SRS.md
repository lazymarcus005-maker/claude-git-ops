# Feature Request: {Feature Name}

> **Branch**: req-000001  
> **Author**: @your-username  
> **Date**: YYYY-MM-DD

---

## Summary

{Brief description of the feature — what it does and why it's needed.
1-3 paragraphs.}

---

## Services Affected

List all services that will change as part of this feature.

- **{service-name}**: {change type} — {brief reason}
- **{service-name}**: {change type} — {brief reason}

---

## Service Mapping

| Service | Repository | Change Type | Overview |
|---------|-----------|-------------|---------|
| payment-service | gitlab.company.com/backend/payment-service | API Addition | [Overview](request/payment-service/add-payment/overview.md) |
| notification-service | gitlab.company.com/backend/notification-service | Event Consumer | [Overview](request/notification-service/add-payment/overview.md) |
| api-gateway | gitlab.company.com/infra/api-gateway | Route Addition | [Overview](request/api-gateway/add-payment/overview.md) |

> **Change Types**: API Addition, API Modification, Event Consumer, Event Producer,
> Route Addition, DB Migration, Config Change, New Service, Dependency Update

> **Overview Path Pattern**: `request/{service-name}/{feature-slug}/overview.md`

---

## Dependencies

### Upstream Services (this feature depends on)
- None

### Downstream Services (depend on this feature)
- None

---

## Non-functional Requirements

- **Performance**: {latency targets, throughput}
- **Security**: {auth requirements, data classification}
- **Availability**: {SLA requirements}

---

## Out of Scope

- {What is explicitly NOT included in this feature request}

---

## References

- Ticket: {Jira/linear link}
- Design doc: {link}
- Related PRs: {links}

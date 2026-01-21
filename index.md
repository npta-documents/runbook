# NPTA Canada Documentation

Operations guides for the NPTA Key Fulfillment System.

```
Customer Order → Shopify → Zapier → API (assign keys) → GHL (email)
                                        ↓
                              Slack Alert (success/failure)
```

---

## Documents

| Guide | Description |
|-------|-------------|
| [**Go-Live Playbook**](go-live.md) | Canada cutover process, phases, rollback plan |
| [**Operations Runbook**](runbook.md) | Daily monitoring, intervention scenarios, escalation |
| [**Reference Guide**](reference.md) | Google Sheets, features, glossary, key import format |

---

## Quick Reference

| Task | Where | Action |
|------|-------|--------|
| Check order status | Slack | Search by order ID or customer name |
| Retry failed order | Sidebar | Clear Order Cache → Re-trigger Zap |
| Refund/release keys | Sidebar | Release Keys |
| Add inventory | Sheet | Import Keys |

---

## Support

**Damian Delmas**
- Slack: @damian
- Email: damiandelmas@gmail.com

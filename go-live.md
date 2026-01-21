# Canada Go-Live Playbook

[← Back to Index](index.md)

---

## Objective

Seamlessly transition NPTA Canada from manual fulfillment (EDP) to the new Automated Inventory System.

---

## The 4-Hour Production Cutover

We will select a 4-hour window (e.g., 8:00 AM - 12:00 PM EST) to perform the switch. This allows time to:

1. Configure the environment and import initial inventory.
2. Test the system end-to-end before turning it customer-facing.
3. Restore the prior system (EDP) in the event of any unforeseen issues.

*Note: Any orders placed during this 4-hour window will be fulfilled manually (standard procedure).*

---

## Phase 1: Preparation & Safety

Prior to connecting the new system, we will create a full **Snapshot** of the NPTA-CA Shopify store's products. This includes product names, prices, and existing inventory levels to ensure we have a baseline backup.

---

## Phase 2: Execution (Damian)

- **Configure Environment:** Damian sets the production variables and syncs the product catalogue.
- **Entitlements:** Damian and the team will confirm the entitlements set for each product are correct.
- **Import Keys:** All aggregated Canada keys are imported into the system.
- **Final Sync:** The inventory levels are synced to Shopify.

---

## Phase 3: Cutover & Verification (NPTA Team)

- **Disable EDP:** NPTA turns off the legacy (EDP) app.
- **Enable Automation:** Damian enables the Zapier workflows.
- **Live Verification:** We place a draft order on the live store to confirm:
  - Slack notification received ✅
  - Email delivered to test recipient ✅
  - Keys assigned correctly ✅
  - Inventory count decremented ✅

---

## Phase 4: Full Production Launch

- **Activate Customer Communications:** Upon successful verification, Damian updates the Zapier configuration to route fulfillment emails directly to customer email addresses, transitioning the system from the test environment to live production traffic.

---

## Rollback Plan

If a critical issue is identified during verification, we immediately disable the new automation and re-enable EDP. No data or orders are lost.

---

## Notes

For two weeks each Order will be manually switched over from 'Unfulfilled' to 'Fulfilled' in Shopify's Order management page upon verification of success. This gives operational oversight to each Order. Upon confirmation of a successful two weeks we will switch on the Zapier Step which automates this process.

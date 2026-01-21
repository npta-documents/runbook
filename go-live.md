# Canada Go-Live Playbook

[← Back to Index](index.md)

---

## Objective

Seamlessly transition NPTA Canada from manual fulfillment (EDP) to the new automated inventory system.

---

## The 4-Hour Production Cutover

We will select a 4-hour window (e.g., 8:00 AM – 12:00 PM EST) to perform the switch. This allows time to:

1. Configure the environment and import initial inventory.
2. Test the system end-to-end before turning it customer-facing.
3. Restore the prior system (EDP) in the event of any unforeseen issues.

*Note: Any orders placed during this 4-hour window will be fulfilled manually (standard procedure).*

---

## Phase 1: Preparation & Safety

Prior to connecting the new system, we will create a full **snapshot** of the NPTA-CA Shopify store's products. This includes product names, prices, and existing inventory levels to ensure we have a baseline backup.

---

## Phase 2: Execution (Damian)

- **Configure environment:** Set production variables and sync the product catalogue.
- **Entitlements:** Confirm the entitlements set for each product are correct.
- **Import keys:** All aggregated Canada keys are imported into the system.
- **Final sync:** Inventory levels are synced to Shopify.

---

## Phase 3: Cutover & Verification (NPTA Team)

- **Disable EDP:** NPTA turns off the legacy EDP app.
- **Enable automation:** Damian enables the Zapier workflows.
- **Live verification:** Place a test order on the live store to confirm:
  - Slack notification received
  - Email delivered to test recipient
  - Keys assigned correctly
  - Inventory count decremented

---

## Phase 4: Full Production Launch

- **Activate customer communications:** Upon successful verification, Damian updates the Zapier configuration to route fulfillment emails directly to customer email addresses, transitioning the system from test to live production traffic.

---

## Rollback Plan

If a critical issue is identified during verification, we immediately disable the new automation and re-enable EDP. No data or orders are lost.

---

## Notes

For two weeks, each order will be manually marked as "Fulfilled" in Shopify's order management page upon verification of success. This provides operational oversight. After a successful two-week period, we will enable the Zapier step that automates this process.

# Operations Runbook

[← Back to Index](index.md)

---

## Daily Monitoring

**Slack Channel**

All order fulfillments post to Slack:
- **Order Fulfilled** — Keys assigned, email sent
- **Fulfillment Failed** — Action required
- **Fulfillment Skipped** — No action needed, expected behavior (product has `fulfillment=EXTERNAL`)

No action needed for successful fulfillments or expected skips. Only respond to failures.

---

## Intervention Scenarios

### Scenario 1: Order Shows "Fulfillment Failed"

**Symptoms:** Slack alert with error, customer did not receive keys.

#### Reason 1: Keys Out of Stock

**Steps:**
1. Note the order ID from the Slack alert (or Shopify Admin → Orders → ID in URL).
2. Open Google Sheets → NPTA Control Panel (sidebar).
3. Check if keys are available for that product in Key Inventory sheet.
4. If **out of stock**: Import more keys, then proceed to Scenario 2.
5. If **keys available**: Contact support — likely a system error.

#### Reason 2: Entitlements Not Set

**Steps:**
1. Open NPTA Catalogue sheet → find the product by SKU.
2. Check `entitlements` column — if empty, that's the issue.
3. Add entitlements (e.g., `NASM-CPT-FULL`).
4. Click sidebar → **Sync Catalogue**.
5. Clear order cache for the order → retry fulfillment (Scenario 2).

---

### Scenario 2: Retry Failed Fulfillment

**When:** Fulfillment failed and you've fixed the issue (imported keys or added entitlements).

**Steps:**
1. Open Google Sheets → Sidebar → **Clear Order Cache**.
2. Enter order ID (just the number, e.g., `6855909900591`).
3. Click Clear.
4. Open Zapier → Zap History → find the failed run → click **Replay**.


---

### Scenario 3: Customer Requests Refund

**When:** Customer refunded, keys should return to inventory.

**Steps:**
1. Open Google Sheets → Sidebar → **Release Keys**.
2. Enter order ID.
3. Click Release.
4. Confirmation appears: "Released X keys."
5. Verify: Key Inventory count increases.

*Note: Released keys become available for future orders (FIFO).*

---

### Scenario 4: Check Order Status

**When:** Customer asks "Did my order go through?"

**Steps:**
1. Check Slack channel — search by order ID or customer name.

---

### Scenario 5: Low Inventory Alert

**When:** Product availability is low or zero.

**Steps:**
1. Obtain new keys from provider (NASM, NCSF, etc.).
2. Open Google Sheets → Import Keys sheet.
3. Paste keys in format: `CANADA | provider | product_code | component | keycode | seats`
4. Click sidebar → **Import Keys**.
5. Verify: Key Inventory sheet updates, Catalogue "available" column increases.

---

## Safety Features

| Feature | Description |
|---------|-------------|
| **Idempotency** | Same order processed twice returns same keys (no double-assignment) |
| **Validate Gate** | Emails only sent if keys successfully assigned |
| **Slack Alerts** | All failures alert immediately |
| **Inventory Sync** | Availability auto-updates after each order |

---

## Escalation

### When to Escalate

- Multiple consecutive failures
- Zapier errors
- Any issue you cannot resolve with this runbook

### Contact

**Damian Delmas**
- Slack: @damian
- Email: damiandelmas@gmail.com

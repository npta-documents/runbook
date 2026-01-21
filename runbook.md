# Operations Runbook

[← Back to Index](index.md)

---

## Daily Monitoring

**Slack Channel:**

All order fulfillments post to Slack:
- ✅ **Order Fulfilled** — keys assigned, email sent
- ❌ **Fulfillment Failed** — action required
- ⏭️ **Fulfillment Skipped** — no action needed, expected behavior:
  - Product has `fulfillment=EXTERNAL` (NPTA fulfills manually)

**No action needed** for successful fulfillments or expected skips. Only respond to failures.

---

## Intervention Scenarios

### Scenario 1: Order Shows "Fulfillment Failed"

**Symptoms:** Slack alert with error, customer didn't receive keys.

#### Reason 1: Keys out of Stock

**Steps:**
1. Note the Order ID from the Slack alert
2. Open Google Sheets → NPTA Control Panel (sidebar)
3. Check if keys are available for that product in Key Inventory sheet
4. If **out of stock**: Import more keys, then proceed to Scenario 2
5. If **keys available**: Contact Support — likely a system error

#### Reason 2: Entitlements not set for Product

**Steps:**
1. Open NPTA Catalogue sheet → find the product by SKU
2. Check `entitlements` column — if empty, that's the issue
3. Add entitlements (e.g., `NASM-CPT-FULL`)
4. Click sidebar → **Sync Catalogue**
5. Clear Order Cache for the order → retry fulfillment (Scenario 2)

---

### Scenario 2: Customer Needs Manual Fulfillment (Refund/Re-send)

**When:** Keys were assigned but email failed, or need to retry after importing keys.

**Steps:**
1. Open Google Sheets → Sidebar → **Clear Order Cache**
2. Enter Order ID (just the number, e.g., `6855909900591`)
3. Click Clear
4. Re-trigger the associated Zapier run

---

### Scenario 3: Customer Requests Refund (Release Keys)

**When:** Customer refunded, keys should return to inventory.

**Steps:**
1. Open Google Sheets → Sidebar → **Release Keys**
2. Enter Order ID
3. Click Release
4. Verify keys returned: Check Key Inventory sheet, count should increase

**Note:** Released keys become available for future orders (FIFO).

---

### Scenario 4: Check Order Status

**When:** Customer asks "did my order go through?"

**Steps:**
1. Check Slack channel — search by order ID or customer name

---

### Scenario 5: Low Inventory Alert

**When:** Product availability is low or zero.

**Steps:**
1. Obtain new keys from provider (NASM, NCSF, etc.)
2. Open Google Sheets → Import Keys sheet
3. Paste keys in format: `region | provider | product_code | component | keycode | seats`
4. Click sidebar → **Import Keys**
5. Verify: Key Inventory sheet updates, Catalogue "available" column increases

---

## Safety Features (Built-In)

| Feature | What It Does |
|---------|--------------|
| **Idempotency** | Same order processed twice → returns same keys (no double-assignment) |
| **Validate Gate** | Emails only sent if keys successfully assigned |
| **Slack Alerts** | All failures alert immediately |
| **Inventory Sync** | Availability auto-updates after each order |

---

## Escalation

### When to Escalate to Support

- Multiple consecutive failures
- Zapier errors
- Any issue you can't resolve with the runbook

### Contact

**Damian Delmas**
- Slack: @damian
- Email: damiandelmas@gmail.com
- Response: As soon as practical

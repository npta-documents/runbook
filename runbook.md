# NPTA Fulfillment Operations Runbook & Go-Live Playbook

## System Overview

The NPTA Key Fulfillment System automatically assigns course access keys to customers when they purchase products. Product entitlements and keys are managed in Google Sheets and the system automatically allocates them via integrations with Shopify (orders), Zapier (orchestration), and GHL (email delivery).

```
Customer Order → Shopify → Zapier → API (assign keys) → GHL (email)
                                        ↓
                              Slack Alert (success/failure)
```

---

## Canada Go-Live Playbook

### Objective
Seamlessly transition NPTA Canada from manual fulfillment (EDP) to the new Automated Inventory System.

### The 4-Hour Production Cutover
We will select a 4-hour window (e.g., 8:00 AM - 12:00 PM EST) to perform the switch. This allows time to:
1.  Configure the environment and import initial inventory.
2.  Test the system end-to-end before turning it customer-facing.
3.  Restore the prior system (EDP) in the event of any unforeseen issues.

*Note: Any orders placed during this 4-hour window will be fulfilled manually (standard procedure).*

### Phase 1: Preparation & Safety (Shopify CYA)
Prior to connecting the new system, we will create a full **Snapshot** of the NPTA-CA Shopify stores products. This includes product names, prices, and existing inventory levels to ensure we have a baseline backup.

### Phase 2: Execution (Damian)
*   **Configure Environment:** Damian sets the production variables and syncs the product catalogue.
*   **Entitlements:** Damian and the team will confirm the entitlements set for each product are correct.
*   **Import Keys:** All aggregated Canada keys are imported into the system.
*   **Final Sync:** The inventory levels are synced to Shopify.

### Phase 3: Cutover & Verification (NPTA Team)
*   **Disable EDP:** NPTA turns off the legacy (EDP) app.
*   **Enable Automation:** Damian enables the Zapier workflows.
*   **Live Verification:** We place a draft order on the live store to confirm:
    - Slack notification received ✅
    - Email delivered to test recipient ✅
    - Keys assigned correctly ✅
    - Inventory count decremented ✅

### Phase 4: Full Production Launch
*   **Activate Customer Communications:** Upon successful verification, Damian updates the Zapier configuration to route fulfillment emails directly to customer email addresses, transitioning the system from the test environment to live production traffic.

### Rollback Plan
If a critical issue is identified during verification, we immediately disable the new automation and re-enable EDP. No data or orders are lost.

### Notes
For two weeks each Order will be manually switched over from 'Unfulfilled' to 'Fulfilled' in Shopify's Order management page upon verification of success. This gives operational oversight to each Order. Upon confirmation of a successful two weeks we will switch on the Zapier Step which automates this process.

---

## Google Sheets Reference

| Sheet | Purpose | Editable? |
|-------|---------|-----------|
| **NPTA Catalogue** | Product definitions with SKUs, entitlements, and fulfillment mode | Yes — entitlements, fulfillment |
| **Key Inventory** | Live stock levels by product/component (total, used, available) | No — auto-refreshed |
| **Import Keys** | Staging area — paste new keys here, run Import | Yes — paste zone |
| **Import History** | Audit log of all imported keys with timestamps | No — auto-populated |

### Quick Notes

- **Catalogue `entitlements` column** — must match key codes composite key (e.g., `NASM-CPT-FULL`)
- **Catalogue `fulfillment` column** — `AUTO` (system) or `EXTERNAL` (manual)
- **Key Inventory auto-updates** after every order and import
- **Import History** is your recovery source if keys need reconstruction

## NPTA Catalogue

### provider
Set by the `provider:xyz` tag on the Shopify product (e.g., `provider:nasm`).

### SKU
Unique identifier assigned to each Shopify product.

### category
Product type: Certification, Specialization, or Bundle.

### entitlements
Access granted on purchase. Format: `PROVIDER-PRODUCT_CODE-LEVEL` (e.g., `NASM-CPT-FULL`).

### active
Auto-populated from Shopify product status. TRUE = Shopify "active", FALSE = Shopify "draft". To deactivate a product, set it to Draft in Shopify then Sync.

### fulfillment
- **AUTO** — System assigns keys automatically via Zapier → GHL email
- **EXTERNAL** — NPTA fulfills manually (e.g., KBStronger products)

### available
Live stock count. For bundles, calculated as the minimum across all component products.

---

## Main Features

### Sync from Shopify
Pulls latest product data from Shopify store. Use when products are added/modified in Shopify and need to reflect in the system.

### Sync Catalogue
Pushes entitlement mappings from Google Sheets to the fulfillment API. Required after changing the `entitlements` column for any product.

### Import Keys
Bulk-loads access keys from Import Keys sheet into inventory. Keys become available for assignment immediately and inventory is refreshed locally and in Shopify.

### Refresh Inventory
Recalculates available counts and syncs to Shopify's inventory levels. Use after imports or to verify counts.

## Advanced

### Release Keys
Returns assigned keys to inventory pool for a given Order ID. Use for refunds. Keys re-enter FIFO queue.

### Clear Order Cache
Removes idempotency lock for an Order ID. Allows re-processing of a failed order. **Required before retrying fulfillment.**

## Adding Products from Shopify

### Step 1: Create Product in Shopify

1. Create the product with name and price
2. Assign a **unique SKU** (e.g., `NSTB` for NPTA Strength Training Bundle)
3. Add a **provider tag**: `provider:nasm`, `provider:ncsf`, or `provider:multi` for bundles
4. Set **product type**: `Certification`, `Specialization`, or `Bundle`

### Step 2: Sync from Shopify

1. Open Google Sheets → NPTA Control Panel (sidebar)
2. Click **Sync from Shopify**
3. New product appears in NPTA Catalogue

### Step 3: Assign Entitlements

In the `entitlements` column, enter what access this product grants:

```
PROVIDER-COURSE-LEVEL
```

**Examples:**

| SKU | Entitlements | What Customer Gets |
|-----|--------------|-------------------|
| `CNC` | `NASM-CNC-FULL` | CNC course + exam |
| `CNCe` | `NASM-CNC-EXAM` | CNC exam only |
| `NSTB` | `ELEIKO-STR1-FULL,NASM-CPT-FULL,NCSF-STS-FULL` | 3 courses across 3 providers |

> **Note:** SKU and product_code don't have to match. For example, SKU `CPTa` (full access product) and SKU `CPT` (exam-only product) can both map to product_code `CPT` with different access levels: `NASM-CPT-FULL` vs `NASM-CPT-EXAM`.

### Step 4: Sync Catalogue

1. Click **Sync Catalogue** in sidebar
2. Product + entitlements pushed to database
3. Ready for orders

---

## Adding Courses Without a Shopify Product

Use this when keys exist for courses not sold standalone (bundles, promos, multi-provider).

### Step 1: Import Keys

Import keys with any product_code — no matching Shopify product required:

```
CANADA | ELEIKO | STR1 | combined | KEY-123 | 15
```

### Step 2: Reference via Entitlements

A Shopify product can grant access to these keys through its entitlements:

```
entitlements: ELEIKO-STR1-FULL,NASM-CPT-FULL
```

### Step 3: Sync & Sell

1. **Sync Catalogue**
2. Customer purchases → system allocates keys by entitlement

**Example:** A "Career Bundle" Shopify product with entitlements `NASM-CPT-FULL,NASM-CNC-FULL,NCSF-STS-FULL` will allocate keys from three different courses across two providers — even though only one product exists in Shopify. 

## Creating Custom Bundles

The entitlements system is fully flexible. You can create any Shopify product and assign any combination of courses from any provider.

**To create a custom bundle:**
1. Create a Shopify product with any name, price, and SKU
2. Tag it `provider:multi` (or single provider if applicable)
3. Sync from Shopify
4. Assign entitlements for whatever courses you want to include
5. Sync Catalogue

The system doesn't enforce any relationship between product name, SKU, and entitlements — you define what each product grants.

---

## Daily Operations

### Monitoring

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

Reason 1: Keys out of Stock

**Steps:**
1. Note the Order ID from the Slack alert
2. Open Google Sheets → NPTA Control Panel (sidebar)
3. Check if keys are available for that product in Key Inventory sheet
4. If **out of stock**: Import more keys, then proceed to Scenario 2
5. If **keys available**: Contact Support — likely a system error

Reason 2: Entitlements not set for Product

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

---

## Quick Reference

| Task | Where | Action |
|------|-------|--------|
| Check order status | Slack | Search by order ID or customer name |
| Retry failed order | Sidebar | Clear Order Cache → Re-trigger Zap run |
| Refund/release keys | Sidebar | Release Keys |
| Add inventory | Sheet | Import Keys |

---

## Appendix: Glossary

| Term | Meaning |
|------|---------|
| **component** | Key type: `combined` (full access), `content` (course only), `exam` (exam only) |
| **FIFO** | First-In-First-Out — oldest keys are assigned first |
| **entitlement** | Access grant format: `PROVIDER-PRODUCT_CODE-LEVEL` (e.g., `NASM-CPT-FULL`) |
| **NSBK** | NASM Stock Bundle Key — prefix for multi-course bundle keys |

---

## Appendix: Key Import Format

### Single Course Keys
```
region | provider | product_code | component | keycode | seats
CANADA  NASM      CPT            combined    EX-2t572t1232321a  15
CANADA  NASM      CPT            content     EX-2t572t1232321b  15
CANADA  NASM      CPT            exam        EX-2t572t1232321c  20
```

### Stock Bundle Keys (Multiple Courses)
*note: when importing NASM stock bundle keys please use the NSBK_ prefix*

```
region | provider | product_code       | component | keycode              | seats
CANADA    NASM      NSBK_CPT-CNC-CES     combined    EX-2t572t1232325a  15
CANADA    NASM      NSBK_CPT-CNC-CES     content     EX-2t572t1232325b  15
CANADA    NASM      NSBK_CPT-CNC-CES     exam        EX-2t572t1232325c  20
```
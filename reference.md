# Reference Guide

[<- Back to Index](index.md)

---

## Google Sheets Overview

| Sheet | Purpose | Editable? |
|-------|---------|-----------|
| **NPTA Catalogue** | Product definitions with SKUs, entitlements, and fulfillment mode | Yes |
| **Key Inventory** | Live stock levels by product/component (total, used, available) | No (auto-refreshed) |
| **Import Keys** | Staging area - paste new keys here, run Import | Yes |
| **Import History** | Audit log of all imported keys with timestamps | No (auto-populated) |

### Quick Notes

- **Catalogue `entitlements` column** - Must match the key format (e.g., `NASM-CPT-FULL`).
- **Catalogue `fulfillment` column** - `AUTO` (system) or `EXTERNAL` (manual).
- **Key Inventory** auto-updates after every order and import.
- **Import History** is your recovery source if keys need reconstruction.

---

## NPTA Catalogue Columns

| Column | Description |
|--------|-------------|
| **provider** | Set by the `provider:xyz` tag on the Shopify product (e.g., `provider:nasm`) |
| **SKU** | Unique identifier assigned to each Shopify product |
| **category** | Product type: Certification, Specialization, or Bundle |
| **entitlements** | Access granted on purchase. Format: `PROVIDER-PRODUCT_CODE-LEVEL` |
| **active** | Auto-populated from Shopify product status. TRUE = active, FALSE = draft |
| **fulfillment** | `AUTO` (system assigns keys) or `EXTERNAL` (NPTA fulfills manually) |
| **available** | Live stock count. For bundles, calculated as minimum across all components |

---

## Sidebar Features

### Main

| Button | Action |
|--------|--------|
| **Sync from Shopify** | Pulls latest product data from Shopify store |
| **Sync Catalogue** | Pushes entitlement mappings from Sheets to the fulfillment API |
| **Import Keys** | Bulk-loads access keys from Import Keys sheet into inventory |
| **Refresh Inventory** | Recalculates available counts and syncs to Shopify |

### Advanced

| Button | Action |
|--------|--------|
| **Release Keys** | Returns assigned keys to inventory pool for a given order ID (refunds) |
| **Clear Order Cache** | Removes idempotency lock for an order ID (allows re-processing) |

---

## Adding Products from Shopify

### Step 1: Create Product in Shopify

1. Create the product with name and price.
2. Assign a unique SKU (e.g., `NSTB` for NPTA Strength Training Bundle).
3. Add a provider tag: `provider:nasm`, `provider:ncsf`, or `provider:multi` for bundles.
4. Set product type: Certification, Specialization, or Bundle.

### Step 2: Sync from Shopify

1. Open Google Sheets -> NPTA Control Panel (sidebar).
2. Click **Sync from Shopify**.
3. New product appears in NPTA Catalogue.

### Step 3: Assign Entitlements

In the `entitlements` column, enter what access this product grants:

```
PROVIDER-PRODUCT_CODE-LEVEL
```

**Examples:**

| SKU | Entitlements | What Customer Gets |
|-----|--------------|-------------------|
| `CNC` | `NASM-CNC-FULL` | CNC course + exam |
| `CNCe` | `NASM-CNC-EXAM` | CNC exam only |
| `NSTB` | `ELEIKO-STR1-FULL,NASM-CPT-FULL,NCSF-STS-FULL` | 3 courses across 3 providers |

> **Note:** SKU and product code do not have to match. For example, SKU `CPTa` (full access) and SKU `CPT` (exam-only) can both map to product code `CPT` with different access levels.

### Step 4: Sync Catalogue

1. Click **Sync Catalogue** in sidebar.
2. Product + entitlements pushed to database.
3. Ready for orders.

---

## Adding Courses Without a Shopify Product

Use this when keys exist for courses not sold standalone (bundles, promos, multi-provider).

### Step 1: Import Keys

Import keys with any product code - no matching Shopify product required:

```
CANADA | ELEIKO | STR1 | combined | KEY-123 | 15
```

### Step 2: Reference via Entitlements

A Shopify product can grant access to these keys through its entitlements:

```
entitlements: ELEIKO-STR1-FULL,NASM-CPT-FULL
```

### Step 3: Sync & Sell

1. Click **Sync Catalogue**.
2. Customer purchases -> system allocates keys by entitlement.

**Example:** A "Career Bundle" Shopify product with entitlements `NASM-CPT-FULL,NASM-CNC-FULL,NCSF-STS-FULL` will allocate keys from three different courses across two providers - even though only one product exists in Shopify.

---

## Creating Custom Bundles

The entitlements system is fully flexible. You can create any Shopify product and assign any combination of courses from any provider.

**To create a custom bundle:**
1. Create a Shopify product with any name, price, and SKU.
2. Tag it `provider:multi` (or single provider if applicable).
3. Sync from Shopify.
4. Assign entitlements for whatever courses you want to include.
5. Sync Catalogue.

The system does not enforce any relationship between product name, SKU, and entitlements - you define what each product grants.

---

## Glossary

| Term | Definition |
|------|------------|
| **component** | Key type: `combined` (full access), `content` (course only), `exam` (exam only) |
| **FIFO** | First-In-First-Out - oldest keys are assigned first |
| **entitlement** | Access grant format: `PROVIDER-PRODUCT_CODE-LEVEL` (e.g., `NASM-CPT-FULL`) |
| **NSBK** | NASM Stock Bundle Key - the system automatically uses these when a customer orders exactly the matching courses |

---

## Key Import Format

### Single Course Keys

```
region | provider | product_code | component | keycode | seats
CANADA   NASM       CPT            combined    EX-2t572t1232321a  15
CANADA   NASM       CPT            content     EX-2t572t1232321b  15
CANADA   NASM       CPT            exam        EX-2t572t1232321c  20
```

### Stock Bundle Keys

*Note: When importing NASM stock bundle keys, use the `NSBK_` prefix.*

```
region | provider | product_code       | component | keycode              | seats
CANADA   NASM       NSBK_CPT-CNC-CES     combined    EX-2t572t1232325a  15
CANADA   NASM       NSBK_CPT-CNC-CES     content     EX-2t572t1232325b  15
CANADA   NASM       NSBK_CPT-CNC-CES     exam        EX-2t572t1232325c  20
```

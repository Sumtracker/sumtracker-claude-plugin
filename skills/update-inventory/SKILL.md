---
name: update-inventory
description: Update the inventory of a SKU or a list of SKUs in Sumtracker. Use when the user wants to change, set, correct, or adjust the stock of one or more products at a warehouse, for example after a stock count, a damaged or lost shipment, or a manual recount.
---

To update inventory in Sumtracker:

1. **Find out the warehouse.** Call `list_warehouses` to see how many warehouses the account has. If the account has only one, use it without asking and skip step 2. If it has more than one and the user hasn't said which, ask which warehouse to update. Never assume a warehouse.

2. **Match the warehouse.** Find the closest match for what the user said among the account's warehouses, by name or code. If more than one warehouse is a similar match, ask the user which specific warehouse they mean, listing the candidates by name and code. Don't continue until it's unambiguous.

3. **Verify before changing anything.** Look up each SKU with `list_stock_levels`, using the warehouse's `warehouse_id` and the `sku` or `search` filter. If a SKU isn't found, tell the user and don't guess a similar one. Then show a summary table with one row per SKU and these columns:

   | SKU | Product name | Variant name | Warehouse | Current inventory | New inventory |
   |---|---|---|---|---|---|

   Use `in_stock` as the current inventory. Ask the user to confirm. Make no changes until they clearly say yes. If they correct anything, redo the lookup and show the table again.

4. **Update through the stock adjustment line.** Use the Sumtracker connector's stock adjustment tool to set each confirmed SKU to its new inventory at the chosen warehouse. This skill covers the direct adjust stock method only. Don't use the stock adjustment form here. Support for the form will be added to this skill later.

5. **Ask for a reason (optional).** When you ask for confirmation in step 3, also ask whether the user wants to record a reason for the adjustment, such as "stock count" or "damaged goods". If they give one, record it on every adjustment in the batch. If they skip it, leave it blank. Don't ask again.

6. **Show the result.** After the changes are done, show a table of what changed:

   | SKU | Product name | Variant name | Warehouse | Previous inventory | New inventory | Status |
   |---|---|---|---|---|---|---|

   Mark each row as updated or failed. For any failure, give the reason from the tool and say which SKUs were not changed. Don't retry a failed update without asking.

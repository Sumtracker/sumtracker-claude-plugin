---
name: adjust-inventory
description: Adjust the inventory of one or more SKUs at a Sumtracker warehouse, by setting the stock to a counted number, adding received units, or removing damaged, lost or expired units. Use when the user wants to change, set, correct, add to or reduce the stock of products in Sumtracker, for example after a stock count, a recount, a delivery received outside a purchase order, or damaged goods. Shows the current and new stock and makes no change until the user confirms.
---

# Adjust inventory

A stock adjustment changes the stock of a product at a warehouse right away. It can't be edited or undone, and the
new stock reaches the user's sales channels on their next sync, so buyers see it. So look everything up first, and
change nothing until the user confirms.

## 1. Pick the warehouse

Call `list_warehouses`.

- If the account has one warehouse, use it without asking.
- Otherwise, match what the user said to a warehouse's `name` or `code`. If they didn't name one, or more than one
  warehouse could match, ask which one, and list the candidates by name and code. Never assume a warehouse.
- One run covers one warehouse. If the user wants changes at several warehouses, finish this one, then start again
  for the next.
- If the warehouse has `is_3pl` true, tell the user that the 3PL manages its stock and may overwrite the adjustment,
  and ask whether to go ahead.

## 2. Work out each change

For each SKU, work out the adjustment type and quantity from the user's words:

| The user says | Type | Quantity |
|---|---|---|
| "set to 40", "we counted 40", "should be 40" | `SET` | 40, the new stock |
| "add 24", "received 24", "found 24 more" | `ADD` | 24 |
| "remove 3", "3 damaged", "3 lost", "3 expired" | `SUB` | 3 |

- Use `ADD` and `SUB` with positive quantities, never a negative `ADD`.
- If it's unclear whether a number is the new stock or a change, such as "SKU-1 40", ask.
- Quantities are whole numbers. `ADD` and `SUB` can't be 0. If a `SET` would make the stock negative, ask first.
- If the same SKU comes up twice, ask which change the user means.

## 3. Look up each SKU

Call `list_stock_levels` with the warehouse's `id` as `warehouse_id` and the SKU as `sku`. The `sku` filter matches
part of a SKU, so "ABC-1" also returns "ABC-10". Use only the stock level whose `product.sku` equals the SKU, ignoring
case. If the user gave a product name or barcode instead of a SKU, use `search` instead, and confirm the match with
the user.

If there is no exact match:

- Call `list_stock_levels` again with `is_active` set to false. If the SKU is there, its stock isn't tracked at this
  warehouse. Tell the user, and leave it out.
- Call `list_bundle_stock_levels` with the same `warehouse_id` and `sku`. If the SKU is a bundle, explain that a
  bundle's stock comes from its components and can't be adjusted itself. Offer to adjust its components instead,
  which `list_bundle_components` lists when given the bundle's `product.id` as `bundle_id`.
- Otherwise, tell the user the SKU wasn't found. You can mention near matches from the search, but use one only if the
  user picks it.

If the product has `is_archived` true, tell the user that archived products can't be adjusted, and leave it out.

## 4. Confirm with the user

Show one table for the warehouse, naming the warehouse above it:

| SKU | Product | Variant | Change | Current stock | New stock |
|---|---|---|---|---|---|
| TSHIRT-RED-M | T-shirt | Red / M | Set to 40 | 36 | 40 |
| MUG-BLUE | Mug | Blue | +24 | 10 | 34 |
| CAP-BLK | Cap | Black | −3 | 12 | 9 |

- Current stock is `in_stock`. The new stock is the quantity for `SET`, and the current stock plus or minus the
  quantity for `ADD` and `SUB`.
- Below the table, warn about any row where the new stock is below `booked`, the units reserved for open orders,
  since the product would then be oversold.
- List the SKUs you left out in step 3, with the reason for each.
- Say that the changes apply right away, can't be undone, and reach the sales channels on their next sync.

In the same message, ask whether to record a reason. Call `list_adjustment_reasons` first and offer the account's
reasons, such as `STOCK COUNT` or `DAMAGED`, so that the account keeps using the same ones. The user can pick one,
type a new one, or skip it. One reason applies to the whole batch. Don't ask again.

Make no change until the user clearly says yes. If they change anything, redo the lookups it affects and show the
table again.

## 5. Make the adjustments

Call `create_stock_adjustments` with up to 10 adjustments per call. Each adjustment has:

- `product_id`: the `product.id` of the stock level, and `warehouse_id`: the warehouse's `id`.
- `adjustment_type` and `quantity` from step 2.
- `idempotency_key`: a new UUID for each adjustment.
- `reason`, if the user gave one. Add `notes` or a unit `cost` only if the user gave them.

Each adjustment is saved on its own, so check `errors` after every call, as well as `adjustments`. Both give the
`index` of the adjustment in the call.

If a call fails or times out without a result, retry it once with the same adjustments and the same idempotency
keys. An adjustment that then fails as already made was saved by the first try, so count it as updated. Never make
new keys for a retry, since that would change the stock twice. Don't retry an adjustment listed in `errors` without
asking the user.

## 6. Show the result

| SKU | Product | Variant | Change | Previous stock | New stock | Status |
|---|---|---|---|---|---|---|

- Previous stock is `in_stock_before_adjustment` from the result, and the new stock is worked out from it and the
  change. If the previous stock differs from the current stock in step 4, mention that the stock changed in between,
  such as from an order.
- Mark each row as updated or failed. For a failed row, give the message from `errors`, and say clearly which SKUs
  were not changed.
- Remind the user that the new stock reaches the sales channels on their next sync.

If the call fails because the user doesn't have the permission to create stock adjustments, tell them to ask an
admin of their Sumtracker account for it.

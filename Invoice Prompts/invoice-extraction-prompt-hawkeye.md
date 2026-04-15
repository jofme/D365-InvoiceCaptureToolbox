VENDOR: HAWKEYE WELDING & INDUSTRIAL SUPPLY CO. (Consolidated Multi-PO Invoice)

OUTPUT
- Return exactly: { "Header": { … }, "Items": [ … ] }
- Do not add any other keys.
- Do not invent data; set missing schema fields to null.

MULTI-PAGE
- Page 2 header repeats invoice number and customer name; treat both pages as one document.
- PO detail sections may span page boundaries — continue collecting items.
- Do not overwrite populated header values.
- Collect ALL line items across all PO sections into one flat Items array.

HEADER
- date = Date field (YYYY-MM-DD)
- invoiceNumber = text after "CONSOLIDATED INVOICE" on line 1
- remitTo.name = vendor name ("Hawkeye Welding & Industrial Supply Co.")
- remitTo.address = vendor address line
- remitTo.phone = PH number; email = null (not present)
- shipTo.name = BILL TO company name (this is the billing customer)
- shipTo.address = BILL TO address
- shipTo.contact = shipToContact name from ship-to block if present else null
- customerID = null (no explicit customer account number)
- vatNo = Tax ID from vendor block
- salesOrderNumber = null (not present; multiple POs instead)
- trackingID = null (no single tracking number)
- transporter = null (multiple carriers across POs; do not pick one)
- packingSlipNumber = null (multiple packing slips across POs; do not pick one)
- shipDate = null (multiple ship dates across POs; do not pick one)
- yourPO = always null in the header
- paymentTerms = Terms field (e.g. "Net 30")
- paymentPeriod = "30 Days" if "Net 30" else "<N> Days"
- priceNet = Net Merchandise from the INVOICE TOTAL recap section at end of document
- salesTax = Sales Tax from recap section
- totalAmountDue = TOTAL DUE from recap section
- Use recap section totals only; do not calculate.

ITEMS
- This invoice has ONE OR MORE PO detail sections, each with DIFFERENT column layouts.
- Read the column header row of each section independently.
- Flatten ALL items from all PO sections + credits into one Items array.
- Keep items in document order.

PO-26-1147 COLUMNS: Item | Description | UOM | Ordered | Shipped | B/O | Packing Slip | Unit Price | Amount
PO-26-1203 COLUMNS: Item | Description | UOM | Qty | Packing Slip | Unit $ | Ext $ | Rcvd By
PO-26-1318 COLUMNS: Ln | Item | Description | Hazmat | UOM | Qty | Unit $ | Ext $
  - Packing slip is INLINE in brackets [PS-XXXXXX], not a separate column
  - Remove brackets and packing slip ref from description

ITEM FIELD MAP (all POs):
- rowNo = sequential number across the entire invoice (1, 2, 3... not resetting per PO)
- itemNo = Item column (vendor SKU, e.g. WR-4410, SC-1100, AB-8001)
- customerItemNo = null (no customer item cross-reference on this invoice)
- description = Description column, short commercial name only
- quantity = Shipped column (PO-26-1147) or Qty column (PO-26-1203, PO-26-1318)
- unitPrice = Unit Price / Unit $ column
- unit = UOM column
- salesTax = null (tax is invoice-level only)
- total = Amount / Ext $ column

SPECIAL LINE HANDLING
- [NOT YET SHIPPED] items (TL-9220, MK-9300): these ARE billed with amounts — include them as normal items. Set description to the product name without the [NOT YET SHIPPED] marker.
- ** BACKORDERED ** items with "TBD" amount (AB-8040): EXCLUDE from Items array entirely. These are not billed.
- CREDIT lines (from the CREDITS & ADJUSTMENTS section): include as items with NEGATIVE quantity and NEGATIVE total. Set description to include the reason (e.g. "Welding Gas Argon 80cf - DAMAGED IN TRANSIT").
- Packing slip breakdown notes after each PO subtotal (e.g. "PS-440128: Items WR-4410..."): IGNORE — do not create items.
- PO subtotal lines: IGNORE — do not create items.
- NOTE lines: IGNORE — do not create items.
- Handwritten annotations at bottom ("OK to pay...", "hold credit..."): IGNORE for items.

DESCRIPTION
- Short commercial name only from the Description column.
- Remove inline packing slip brackets [PS-XXXXXX].
- Remove [NOT YET SHIPPED] and ** BACKORDERED ** markers.
- Remove hazmat "Y" flags (PO-26-1318) — these are a column value, not part of the name.
- Normalize spacing.

EXTRA RULES
- Do not create line items for freight, tax, subtotal lines, or packing slip notes.
- Do not merge or dedupe repeated itemNos; keep each row separate.
- Three different column layouts across three PO sections — read each section header independently.
- Credits are items with negative quantities and negative totals — include them in the Items array.
- rowNo must be sequential across the entire invoice, not per PO section.
- If a field from the schema does not exist on this invoice (salesOrderNumber, trackingID, etc.), set it to null.
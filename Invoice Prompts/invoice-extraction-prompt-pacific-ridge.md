VENDOR: PACIFIC RIDGE ELECTRONICS SUPPLY INC (AS400 PRE-INV-4P)

OUTPUT
- Return exactly: { "Header": { … }, "Items": [ … ] }
- Do not add any other keys.
- Do not invent data; set missing schema fields to null.

MULTI-PAGE
- Header repeats per page; treat all pages as one invoice.
- Do not overwrite populated header values.
- Collect all LN rows across all pages.
- Ignore PAGE SUBTOTAL.

HEADER
- date = INVOICE DATE (YYYY-MM-DD)
- invoiceNumber = INVOICE NO.
- remitTo.name/address/phone = vendor block + PH; email only if present else null
- shipTo.name/address = SHIP TO block; contact only if present else null
- customerID = CUSTOMER ACCT
- vatNo = TAX ID
- salesOrderNumber = OUR ORDER NO.
- yourPO = YOUR P.O. NO.
- paymentTerms = TERMS
- paymentPeriod = "<N> Days" if NET N else null
- shipDate = SHIP DATE (YYYY-MM-DD)
- transporter = SHIP VIA
- currency = CURRENCY if present else null
- priceNet = final MERCHANDISE TOTAL
- salesTax = final SALES TAX
- totalAmountDue = final INVOICE TOTAL / PLEASE PAY amount
- Use final totals only; do not calculate.

PACKING SLIP / TRACKING
- packingSlipNumber = PRO NUMBER from last-page shipping block
- Ignore per-line "PACKING SLIP: PS-xxxx" values
- trackingID = null unless explicitly labeled as tracking

ITEMS (one per LN)
Source columns:
LN | ITEM NO. | DESCRIPTION | CUST REF | UOM | WH | QTY | PRICE | AMOUNT

Map:
- rowNo = LN
- itemNo = ITEM NO. (vendor item id)
- customerItemNo = CUST REF (customer item id); if missing, null
- quantity = QTY
- unitPrice = PRICE
- unit = UOM
- total = AMOUNT
- salesTax = null (invoice-level tax only)

DESCRIPTION
- Short commercial name only (single concise phrase).
- Prefer the compact description from the main row.
- Exclude quantities, pricing, WH/UOM, compliance, COO/HTS/WT/PKG, serials, lots, certs, dates, PO line, del note, packing slip, shipping notes.
- Stop when encountering: ROHS, REACH, WEEE, ENERGY STAR, COO:, HTS:, WT/UNIT:, PKG:, S/N:, LOT:, CERT:, FCC, UL, ETL, MFR DATE, SHELF LIFE, PO LINE, DEL NOTE, PACKING SLIP.
- Normalize spacing and spelling.

EXTRA RULES
- Do not create line items for freight or tax.
- Do not merge or dedupe repeated itemNos; keep each LN separate.
# Printable material forms (all sites)

One page (`print.html`) and one Supabase function (`bol-print`) print three forms for every site:

| Form | Airtable header table | Lines table | `type=` |
|---|---|---|---|
| Field Material Request (pull BOL) | Pull Requests | Pull Lines | `pull` |
| Truck Receiving Report | Truck Receipts | Receiving Lines | `receipt` |
| Supplier Material Order | Supplier Orders | Order Lines | `order` |

Each base has a **BOL Link** formula field on each header table. The Print BOL button opens it:

```
print.html?type=pull&site=Buckeye%202.0&groups=Work%20Packages&base=app…&t=tbl…(header)&lt=tbl…(lines)&tz=America%2FPhoenix&id=<RECORD_ID>
```

## The BOL contract

The function reads **only** these fields. Each base fills them with formulas/rollups from its own fields,
so renaming ordinary fields never breaks printing. Don't rename or delete a `BOL …` field.

**Header table**

| Field | pull | receipt | order | Notes |
|---|---|---|---|---|
| BOL Number | ✓ | ✓ | ✓ | text |
| BOL Requested By | ✓ | | | |
| BOL Date Needed | ✓ | | ✓ | `DATETIME_FORMAT(…, 'M/D/YYYY')` |
| BOL Location | ✓ | | | delivery location |
| BOL Date Received | | ✓ | | |
| BOL Carrier | | ✓ | | |
| BOL Paperwork | | ✓ | | BOL / packing slip # |
| BOL Received By | | ✓ | | |
| BOL Supplier / BOL PO / BOL Ordered By | | | ✓ | |
| BOL Notes | | ✓ | ✓ | printed in the band under the info row |
| BOL Groups | ✓ | ✓ | ✓ | one entry per line (rollup `ARRAYJOIN(values, "\n")`): bays, work packages, cells, CWP… |
| BOL Line IDs | ✓ | ✓ | ✓ | rollup of the lines' BOL Line ID, `ARRAYJOIN(values, ",")` |

**Lines table**

| Field | pull | receipt | order |
|---|---|---|---|
| BOL Line ID (`RECORD_ID()`) | ✓ | ✓ | ✓ |
| BOL Sort (line order; number or text) | ✓ | ✓ | ✓ |
| BOL Qty | ✓ | ✓ | ✓ |
| BOL Part # | ✓ | ✓ | ✓ |
| BOL Description | ✓ | ✓ | ✓ |
| BOL Damaged / BOL Condition / BOL Yard | | ✓ | |
| BOL Group (feeds the header's BOL Groups) | | optional | |

If a base is missing one, the page says "This base is missing the "BOL …" field" — add it or rename it back.

## Adding a site

1. Add the BOL fields and BOL Link fields to the new base (Claude can do this).
2. Add the buttons: Button field → Open URL → `{BOL Link}` on each header table, and on the interface detail pages.
3. Nothing in Supabase or GitHub changes, as long as the token's access is "All current and future bases" in the workspace.

## One-time setup

- **Token:** airtable.com/create/tokens → scope `data.records:read` only → Access: "All current and future bases" in the projects workspace. Save it in Supabase → Edge Functions → Secrets as `BOL_AIRTABLE_TOKEN`.
- **Function:** Supabase → Edge Functions → Deploy new function → name `bol-print`, paste `bol-print/index.ts` → Details → turn **Verify JWT** off.
- **Page:** upload `print.html` to this repo.

Preview a layout without Airtable: `print.html?type=receipt&id=demo&lines=80`.

The older `index.html` (Palestine), `buckeye.html` and `order.html` pages keep working on their own functions until each base's buttons are pointed at BOL Link. Retire them after that.

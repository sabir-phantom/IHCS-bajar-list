# Iqbal Hossain Catering — Order & Bajar List System

Working plan and project record. Last updated 2026-09-27.

---

## 1. What this project is

Two self-contained HTML pages that replace a handwritten workflow: taking a catering
order, and turning it into a market shopping list on the firm's own printed forms.

Everything runs from local files. No server, no build step, no framework, no internet.
Each page is a single `.html` file with its CSS, JavaScript, data and fonts inside it,
so it works by double-click and survives being copied to another machine.

### Files

```
Bangla Menu/
├── plan.md            ← this file
├── final/             ← the live system
│   ├── order_form.html    order entry, history, multi-time meals, auto-set menu,
│   │                      universal item search, single-page print guard  (293 KB)
│   ├── bajarlist.html     recipes, quantity calculation, chef pages,
│   │                      market sheets                                (327 KB)
│   ├── guide-en.html      user + maintainer guide, English
│   ├── guide-bn.html      user + maintainer guide, Bangla
│   └── bajar-data.json    the data file — CREATED BY THE USER, not yet set up
└── old/               superseded order form versions, reference only
```

Source material lives outside this folder at `D:\Sabir\Work\IC\Menu\Recipes\Recipes`
(the master `bazaar list.pdf` and the individual recipe cards).

---

## 2. Current state

**Done and verified.**

| Area | State |
|---|---|
| Order → bajar list handoff | Set menu tag, headcount, date, venue and item list pass through the URL |
| Recipes | 28 dishes, per 100 guests, all ingredients resolve to form rows |
| Forms | 3 printed sheets, ~260 rows, fully editable (names, units, rows, headings) |
| Packages & auto-select | Built-in packages auto-check their items on the order form; switching packages clears the previous package's auto-checked items without touching anything checked by hand; manually touched items are always preserved; live-synced from `bajarlist.html` via `localStorage` (`iqbal_catering_bajar_setmenus`) |
| Multi-time meals | সকাল / দুপুর / বিকাল / সন্ধ্যা / রাত as multi-select checkboxes; selecting 2+ times splits the form into a dedicated headed section per time, each with its own category blocks and "আরেকটি মেনু যোগ করুন" button; single or no time keeps the flat layout |
| Universal item search | A search bar above all blocks searches every category at once; results show a category badge per item; multi-time orders show a time-slot dropdown per result; already-checked items are marked and blocked from double-adding; clicking outside dismisses results |
| Review screen | Per-100 and per-order columns, per-ingredient breakdown, folding, confirmations |
| Chef assignment | Each dish of the package can go to বাবুর্চি S / A / C. A and C's ingredients leave the three normal sheets and print on that cook's own page; shares are split in proportion to each dish's contribution, so nothing is bought twice and nothing is mixed between cooks |
| Printing | Print styles shared between screen preview and printer; column count auto-chosen (1 / 2 / 3) based on item count vs. A4 printable height; scale guard shrinks font to ≥70% if even 3 columns overflow; manual font-size control overrides auto-scale; multi-time orders print with a bold underlined heading per time slot |
| Fonts | Embedded — zero external requests |
| Data file | Built and tested; **not yet switched on by the user** |
| Documentation | Both guides written and current |

**Not done.**

- The data file has never been created for real. The file-picker step cannot be
  exercised in the development environment; it needs one manual run in Chrome or Edge.
- No test print on the real printer.
- The six PDF recipe cards were transcribed from scans; a handful of values are
  flagged below and unconfirmed.

---

## 3. How the data fits together

```
recipe  →  ingredient  →  form row  →  printed line
           (per 100)      (unique)     (× headcount ÷ 100)
```

### Chef split

A dish can be handed to বাবুর্চি A or C. `chefSplit()` divides every row's quantity
between S / A / C in proportion to how much each chef's dishes contribute to that row,
taken from the same `sources` that power the ▸ breakdown. The three shares always add
back to the row total (verified: drift 0 across all rows). A row with no recipe behind
it — hand-typed, or a package default like ভাতের চাউল — belongs to no dish and stays
with S. Chef pages print first in the packet, one merged table per cook, and reuse the
same 22 / 56 line rules as the other sheets.

The assignment lives in memory only and is never saved. That is deliberate: a stale
assignment restored from last week would silently remove items from the shopping sheets.

**The rule that keeps it honest:** every ingredient names exactly one form row.
Two dishes using the same item therefore add into one line instead of being bought
twice. Ghee was the only exception found (written on three rows across the paper
cards) and has been consolidated onto `m5`.

### Row ids

| Prefix | Sheet |
|---|---|
| `p1…p87` | পার্টি বাজার লিস্ট, numbered as on the printed form |
| `n1…n34` | items added to the পার্টি sheet |
| `v1…v31` | সবজী sheet |
| `w1…w7` | items added to the সবজী sheet |
| `m1…m52` | বাজার লিস্ট sheet |
| `r1…r17` | রুটির মালামাল section |
| `u…` `rec…` `pkg…` | created in the app at runtime |

### Storage layers

1. **In the file** — recipes, form rows, built-in packages. Survives everything.
2. **Browser storage** — the user's edits and, critically, the order history.
   Seven `iqbal_catering_*` keys. Fragile: cleared history wipes it.
3. **`bajar-data.json`** — one file holding both pages' data, written on every change.
   Shape: `{ app, version: 2, savedAt, per100, forms, setMenus, recipes, orderForm: { menu, history } }`.
   Each page rewrites only its own section, so neither can clobber the other.
   Needs Chrome or Edge (File System Access API); the file handle is remembered in
   IndexedDB `iqbal-bajar` → `handles` → `dataFile`.

### Set-menu to order-item mapping

`bajarlist.html` stores packages under recipe-key dish names (e.g. `kachi`,
`chickenRoast`, `borhani`). `order_form.html` displays fuller item names (e.g.
`শাহী মাটন কাচ্চি বিরিয়ানী (চিনিগুড়া)`, `বোরহানী`). The one manual bridge
between them is `DISH_TO_ORDER_ITEM` in `order_form.html`.

**Rule:** a new *package* that uses only dishes already listed there needs no change —
it auto-syncs via `localStorage`. Only a brand-new *dish* (recipe key) introduced
in `bajarlist.html`'s `RECIPE_LIBRARY` and used in a package needs a new line added
to `DISH_TO_ORDER_ITEM`.

### Auto-checked item tracking

`autoCheckedItems` (a `Set` in `order_form.html`) tracks which items the most recent
package selection checked automatically. Switching to a different package calls
`clearAutoCheckedItems()` to remove only those items before checking the new set.
Any item the user clicks by hand is removed from `autoCheckedItems` at that moment,
so it is never swept away by a later package change. The set resets on new order and
on editing a history order.

---

## 4. Open items

### Needs the owner's answer

- [ ] **চিকেন রোস্ট / জালি কাবাব carry no chicken.** Follows the master bazaar list,
      but it means the standard কাচ্চি package buys no মুরগী at all. Confirm that
      chicken is bought separately like the meat, or give an amount per 100.
- [ ] **মোতানজান জদ্দা** — গোলাপজল and কেওড়া জল recorded as 1 bottle each
      (card says 150 গ্রাম, form counts bottles).
- [ ] **মোতানজান জদ্দা** — four spices at 5 গ্রাম each = 20 গ্রাম, where the card's
      combined line reads 15 গ্রাম.
- [ ] **মুরগ পোলাও** — মোরগ ৫০ পিছ sits on the দেশী মোরগ row, though "দেশী" was
      struck out on the card.
- [ ] Three handwritten additions on row 88 of the cards, unreadable in the scans:
      সাদা পোলাও 250 গ্রাম, মিক্সড ভেজিটেবল 200 গ্রাম, চাটনি 20 গ্রাম.
- [ ] Should রুটি কালিয়া and মোরগ পোলাও offer dessert options?

### Needs a real-world run

- [ ] Create `bajar-data.json` from the bajar list, then open it from the order form.
      Confirm both bars show a green dot, and that it survives closing and reopening.
- [ ] Test print all three sheets on the actual printer, A4, default margins.
- [ ] Confirm the order history survived the move into `final/`.
- [ ] Test print the order sheet with a multi-time order (e.g. দুপুর + রাত) and
      confirm the time headings and column layout are correct on paper.

---

## 5. Roadmap

Ordered by value, not effort. Nothing here is started.

1. **Cost estimate.** Hold a current price per row; total the bajar list into an
   estimated cost and a cost per guest. Useful when quoting. Would not print on the
   market sheets unless asked.
2. **Learn from what was actually bought.** After the market trip, record real
   quantities against the planned ones; over several events, suggest better per-100
   values. Turns the app from a calculator into a record.
3. **Combined list for same-day events.** Merge two or more orders into one shopping
   trip, while still printing each event's sheet separately.
4. **Per-shop splitting.** The three sheets already roughly match grocery, vegetable
   market and disposables. Make that explicit so different people can be handed
   different sheets.
5. **Remaining recipe cards.** The folder holds cards not yet imported (Kachhi,
   Borhani, Firni, Zorda, Jali Kabab, Roast+Grill, Rezala, Motanjan, Beef+Ruti,
   Fish fillet+Salad, Chicken Jhal fry+Korai Gosht, and a "Recipe by Nadim sir"
   folder). Most duplicate the master list; worth a pass to see which add detail.

---

## 6. Maintenance notes

**Adding a recipe.** Use the রেসিপি লাইব্রেরি in the app — it saves to browser storage
and the data file. To make a recipe permanent for every copy of the file, add it to
`RECIPE_LIBRARY` in `bajarlist.html` as
`{ label, dessert?, ingredients: [[rowId, qtyPer100, unit], …] }`.

**Unit kinds must match.** A weight amount on a row whose unit is `pcs` is multiplied
a thousandfold. This bug happened once (বড় পেঁপে, 3 কেজি read as 3000 পিছ). Before
shipping recipe changes, run the check in the browser console:

```js
Object.keys(RECIPE_LIBRARY).forEach(k => (RECIPE_LIBRARY[k].ingredients || []).forEach(e => {
  const id = recipeRowId(e[0]);
  if (!id) console.warn('unresolved', RECIPE_LIBRARY[k].label, e[0]);
  else if (unitInfo(ROW_INDEX[id].unit).type !== unitInfo(e[2]).type)
    console.warn('unit mismatch', RECIPE_LIBRARY[k].label, ROW_INDEX[id].label);
}));
```

**Two lists to keep in sync.** Built-in packages appear in `SET_MENUS` in
`bajarlist.html` and in `BAJAR_SET_MENUS` in `order_form.html`. The keys must match.
New packages built by the user in the app sync automatically via `localStorage`
(`iqbal_catering_bajar_setmenus`) and need no manual update in `order_form.html`.
Only a hardcoded built-in package requires a matching entry in both files.

**Adding a dish to `DISH_TO_ORDER_ITEM`.** Find `DISH_TO_ORDER_ITEM` in
`order_form.html` and add:
```js
newDishKey: 'অর্ডার ফর্মে যেভাবে দেখায়',
```
Both keys (dish key in `RECIPE_LIBRARY` and the order-form item name) must match
exactly, including capitalisation and spacing.

**Print column thresholds.** One column holds ≈ 38 lines against A4's 1032 px
printable height (180 px header + 22 px per line). Beyond 38 the layout switches
to 2 columns; beyond 76 it switches to 3. Beyond 114 the scale guard kicks in.
These are calculated from A4 dimensions — re-check only if the header height or
font size changes significantly.

**Editing the files.** They are large single files; use anchored search-and-replace
rather than line numbers, and check the JavaScript parses afterwards:

```bash
node --check <extracted script>
```

---

## 7. Decisions worth remembering

- **Single-file pages over a shared library.** Portability beats tidiness here: any
  page must work alone when copied.
- **A JSON file over a real database.** No install, readable forever, copyable to a
  USB stick. A database would need software on every machine.
- **The master `bazaar list.pdf` governs quantities.** Where it disagreed with values
  given verbally, the master won (লবঙ্গ 10, হলুদ গুড়া 100, সাদা সরিষা 150).
- **Only rows with a quantity print.** Chosen over printing the whole blank form.
- **Unreadable values are flagged, never guessed.** Every uncertain transcription is
  listed in section 4 rather than quietly entered.
- **Multi-time uses dedicated blocks, not per-item tags.** When 2+ times are selected,
  the form shows a headed section per time (সকাল, দুপুর, etc.), each with its own
  category blocks. The alternative (tagging each item with which times it belongs to)
  was tried first and replaced in favour of this cleaner structure.
- **Auto-select clears only its own items.** When switching packages, only items the
  previous package auto-checked are removed; anything the user ticked by hand
  survives the switch. A manual click on an auto-checked item transfers ownership
  to the user at that moment.
- **Chef assignment is per job and never persisted.** Convenience would cost
  correctness here: the failure mode of a remembered assignment is items quietly
  missing from the market sheet, which is not discovered until the market.
- **Chef shares are proportional, not all-or-nothing.** An overridden or hand-typed
  quantity on a shared row is divided the same way the recipe would have divided it.
  Count units are rounded up per cook, so a split of 7 পিছ gives 4 and 4 — over by one
  rather than short.
- **Print scale is calculated, not stored.** Column count and zoom are derived from
  the actual rendered line count at print time, so no value needs updating when
  menu content grows or shrinks.

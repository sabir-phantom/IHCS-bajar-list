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
│   ├── order_form.html    order entry, history, per-category item search,
│   │                      set menu tag, order sheet printing            (276 KB)
│   ├── bajarlist.html     recipes, quantity calculation, chef pages,
│   │                      market sheets                                (327 KB)
│   ├── guide-en.html      user + maintainer guide, English
│   ├── guide-bn.html      user + maintainer guide, Bangla
│   └── bajar-data.json    the data file — created 22 Sep, holds 1 saved order
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
| Packages | 3 built in: স্ট্যান্ডার্ড কাচ্চি, রুটি কালিয়া, মোরগ পোলাও; more can be created in the app. The order form's dropdown reads them live from `localStorage` (`iqbal_catering_bajar_setmenus`) |
| Review screen | Per-100 and per-order columns, per-ingredient breakdown, folding, confirmations |
| Chef assignment | Each dish of the package can go to বাবুর্চি S / A / C. A and C's ingredients leave the three normal sheets and print on that cook's own page; shares are split in proportion to each dish's contribution, so nothing is bought twice and nothing is mixed between cooks |
| Printing | Only filled rows, no price columns, one page per sheet, measured against A4: one column to 22 lines, two columns beyond, dense type beyond 56 |
| Fonts | Embedded — zero external requests |
| Data file | `final/bajar-data.json` exists and holds 1 saved order (22 Sep); re-open it from both pages after the recent changes |
| Documentation | Both guides written and current |

**Not done.**

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

### Set menus across the two pages

`bajarlist.html` owns the packages. `order_form.html` carries its own built-in list in
`BAJAR_SET_MENUS` and merges anything the user created, read live from `localStorage`
(`iqbal_catering_bajar_setmenus`). Packages made in the app therefore appear in the
order form's dropdown without any edit; only a new *hardcoded* package needs a matching
entry in both files.

The order form passes the chosen package to the bajar list by key, through the URL.
It does not know which dishes a package contains — that stays in `bajarlist.html`.

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

- [ ] `bajar-data.json` exists (22 Sep, 1 saved order). Open it from **both** pages and
      confirm each shows a green dot — `bajarlist.html` has changed a lot since then.
- [ ] Test print all three sheets on the actual printer, A4, default margins.
- [ ] Test print a job with a dish given to বাবুর্চি A and another to C: confirm the
      two cook pages come out first and the shares add up to the full amount.

---

## 5. Roadmap

Ordered by value, not effort. Nothing here is started.

Items 1–3 were written up as done in an earlier draft of this file, but no file in the
project contains them: `autoCheckedItems` and `DISH_TO_ORDER_ITEM` appear nowhere, the
time field is a single-select `<option>` list, and the search box is per-category. They
are kept here in full as specifications, because the thinking behind them is worth
keeping.

1. **Package auto-select on the order form.** Choosing a package ticks its items
   automatically. Switching package clears only the items that package ticked —
   anything ticked by hand survives, and a hand click on an auto-ticked item transfers
   ownership to the user from that moment. Resets on a new order and on editing a
   history order. Needs a bridge from recipe keys (`kachi`, `borhani`) to the fuller
   names the order form shows (`শাহী মাটন কাচ্চি বিরিয়ানী (চিনিগুড়া)`, `বোরহানী`) —
   a `DISH_TO_ORDER_ITEM` map, the one place a brand-new dish would need a manual line.
2. **Multi-time meals.** সকাল / দুপুর / বিকাল / সন্ধ্যা / রাত as multi-select
   checkboxes instead of the present single dropdown. Two or more times split the form
   into a headed section per time, each with its own category blocks and its own
   "আরেকটি মেনু যোগ করুন" button; one time or none keeps today's flat layout. Printing
   gets a bold underlined heading per time slot. Decided while designing it: dedicated
   blocks per time, not a tag on each item saying which times it belongs to.
3. **Universal item search.** One search bar above all the category blocks, searching
   every category at once, each result badged with its category. In a multi-time order
   each result carries a time-slot dropdown. Items already ticked are marked and
   blocked from being added twice; clicking outside dismisses the results.
4. **Cost estimate.** Hold a current price per row; total the bajar list into an
   estimated cost and a cost per guest. Useful when quoting. Would not print on the
   market sheets unless asked.
5. **Learn from what was actually bought.** After the market trip, record real
   quantities against the planned ones; over several events, suggest better per-100
   values. Turns the app from a calculator into a record.
6. **Combined list for same-day events.** Merge two or more orders into one shopping
   trip, while still printing each event's sheet separately.
7. **Per-shop splitting.** The three sheets already roughly match grocery, vegetable
   market and disposables. Make that explicit so different people can be handed
   different sheets.
8. **Remaining recipe cards.** The folder holds cards not yet imported (Kachhi,
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

**Print thresholds are measured, not guessed.** One column holds 22 lines; beyond that
the sheet splits into two columns, and beyond 56 the type tightens. Measured against
A4's 703 × 1032 px printable area with the embedded font — a chef page of 41 rows came
out 782 px tall, and a forced 70-row page 748 px. Re-measure if the font or the row
padding changes.

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
- **Chef assignment is per job and never persisted.** Convenience would cost
  correctness here: the failure mode of a remembered assignment is items quietly
  missing from the market sheet, which is not discovered until the market.
- **Chef shares are proportional, not all-or-nothing.** An overridden or hand-typed
  quantity on a shared row is divided the same way the recipe would have divided it.
  Count units are rounded up per cook, so a split of 7 পিছ gives 4 and 4 — over by one
  rather than short.
- **Write down only what the files actually do.** An earlier draft of this plan
  described order-form features that had never been built. Anything not in the code
  belongs in the roadmap, not in the state table.

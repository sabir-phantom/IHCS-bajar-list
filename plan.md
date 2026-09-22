# Iqbal Hossain Catering — Order & Bajar List System

Working plan and project record. Last updated 2026-09-22.

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
│   ├── order_form.html    order entry, history, order sheet printing  (276 KB)
│   ├── bajarlist.html     recipes, quantity calculation, market sheets (327 KB)
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
| Packages | 3 built in: স্ট্যান্ডার্ড কাচ্চি, রুটি কালিয়া, মোরগ পোলাও; more can be created in the app |
| Review screen | Per-100 and per-order columns, per-ingredient breakdown, folding, confirmations |
| Printing | Only filled rows, no price columns, one page per sheet, measured against A4 |
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
This duplication is deliberate: a shared `.js` file would break the moment someone
copies only the HTML.

**Print thresholds are measured, not guessed.** One column holds 22 lines; beyond
that it splits into two columns, and beyond 56 the text tightens. Measured against
A4's 703 × 1032 px printable area with the embedded font. Re-measure if the font or
the row padding changes.

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

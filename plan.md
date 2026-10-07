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
│   ├── index.html         recipes, quantity calculation, chef pages,
│   │                      market sheets                                (327 KB)
│   ├── guide-en.html      user + maintainer guide, English
│   ├── guide-bn.html      user + maintainer guide, Bangla
│   └── bajar-data.json    the data file — created 22 Sep, holds 1 saved order
└── old/               superseded order form versions, reference only
```

The bajar list is named `index.html` because the repository is published with GitHub
Pages (`.github/workflows/static.yml` serves the `final/` folder), so it is the site's
landing page. `order_form.html` opens it by that name.

Source material lives outside this folder at `D:\Sabir\Work\IC\Menu\Recipes\Recipes`
(the master `bazaar list.pdf` and the individual recipe cards).

---

## 2. Current state

**Done and verified.**

| Area | State |
|---|---|
| Order → bajar list handoff | Set menu tag, headcount, date, venue and item list pass through the URL |
| Recipes | 34 dishes, per 100 guests, all ingredients resolve to form rows |
| Forms | 3 printed sheets, ~260 rows, fully editable (names, units, rows, headings) |
| Packages | 3 built in: স্ট্যান্ডার্ড কাচ্চি, রুটি কালিয়া, মোরগ পোলাও; more can be created in the app. The order form's dropdown reads them live from `localStorage` (`iqbal_catering_bajar_setmenus`) |
| Review screen | Per-100 and per-order columns, per-ingredient breakdown, folding, confirmations, and a sticky search box that filters all three sheets at once |
| Recipe editor | Ingredients are chosen through a searchable box with a menu the page draws itself; an unknown name offers to create the form row; narrow windows stack each ingredient as a card |
| Chef assignment | Each dish of the package can go to বাবুর্চি S / A / C. A and C's ingredients leave the three normal sheets and print on a page of that cook's own, after them. Shares are split in proportion to each dish's contribution, so nothing is bought twice and nothing is mixed between cooks |
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

### Sheet order

`SHEET_ORDER` (`party`, `main`, `produce`) decides the order the sheets are reviewed and
printed in. It is deliberately separate from the order `forms` stores them in, so a data
file written under the old order still comes out right, and a sheet the user creates
keeps its place at the end. The review screen renders in this order too, while each row
still carries its original sheet index, so edits land on the right sheet.

### The whole roasts

আস্ত খাসি and আস্ত মুরগির রোস্ট were added from the owner's list. Both are dressed with
লেটুস পাতা, বিট and মুলা; the খাসি also takes চান্দি তবক, মার্বেল and one সুই-সুতা set.
The papaya is recorded as 1.5 কেজি of বড় পেঁপে rather than "1 piece": that row is kept in
কেজি because মিক্সড ভেজিটেবল buys 3 কেজি of it, and one ingredient stays on one row. The
owner gave 1.5 কেজি as the weight of a whole one.

Neither carries the animal itself — like the rest of the meat that is the owner's call,
typed into খাশীর মাংস or the মোরগ row by hand.

বিট and মুলা were কেজি rows and are now পিছ, which was safe because no recipe used
either. **A data file written before this change still holds them as কেজি**, so after
loading an old one, check those two rows read পিছ.

`DISH_MATCHERS` learned both dishes, and `chickenRoast` now excludes আস্ত — without that
it swallowed "আস্ত মুরগির রোস্ট" on the way past, since it matches any রোস্ট.

### Phone layout

A `@media (max-width: 620px)` block carries the phone rules: package slots stack, the
checklist drops to one column, and a row wraps so the dish name sits beside its tick box
with the three controls on the line below. Each control is wrapped in a `.ctl` span that
is `display: contents` on a wide screen — the desktop layout is untouched — and becomes a
labelled column on a phone, so the controls are still named once the header row is hidden.
Controls stay hidden on unticked rows. The review tables scroll sideways inside their own
box rather than widening the page.

Print is deliberately outside all of this: the `@media print` sizes were measured against
A4 and are not touched by the screen rules.

### Settings live in one dialog

The data-file bar, the cook list and the backup links were three separate blocks at the
foot of the page. They are now three sections of a single ⚙ সেটিংস dialog, opened from
one line at the bottom. The elements kept their ids and moved wholesale, so every handler
that writes to them — `setDataStatus`, `renderChefAdmin`, the import file input — is
untouched.

The line that opens it still reports the data state in words and with the coloured dot,
because that warning is the one thing that must not need a click to be seen: a data file
left unconnected is how a month of order history is lost.

### Cook colours

A cook's colour is stored on the cook (`chefs[].color`), chosen from `CHEF_COLORS` when
they are added, taking the first shade nobody is using. It was keyed on list position
before, which meant removing a cook silently repainted everyone below them. A checklist
row wears its cook's colour; the standard cook keeps the gold. A list saved before
colours existed gets one assigned per cook on load.

### Dishes that default to a cook

`CHEF_DEFAULTS` maps ten dish keys to a cook: six to A (`sherwaniRuti`, `rumaliRuti`,
`beefKalia`, `koraiGosht`, `jhalFry`, `muttonGlassy`) and four to C (`fishFillet`,
`russianSalad`, `dressingSalad`, `doryFish`). `chefOf()` reads it only when the job holds
no assignment at all, so an explicit choice always wins — which is why it tests
`chefAssign[k] !== undefined` rather than truthiness: picking **S** by hand must not fall
back to the default. An id no longer in `chefIds()` (a removed cook) falls to S, so a
default can never strand ingredients off every sheet.

This softens the rule written under *Chef split*: assignments were deliberately not saved
so a forgotten one could never quietly empty the main sheets. Ten dishes now carry one by
default, at the owner's request. The guard above keeps the dangerous half of that risk
closed — nothing is ever assigned to a cook who does not exist — but these ten do move
their ingredients onto a cook's page without anyone asking.

### Items with no recipe

An item can be on a job with no ingredients at all: bought ready-made, sourced by the
cook, or simply not written down yet. There is no flag for it — `hasNoIngredients(key)`
is true exactly while the item holds no `ingredients` and no `editable` lines, so filling
rows in later turns it into an ordinary recipe with nothing to migrate.

`saveRecipe()` confirms rather than refuses when no ingredient rows are given. The
quantity pipeline needed no guarding: `recipePer100()` iterates `dish.ingredients`, so an
empty list contributes nothing and no other amount moves (verified byte-identical).

It is marked উপকরণ নেই in the checklist (`.no-rec`) and the recipe library, and the note
is appended once in `chefSplit()` where `split.dishes` is built — so every printed
রান্নার আইটেম line shows `উপকরণ নেই: নাম` without touching each print site.

**Known limit, accepted by the owner.** Cook pages are built from ingredient rows
(`out.active` keeps only cooks with at least one), so a cook given *only* recipe-less
items gets no printed page and those items appear nowhere on paper. A cook with a mix is
unaffected. The alternative — printing an empty sheet — was declined.

### The dishes are a checklist, not the package

`picked[dishKey]` = `'vip' | 'packet' | 'both'` is now the source of truth for what this
job cooks; `pickedBy[dishKey]` records which preset ticked it. Choosing a package calls
`applyPreset()`, which clears only its own previous picks and then ticks its dishes, so a
hand-ticked item survives a package change. Every recipe is listed with a tick box in two
columns, in library order — the list never reorders, which is what made an earlier
"chosen items float to the top" version unusable.

`dishBaseGuests()` reads the group off `picked`, so a dish marked ভি.আই.পি costs for the
VIP count, প্যাকেট for the packet count, and উভয় for the total. Desserts are ordinary
checklist items; the per-group dessert radios and `chosenDessert` are gone.

An incoming order ticks what it names, including dishes no package covers — so আস্ত খাসি
ordered against the কাচ্চি package is costed rather than only reported missing. Order of
startup matters: `applyIncomingOrder()` fills the package slots and headcounts first,
then the presets run.

### Two menus in one job

VIP guests and packet guests sometimes eat different menus, so there is a package per
group: `GROUPS` = vip, packet. One package alone feeds everybody — `guestsOfGroup()`
returns the whole headcount when only one group is active — which is exactly the old
behaviour, so nothing changes for an ordinary job.

`dishBaseGuests()` maps each dish to the headcount of the group(s) cooking it, summing
when a dish is in both menus. `allDishKeys()` is the union across groups, and dessert is
chosen per group. A package-level value (a saved default, or an owner-confirmed one in
`SET_MENU_PER100`) belongs to its own package, so `effectivePer100()` sums each group's
value at that group's headcount: ভাতের চাউল at 600/100 on কাচ্চি gives 3 kg for 500 VIP
guests rather than for all 700.

Saving per-100 defaults is refused while two menus are active, since there is no way to
tell which package the value belongs to.

Verified: সয়াবিন তেল across both menus is 15 + 10 + 4 = 29 ltr, an ingredient both menus
need lands on one row as the sum, and the chef split still has zero drift.

### Headcounts, per dish

Two headcounts now: ভি.আই.পি and প্যাকেট, costed on their sum. `dishGuests[dishKey]` holds
a number typed against one dish; `guestsFor()` falls back to the total, so a dish only
differs when it was set by hand. Per job, never saved.

This changes the shape of the calculation. A row fed by recipes is no longer
`per100 × total/100`: each dish contributes `per100 × itsOwnHeadcount/100` and the row is
their sum (`recipe.amount`). A value that has no dish behind it — hand-typed, or a package
default — still scales on the total, which `effectivePer100().fixed` marks. `chefSplit()`
weighs on the **scaled** amounts rather than per-100 figures, since with one dish at 700
and another at 500 the per-100 numbers no longer reflect real shares.

Verified by hand: সয়াবিন তেল (কাচ্চি 3 ltr/100 + চিকেন রোস্ট 2 ltr/100) gives 35 ltr with
both at 700, and 31 with the roast at 500. Split between two cooks under mixed headcounts:
21 + 10 = 31, drift 0.

### The across-the-board reduction

`reducePercent()` takes a percentage off every row last, after everything else, so it
reduces whatever the row ended up with. `reduceExcluded[rowId]` keeps a row whole; the
review table shows that tick in a পুরোটাই column, which only appears while a percentage is
set. Per job, not saved — a stale tick would silently leave an ingredient out of a cut.

### Chefs are a list

The footer line is read-only until সম্পাদনা is pressed: the owner found it too open to
edit or delete, and a deleted cook silently returns their dishes to the standard sheets.
ডিফল্টে ফেরান restores just A and C.


`chefs` is a saved list of `{id, name}`, with `S` implicit and always present. Cooks are
added by name at the foot of the page, deliberately understated. Removing one returns its
dishes to standard and says how many first. Colours come from `CHEF_TINTS`/`CHEF_INKS` by
position, so a third and fourth cook stay distinguishable; badges show the letter for the
built-in cooks and the first characters of the name otherwise.

### Chef split

A dish can be handed to বাবুর্চি A or C. `chefSplit()` divides every row's quantity
between S / A / C in proportion to how much each chef's dishes contribute to that row,
taken from the same `sources` that power the ▸ breakdown. The three shares always add
back to the row total (verified: drift 0 across all rows). A row with no recipe behind
it — hand-typed, or a package default like ভাতের চাউল — belongs to no dish and stays
with S.

The cooks' sheets print last in the packet, one page and one merged table per cook, A
before C — a cook is handed their own sheet, never half of one. Each page uses the same
rules as the market sheets: one column to 22 rows, two columns beyond, dense type beyond
56. Measured against the 1032px A4 page, one row at a time: 22 rows fill 989px in one
column, 45 fill 828px in two, 56 fill 963px in two, and the true ceiling is 106 rows at
1020px — 107 reaches 1035px and spills. `CHEF_PAGE_MAX` is deliberately set at 100, five
rows short of the ceiling, because a long Bangla name can wrap to a second line and cost
height; past it the review screen warns in red rather than quietly printing a second
sheet. The ceiling is unreachable in practice: the whole master is 227 rows and the
standard package uses 41.

### Order form handoff — verified

Tested over `http://localhost` (the desktop preview pane strips query strings from
`file://`, which is why this went unverified for so long). `order_form.html`'s
`openBajarList()` refuses without a set menu or with no items ticked, otherwise opens
`index.html?setMenu=&setMenuPacket=&vip=&packet=&date=&place=&label=&items=[…]`. On arrival the package
is selected, the four header fields are filled, the dessert named in `items` is ticked,
the review table is computed, and a notice names the set menu and lists any ordered item
with no recipe behind it. Confirmed with the standard কাচ্চি package at 250 guests and
the real ভি.আই.পি menu list: 42 rows carried a quantity, শাহী জর্দা was picked up as the
dessert, and seven items were flagged as having no recipe.

That flag list is worth acting on. Three of them are real dishes whose ingredients
therefore never reach the bajar list at all:

- [ ] **আস্ত খাশীর রোস্ট** — no recipe, no form row
- [ ] **চিকেন সাসলিক** — no recipe, no form row
- [ ] **ভেটকী গ্রীল** — no recipe, no form row

The other four are consumables that do have form rows to type a quantity into, so the
flag is correct but harmless: মিনারেল ওয়াটার (rows পানি ১.৫ লিটার / ৫০০ মিলি),
শাহী পান বক্স (বক্স পান), টিস্যু/তাওয়েল/সাবান (টিস্যু পেপার, টিস্যু বক্স),
আলু বোখারার চাটনী (আলু বোখারা চাটনি).

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

`index.html` owns the packages. `order_form.html` carries its own built-in list in
`BAJAR_SET_MENUS` and merges anything the user created, read live from `localStorage`
(`iqbal_catering_bajar_setmenus`). Packages made in the app therefore appear in the
order form's dropdown without any edit; only a new *hardcoded* package needs a matching
entry in both files.

The order form passes the chosen package to the bajar list by key, through the URL.
It does not know which dishes a package contains — that stays in `index.html`.

---

## 4. Open items

### Needs the owner's answer

- **তেহারি এলাচি / দারুচিনি are wanted in pieces** but rows `p25` and `p26` are weight
  rows, and মোতানজান (5/5), বিফ কালা ভুনা (20/20), মুরগ পোলাও (50/75) and
  মিক্সড ভেজিটেবল (20/30) all buy from them in grams. Switching the rows to pieces
  would silently reinterpret those four. Either the gram weight of ১০ এলাচি and ২০
  দারুচিনি, or piece counts for those four recipes, unblocks it. Stored as 10 gm and
  20 gm meanwhile.
- **কেওড়া জল is on the ঝাল ফ্রাই (50 gm) and কড়াই গোস্ত (100 gm) sheets**, but form
  row `p44` is counted in bottles. Left out of both rather than guessed at.
- **ডাস্টার has no form row**, so the ১–২ টি on the three salad/fillet sheets is left
  out. টুকরি (`p74`) does have one and is in.
- **মাটন রেজালা and বিফ রেজালা carry no meat**, because the বিফ / মাটন রেজালা they
  were copied from has only three lines and no মাংস. The original is still in the library.
- **রাশিয়ান সালাদ তেজপাতা 250 gm** is printed on the sheet and entered as such, but it
  is a lot of bay leaf for a salad — worth a second look.

- **তেহারি has no rice and no meat.** The hand-written sheets carry a quantity for 21
  ingredients but none for পোলাউ চাল or any meat — both were ticked with no number, as
  were চিনি, ডিম, টমেটো সস্, ফয়েল and রং. The recipe is in the library and correct as
  far as it goes, but it will not buy rice or meat until those figures are added.
- **তেহারি এলাচি 10 and দারুচিনি 20 were given without a unit** and are stored as grams,
  matching every other spice on that sheet. Worth confirming once.

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
      confirm each shows a green dot — `index.html` has changed a lot since then.
- [ ] Test print all three sheets on the actual printer, A4, default margins.
- [ ] Confirm with the owner which dishes are normally VIP-only, so the per-dish headcount
      is typed rather than remembered wrongly.
- [ ] Test print a job with a dish given to বাবুর্চি A and another to C: confirm the
      two cook pages come out first and the shares add up to the full amount.

---

### Recipe editor and list search

`PICK_TO_ROW` / `ROW_TO_PICK` give every form row a display name that is unique. 18 names
sit on more than one row (ঘি on three, ডিম on four), so the name alone cannot identify a
row and the duplicates carry their sheet: `ঘি — পার্টি বাজার লিস্ট`. A bare ambiguous name
resolves to nothing rather than guessing, which is what keeps "one ingredient, one row"
true when a recipe is edited.

The suggestion menu is drawn by the page, not by a `<datalist>`. The ingredient input sits
inside `.rec-table-wrap`, which scrolls sideways on a narrow window; once scrolled, the
input is clipped and a native popup anchored to it lands somewhere else. The menu is now
fixed-positioned against the input's own rectangle and repositions on scroll.

Typing an unknown name offers to create the row: pick a sheet and unit, and it is appended
to that form, indexed, and bound to the recipe in one step. This is the only route by which
a brand-new ingredient enters the system.

**Two bugs worth remembering**, both the same shape:

- The ✕ button did nothing. Clicking it blurred the focused ingredient box, the blur fired
  `change`, and `change` redrew the table — so the button was detached before its click
  could be delivered. Fixed by acting on `mousedown` with `preventDefault`, and by
  inserting the create-row offer into one cell instead of redrawing.
- A synthetic `.click()` in a test involves no focus, so it passed while the real button
  was dead. **Reproduce pointer bugs with the pointer**, or at least with the full
  mousedown → blur → click sequence.

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
`RECIPE_LIBRARY` in `index.html` as
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
`index.html` and in `BAJAR_SET_MENUS` in `order_form.html`. The keys must match.
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

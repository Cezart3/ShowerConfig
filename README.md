# ShowerConfig

A **desktop configurator for custom shower cabins**, built for a real
manufacturing workflow: it walks a salesperson through cabin type, dimensions,
glass, hardware and finish, then **computes the bill of materials and the price**
and exports a **PDF quote** for the customer.

> ### 🔒 Why this repository is a description only
> The full source code is kept **private** because it was **built for a client** —
> it encodes their product catalog, pricing rules and hardware data, which belongs
> to them, so the code stays closed. This page documents what it does and how it's
> built.
>
> Happy to demo it or walk through the code on request.

---

## What it does

A guided, **step-by-step wizard** that mirrors how a cabin is actually quoted. A
`StepNavigator` drives the flow and the path adapts to the product chosen — a
hinged cabin branches into sub-types, a fixed panel skips straight to dimensions:

1. **Cabin type** — pick from the supported models (see below). The choice is the
   pivot of the whole configuration: it decides which hardware is preselected and
   how the glass is later cut.
2. **Dimensions** — enter measurements; the app computes the **surface area and
   perimeter of every glass panel** individually.
3. **Glass** — type (cut / tempered-toughened), shape and template options, with
   per-panel surcharges.
4. **Hardware & finish** — the hardware list arrives **already filled in from the
   cabin type**; the salesperson adjusts only the exceptions. Color and finish are
   selected here, and hardware that carries finish-specific or special pricing is
   handled automatically.
5. **Quote** — the **price calculator** sums glass (area × rate + grinding,
   drilling, extra cut-outs, shape/template surcharge), hardware and profiles,
   applies the live **RON exchange rate from the BNR (National Bank of Romania)
   API**, and the result is exported as a **PDF offer**.

An **admin panel** (login-gated) lets the business edit the catalog — add/remove
products, modify elements and prices, change settings — so the configurator stays
up to date without touching code.

## Cabin types → automatic hardware preselection

This is the core of the tool. The shop sells **several cabin models**, and each
model has a known *recipe* of hardware: which hinges, how many handles, which
rigidizing profiles and connectors, which seals, and what profile lengths. Instead
of making the operator rebuild that list every time (slow and error-prone), each
cabin type ships with its hardware set **predefined**, and selecting a type
**preselects every piece with the right quantity**.

Supported models today:

| Model | Typical use | Example preselected hardware |
|-------|-------------|------------------------------|
| **Type 1 / Type 2** | Single hinged door | 2x hinges, 1x button handle, rigidizing profiles + connectors, 3 seal types, U/GPU profiles |
| **Type 3 / Type 4** | Hinged door + fixed side | heavier hinge model, extra connector profiles |
| **Type 5 / Type 6** | Double / corner configurations | 4x hinges, 2x handles, doubled profiles and seals |
| **Fixed panel** | Walk-in fixed glass | hinges + profiles + seals, no door handle |

*(Exact product codes and quantities live in the private catalog; the table above
is illustrative.)*

Concretely, every model is described by a `CabinTypeInfo` object that groups
hardware by category and stores profile lengths:

```
CabinTypeInfo "type_1"
  hinges            -> { SH301: 2 }
  button handles    -> { BR20: 1 }
  profiles/connect. -> { GT02-304: 1, C34: 1, C35: 1 }
  seals             -> { S01: 1, S02: 1, S10: 1 }
  profile lengths   -> { U20: 1, GPU: 1 }
```

When the hardware step opens, the panel reads the selected model and **ticks the
matching items and fills their quantity spinners automatically**; the operator can
still add, remove or re-quantify anything. A live "recipe" summary shows what the
chosen type implies. The same `CabinTypeInfo` is reused downstream by the **glass
dimension calculator** — hinges subtract from panel height, seals from both sides,
each profile its own amount — so the cut sizes are consistent with the hardware
that was actually selected. **One choice up front drives hardware, dimensions and
price**, which is the entire point of the tool.

## Tech stack

- **Java 21** + **Swing** with **FlatLaf** for a modern desktop UI
- **MySQL** (via `mysql-connector-j`) — product catalog, glass and pricing data;
  DAO layer for access
- **Apache Batik** — renders the live **SVG preview** of the configured cabin
- **iText** — generates the **PDF quote**
- **BNR exchange-rate API** — live RON conversion (cached, with a bundled
  truststore for the HTTPS call)
- **GSON** — JSON serialization; **dotenv-java** — DB credentials via `.env`
- **Maven** (assembly plugin → fat JAR) packaged into a **Windows MSI installer**
  for non-technical client machines

## Architecture (high level)

```
gui/steps/        the wizard panels (type, sub-type, dimensions, glass, hardware, color, ...)
gui/admin/        catalog & settings editor (login-gated)
gui/navigation/   StepNavigator + side navigation (adaptive, type-dependent flow)
gui/preview/      SVG cabin preview (Batik)
model/cabin/      cabin types + their predefined hardware sets (the recipes)
model/            glass, product
db/               DAO layer over MySQL (products, glass)
util/calculator/  glass dimension + price calculation
util/export/      PDF quote exporter (iText)
util/api/         BNR exchange-rate client
util/session/     login session + current configuration state
```

## Design notes

- **Catalog lives in the database, recipes live in code.** Individual products
  (codes, prices, finishes) are editable by the client through the admin panel
  without a rebuild; the per-type hardware *recipes* are versioned in code because
  they're structural to each model, not day-to-day data.
- **Single source of truth for a configuration.** A session object holds the
  current selection (type, dimensions, glass options, hardware quantities) and is
  read by the calculators, the preview and the PDF exporter alike, so they never
  disagree.
- **Built for a non-technical operator.** Sensible defaults from the cabin type,
  an always-visible recipe summary, and a one-click PDF keep the quoting flow fast
  on the shop floor.

## Status

✅ **Delivered and in production use**, packaged as a Windows installer with a
database restore script and an end-user setup guide.

---

### Contact

- 💼 [LinkedIn](https://www.linkedin.com/in/tocaciu-cezar-0865373b6/)
- 📧 cezartocaciu233@gmail.com

<sub>© 2026 Cezar Tocaciu. Code private (client work). This page is documentation only.</sub>

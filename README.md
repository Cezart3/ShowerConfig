# ShowerConfig

A **desktop configurator for custom shower cabins**, built for a real
manufacturing workflow: it walks a salesperson through cabin type, dimensions,
glass, hardware and finish, then **computes the bill of materials and the price**
and exports a **PDF quote** for the customer.

> ### 🔒 Why this repository is a description only
> The full source code is kept **private** because this is **commissioned client
> work** — it was built for **two real businesses** that produce shower cabins, and
> it encodes their product catalog, pricing rules and hardware data. That belongs
> to the clients, so the code stays closed. This page documents what it does and
> how it's built.
>
> Happy to demo it or walk through the code on request.

---

## What it does

A guided, **step-by-step wizard** that mirrors how a cabin is actually quoted:

1. **Cabin type** — pick from the supported models, each with its own predefined
   hardware set (hinges, handles, rigidizing profiles, connectors, seals).
2. **Dimensions** — enter measurements; the app computes the **surface area and
   perimeter of every glass panel** individually.
3. **Glass** — type (cut / tempered-toughened), shape and template options, with
   per-panel surcharges.
4. **Hardware & finish** — color and finish selection, with hardware that carries
   finish-specific or special pricing handled automatically.
5. **Quote** — the **price calculator** sums glass (area × rate + grinding,
   drilling, extra cut-outs, shape/template surcharge), hardware and profiles,
   applies the live **RON exchange rate from the BNR (National Bank of Romania)
   API**, and the result is exported as a **PDF offer**.

An **admin panel** (login-gated) lets the business edit the catalog — add/remove
products, modify elements and prices, change settings — so the configurator stays
up to date without touching code.

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
gui/steps/        the wizard panels (type, dimensions, glass, hardware, color, ...)
gui/admin/        catalog & settings editor (login-gated)
gui/preview/      SVG cabin preview (Batik)
model/            cabin types + predefined hardware sets, glass, product
db/               DAO layer over MySQL (products, glass)
util/calculator/  glass dimension + price calculation
util/export/      PDF quote exporter (iText)
util/api/         BNR exchange-rate client
util/session/     login session + current configuration state
```

## Status

✅ **Delivered and in use** by two manufacturing clients, packaged as a Windows
installer with a database restore script and an end-user setup guide.

---

### Contact

- 💼 [LinkedIn](https://www.linkedin.com/in/tocaciu-cezar-0865373b6/)
- 📧 cezartocaciu233@gmail.com

<sub>© 2026 Cezar Tocaciu. Code private (client work). This page is documentation only.</sub>

# Prompt v2 — Simulador de Leasing en UN SOLO ARCHIVO HTML (GitHub Pages)


---

## ROLE

You are a senior front-end developer. Build the **entire thing in one pass**: one working file, no placeholders, no TODOs, no build step. I need to demo it in under an hour.

## GOAL

A Spanish-language web page that lets my sister and my elderly parents **see and compare what happens to her apartment debt in Colombia** under different payment decisions. It is a decision tool for non-financial people, not a spreadsheet.

## TECH STACK (fixed, do not substitute)

- **One single self-contained `index.html` file.** HTML + CSS + vanilla JavaScript, all inline. No React, no build step, no bundler, no npm, no framework.
- **Charts: Plotly.js** loaded from a CDN (`https://cdn.plotly.dev/plotly-3.x.min.js` or cdnjs). That is the only external dependency allowed.
- Must work by **double-clicking the file locally** and by being served as a static file from **GitHub Pages** — no server, no backend, no Python, no API calls. All math runs in the browser.
- Must work on a phone and on a laptop. No cold start, no loading spinner beyond the CDN fetch.
- Deliver a `README.md` with the two lines needed to publish it: commit `index.html` to the repo root, then Settings → Pages → Deploy from branch → `main` / root.

## REAL DATA (from the Davivienda statement — hard-code as defaults, but make every value editable in the UI)

| Concepto | Valor |
|---|---|
| Producto | Leasing Habitacional, Banco Davivienda |
| Saldo actual (sitio del banco, hoy) | **$254,090,608.41 COP** |
| Cuota mensual (canon) | **$3,441,000 COP** |
| Tasa de interés **cobrada** | **13.00 % Efectivo Anual** |
| Tasa de interés **pactada** | **20.98 % Efectivo Anual** |
| Sistema de amortización | BAJA $ 0 % LEAS (cuota fija en pesos, sin UVR, sin indexación) |
| Plazo total | 180 meses |
| Cánones pagados | 30 |
| Cánones pendientes según banco | 150 |
| Opción de compra | 0.00 % |
| Seguros y gastos administrativos | $0.00 (dejar campo editable por si cambian) |
| Fecha de corte / próximo pago | 08 Sep 2026 |

**Verification data from last period (use it to prove the math is right):** saldo 08-Jul-2026 = $255,745,119.37 → intereses corrientes $2,618,017.11 + abono a capital $821,982.89 = pago $3,440,000 → saldo 08-Ago-2026 = $255,090,697.08.

## FINANCIAL MODEL (be precise — this is the part that must not be wrong)

1. Monthly rate from the **effective annual** rate: `r = Math.pow(1 + EA, 1/12) - 1`. At 13 % EA → `r = 1.023684 %` monthly. At 20.98 % EA → `r = 1.599787 %` monthly. **Do not use EA/12.**
2. Monthly loop, in this order: `interes = saldo * r`; `abonoCapital = cuota + abonoExtra - interes`; `saldo = saldo - abonoCapital`.
3. Cap the last payment so the balance lands exactly on 0 (never negative).
4. **Two ways to apply an extra payment** — exactly the choice the bank's own payment coupon offers, so make it a visible radio button:
   - **"Abono a capital → disminuir el PLAZO"** (cuota stays the same, loan ends earlier). *Note in the UI: this is what Davivienda applies by default if you don't tell them otherwise.*
   - **"Abono a capital → disminuir la CUOTA"** (term stays the same, monthly payment drops; recompute the payment with the standard annuity formula each time an extra payment is made).
5. Guard against a non-amortizing case: if `cuota <= saldo * r`, **stop the loop** and show a big warning card — *"Con esta tasa la cuota no alcanza a cubrir los intereses: la deuda crecería en vez de bajar."* Never let the loop run forever; hard-cap at 600 iterations.

## SCENARIOS (all four visible and comparable at once)

1. **Solo la cuota normal** — baseline, nothing extra.
2. **Abono mensual extra** — slider (default $500,000; range $0 – $5,000,000, steps of $100,000) with the plazo/cuota radio button above.
3. **Abono único grande (una sola vez)** — amount + the month it is made (a prima, un ahorro), same plazo/cuota choice.
4. **Pagar toda la deuda hoy** — show today's payoff amount and how much interest that avoids vs. the baseline.

Plus a **fifth section as a warning: "¿Y si se acaba la cobertura de tasa?"** — the statement shows a *pactada* rate of 20.98 % but a *cobrada* rate of 13 % (a FRECH-type rate subsidy that lasts a limited number of years). Let the user pick the month the rate jumps to 20.98 %. **Important:** at 20.98 % the monthly interest on the current balance is ≈ $4,064,700, which is **more than the $3,441,000 payment** — the debt would never amortize. Surface that plainly, in plain Spanish, as the single most important risk on the page. Do **not** hide it in a footnote.

## OUTPUT ON SCREEN

- Four big KPI cards: **Cuándo termina** (mes/año and "X años Y meses"), **Intereses totales que pagará**, **Cuánto se ahorra vs. la cuota normal**, **Cuántos meses se ahorra**. These must update live as the sliders move (recalculate on `input`, it is instant — a few hundred iterations).
- Chart 1: **saldo de la deuda mes a mes**, one line per scenario.
- Chart 2: **interés vs. capital** per month (stacked area) — makes visible that today most of the payment is interest.
- Chart 3: **barras horizontales comparando intereses totales** de los escenarios.
- A "resumen en palabras" box under the charts: 2–3 plain sentences, auto-generated, e.g. *"Si abonas $500.000 más cada mes, terminas de pagar en agosto de 2035 en vez de julio de 2038, y te ahorras $58.999.000 en intereses."*
- A collapsible **tabla de amortización** (year by year by default) and a **"Descargar tabla en CSV"** button implemented with a Blob download — no library.
- Optional but nice: a **"Imprimir / Guardar como PDF"** button (`window.print()`) with a `@media print` stylesheet, so my parents can hold it on paper.

**Reference numbers to validate your implementation** (13 % EA, saldo $254,090,608.41, cuota $3,441,000, modo "disminuir plazo"):

| Abono extra mensual | Meses restantes | Intereses totales |
|---|---|---|
| $0 | 139 | $222,364,788 |
| $300,000 | 117 | $182,427,113 |
| $500,000 | 106 | $163,362,688 |
| $1,000,000 | 87 | $130,137,983 |
| $2,000,000 | 64 | $93,261,917 |

If your numbers don't match these within a few thousand pesos, your rate conversion or loop order is wrong. Fix it before moving on. (The baseline is 139 months, not the 150 the bank shows — because the charged rate is below the contracted rate.) Add a small self-check function that runs these five cases on page load and logs pass/fail to the console.

## ACCESSIBILITY — NON-NEGOTIABLE (my parents are elderly; there are vision problems in the family)

- **Base font size 22 px minimum**; KPI numbers **48–64 px, bold**; headings 34 px+. Set it on `html { font-size: 20px }` and use `rem` throughout.
- **High contrast**: near-black `#111111` text on white `#FFFFFF`. Add a **"Modo alto contraste"** toggle that switches to `#FFFFFF` on `#0B0B0B`. Never gray-on-gray, never thin fonts, no font weight below 500.
- **Color-blind safe palette**, and never use color as the only signal — always pair it with a text label and a distinct line style. Use: `#0072B2` (azul), `#E69F00` (naranja), `#009E73` (verde), `#D55E00` (rojo-naranja), `#CC79A7` (rosa). Keep every line **4 px thick** and every Plotly axis/tick font ≥ 18 px (set it explicitly in the layout — Plotly's defaults are far too small).
- Sliders with **large handles** (style `input[type=range]` explicitly, at least 36 px thumbs) and a big number shown next to each one. Big touch targets, generous spacing, no dense tables on the first screen.
- Semantic HTML, `<label>` on every control, `lang="es"` on the `<html>` tag.

## LANGUAGE

- **Everything the user sees must be in Spanish (Colombia)** — labels, buttons, tooltips, chart axes, warnings.
- Plain language, no financial jargon. Say *"lo que pagas de más por los intereses"*, not *"costo financiero"*. Explain "abono a capital" in one short sentence where it first appears.
- Format money as Colombian pesos with dots as thousand separators and no decimals: `$254.090.608` (`toLocaleString('es-CO')`). Format dates as `agosto 2035`.
- Title: **"Simulador del apartamento — ¿Qué pasa si abonamos más?"** Add a short intro line explaining what the tool does and that the numbers come from the Davivienda statement of 08-Sep-2026.

## GUARDRAILS

- Do **not** invent numbers, rates, or fees. If something isn't in the data above, make it an editable input with a sensible default labeled *"supuesto"*.
- Visible disclaimer at the bottom: *"Estimación educativa basada en el extracto del 08/09/2026. Las cifras reales pueden variar por seguros, fechas de pago y decisiones del banco. Confirme siempre con Davivienda antes de tomar una decisión."*
- No analytics, no trackers, no external requests other than the Plotly CDN.
- **Do not put my sister's name, the contract number, or any other personal identifier in the file** — this will be on a public GitHub Pages URL. Refer to it only as "el apartamento".

## DELIVERABLES

`index.html` (self-contained) and `README.md`. Tell me the one command to open it locally and the two clicks to publish it on GitHub Pages.

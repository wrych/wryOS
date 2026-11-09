# wryOS — Product Specification

**Status: draft 1, for review.** Nothing here is built yet. Requirement ids
(`ST-1`, `PR-3`, …) are stable so that issues, pull requests and tests can cite
them.

## 1. What wryOS is

wryOS lets a small CNC manufacturer sell customisable parts online and guides
each order from the shopper's click to the finished, shipped part.

- **For shoppers:** ordering a CNC-made part is as easy as ordering a screw.
  They pick a product, type their text, see exactly what they will get, see the
  price, pay.
- **For the shop:** no IT knowledge needed. wryOS tells the operator what to do
  next: which stock to cut to what size, which file to load on which machine,
  what to pack and ship.
- **For integrators:** every step a person does by hand can later be done by a
  device or another system through a documented interface.

wryOS starts with wood (signs and similar flat parts) and is built so metal can
follow.

**Simplicity is a requirement, not a wish.** Every screen has one obvious next
action. A feature that needs explaining is redesigned or dropped. A requirement
below that cannot be met simply is cut back before it is built, and anything
not needed for the current milestone waits.

## 2. Decisions taken so far

| Topic            | Decision                                                                                                                                   |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Audience         | Multi-shop SaaS. Open source; anyone can self-host. The hosted service may charge a small fee to cover costs.                              |
| Storefront       | Each shop has its own branded store. No shared marketplace.                                                                                |
| Customisation    | Text (and listed options such as font, size, wood) on designs the shop created. No free dimensions, no file upload in version 1.           |
| Design source    | A parametric Fusion template with its CAM setup is the truth for the cut part. A web preview mirrors its parameters.                       |
| Order to G-code  | A wryOS add-in runs inside Fusion on a PC in the shop. It sits behind a seam so a cloud CAM service can replace it.                        |
| Price            | A formula per product (base price plus surcharges per option). It sits behind a seam so a CAM-based cost estimate can replace it.          |
| Approval         | A person approves each new design once. Approved designs flow into the backlog without a per-order check.                                  |
| Shop floor       | Guided manual steps first; each step is also an integration point.                                                                         |
| Machines         | Controller-neutral. Manual file transfer first; machine connectors plug in later.                                                          |
| Business scope   | Online payment, shipping, invoices and VAT, material stock tracking are all in version 1.                                                  |
| Market           | Switzerland only: CHF, Swiss VAT, TWINT and cards, Swiss Post, German first with further languages prepared.                               |
| Analytics        | Mixpanel, for the store and the back office, without personal data and only when configured.                                               |
| Technology       | TypeScript and Postgres; self-hosting from one Docker Compose file.                                                                        |
| First milestone  | The storefront is close to finished, to test demand. Every later step exists in a simple form so a real order can be fulfilled end to end. |

Technology, the Fusion bridge, the multi-shop model and the integration
interface each get an ADR before code lands.

## 3. Who uses it

| Person        | What they want                                                             |
| ------------- | -------------------------------------------------------------------------- |
| Shopper       | A personalised part, a clear price, no surprises.                          |
| Shop owner    | Sell products, set prices, see money and stock, without an IT department.  |
| Operator      | Know what to make next and how. Often the same person as the owner.        |
| Designer      | Turn a Fusion design into a sellable product. Often the owner too.         |
| Integrator    | Connect a machine, a stock shelf or another system to the shop.            |
| Host          | Run wryOS for one or many shops.                                           |

## 4. The journey of an order

1. The shopper customises a product and sees preview and price update live.
2. The shopper pays. Stock for the order is reserved.
3. The order appears in the shop's backlog.
4. The Fusion add-in fills the template with the shopper's values and returns
   G-code, the stock size needed and the estimated machine time.
5. The operator is guided: cut stock, load it, load the file, run, finish,
   check quality, pack.
6. wryOS creates the shipping label; the shopper gets a tracking e-mail.
7. The invoice is issued and the order is closed.

The shopper can see the order's status at every step.

## 5. Requirements

### 5.1 Store (ST)

- **ST-1** Each shop has its own store with its name, logo, colours and texts,
  on a wryOS address or the shop's own domain.
- **ST-2** A product page shows a live preview that updates as the shopper
  types, without a page reload and without waiting for the server.
- **ST-3** The preview shows the real material look (wood type, engraving
  depth, finish) and the real proportions, and can be turned and zoomed.
- **ST-4** The price is shown at all times and updates with every change.
- **ST-5** A customisation that cannot be made (text too long, a character the
  font lacks, a letter too thin for the tool) is refused while the shopper
  types, with a plain explanation. What can be ordered can be made.
- **ST-6** A product whose material is out of stock is shown as unavailable or
  with a longer delivery time, as the shop chooses.
- **ST-7** The shopper sees an expected delivery date before paying.
- **ST-8** Checkout needs no account. It asks only for what delivery and the
  invoice need.
- **ST-9** The shopper confirms the final preview before paying. That exact
  preview is stored with the order.
- **ST-10** The shopper receives an order confirmation, a shipping notice with
  tracking, and can open an order status page from a link.
- **ST-11** The store works on a phone as well as on a desktop, and is usable
  with keyboard and screen reader.
- **ST-12** The store is in German; all texts are translatable.

### 5.2 Products and designs (PD)

- **PD-1** A designer registers a Fusion template as a product and names the
  parameters shoppers may change (for example text, font, size, wood).
- **PD-2** Each parameter has limits set by the shop: allowed values, lengths,
  characters.
- **PD-3** The web preview is built from the same parameters. Before a product
  can be published, wryOS shows the preview next to Fusion's own rendering for
  a set of test values, and a person approves the match.
- **PD-4** Approval also requires one successful G-code run through the
  pipeline for the test values.
- **PD-5** A changed template is a new version and needs approval again. Orders
  keep the version they were placed with.
- **PD-6** A product can be drafted, published, paused and retired.

### 5.3 Pricing (PR)

- **PR-1** A product's price is a base price plus surcharges per option, set by
  the shop in a form, not in code.
- **PR-2** The price shown in the store is the price charged. The server
  recalculates it at checkout.
- **PR-3** Prices are shown including VAT, with shipping shown before payment.
- **PR-4** Price calculation is one replaceable module with a fixed interface:
  product, chosen values and shop in; price and its breakdown out.
- **PR-5** For each finished order the shop sees real material used and real
  machine time next to the price charged, to tune prices.

### 5.4 Orders and backlog (OR)

- **OR-1** A paid order appears in the shop's backlog with its status, due date
  and what is blocking it, if anything.
- **OR-2** The backlog is ordered by due date; the shop can reorder it.
- **OR-3** An order can be cancelled and refunded until production starts.
  After that the shop decides.
- **OR-4** A failed part (broken, wrong, poor quality) can be remade with one
  action; the extra stock is booked and the reason recorded.
- **OR-5** Every status change is recorded with time and who or what caused it.

### 5.5 From order to G-code (CAM)

- **CAM-1** For each order, wryOS produces a job: template version plus the
  shopper's values.
- **CAM-2** A CAM worker turns a job into G-code for a named machine, the stock
  size needed, the tools needed and the estimated machine time.
- **CAM-3** The first CAM worker is an add-in inside Fusion on a shop PC. It
  asks wryOS for work; wryOS never needs to reach into the shop's network.
- **CAM-4** The CAM worker interface is fixed and documented, so a cloud CAM
  service can replace the add-in without changes elsewhere.
- **CAM-5** A job that fails (Fusion error, toolpath warning, part outside the
  machine's range) blocks its order with a plain explanation in the backlog. It
  is never silently skipped.
- **CAM-6** The owner sees whether the shop PC is connected and how many jobs
  are waiting.
- **CAM-7** Installing the add-in is a guided setup: download, open in Fusion,
  paste a pairing code.

### 5.6 Shop floor (SF)

- **SF-1** The operator sees one clear "next thing to do", on a tablet or phone
  at the machine.
- **SF-2** A job is a list of steps, each confirmed when done: fetch stock, cut
  stock to size, fit tool, load stock, load file, run, unload, finish, check,
  pack.
- **SF-3** Each step states exactly what is needed: which material, which
  dimensions, which tool, which file, which machine.
- **SF-4** The G-code file is offered for download, named so it cannot be mixed
  up with another job.
- **SF-5** The operator can report a problem at any step; the order is then
  blocked and the owner is told.
- **SF-6** Orders that share material and machine can be grouped so stock is
  cut and loaded once.

### 5.7 Machines (MA)

- **MA-1** The shop lists its machines: name, working area, tools available,
  materials it can cut.
- **MA-2** Each machine has a status: free, running, needs attention, out of
  service.
- **MA-3** Running hours and jobs done are counted per machine and per tool;
  maintenance and tool-change reminders follow from limits the shop sets.
- **MA-4** A product names the machines that may cut it. wryOS only schedules a
  job on one of them.

### 5.8 Materials and stock (MT)

- **MT-1** The shop lists its materials: type, thickness, sheet or board size,
  cost, supplier.
- **MT-2** Stock is counted per material and size. Receiving, using, scrapping
  and correcting stock are all recorded.
- **MT-3** A paid order reserves its stock. A cancelled order releases it.
- **MT-4** The shop sets a minimum per material. Below it, the owner is warned
  and the store reacts as in ST-6.
- **MT-5** Usable offcuts can be put back into stock with their size.

### 5.9 Payment, invoices and tax (PAY)

- **PAY-1** Shoppers pay by card or TWINT through a payment provider. wryOS
  never stores card data.
- **PAY-2** Money goes to the shop's own account with the payment provider.
- **PAY-3** Production starts only for paid orders.
- **PAY-4** Each order gets a numbered invoice as a PDF that meets Swiss
  requirements, including for shops not liable for VAT.
- **PAY-5** Refunds and credit notes are issued from the order.
- **PAY-6** The shop can export orders, invoices and VAT totals for its
  accountant for any period.

### 5.10 Shipping (SH)

- **SH-1** Shipping cost is calculated at checkout from the parcel's size and
  weight, which come from the product.
- **SH-2** The shop prints a Swiss Post label from the order with one action.
- **SH-3** The tracking number is stored and sent to the shopper.
- **SH-4** Pickup at the shop can be offered as an alternative.

### 5.11 Integration (IN)

- **IN-1** Everything a person can do in the back office and on the shop floor
  can be done through a documented interface.
- **IN-2** Every step on the shop floor can be assigned to a person or to a
  connected device. A device receives a request and confirms completion, for
  example: an automated shelf receives "provide this stock", answers "stock is
  ready".
- **IN-3** wryOS announces events (order paid, job ready, step done, stock low,
  machine needs attention) so other systems can react.
- **IN-4** A device that does not answer in time hands its step back to a
  person, visibly. A job never waits unseen.
- **IN-5** Each connected system has its own access key with only the
  permissions it needs, which the owner can revoke.

### 5.12 Shops, people and hosting (SYS)

- **SYS-1** A shop's data is visible only to that shop. This is enforced on the
  server for every request.
- **SYS-2** A shop invites people and gives them permissions (manage products,
  see money, work the floor).
- **SYS-3** A new shop is guided through setup: shop details, payment account,
  first material, first machine, Fusion pairing, first product.
- **SYS-4** wryOS can be installed on one's own server with one documented
  command, and updated the same way.
- **SYS-5** The hosted service can charge shops a fee; a self-hosted
  installation has no fee and no dependency on the hosted service.
- **SYS-6** Shopper data is kept only as long as the order, the warranty and
  the law require, and can be exported or deleted on request.

### 5.13 Analytics (AN)

- **AN-1** The store records how shoppers move from product view to
  customisation, checkout and payment, so the shop sees where they leave.
- **AN-2** The back office and shop floor record which features are used and
  how long each order step takes.
- **AN-3** Events carry no personal data: no name, address, e-mail, and never
  the text a shopper typed. Shoppers are counted with a random id.
- **AN-4** Store analytics follow the shopper's consent choice under Swiss data
  protection law; refusing changes nothing else in the store.
- **AN-5** Events are sent to Mixpanel, with data kept in the EU. All events
  pass through one function, named in one list, so the list is the full record
  of what is collected.
- **AN-6** Analytics are off unless a Mixpanel key is configured. A self-hosted
  installation works fully without it.
- **AN-7** Each shop sees only its own figures.

## 6. First milestone

Goal: find out whether shoppers want this, and be able to deliver if they do.

| Area              | In the first milestone                                                                |
| ----------------- | ------------------------------------------------------------------------------------- |
| Store             | Complete: ST-1 to ST-12, for one shop and a handful of sign products.                 |
| Pricing           | Complete: PR-1 to PR-4.                                                               |
| Payment           | PAY-1 to PAY-4.                                                                       |
| Products          | PD-1 to PD-6, set up with help rather than a polished editor.                         |
| Order to G-code   | CAM-1 to CAM-5, working for the first products.                                       |
| Backlog and floor | OR-1, OR-3, OR-5, SF-1 to SF-5 in a plain form.                                       |
| Materials         | MT-1 to MT-4.                                                                         |
| Machines          | MA-1, MA-2, MA-4.                                                                     |
| Shipping          | SH-1 with fixed rates; labels made by hand at Swiss Post; SH-3 typed in.              |
| Integration       | The interfaces exist because wryOS's own screens use them (IN-1). No devices yet.     |
| Analytics         | AN-1 and AN-3 to AN-6: the store funnel is what measures demand. AN-2 follows.        |
| Shops             | Data is kept per shop from the first day (SYS-1). Self-service sign-up comes later.   |

Later: self-service shop onboarding, Swiss Post label printing, accounting
export, machine connectors, device integrations, grouping of jobs, cost-based
pricing, free dimensions, uploads, metal.

## 7. Not in version 1

- Shoppers uploading their own drawings.
- Free dimensions.
- A shared marketplace across shops.
- Sales outside Switzerland.
- Sending G-code to a machine or reading machine status automatically.
- Metal-specific needs (coolant, fixtures, tolerances, material certificates).

## 8. Open questions

1. **Preview fidelity.** Is a faithful 2.5D rendering (engraved text on a
   textured board, turnable) enough, or must the preview show true 3D shapes
   such as V-carved bevels?
2. **Fonts.** Which fonts are offered, and are they licensed for web preview
   and for manufacturing?
3. **Proof of demand.** May the first store take real payments before a machine
   exists, with a stated longer delivery time, or should it collect
   pre-orders or a waiting list only?
4. **Payment provider.** Stripe supports TWINT and payouts to each shop's own
   account; is a US provider acceptable, or is a Swiss one (Payrexx, Datatrans)
   preferred?
5. **Returns.** Personalised goods are normally excluded from returns. What
   does the shop promise when a part arrives damaged or wrong?
6. **Finishing.** Are oiling, painting or mounting hardware options a shopper
   can choose, and so steps on the shop floor?
7. **First machine.** Which CNC and controller is planned? It decides the first
   Fusion post-processor and the first machine connector.
8. **Fusion licence.** Unattended generation needs a commercial Fusion licence
   on the shop PC; the terms for automated use need checking before CAM-3 is
   built.
9. **Hosted fee.** A fixed monthly amount, or a small share per order?
10. **Analytics for shops.** Does each shop see its funnel inside wryOS
    (AN-7), or is Mixpanel only for whoever hosts the installation at first?
11. **Shop languages.** Is the back office German only at first, or also
    English for the open-source community?

# GQ Claim Center

UPS / USPS / FedEx claim tracker with a Commercial Invoice generator — built as a single
self-contained HTML page with a LEGO-brick interface.

Enter the tracking number, pick the carrier, fill in the customer name / address and the
line items, and the app prints a Commercial Invoice **using the company's existing Word
template layout** (the file name follows the SOP: the template's `xx` is replaced with the
tracking number). Every claim is stored in the app so status, claim amount, file date and
paid date can be updated as the claim moves along.

## Running it

**Pure front-end. No backend, no install.** Open `index.html` in a browser.

To serve it on the network:

```bash
# plain static server
python -m http.server 8080

# or Docker
docker build -t gq-claim . && docker run -p 8080:8080 gq-claim
```

## Tabs

| Tab | What it does |
|---|---|
| ✏️ New Claim | A blank claim: carrier, tracking, order # / reference #, Bill to / Ship to, line items, tax / shipping / insurance, claim amount, file date, notes. Saving clears it for the next one |
| 📋 Claims | Every claim at a glance — **Edit opens the same form in a dialog** — tiles for open / paid / claimed total / recovered / average days to payment; search, filter by carrier and status, change status inline, export CSV |
| 🧾 Commercial Invoice | Pick a claim, adjust the file name, download Word (.doc) or PDF |
| ⚙️ Settings | Seller profiles (letterhead, address, logo, Market), file-name pattern, contact email / phone, backup and restore |

## The Commercial Invoice

The layout reproduces `UPS / USPS / FedEx_Commercial_Invoice_Templatexx.doc`:
centred **Invoice** heading, logo on the left, company block on the right, two grey rules,
Bill to / Ship to, `Tracking # / Order # / REFERENCE # / Market`, the
`Qty. / SKU / Title / Price / Net Price` table, then Subtotal / Tax / Shipping and handling /
Insurance / Total.

* **The tracking label follows the carrier** — FedEx prints `FedEx Tracking #`, UPS prints
  `UPS Tracking #`, USPS prints plain `Tracking #`.
* **The seller profile switches automatically** — USPS uses the K2Motor letterhead, UPS and
  FedEx use the GENIQUA letterhead (editable in Settings).
* **Zero amounts print as `$00.00`**, exactly like the Word template (switchable to `$0.00`).
* **The Word file is packed as MHTML**, so the logo and table borders survive in Word — the
  recipient opens a complete invoice.
* File name defaults to `{carrier}_Commercial_Invoice_Template{tracking}.doc` and is
  editable per claim. Placeholders: `{carrier}` `{tracking}` `{order}` `{claim}` `{date}`.
* **PDF is generated in the browser without any library** — Helvetica text plus the
  logo as an embedded JPEG — so it works offline and the text stays selectable.
  Long item lists flow onto extra pages with the header repeated.

### After the invoice is generated

The carrier only issues a claim number **after** the claim is submitted on their site, so the
app pops up a dialog right after the invoice is produced asking for the **Carrier Claim #**,
the file date, the status and the claim amount. Skip it with “Not filed yet — later” and fill
it in on the claim itself whenever the number arrives.

## Status flow

`Draft → Filed → In review → Docs required → Approved / Denied → Paid → Closed`

* A new claim starts with today's date in File date.
* Moving to **Filed / In review** fills in today's file date when it is still blank.
* Moving to **Paid** fills in the paid date and the paid amount (approved amount, or the
  claim amount when there is none).
* Status changes and invoice generation are written to that claim's activity log.

## Little things

* Type a 5-digit US ZIP and the City / State fill themselves (looked up once, then
  cached locally). No internet — it says so once and leaves the fields to you.
* The seller profile switches with the carrier, and the invoice's `Market:` line
  always follows the profile.

## Where the data lives

In this browser's `localStorage` — nothing is uploaded anywhere. Before switching computers
or clearing browser data, use **Settings → Backup & restore → Export all JSON**.

## Security note

The app stores **no carrier account credentials**. Keep UPS / FedEx logins wherever the
company already keeps them — not in the notes field and not in a JSON backup.

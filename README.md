# Auto Invoice Tracker (n8n template)

Every morning this n8n workflow looks through Gmail for invoices, receipts and order confirmations. Claude reads each one, and the details go into a Google Sheet.

## How it works

1. At 8am it searches Gmail for the last day's emails that have a PDF attached, or a subject like invoice, receipt, order confirmation, booking, itinerary or ticket. It also searches for the Dutch, German, Spanish and Italian words (factuur, rechnung, recibo, fattura and a few more).
2. If the email has a PDF, it reads the text from the PDF. Otherwise it uses the email body.
3. Claude checks whether it's a purchase. Shipping notices, newsletters and promos with no price get dropped.
4. For the ones that are, Claude pulls out the vendor, invoice number, invoice date, amount, currency, a category (Flight, Hotel, Taxi, Train, Meals, Shopping, Subscription or Other) and a short description. If something isn't in the email, the cell stays empty.
5. It adds a row to the sheet. Rows are matched on the Gmail link, so running it twice on the same email updates the row and doesn't add a duplicate.

## Setup

1. Import `auto-invoice-tracker.json` into n8n.
2. Connect your own accounts on these nodes. n8n will ask for them when you open each one:
   - Gmail, on "2. Find recent invoice emails"
   - Anthropic API key, on "Claude Sonnet 4.5"
   - Google Sheets, on "8. Add row to tracker"
3. In "8. Add row to tracker", replace `YOUR_SHEET_ID` with your own sheet. It writes to a tab called `Gmail invoices`, so either name your tab that or change the Sheet field. The tab needs these column headers: `Date Received`, `Vendor`, `Invoice Number`, `Invoice Date`, `Amount`, `Currency`, `Category`, `Description`, `Gmail Link`.
4. Click Execute workflow once to fill in the backlog. Manual runs look back 30 days, and the daily run looks back 1.
5. Publish the workflow to turn on the 8am run.

## Good to know

- The Gmail search casts a wide net and leaves the sorting to Claude. If too many non-receipts are getting through to Claude, narrow the `q` filter on the Gmail node.
- It only checks the first attachment for a PDF. If an email has several attachments and the invoice isn't first, it reads the email body instead.
- Text is cut at 15,000 characters before it goes to Claude, which keeps the cost down on long PDFs.
- Now and then you might see "Model output doesn't fit required format". Running it again usually fixes it.

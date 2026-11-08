# Gold Jewelry POS – Specification

This is the implementation reference I wrote for the development team that built the production version of Gold Jewelry POS, a point-of-sale system for a gold jewelry store. It covers the POS (counter) pages and describes each one: its layout, workflow and business rules, and, most importantly, why each decision was made. The clickable prototypes in this repository show every page in action. All customer names, phone numbers and figures are fictional.

## About the prototype

- The pages in this repository are standalone prototypes with sample data.
- They don't share data between pages, and they don't print.
- Receipt printing, and transactions from the other pages feeding into Cash Management, are requirements for the production system.

## General principles

- The workflow uses cards and a compact design to make every task at the counter easier.
- The Sale, Process Return, New Repair and Used Gold pages are not scrollable: everything fits on one screen.
- Two dedicated pages cover money that doesn't come from a sale:
  - **Used Gold** tracks the gold the store buys from customers.
  - **Cash Management** tracks the cash flowing in and out of the store.
- Payment methods are easy to select and auto-calculated. This ensures correct numbers.
  - Customer transactions (sales, returns, repairs) offer **Cash, Card, or Split**.
  - Used gold purchases offer **Cash or E-Transfer**.
  - Cash Management entries are **cash only**, because they record the cash box itself.
- Light and dark mode on all pages. The choice is remembered across pages.

## POS Dashboard

- The POS landing page has 6 action buttons in a 3×2 grid:
  - Start New Sale, Process Return, New Repair, Repair Pickup, Used Gold, Cash Management
- Each button has a gold bar to visually distinguish it, an icon, a label and a short description.
- Buttons are compact and spaced out so the page doesn't feel cramped on the iPad.
- Below the buttons, 4 stat cards:
  1. **Today's Sales:** blurred by default so staff can't see revenue numbers.
     - When tapped, a 4-digit PIN keypad appears. The correct PIN reveals the number for 5 seconds, then it auto-blurs again.
  2. **Store Cash Box:** shows the current balance and today's net change.
  3. **In Progress Repairs:** shows the count and the overdue count.
     - When tapped, opens a window listing all unfinished repairs with customer name, phone number, slot number, repair notes, cost and due date (overdue highlighted in red).
  4. **Used Gold On Hand:** shows total weight, average purity and current value.
- **Recent Transactions** section:
  - Each row shows customer name, receipt number, amount (green), payment method (color-coded tag: Cash green, Card purple, Split amber), date/time, and a View Receipt button.

## POS Sale Page

- Two-column layout, everything on one screen, no scrolling:
  - **Left:** Cart (items being sold). Trade-in and Exchange credit panels appear here under the cart when active.
  - **Right:** Customer, Discount, Payment, Summary, Save.
  - **Top bar:** light/dark toggle, Notes, Exchange, Trade-in, Scan, Back to POS.
- **Cart:** items are compact rows, not a wide table.
  - Each row shows product name, karat + type + SKU, qty, qty available, weight, $/gram and subtotal.
  - Qty, weight, $/gram and subtotal are all editable directly in the row. Editing the subtotal back-calculates $/gram.
  - An "Apply to all [karat]K" button appears when focusing any $/gram input.
- **Add Item modal:** Product name, Product type (Gold/Silver), Karat, Weight, Quantity, Category, Qty available.
- **Customer selection:** green highlight when a customer is selected; search + "New customer" when not. It is obvious at a glance whether a customer is attached.
  - The selected card shows a tier badge by lifetime spend (metallic colors; VIP shines and glows), plus item count and lifetime spend:
    - VIP = $100k+ (Jesse Simmons)
    - GOLD = $50k+ (Morgan Hayes)
    - SILVER = $25k+ (Taylor Ramos)
    - BRONZE = $8k+ (Skyler King)
    - REGULAR = under $8k (Devon Bailey)
  - The new-customer phone field auto-formats as you type (e.g. 780-555-0123).
- **Payment:** Cash (green), Card (purple), Split (amber).
  - Cash or Card: one tap, the amount auto-fills and the other field is disabled.
  - Split: both fields are enabled; edit one and the other auto-calculates the remainder.
- **Discount:** compact row with a $ or % toggle.
- **Trade-in:**
  - The "Trade-in" button in the top bar opens a modal with karat rows (24K–10K), gold-colored badges, and editable weight and $/gram.
  - "Add karat row" adds custom karats (9K, 19K, etc.).
  - No seller (the customer is already selected) and no payment method (it's a credit).
- **Exchange** (return a past purchase as credit):
  - The red "Exchange" button in the top bar opens a modal to search the original transaction by name, phone or receipt #.
  - Staff pick which items to return (qty selector when more than one was purchased), set each item's destination (Return to stock or Melt), and a return reason. All are required before the credit applies.
- Trade-in and Exchange feed the same net total. Three scenarios, handled live:
  - **Credit < cart total:** the customer pays the difference via cash/card/split. The payment section is visible and the button reads "Save sale — $X".
  - **Credit > cart total:** the store pays the customer the difference (cash out). The payment section is hidden, a red line shows "Cash owed to customer — $X", and the button reads "Save sale — pay customer $X".
  - **Credit = cart total:** even exchange. "Even exchange — no payment needed", no payment section, and the button reads "Save sale — even exchange".
- **Why trade-ins and exchanges happen on the sale page, not a separate page:** one transaction, one receipt, and the cash counter only sees the net cash that actually moved. Splitting it across two pages creates two records the system can't link.
- **Summary:** Subtotal, Trade-in credit, Exchange credit, Discount, Total (the label flips to "Customer pays" when money is owed).
- **Save button:** shows the total and stays disabled until the cart has items and the total is valid (its label changes per the scenarios above).

## Process Return

- Two-screen system: Search → Process Return.
- **Screen 1 (Search):**
  - Three filter tabs: Name, Phone, Receipt #.
  - Search results appear as cards.
  - Each card shows a readable date (Apr 17, 2026), item description, total amount and receipt number.
- **Screen 2 (Process Return):**
  - Two columns: items on the left; transaction details, refund summary and submit on the right.
  - Transaction details at the top of the right panel: customer name, phone, date, original total, payment method.
  - A View Receipt button in the transaction summary pulls up the original receipt.
  - Items are displayed as expandable cards.
  - Clicking the checkbox on an item expands it, showing:
    - Return reason (dropdown: Changed mind, Wrong item, Defective, Size issue, Gift return, Other)
    - Condition (pill buttons: Good, Needs polish, Damaged)
    - Return option (Return to stock or Melt, displayed as two large toggle cards with icons)
    - Qty to return, if they bought more than one of the same item
  - Unchecked items stay collapsed as a single clean row, reducing visual clutter.
  - The running refund summary on the right updates live as items are selected.
  - The submit button stays disabled with a validation message until every selected item has a reason, condition and return option.
- **Refund Payment modal:**
  - Clicking Process Return opens a modal with Cash, Card and Split tabs.
  - There is no default selection: staff must choose the refund method.
  - Split auto-calculates: type the cash amount and the card amount fills the remainder automatically.
  - The confirmation logs exactly how the refund was paid, e.g. "Cash $800 / Card $229.30".
  - This data feeds into the cash counter to track every dollar in and out.
- Card-based layout throughout, so everything fits on an iPad without horizontal scrolling.

## New Repair (Repair Intake)

- Two-column layout, no scrolling:
  - **Left:** Repair notes (a big text area that takes most of the space, since it's the primary input) + Pricing.
  - **Right:** Customer selection + Slot assignment + Due date + Receiver checkbox.
- **Repair notes** are mandatory and printed directly on the receipt.
  - Format, one item per line:
    ```
    Ring to size 7 (weight 6.34)
    Weld Bracelet (weight 14.8)
    ```
  - **Store copy:**
    ```
    Jesse Simmons
    780-555-0101
    ------------
    Ring to size 7 (weight 6.34)
    Weld Bracelet (weight 14.8)
    ```
  - **Customer copy:** the customer's receipt shows the store details, repair code, drop-off date, due date, the repair notes, the cost, and the payment status (paid, or the amount due at pickup).
- **Pricing:** a Cost input and 4 payment tabs: Unpaid, Cash, Card, Split.
  - A payment status line confirms the choice: "Paid in full — cash", "Fully paid — split $X cash / $Y card", or "Unpaid — $45 due at pickup".
- **Slot:** auto-assigns the next available number (skipping occupied slots), displayed as a blue badge showing the slot number.
- **Due date:** defaults to 3 days from today, so staff don't have to pick it every time.
- **"Receiver is different from customer" checkbox:** when checked, a name input appears below.
- **Customer selection** uses the same green highlight system as all other pages:
  - When selected: green border, green avatar with initials, green phone number.
  - When not selected: search input + "New customer" button.
  - The "New customer" phone field auto-formats as you type (e.g. 780-555-0123), same as the sale page.
- **Save button:** shows the cost and stays disabled, listing the specific missing fields (customer, notes, due date), until all required fields are filled.

## Repair Orders (Kanban Board)

- A 2-column kanban board: In Progress → Done.
  - Each column has a colored accent bar (amber for In Progress, green for Done) and a count.
- Each repair is a card showing:
  - repair code, slot number, customer name, phone, repair notes, cost
  - due date (color-coded: red overdue, amber due today, gray upcoming)
  - payment status (green "Paid — [method]", red "Unpaid")
  - on Done cards, a notify badge (📞 Notified or ⏳ Awaiting call), plus a "Called X days ago" line once notified
- **Stats bar** across the top: In Progress count, Done count, Awaiting call count, and total Unpaid ($).
- **Card actions** change by status:
  - In Progress: Mark ready + Details + Edit
  - Done / awaiting call: Notified + Picked up + Details + Edit
  - Done / notified: Picked up + Again + Details + Edit
  - "Mark ready" opens a prompt, "Did you notify the customer?" (Yes, notified / Not yet), sending the repair to Done as either notified or awaiting call.
  - "Again" re-calls the customer and updates the notified timestamp.
- **View Details** opens a modal with everything: customer info, repair code, slot, cost, payment, due date, drop-off date, a days-in-shop counter, and a timeline showing every stage the repair went through.
- **Edit modal:** change notes, cost, due date and payment status, plus a Cancel Repair button.
- **Mark Picked Up** on an unpaid repair opens a payment modal (Cash, Card, or Split with auto-calc) before completing. Already-paid repairs skip the modal and go straight to completed.
- **Search** works by name, phone or repair code.
- **Status filter:** All active / In progress / Done / Awaiting call. Picking one hides the other column and the remaining one fills the full width; "Awaiting call" narrows Done to repairs not yet called.
- **Active/Completed tabs:** Completed shows picked-up and cancelled repairs, with search.

## Used Gold Purchase

- A standalone page for when customers sell gold without buying anything.
- **Left side:** karat rows (24K, 22K, 21K, 18K, 14K, 10K) pre-filled with default buy prices.
  - Each row has a gold-toned badge that goes from rich gold (24K) to muted bronze (10K).
  - Weight and price/gram are both editable.
  - "Add karat row" adds custom karats (9K, 12K, 19K).
- **Right side:** Seller selection (green highlight, same system), Payment method (Cash or E-Transfer), Notes, and a Summary with a per-karat breakdown.
  - The "New seller" phone field auto-formats as you type (e.g. 780-555-0123), same as the sale page.
- Subtotals are red because this is money **leaving the store**, whether as cash from the store box or an e-transfer to the seller.
- The Save button says "Pay $2,855.50 — cash out" to make the direction of money clear.
- E-Transfer is offered because the store sometimes pays sellers electronically.

## Cash Management

- **Store cash box:** shows the current balance and today's in/out totals.
  - Cash in: sales, repair payments, manual cash in.
  - Cash out: used gold purchases, refunds, expenses, and cash paid to customers when a trade-in or exchange credit exceeds the cart total.
- **2 action buttons:**
  - **Add Expense:** category dropdown (Shipping, Salaries, Donations, Supplies, Rent, Owner Withdrawal, Transfer to supplier, Other), an amount, and a mandatory notes field ("What is this expense for?").
    - An expense can't be larger than the cash in the store box.
    - Owner Withdrawal requires a 4-digit PIN before processing (1234 in the demo).
      - The PIN isn't about preventing the owner from taking money (it's their store).
      - It's about preventing employees from logging fake withdrawals under "Owner Withdrawal" to cover theft.
    - All other categories process immediately, without a PIN.
  - **Manual Cash In:** for logging cash from sales or repairs that weren't auto-logged (source: Sale (cash portion), Repair payment, or Other).
- **Transaction log:**
  - Searchable by description, notes or category.
  - Filterable by type (Cash in / Cash out).
  - Each row shows the date, description with notes, amount (green for in, red for out) and category.
- **Corrections, not edits:**
  - Past entries are never editable, because that would destroy the audit trail.
  - Instead, each entry has Correct and Void buttons.
  - **Correct:** opens a modal showing the entry and its current amount, and lets staff enter the corrected amount and a required reason.
    - The system calculates the difference from the entry's current amount (the original plus any earlier corrections) and creates an adjustment entry linked to it (amber left border in the log).
    - A live preview shows exactly what will happen, e.g. "Difference: −$150.00 → will remove $150.00".
    - The original entry stays untouched.
    - Correction rows can't be corrected or voided themselves. To change an amount again, staff correct the original entry, so every adjustment stays tied to the entry it belongs to.
  - **Void:** requires a reason and creates one reversal entry (red left border) for the entry's full current amount, including all its corrections.
    - The original entry and its corrections get strikethrough text and a red "VOIDED" tag, so nothing is left counting toward the balance.
    - All entries stay in the log permanently.
    - A voided entry can't be corrected or voided again.
  - **Today's in/out totals** count each live entry at its current amount, corrections included. Voided entries and their reversals are left out, so a voided transaction nets to zero.
  - A void fixes a record that should never have been entered. A refund is the way to record money actually leaving the store.
  - This is standard accounting practice: you never rewrite history, you record corrections.

## Planned improvements

- Add time filters to the Cash Management transaction log.

# Bahi (बही) — inventory, accounts and billing on Google Sheets

Bahi has two parts:

- **The Google Sheet is the database.** Every invoice, bill, receipt, item and party saved in the Bahi page is written into tabs of the Sheet.
- **The web page (`web/index.html`) is what staff use.** It talks to a small Apps Script attached to the Sheet.

Changes flow both ways:

- **From the page to the Sheet:** a saved entry appears in the Sheet straight away.
- **From the Sheet to the page:** if someone edits an item rate, adds a party, or marks an invoice cancelled directly in the Sheet, every open Bahi page picks it up within about 5 seconds.

**Version 1.1.0** adds:
- A Hindi/English switch and System/Light/Dark themes.
- Financial years you add as you need them.
- Instant opening from a local copy on each device.
- Security hardening throughout.
- "Powered by CoreStack Technologies" with the version, build number and build date. These appear at the bottom of the side menu and the sign-in page; click them for details.

```
bahi-sheets/
├── apps-script/Code.gs          paste into the Sheet's Apps Script editor
├── apps-script/appsscript.json  optional: settings file (time zone, web app access)
├── web/index.html               the Bahi page
├── web/config.js                paste your web app address here
└── README.md
```

---

## 1. Create the Sheet and the script / शीट और स्क्रिप्ट बनाएँ

1. Create a new, blank Google Sheet, for example "Bahi - Gita Press".
2. In the Sheet, open **Extensions → Apps Script**.
3. Delete the sample code in `Code.gs`, paste all of `apps-script/Code.gs`, and click **Save**.
4. **Optional:** in **Project Settings**, tick **Show "appsscript.json" manifest file**, then replace that file's contents with `apps-script/appsscript.json`. This sets the time zone to India.

## 2. Run setup once / सेटअप चलाएँ

1. In the function list at the top of the editor, choose **setup** and click **Run**.
2. Google asks for permission the first time. Choose your account, then **Advanced → Go to (project name) → Allow**. The script only works inside this Sheet.
3. When it finishes, the **Execution log** shows:
   - your **owner email**, which is the Google account you are logged in with
   - a **temporary password**, which is also in the **Users** tab until you first sign in

Setup does three things:

- **Creates the tabs:** Settings, Users, Items, Parties, Ledgers, Openings, Vouchers, Lines, Entries, Counters, Audit.
- **Formats the columns,** so codes like `001` stay as text.
- **Adds two triggers,** so edits made directly in the Sheet reach the page.

After this, the Sheet has a **Bahi** menu with *Set up or repair sheets*, *Rebuild Lines and Entries tabs* and *Sign everyone out*.

## 3. Deploy the web app / वेब ऐप डिप्लॉय करें

1. In the Apps Script editor, click **Deploy → New deployment**.
2. Click the gear next to *Select type* and choose **Web app**.
3. Set:
   - **Execute as:** Me
   - **Who has access:** Anyone
4. Click **Deploy** and copy the **Web app URL**. It ends in `/exec`.

"Anyone" only means the page can reach the script. Every request still needs a Bahi sign-in, and access levels are checked by the script itself.

## 4. Connect and host the page / पेज जोड़ें और ऑनलाइन करें

1. Open `web/config.js` and paste the URL:

   ```js
   window.BAHI_CONFIG = {
     scriptUrl: 'https://script.google.com/macros/s/AKfy..../exec'
   };
   ```

2. Put `index.html` and `config.js` (the two files together) somewhere people can open them. Any one of these works:
   - **GitHub Pages:** upload both files to a repository, then go to **Settings → Pages → Deploy from a branch**.
   - **Cloudflare Pages or Netlify:** drag the `web` folder onto the upload area.
   - **Any web server** at Gita Press: copy the two files into a folder it serves.
   - **Directly from a computer:** you can usually just double-click `index.html`. If a browser refuses to connect from a local file, use one of the hosted options instead.

## 5. First sign-in / पहली बार लॉग-इन

1. Open the page and sign in with the owner email and the temporary password from step 2. You will be asked to choose your own password.
2. Open **Company** and fill in the name, Hindi name, GSTIN, state, bank details and invoice prefix.
3. Load your masters:
   - **Inventory → Import CSV:** paste your item list from Excel or Tally.
   - **Parties → Import CSV:** paste your parties.
   - **Ledgers:** add your bank accounts and expense heads.
   - **Opening balances** (on the Ledgers page): enter cash in hand and capital.

   You can also type or paste masters straight into the Items, Parties, Ledgers and Openings tabs of the Sheet.
4. Check **Trial balance**. It should show no "Difference in opening balances" row before billing starts.

## 6. Add staff / स्टाफ जोड़ें

**In the page:** **Company → People and access → Add a person**. Enter their name, email, access level and a temporary password, then give them the password yourself. At first sign-in they choose their own.

**In the Sheet:** add a row in **Users** with Email, Name, Role (`staff`, `viewer`, `admin` or `owner`), Active `TRUE`, and a temporary password in **Set Password**. The temporary password is replaced by a secure hash the first time they sign in.

Only an **owner** can create or change the structure of the books. Admins run the books day to day but cannot create years, people or opening balances.

| | Owner | Admin | Staff | Viewer |
|---|---|---|---|---|
| Add financial years, do the year-end carry forward | ✓ | | | |
| Change company details and the invoice prefix | ✓ | | | |
| Set opening balances (items, parties, ledgers, cash, capital) | ✓ | | | |
| Close and reopen the books; change entries in a closed period | ✓ | | | |
| Add people, change access, turn access off | ✓ | | | |
| Reset passwords | anyone | staff and viewers only | | |
| Delete unused items, parties and ledgers | ✓ | ✓ | | |
| Make, edit and cancel entries; add items, parties and ledgers | ✓ | ✓ | ✓ | |
| See everything | ✓ | ✓ | ✓ | ✓ |

- **No access:** the person cannot sign in. In the Sheet, this is Active = FALSE.
- **Enforced by the script:** the Google Sheet script checks every one of these rules itself, so a page that has been tampered with still can't get past them.
- **Imports by admins or staff:** opening quantities and balances in the file are skipped.
- **The same summary in the app:** it appears under **Company → People and access**.

Staff don't need access to the Google Sheet at all; they only use the page. **Share the Sheet itself only with the people who look after the books.** Anyone who can edit the Sheet can change anything in it directly, including the Users tab.

## 7. Working in the Sheet directly / शीट में सीधे काम

| Tab | What it holds | Edit in the Sheet? |
|---|---|---|
| Settings | Company details, invoice prefix, `books_closed_until`, `financial_years` | Yes |
| Users | People and access | Yes: Email, Name, Role, Active, Set Password. Never edit Password Hash or Salt |
| Items | Items | Yes. Rows you add get an ID automatically |
| Parties | Customers and suppliers. Type is `Customer` or `Supplier` | Yes |
| Ledgers | Bank accounts, expense and income heads. Group uses the names shown in Bahi, e.g. `Bank accounts`, `Indirect expenses` | Yes |
| Openings | Opening stock and balances for each financial year. **Kind** is Item, Party, Ledger or System. **Ref** is the item code, party name, ledger name or ID | Yes |
| Vouchers | One row per entry | Only **Date, Status** (`active` or `cancelled`), **Narration, Ref No, Ref Date**. Other columns are written by Bahi |
| Lines | One row per item line, for filters and pivot tables | No. Rebuilt automatically |
| Entries | Double-entry postings (ledger, debit, credit) for every entry, for ledger pivots | No. Rebuilt automatically |
| Counters | Last number used in each series | No |
| Audit | Every create, edit, cancel and restore, with the version it replaced | No |

**Rules for vouchers in the Sheet:**

- Don't delete voucher rows. Set Status to `cancelled` instead, so the number stays in the books as GST rules expect.
- Don't type new vouchers into the Sheet by hand. Enter them in the page, which works out GST, totals and numbering.

## 8. How it keeps the books safe / सुरक्षा

- **Live updates.** Each open page checks the Sheet every 4 seconds and fetches only what changed. The top bar shows **Live** while connected.
- **Numbering.**
  - Invoice and voucher numbers are given by the script at the moment of saving, one save at a time.
  - Numbers are therefore sequential, gap-free and never duplicated, even when several people bill together.
  - Cancelled entries keep their number.
- **Editing.**
  - If two people edit the same entry, the second save is refused with a message; nobody's change is silently overwritten.
  - **History** on any entry shows who created, changed, cancelled or restored it.
- **Close the books.** In **Company → Close the books**, an owner sets a date. After that, nobody except an owner can add, change or cancel entries on or before it.
- **Backups.**
  - Google keeps the Sheet's version history under **File → Version history**.
  - **Company → Download full backup** gives you a second copy.

## Updating the script later

1. Paste the new `Code.gs` and click **Save**.
2. Go to **Deploy → Manage deployments**, click the pencil, choose **Version: New version**, then **Deploy**.

This keeps the web app URL the same, so `config.js` doesn't change. Run **Bahi → Set up or repair sheets** if the new version adds columns.

## Financial years / वित्त वर्ष

- **A new install opens only the current financial year,** for example FY 2026-27. The top bar shows "Company not set up" until Company details are saved, and the year selector shows just that year.
- **To add another year,** open the year selector at the top and choose **+ Add a financial year…**, or go to **Company → Financial years → Add a financial year**. Only an owner can do this. You can add the coming year, or earlier years for old records. Years can be added but not removed.
- **Entries can only be saved in years that have been added.** The Google Sheet script enforces this, not just the page.
- **Year end, carry forward** adds the next year automatically if it isn't there yet.
- **In the Sheet,** the list is the `financial_years` row in the Settings tab, e.g. `2025-26, 2026-27`.
- **When a new year starts on 1 April,** the dashboard shows a reminder. Owners get an **Add FY** button; everyone else is asked to contact an owner.

## Language and theme / भाषा और थीम

- **Language:** the **हिंदी / English** button in the top bar (and on the sign-in page) switches the whole interface. Names and data you type are never changed. Printed invoices stay bilingual either way.
- **Theme:** the theme button cycles **System → Light → Dark**. On **System** (the default), Bahi follows the phone or computer's own light/dark setting and switches with it.
- **Per device:** both choices are remembered on each device.

## Speed / गति

- **Instant opening:** each device keeps a local copy of the year it's working in. The page opens straight away and fetches only what changed since last time.
- **Only changes travel:** after the first load, each 4-second check fetches only changed rows, and the screen redraws only when something actually changed.
- **Faster saves:** a save looks up its own row instead of reading the whole Vouchers tab.
- **Small page:** about 78 KB when compressed, which is what most hosts send.

## Security / सुरक्षा

**In the page:**
- A strict Content Security Policy. Only the page's own code can run, and it can only talk to Google's script address.
- It refuses to open inside another website's frame.
- It sends no referrer.
- Everything shown on screen is escaped, so text typed into names or narrations can't run as code.
- CSV exports are guarded against spreadsheet formulas.

**Sign-in:**
- Passwords are stored only as salted, repeatedly hashed values (1,000 rounds). Older hashes upgrade automatically at the next sign-in.
- 8 wrong passwords lock that email for 15 minutes.
- Changing a password signs that person out everywhere else.
- Sign-ins last 30 days.
- **Sign out** wipes the local copy from the device.

**In the script:**
- Every request is checked for access level.
- Totals and GST are recalculated by the script, so a tampered page can't save wrong figures.
- Inputs are length- and type-checked.
- Text is stored as plain text in the Sheet, so it can't become a formula.
- Numbers are given one save at a time.
- Every change is kept in the Audit tab.

**Your part:**
- Host the page over **HTTPS**; GitHub Pages, Netlify and Cloudflare all do this.
- Share the Google Sheet only with the people who manage the books.
- Use **Bahi → Sign everyone out** in the Sheet if a device is lost.

## Updating from version 1.0 / पुराने संस्करण से अपडेट

1. Paste the new `Code.gs` into the Apps Script editor and click **Save**.
2. Run **setup** once. It adds the `financial_years` row, using the current year plus any year that already has entries, and keeps all data.
3. Go to **Deploy → Manage deployments**, click the pencil, choose **Version: New version**, then **Deploy**. The URL stays the same.
4. Replace `index.html` on your host. Keep your own `config.js`.

Existing passwords keep working. If the page and the script are on different versions, owners see a reminder on screen.

## Limits worth knowing

- **Update speed.** Updates arrive within about 5 seconds, not instantly, because the page checks the Sheet in turns. Each save takes about 1–2 seconds, because Google Sheets writes are slower than a database.
- **Team size.** Google allows around 30 script runs at the same moment per account, which comfortably covers about 15–20 people with the page open. Larger teams should use the Supabase version.
- **Sheet size.** A Google Sheet holds up to 10 million cells. Bahi uses roughly 100 cells per invoice across Vouchers, Lines and Entries, so one Sheet lasts several years at tens of thousands of invoices a year. Start a fresh Sheet when needed, carrying balances forward.
- **One year at a time.** The page loads one financial year at a time and works out stock, ledgers, GST and reports in the browser.

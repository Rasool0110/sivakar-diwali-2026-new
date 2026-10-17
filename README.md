# Sivakar Diwali Shop — Build from Scratch

This folder is a fresh, separate project for the Sivakar crackers shop. It does not require editing your old site or old database.

## What this build does

- Customer catalogue with product photos/videos, search, category filters, prices, discounts, quantity controls and cart totals.
- Admin sign-in and product management: add/edit/hide/delete products, stock, selling unit, prices, discount, media uploads and private cost/margin estimate.
- **Real order records in Supabase.** When a customer places an order, the database re-checks the latest stock, records the order and deducts/reserves stock in one transaction.
- **Overselling protection.** If two customers try to buy the last unit, the database lock/check means only the order that can be fulfilled succeeds. The other customer receives a stock error.
- Live customer catalogue stock refresh when product stock changes, plus stock refresh after checkout.
- Admin order list with customer details, items, totals, status, Confirm, and Cancel & Restore Stock actions.
- Customer is redirected to WhatsApp with a message containing the saved order reference and server-calculated prices. Customer must press **Send** in WhatsApp.

Orders start as **Pending Confirmation**. Creating the order reserves/deducts stock even if the customer never presses Send in WhatsApp; this is necessary because a website cannot detect whether the WhatsApp message was actually sent. Check pending orders regularly and cancel requests that are not genuine; the Cancel action restores stock. No online payment is taken.

## Part 1 — Create the new Supabase project

1. Go to <https://supabase.com/> and sign in.
2. Choose **New project**. Give it a name such as `sivakar-diwali-2026-new`.
3. Set a strong database password and save it in a password manager. Choose a region close to your customers, if available.
4. Wait for the project to finish provisioning.
5. Open **Project Settings → API** (the exact menu label may vary). Copy the **Project URL** and the **anon/public/publishable key**.
6. Never use a `service_role` or secret key in website code. The public key is expected to be visible in the browser; the database security policies protect data.

## Part 2 — Create all database tables and security rules

1. In the new Supabase project, open **SQL Editor**.
2. Choose a new query.
3. Open this project's `supabase/schema.sql` in a text editor, select all, copy, and paste it into SQL Editor.
4. Click **Run** and wait for successful completion. This creates the product catalogue, private cost table, admin membership, media bucket policies, order tables, stock-safe order functions and live-stock publication setup.
5. If SQL reports an error, stop and fix it before publishing. Do not paste the SQL into your old Supabase project by mistake.

## Part 3 — Create your admin login

1. In Supabase, open **Authentication → Users**.
2. Choose **Add user / Create new user**. Enter the shop owner's email and a strong password. If asked, confirm/auto-confirm the user so you can sign in with that password.
3. Copy that user's **UUID/ID**.
4. Return to **SQL Editor** and run the following after replacing the placeholder with the real UUID:

   ```sql
   insert into public.app_admins (user_id)
   values ('PASTE_THE_ADMIN_USER_UUID_HERE');
   ```

5. Keep the UUID inside the SQL statement in quotes. This grants admin privileges only to that Auth user. Do not share the admin password.

## Part 4 — Connect the website to Supabase

1. Open `supabase-config.js` in a text editor such as VS Code or Notepad.
2. Replace the placeholder values:

   ```js
   window.SIVAKAR_CONFIG = {
     SUPABASE_URL: 'https://YOUR-PROJECT-REF.supabase.co',
     SUPABASE_ANON_KEY: 'YOUR_PUBLIC_ANON_OR_PUBLISHABLE_KEY',
     SHOP_WHATSAPP_NUMBER: '91XXXXXXXXXX',
     SHOP_NAME: 'Sivakar'
   };
   ```

3. Use your actual Project URL and anon/public/publishable key from Part 1. Keep the quote marks and commas.
4. For `SHOP_WHATSAPP_NUMBER`, use the shop's WhatsApp number with country code and digits only. For India, start with `91` then the 10-digit mobile number. Do not include `+`, spaces or hyphens. The example above is a placeholder, not a real number.
5. Save the file.

## Part 5 — Test locally before publishing

You need Python or another static web server because browser modules and Supabase work more reliably over `http://localhost` than by opening the HTML file directly.

### Option A: Python

1. Install Python from <https://www.python.org/downloads/> if it is not installed. During Windows installation, enable **Add Python to PATH**.
2. Open the project folder in File Explorer.
3. Click the folder address bar, type `cmd`, and press Enter to open Command Prompt in that folder.
4. Run:

   ```bash
   python -m http.server 8000
   ```

   If `python` is not recognized, try `py -m http.server 8000`.
5. Open <http://localhost:8000> for the customer shop.
6. Open <http://localhost:8000/admin.html> for the admin page.
7. Leave the Command Prompt window running while testing. Press `Ctrl+C` to stop the server later.

### Test the admin page

1. Sign in using the Auth user created in Part 3.
2. Add one clearly named **test product** with a small stock quantity (for example, 2), a test price, and an image. Do not use a real customer's order for testing.
3. Save the product and confirm it appears in the inventory table.
4. Open the customer shop and verify the product, unit price, discount, image and stock are correct.
5. In admin, edit the product and test the discount set to `0` and then to a positive number.
6. Upload videos only if needed; compress them first. Each image/video must be 50 MB or smaller.

### Test order and stock behavior

1. In the customer shop, select one test product and quantity `1`.
2. Review the cart, enter your own test name/mobile/address, then click **Place order on WhatsApp**.
3. The site should save the order, reserve/deduct stock and then open WhatsApp with the order reference. You do not need to send the message during testing.
4. Return to the admin page and refresh **Customer orders**. The order should appear as **Pending Confirmation**, with the same reference and total.
5. Check inventory: stock should have decreased by the quantity ordered. Another browser/customer view should show the updated stock; the page subscribes to live product changes.
6. In admin, use **Cancel & Restore Stock** on the test order. Confirm that stock increases back by the ordered quantity and the order status becomes cancelled.
7. Test insufficient stock: set stock to `1`, try to order quantity `2`, and verify the customer UI prevents it. For a true database-level check, try ordering the last unit from two separate browsers; only one request should succeed.
8. If any step fails, do not publish yet. Check the message shown on screen and Supabase **Logs**.

## Part 6 — Publish using a new GitHub repository

1. Sign in to <https://github.com/> and create a **new public repository** such as `sivakar-diwali-shop-new`. Do not overwrite your old website repository.
2. Upload the files and folders from this project: `index.html`, `admin.html`, `supabase-config.js`, `README.md`, and the entire `supabase` folder. Do not upload passwords or any service-role key.
3. In repository **Settings → Pages**, choose **Deploy from a branch**, select the main branch and root (`/`), then save.
4. Wait for GitHub Pages to publish. Open the provided Pages URL and add `/admin.html` to test the admin page.
5. Make sure the public customer page and admin sign-in work at the hosted URL. Test an order and cancellation again before sharing the site with customers.
6. The `supabase-config.js` public key can be seen by site visitors; that is normal. The service-role key must never be put there.

## Product and inventory rules (Option A)

Each product has one selling unit: Box, Piece, Packet or Bag. Customers order in that unit only. For a Box product, `stock_quantity` means the number of full boxes available; `pieces_per_box` is only an internal planning multiplier. Example: 12 boxes × 10 pieces per box = 120 internal units, but customers still order full boxes. For Piece/Packet/Bag products, internal units equal stock quantity.

- Discounted price = actual price × (1 − discount percentage / 100), rounded to 2 decimals by the order function.
- A zero discount does not display a discount badge on the customer page.
- Cost price and margin are stored in `product_costs`, separate from public product data. Margin is only an estimate before rent, transport, staff, packaging, taxes, damaged stock and other overheads.

## Order and stock rules

- Stock is reserved/deducted when the customer clicks Place Order and the database successfully creates the order, before WhatsApp opens.
- Prices and stock are re-checked by the database at checkout. The browser's displayed price/stock is not trusted for final order totals.
- A new order is `pending_confirmation`. Confirm only after you have checked the WhatsApp request/customer details.
- If a customer never sends the WhatsApp message or abandons the request, cancel it from the admin page to restore stock.
- Cancelling an order restores its stock once. Repeated cancellation does not repeatedly add stock.
- Do not cancel a fulfilled order without checking inventory. If you cancel a confirmed but unfulfilled order, the reserved stock is restored.
- WhatsApp cannot tell the website whether the customer actually pressed Send. The order may therefore appear in admin even if no WhatsApp message was sent.
- There is no online payment or automatic delivery fee calculation.

## Troubleshooting

- **“Setup required”**: check the URL and public key in `supabase-config.js`, including quotes and commas.
- **Admin says not authorized**: confirm you inserted the correct Auth UUID into `public.app_admins`.
- **Catalogue is empty**: add a product in admin and ensure it is published and has stock.
- **Media upload fails**: confirm the schema ran successfully, the admin user is authorized, the file type is supported and file size is below 50 MB.
- **Order RPC missing / function not found**: rerun the full `supabase/schema.sql` in the new project.
- **Live stock does not refresh**: reload the page; checkout always revalidates stock on the server even if live updates are delayed.
- **WhatsApp does not open**: check `SHOP_WHATSAPP_NUMBER`; use digits only with country code.

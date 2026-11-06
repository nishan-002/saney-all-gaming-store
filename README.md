# SANEY GAMING STORE

A React/Vite storefront with an Express API and PostgreSQL database for game top-ups. Payment proofs are always pending until an administrator verifies them; uploading a screenshot never marks an order as paid.

## Requirements

- Node.js 20 or newer
- PostgreSQL 14 or newer

## Setup

1. Create a PostgreSQL database named `saney_store`.
2. Copy `.env.example` to `.env`, set `DATABASE_URL`, and replace `JWT_SECRET` with a long random secret.
3. Install dependencies with `npm install`.
4. Create the tables and initial catalog with `npm run db:init`.
5. Create the first admin interactively with `npm run admin:create` (the password is prompted securely in the terminal and is never stored in source files).
6. Start the client and API with `npm run dev`. The storefront is at `http://localhost:5173`; the API listens on port 4000.

When updating an existing database with schema changes or the starter catalog, run `npm run db:init` again. The catalog includes mobile games, PC games, and gift cards; entries without confirmed pricing are shown as contact-for-price and cannot be ordered until an administrator sets a price.

If you forget the store admin sign-in email or password, run `npm run admin:recover` in an interactive terminal connected to the configured database. It lists admin accounts and securely prompts for a replacement email and password; passwords are not echoed or saved in source files. This recovers the store login only, not a Google/Gmail account.

Admins can add or edit products and set regular and offer prices in **Admin → Products**. Offer prices must not exceed the regular price. Manage the auto-scrolling image carousel at the top of customer pages in **Admin → Banners**; upload offer or notice banner images, add optional internal links, set display order, and hide or show banners without deleting them. Short scrolling text notices can be managed in **Admin → Ticker**. Banner images use the existing public image upload storage.

Change the admin sign-in email and password in **Admin → Settings**. Both changes require the current password. The separate **Support email** field in Store settings controls the customer-facing contact email; changing it does not change the admin login.

## Online hosting (Render)

The root `render.yaml` defines a paid Render web service, PostgreSQL database, and persistent disk for uploaded banners and private payment proofs. Push this project to a private GitHub repository, then in Render choose **New → Blueprint**, connect the repository, review the current prices, and approve deployment. Render generates `JWT_SECRET`, connects the database, initializes the schema before deployment, and mounts persistent storage. Do not approve the Blueprint until you have reviewed its charges.

After deployment, create the first administrator with `npm run admin:create` from a trusted local terminal configured with the Render database's external connection URL. Set that URL only in your local environment; never commit it. Use a new strong admin password and rotate any credentials that have ever been shared or stored in an example file. Keep the Render-generated `JWT_SECRET` private.

The app supports manual eSewa/Khalti/bank/QR orders. These orders stay pending until an administrator verifies payment; uploading proof does not settle payment. Automated provider verification is not configured. Before inviting customers, verify the live site over HTTPS, test an order end to end, confirm private proof access as an admin, and ensure the production database has backups.

Production: run `npm run build`, then `npm start` with `NODE_ENV=production`; Express serves the built client. Place the app behind HTTPS and configure a production database and persistent upload storage.

## Payments

Manual eSewa/Khalti/bank/QR orders can be submitted with a payment reference and optional proof image. They remain pending until an admin reviews them in `/admin`; proof files are private and admin-only. No screenshot or client callback approves an order. `server/payments/adapter.js` defines a server-side provider verification contract that checks provider settlement, order reference, and exact NPR amount. It is intentionally not connected to a live provider yet: complete the provider-specific signed/lookup call in that adapter after merchant onboarding, and only then wire its verified result to the protected order update path. `ESEWA_MERCHANT_CODE` and `KHALTI_SECRET_KEY` are server-only placeholders; never put them in Vite variables or browser code.

## Included workflows

- Product search, category filtering, product details, a persistent multi-item cart, checkout, and order-progress timelines.
- Product sorting, in-stock/offer filters, customer wishlists, and editable product delivery, region, and FAQ details.
- Customer registration/login, order history, notification inbox, and review submission.
- Protected `/admin` for products, searchable payment review with quick approve/reject actions, order fulfillment, announcements, notifications, reviews, and store settings.
- Admin overview analytics for verified sales trends, average approved order value, low stock, and top-selling products.
- Admin activity log entries for product changes and order payment/status updates.
- Announcement ticker, editable social/contact/payment instructions, maintenance setting, and image uploads.

The database initializer creates the order-item table, wishlist and admin audit tables, and product detail fields; it also migrates existing single-product orders. Rerun `npm run db:init` when updating an existing database before starting the updated app.

The initial product photos and hero background are remote Unsplash image URLs used as replaceable visual placeholders; replace them with licensed store assets from the admin product editor and settings page.

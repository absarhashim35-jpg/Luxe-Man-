# Luxe — Premium Men's Fashion E-Commerce (Laravel)

A full Laravel + MySQL e-commerce application: catalog with filters, cart, checkout,
order emails (admin + customer), email-verified auth, customer account area, order
tracking, and an admin panel for products/categories/orders/customers/coupons/reviews.

Theme: white background, black type, luxury gold (`#C9A96E`) accents.

---

## 1. What's in this package

This is the **application code**, not a pre-installed Laravel project (this build
environment has no access to Packagist, so `composer install` couldn't be run here).
You'll create a fresh Laravel skeleton locally and drop these files in — takes about
5 minutes.

```
app/            Models, Controllers (public + Admin + Auth), Mail classes, middleware
database/       Migrations (13 tables) + seeders (13 categories, 39 products, 2 users)
resources/views Blade templates — storefront, account, admin, emails
routes/web.php  All routes
public/css/js   Luxury theme stylesheet + interactivity (AJAX cart/wishlist, zoom, etc.)
config/mail.php Adds the `admin_order_email` config key used for order notifications
bootstrap/app.php  Registers the `is_admin` route middleware
```

---

## 2. Setup (XAMPP / local development)

**Requirements:** PHP 8.2+, Composer, MySQL (XAMPP ships all three).

```bash
# 1. Create a fresh Laravel 11 project
composer create-project laravel/laravel luxe-tmp
cd luxe-tmp
composer require laravel/ui

# 2. Copy this package's files over the fresh install (overwrite when prompted)
#    — copy app/, database/, resources/views/, routes/web.php, public/css/,
#      public/js/, config/mail.php, bootstrap/app.php, composer.json (merge the
#      "files": ["app/helpers.php"] autoload entry), and .env.example into luxe-tmp/

composer dump-autoload

# 3. Create the database in phpMyAdmin (or via CLI)
mysql -u root -e "CREATE DATABASE luxe_menswear"

# 4. Configure environment
cp .env.example .env
php artisan key:generate
# then edit .env: DB_* credentials, MAIL_* (see section 4), ADMIN_ORDER_EMAIL

# 5. Migrate + seed (13 categories, 39 demo products, admin + demo customer)
php artisan migrate --seed

# 6. Link storage (for admin-uploaded product images)
php artisan storage:link

# 7. Serve
php artisan serve
```

Visit `http://localhost:8000`.

### Demo accounts (created by the seeder)

| Role     | Email                     | Password   |
|----------|---------------------------|------------|
| Admin    | admin@luxemenswear.com    | password   |
| Customer | customer@example.com      | password   |

**Change these passwords before deploying anywhere public.** Admin panel: `/admin`.

---

## 3. Database

13 tables via migrations: `categories`, `products`, `product_images`, `carts` /
`cart_items`, `wishlists` / `wishlist_items`, `addresses`, `coupons`, `orders` /
`order_items`, `reviews`, `settings`, plus Laravel's built-in `users`,
`password_reset_tokens`, `sessions`, `jobs`, `cache`.

Order line items store a **snapshot** of the product name, image, size, color, and
price at time of purchase, so historical orders stay accurate even if a product is
later edited or deleted.

Seeded products use Unsplash placeholder photography (`ProductImage.path` stores a
full URL for seeded items, a relative `storage/` path for anything you upload through
the admin panel — both are resolved correctly by the `image_url()` helper / the
`ProductImage->url` accessor). Swap in your real product photography via
**Admin → Products → Edit → Product Images**.

---

## 4. Email — order notifications & verification

Every completed checkout sends two emails automatically (`CheckoutController::store`):

- **Admin notification** → the address in `ADMIN_ORDER_EMAIL` (`.env`), full order +
  customer + shipping + payment breakdown, black/white/gold HTML template
  (`resources/views/emails/order-admin.blade.php`).
- **Customer confirmation** → the address entered at checkout, same theme
  (`resources/views/emails/order-customer.blade.php`).

Registration fires Laravel's built-in `Registered` event, which sends the standard
email-verification link automatically. Routes protected by the `verified` middleware
(customer account area) are inaccessible until the link is clicked.

### `.env` mail configuration example (Gmail SMTP)

```env
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-16-char-app-password   # Google Account → Security → App Passwords
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS="noreply@luxemenswear.com"
MAIL_FROM_NAME="Luxe"

ADMIN_ORDER_EMAIL=admin@luxemenswear.com
```

For local testing without real email delivery, set `MAIL_MAILER=log` and read sent
emails from `storage/logs/laravel.log`, or use [Mailtrap](https://mailtrap.io) SMTP
credentials in the same fields above.

---

## 5. Feature map (what's implemented)

- **Catalog** — 13 categories, 39 seeded products, category/price/size/color/brand/
  rating filters, search, 5 sort modes, pagination (`ShopController`).
- **Product page** — gallery with thumbnail switching + hover-zoom, size/color
  swatches, quantity stepper, specs/shipping/reviews tabs, related products.
- **Cart** — guest (session-based) and logged-in (user-based) carts that merge
  naturally on login since both persist server-side; AJAX quantity/remove; coupon
  codes (`CartController`).
- **Checkout** — full validated address + payment method form, order creation is
  wrapped in a DB transaction, stock is decremented, snapshot line items are stored
  (`CheckoutController`).
- **Auth** — register/login/logout, forgot/reset password, email verification gate
  on account routes, all using Laravel's native `Auth` facade + notifications.
- **Customer account** — dashboard, profile edit, password change, order history +
  detail, saved addresses, wishlist.
- **Order tracking** — public lookup by order number + email, visual status stepper.
- **Admin panel** (`/admin`, `is_admin` middleware) — dashboard with revenue chart,
  full product/category CRUD with multi-image upload, order management (status +
  payment status + printable invoice), customer list with verification status,
  coupon creation, review moderation, store settings.
- **Security** — CSRF on every form, Form validation on every input, `is_admin`
  route middleware, password hashing via `Hash::make`, Eloquent (no raw SQL),
  file-upload validation (`image|max:4096`) on all uploads.

## 6. What you'll likely want to extend

- **Online payment gateway** — the checkout form already has an "Online Payment"
  radio option and the order model already supports `payment_status: paid`; wire in
  Stripe/PayPal by adding a webhook that calls `$order->update(['payment_status' =>
  'paid'])`.
- **Real product photography** — replace the Unsplash seed URLs via the admin panel.
  On IP grounds Claude can't source enough licensed studio photography for a real
  catalog — this is a placeholder set to be replaced before launch.
- **Contact form email** — `HomeController::submitContact` currently just flashes a
  success message; wire in a `Mail::send()` call the same way order emails work.
- **Review submission UI** — reviews are seeded/admin-moderated; add a form on
  delivered orders if you want customers to submit their own.

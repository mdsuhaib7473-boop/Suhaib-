# GramWise — Installation Guide

GramWise is a plain PHP 8 + MySQL application. It needs no Node.js, no build
step, and no long-running processes — it runs on standard shared hosting
(Hostinger, SiteGround, etc.) exactly like a normal PHP site.

## Requirements

- PHP 8.0+ with the `pdo_mysql`, `mbstring`, `fileinfo`, and `gd` (or similar) extensions enabled
- MySQL 5.7+ or MySQL 8 / MariaDB 10.3+
- Apache with `mod_rewrite` and `mod_headers` enabled (standard on shared hosting)

## 1. Upload the files

Upload the entire contents of this project to your hosting account's web
root (usually `public_html`), so that `index.php`, `.htaccess`,
`database.sql` etc. sit directly inside `public_html`.

## 2. Create a MySQL database

In your hosting control panel (e.g. Hostinger's hPanel → Databases → MySQL
Databases):

1. Create a new database, e.g. `u123_gramwise`.
2. Create a new database user with a strong password.
3. Attach the user to the database with **all privileges**.

## 3. Create the database user

(Done together with step 2 on most shared hosts — the panel creates the
user and assigns privileges in one step.)

## 4. Import `database.sql`

Using phpMyAdmin (or Adminer):

1. Open phpMyAdmin and select your new database.
2. Go to the **Import** tab.
3. Choose the `database.sql` file from this project.
4. Click **Go**. This creates all tables and seeds the initial 23
   conversion types, unit definitions, categories, and 18 starter
   ingredients with real density values.

## 5. Configure database credentials

Edit `config/config.php` and fill in:

```php
define('DB_HOST', 'localhost');           // usually 'localhost' on shared hosting
define('DB_NAME', 'u123_gramwise');
define('DB_USER', 'u123_gramwise_user');
define('DB_PASS', 'your-database-password');

define('SITE_URL', 'https://yourdomain.com'); // no trailing slash
define('SITE_NAME', 'GramWise');
define('ADMIN_EMAIL', 'you@yourdomain.com');

define('APP_SECRET', bin2hex(random_bytes(32))); // or paste a random 64-char string
define('APP_ENV', 'production');
```

Also update `contact_email` and `contact_address` later from
**Admin → Settings** (they're stored in the database, not this file).

## 6. Configure `.htaccess`

The provided `.htaccess` already contains the rewrite rules needed for
clean URLs, security headers and caching. No changes are usually required.
If your host uses `php-fpm` with a different rewrite setup, ask their
support team to confirm `mod_rewrite` and `AllowOverride All` are enabled
for your directory (most shared hosts have this on by default).

If your site is served over HTTPS (recommended — most hosts offer a free
SSL certificate), uncomment the HTTPS redirect block near the top of
`.htaccess`.

## 7. Create your admin account

1. Visit `https://yourdomain.com/install-create-admin.php` in your browser.
2. Fill in a username, email, and a strong password (10+ characters).
3. Submit — this creates your first `super_admin` account.
4. **Delete `install-create-admin.php` from the server immediately after.**
   Leaving it live would let anyone create an admin account.

## 8. Open the website

Visit `https://yourdomain.com/` — you should see the GramWise homepage
with a working live calculator.

## 9. Open `/admin`

Log in at `https://yourdomain.com/admin/login.php` with the account you
just created.

## 10. Configure site settings

Go to **Admin → Settings** and fill in:

- Site name / tagline
- Contact email and address (shown on the Contact and legal pages)
- Leave Google Analytics / Search Console / AdSense disabled until you have
  real IDs to enter

## 11. Generate your first batch of conversion pages

Go to **Admin → Programmatic SEO**:

1. Choose a conversion type (e.g. "Grams to Cups").
2. Choose an ingredient (e.g. "All-Purpose Flour") if the conversion
   requires one.
3. Set a page amount range, e.g. Start `25`, End `500`, Step `25`.
4. Set a conversion table range, e.g. Start `25`, End `1000`, Step `25`.
5. Choose **Draft** status so you can review pages before they go live.
6. Click **Preview Pages** to see exactly which URLs will be created and
   how many already exist (duplicates are automatically skipped).
7. Click **Confirm & Generate**.
8. Review the new pages under **Admin → Conversion Pages**, then bulk-select
   and **Bulk Publish** the ones you're happy with.

Repeat for other conversion types and ingredients. Start with 100–200
well-reviewed pages, then expand gradually based on what's actually
getting search traffic — resist the urge to generate everything at once.

## 12. Add Google Search Console

1. In [Google Search Console](https://search.google.com/search-console),
   add your domain as a property using the HTML tag verification method.
2. Copy the `content="..."` value from the meta tag Google gives you.
3. In **Admin → Settings**, paste it into "Verification Code" under Google
   Search Console and check **Enabled**.
4. Submit your sitemap in Search Console: `https://yourdomain.com/sitemap.xml`.

## 13. Add AdSense code when eligible

Once your site has enough original, high-quality content and you're ready
to apply, sign up at [Google AdSense](https://adsense.google.com). After
approval:

1. In **Admin → Settings**, paste your Publisher/Client ID (e.g.
   `ca-pub-XXXXXXXXXXXXXXX`) under Google AdSense and check **Enabled**.

This project does not and cannot guarantee AdSense approval — that
decision is entirely Google's, based on their policies and your site's
content at the time you apply.

---

## Ongoing maintenance notes

- **Backups**: export your database regularly via phpMyAdmin, and keep a
  copy of the `uploads/` folder (user-uploaded images).
- **Updating ingredients**: use **Admin → Ingredients** — changing a
  density value does *not* automatically recalculate existing generated
  pages; regenerate affected pages via the SEO generator if a density
  correction is significant.
- **Adding new unit types**: insert a row directly into the `units` table
  with the correct `to_base_factor` relative to grams (mass) or
  milliliters (volume), then create a new conversion type in
  **Admin → Conversions** referencing it.
- **Security**: change `APP_SECRET` and your database password before going
  live, keep PHP updated on your hosting plan, and never re-upload
  `install-create-admin.php` to a live site with existing admin accounts.

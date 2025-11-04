# Implementation Plan

## 1. Scope Recap & Approach
- Deliver a Laravel 12 company profile site with Home, About, Services, Portfolio, Contact, and future Blog sections, paired with a custom admin panel that controls all public-facing content.
- Implement AJAX-driven CRUD for services, projects, team members, posts, and contact management using jQuery, Yajra DataTables, and Vanilla JS to keep the interface responsive without full-page reloads.
- Adopt a componentised Blade architecture with Bootstrap 5 for layout, Vite for asset bundling, and strict separation between public and `/admin` areas enforced by Spatie Permission middleware.
- Follow an iterative workflow: finalize UI assets from design handoff, scaffold domain models and migrations, implement public pages, then layer admin CRUD, QA, and deployment automation.

## 2. High-Level Architecture
```
Browser (Public + Admin)
        │
        ▼
Laravel 12 (Blade, Controllers, Policies, Services)
        │            │
        │            ├── Redis (queues, cache, rate limiting)
        │            └── Mailer (Mailgun/Postmark/SES)
        ▼
MySQL 8+ (content, auth, Spatie roles/permissions)
```
- Public requests hit Blade controllers that read cached content and render Bootstrap templates.
- Admin requests are routed through `auth` + `role/permission` middleware, using AJAX endpoints for CRUD operations that feed DataTables.
- Queued jobs handle email delivery, cache warming, and media processing; scheduled commands generate sitemaps and run housekeeping tasks.
- Storage uses the public disk (local or S3) with `storage:link` to serve uploaded assets.

## 3. Environment & Tooling
| Environment | Purpose | Hosting & Configuration |
| ----------- | ------- | ----------------------- |
| Local       | Feature development, QA | Laravel Sail or Valet, `.env.local`, Telescope enabled |
| Staging     | Stakeholder review, regression testing | Forge-managed droplet, HTTPS, seeded with sample data |
| Production  | Live traffic | Forge + Envoyer blue/green deploy, auto-SSL, queue workers |

- Core stack: PHP 8.3, Laravel 12 LTS, MySQL 8+, Redis, Nginx + PHP-FPM.
- Frontend: Blade, Bootstrap 5, Vite, Vanilla JS, jQuery (for AJAX integration with DataTables).
- Packages: spatie/laravel-permission, yajra/laravel-datatables, spatie/laravel-seo, spatie/laravel-sitemap, Laravel Breeze, Laravel Telescope (dev), Sentry (prod monitoring).
- Tooling: PestPHP/PHPUnit, PHP-CS-Fixer or Pint, ESLint (optional), Prettier, Git hooks via Husky, CI via GitHub Actions, deployments via Forge/Envoyer.

## 4. Database Design
| Table | Key Columns | Relationships | Purpose |
| ----- | ----------- | ------------- | ------- |
| users | name, email, password, email_verified_at | `hasMany` posts, contact_notes | Authenticated accounts for admin access |
| roles / permissions | name, guard_name | Many-to-many with users via Spatie pivot tables | Role-based access control |
| model_has_roles / role_has_permissions / model_has_permissions | model_type, model_id, role_id / permission_id | Spatie pivot tables | Link users to roles and permissions |
| services | title, slug, short_description, description, icon_path, display_order, is_published | N/A | Public Services section |
| projects | title, slug, client, category, technology_stack, summary, results, cover_image, is_published | `hasMany` project_images; optional `belongsToMany` tags | Portfolio case studies |
| project_images | project_id, path, caption, display_order | `belongsTo` projects | Image gallery per project |
| teams | name, role_title, bio, photo_path, linkedin_url, order_index, is_visible | N/A | Team profiles |
| posts | title, slug, excerpt, body, cover_image, status, published_at, author_id | `belongsTo` users; `belongsToMany` tags | Blog (phase 2) |
| tags | name, slug | `belongsToMany` posts/projects | Content taxonomy |
| post_tag / project_tag | post_id/project_id, tag_id | Pivot tables | Tag assignments |
| contact_messages | name, email, company, phone, message, status, handled_by, handled_at | `belongsTo` users (handled_by) | Leads from Contact form |
| contact_notes | contact_message_id, user_id, note | `belongsTo` contact_messages & users | Internal follow-up log |
| media | uuid, model_type, model_id, disk, path, collection | Polymorphic | Centralised asset handling (optional) |
| settings | key, value, type, group | N/A | Manage global site metadata, hero copy, social links |

## 5. Routing & Controllers
- Public area (`routes/web.php`):
  - Named routes for home, about, services index/show, portfolio index/show, contact (GET/POST), blog listing/post (feature-flagged for phase 2).
  - Controllers under `App\Http\Controllers\Site` returning Blade views, leveraging route caching and view composers.
- Admin area (`routes/admin.php`) with prefix `/admin` and middleware stack `['web','auth','verified','role:Admin|Editor']`:
  - `DashboardController@index` renders metrics and aggregated counts.
  - Resource-style controllers (e.g., `ServiceController`, `ProjectController`) expose JSON endpoints for DataTables (`index`, `store`, `update`, `destroy`) and serve Blade views for forms.
  - Dedicated `UploadController` for media, `ContactMessageController` for read/archive, and `ProfileController` for password updates.
- API routes (`routes/api.php`) provide minimal JSON endpoints if needed for asynchronous widgets, protected by `auth:sanctum` and permission checks.
- Middleware: `EnsureFrontendRequestsAreStateful`, `SetLocale` (if multi-language later), rate limiting on contact submission, `permission:` guards for CRUD endpoints.

## 6. Frontend Implementation
- Structure Blade layouts (`layouts/app.blade.php`, `layouts/admin.blade.php`) with partials for navigation, footers, modals, and flash alerts.
- Use Bootstrap 5 utility classes, custom SCSS modules compiled through Vite, and CSS variables for brand palette (`#5AB3F1` accent, neutrals for background/typography).
- Vanilla JS modules handle interactive elements: hero animations, scroll effects, form validation feedback, responsive navigation toggles.
- jQuery is scoped to AJAX CRUD interactions; use `$.ajax` wrappers returning JSON, update DOM via template partials.
- Implement progressive enhancement: forms submit normally with server fallback; JS intercepts to send XHR when available.
- Optimize performance with lazy-loaded images (`loading="lazy"`), responsive `<picture>` assets, prefetch of critical CSS, and analytics scripts loaded asynchronously.

## 7. Admin Panel UX Details
- Admin layout: fixed sidebar with accessible keyboard navigation, top bar for quick actions, breadcrumb-based content region.
- Each module uses Yajra DataTables with server-side processing for pagination, sorting, and column filters; responsive design ensures usability on tablets.
- CRUD forms open in modals or dedicated pages; AJAX submissions return validation errors as JSON, rendered inline without reloads.
- Utilize reusable Blade components (`<x-admin.form.input>`, `<x-admin.table.actions>`) for consistency; flash toasts appear via Vanilla JS.
- Spatie Permission drives menu visibility and button availability: Editors can create/update content; only Admin can delete or publish.
- Include media upload previews using FileReader API, with size/type validation prior to submission; integrate drag-and-drop reordering for services/projects where required.

## 8. Security & Compliance
- Enforce HTTPS in staging/production with HSTS; configure trusted proxies for Forge-managed load balancers.
- Laravel-native CSRF tokens on all forms; sanitize inputs with `request->validated()` and HTML Purifier for WYSIWYG content.
- Implement rate limiting (`ThrottleRequests`) on login and contact endpoints; add hCaptcha/reCAPTCHA v3.
- Store passwords with Argon2id hashing, enforce strong password rules, and enable two-factor auth (optional via Laravel Fortify/Breeze).
- Harden headers using Laravel’s `SecureHeaders` middleware (CSP, X-Frame-Options, Referrer-Policy).
- Scheduled jobs purge stale contact records per privacy policy; log data access in Telescope/Sentry to support audits.

## 9. Content Management Flow
1. **Services:** Admin navigates to Services table, fetches server-side data via DataTables, uses modal form for create/edit, order adjusted via drag handles hitting a `PUT /services/{id}/reorder` endpoint.
2. **Portfolio:** Projects managed with multi-step modal or dedicated page; uploads handled via AJAX to media endpoint, gallery order persisted; publish toggle triggers `PATCH /projects/{id}/status`.
3. **Team:** Simple CRUD with optional social links; status toggle controls visibility on public page.
4. **Blog (Phase 2):** WYSIWYG field integrates Trix/TinyMCE; slug auto-generated, tags selected via tokenized input; publish workflow aligned with Editor/Admin permissions.
5. **Contacts:** DataTable displays unread messages; clicking row opens sliding panel with full message, ability to mark as handled, leave internal notes, or archive; export CSV via queued job.

## 10. Testing Strategy
- **Unit tests:** Cover Eloquent models (accessors, scopes), service classes, permission gates.
- **Feature tests:** Ensure public pages render expected data, contact form handles validation and spam checks, admin endpoints enforce permissions and return correct JSON structures.
- **Integration tests:** Simulate AJAX CRUD flows using `actingAs` with roles; verify DataTables responses, file uploads, and queue-triggered mail dispatch.
- **Browser tests (Dusk or Laravel Playwright):** Validate critical UI paths (contact submission, admin CRUD) on key breakpoints.
- Continuous testing via GitHub Actions: run `php artisan test`, `npm run build -- --watch=false`, static analysis (Larastan/Psalm) and coding standards (Pint).

## 11. Deployment Checklist
1. Configure Forge server (PHP 8.3, Nginx, Redis, Supervisor workers) and attach repository.
2. Provision databases, Redis, and storage bucket; set environment secrets and mail credentials.
3. Define Envoyer deployment hook: `composer install --optimize-autoloader`, `php artisan migrate --force`, `php artisan db:seed --class=InitialContentSeeder`, `npm ci && npm run build`, `php artisan storage:link`.
4. Run caches: `config:cache`, `route:cache`, `view:cache`, `event:cache`; restart queue workers and Horizon (if used).
5. Verify health checks (HTTP 200, Horizon dashboard, queue status), rotate logs, and enable monitoring/alerts (Sentry, Forge notifications).
6. Schedule cron: `* * * * * php artisan schedule:run` for heartbeat tasks (sitemap generation, stale contact clean-up).

## 12. Timeline Alignment
| PRD Phase | Duration | Implementation Focus |
| --------- | -------- | -------------------- |
| Research & Sitemap (Week 1) | 1 week | Personas validation, sitemap confirmation, content inventory, wireframe approvals |
| Wireframe & UI Design (Weeks 2-3) | 2 weeks | Deliver responsive Figma mockups, component library, admin UX blueprint |
| Development (Weeks 4-7) | 4 weeks | Scaffold Laravel project, build database/migrations, implement public pages, admin CRUD, integrate SEO + analytics |
| Testing & Launch (Week 8) | 1 week | Regression + accessibility testing, performance tuning, final content load, deployment cutover |

## 13. Open Questions / Decisions Needed
1. Confirm hosting preference (Forge-provisioned VPS vs. managed platform) and budget for Redis/S3/Sentry services.
2. Determine exact analytics stack (Google Analytics 4 vs. Plausible) and cookie consent requirements.
3. Clarify blog launch timing and content workflow (draft/approve roles) to plan migrations and UI scope.
4. Approve email service provider (Mailgun, Postmark, SES) and sender domains for contact notifications.
5. Provide final copy, imagery, and brand assets timeline to align with development milestones.

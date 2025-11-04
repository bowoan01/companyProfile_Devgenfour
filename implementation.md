# Implementation Plan

## 1. Scope Recap & Approach
The project delivers a marketing-driven company profile site with public sections (Home, About, Services, Portfolio, Contact, Phase 2 Blog) and a secure admin panel for content maintenance. Laravel 12 on PHP 8.3 anchors the stack, leveraging Blade for templating, MySQL 8 for persistence, and jQuery-enhanced vanilla JavaScript for interactivity. Development follows Git-based feature branching, code review, and staggered releases (staging → production). Content modules expose CRUD services through RESTful controllers returning JSON for AJAX operations, while public pages consume server-rendered Blade views to maintain SEO and performance.

## 2. High-Level Architecture
```
Browsers (public + admin)
   │
   ├── Blade templates + Bootstrap 5 (CDN)
   │       └── public/js/app.js (public UX)
   │       └── public/js/admin.js (admin AJAX/DataTables)
   │       └── public/css/*.css (custom styling)
   │
   └── Laravel HTTP layer (Controllers, Middleware, Policies)
           └── Application Services (Content managers, Mailers)
           └── Repositories / Eloquent Models
                   └── MySQL 8 database
                   └── Storage (local/public disk for uploads)
           └── Queue workers (Redis driver) for email + background jobs
           └── External integrations (SMTP, Google reCAPTCHA, Analytics API*)
```
*Analytics API integration is optional per PRD.

## 3. Environment & Tooling
| Layer | Details |
| --- | --- |
| Local development | Laragon/Valet on PHP 8.3, Composer, MySQL 8, Redis; `.env.local` mirroring production configs; Mailhog for email previews. |
| Staging | Forge-managed Ubuntu host, Nginx + PHP-FPM, HTTPS via Let's Encrypt, mirrored database schema, limited admin access for UAT. |
| Production | Forge + Envoyer blue/green deploys, autos provisioned SSL, nightly database backups, S3-compatible storage optional for media growth. |
| Core dependencies | Laravel 12, Blade, Eloquent ORM, Laravel Breeze/Fortify for auth scaffolding, Spatie Permission for RBAC, Yajra DataTables for server-side tables, Laravel Mail, Laravel Queue (Redis driver), Laravel Telescope (restricted) for debugging, Laravel DOMPDF for PDF exports (if required). |
| Frontend assets | Bootstrap 5, Popper, jQuery all via CDN; custom CSS in `public/css/app.css`, admin overrides in `public/css/admin.css`; custom scripts in `public/js/app.js` and `public/js/admin.js`; images stored in `public/images` or storage symlink. |
| Tooling | PHP-CS-Fixer for style, PestPHP/PHPUnit for tests, Laravel Pint optional, Laravel IDE helper, PHPStan 1.x for static analysis, Postman/Insomnia collections for API smoke tests. |

## 4. Database Design
| Table | Purpose | Key Fields | Relationships |
| --- | --- | --- | --- |
| users | Admin and potential future public accounts | name, email (unique), password, status, last_login_at | M:N `roles` via Spatie tables; 1:N with posts, projects (created_by) |
| roles | RBAC roles (`super-admin`, `content-manager`) | name, guard_name | M:N `permissions`, M:N `users` |
| permissions | Fine-grained abilities | name, guard_name | M:N `roles`, M:N `users` (direct) |
| services | Public service offerings | title, slug, summary, description, icon_path, display_order, published_at | 1:N attachments via media table (if needed) |
| projects | Portfolio entries | title, slug, category_id, client, technologies, summary, results, cover_path, is_featured, published_at | N:1 `project_categories`; 1:N `project_images`; 1:N relation to `project_tags` pivot |
| project_categories | Filter support for portfolio | name, slug, display_order | 1:N `projects` |
| project_images | Gallery items | project_id, path, caption, display_order | N:1 `projects` |
| project_tags | Pivot for tagging | project_id, tag_id | N:M `tags` |
| tags | Shared tag vocabulary | name, slug, type | N:M `projects`, N:M `posts` |
| team_members | Staff profiles | name, role_title, bio, photo_path, linkedin_url, order | Standalone |
| posts (Phase 2) | Blog entries | title, slug, excerpt, content, cover_path, published_at, status, author_id | N:1 `users`; N:M `tags` |
| contacts | Inbound enquiries | name, email, phone, company, message, source, status, admin_notes | Logged via contact form |
| contact_logs | Admin actions on contacts | contact_id, user_id, status_from, status_to, notes | N:1 `contacts`, N:1 `users` |
| failed_jobs, jobs | Queue infrastructure | payload metadata | Managed by Laravel queue |
| activity_logs | Optional audit trail | causer_id, subject_id, event, payload | Supports content history |
Spatie Permission introduces `model_has_roles`, `model_has_permissions`, and `role_has_permissions`. Timestamps and soft deletes (for content tables) enable rollback.

## 5. Routing & Controllers
- **Public routes (`routes/web.php`)**
  - Static pages served through `PageController`: `/`, `/about`, `/services`, `/contact`.
  - Dynamic listings via `PortfolioController@index`, `PortfolioController@show`, `ServiceController@index`.
  - Optional `/blog` + `/blog/{slug}` handled by `PostController` (Phase 2, behind feature flag).
  - `ContactController@store` processes form submissions with reCAPTCHA validation and dispatches queued emails.
- **Admin routes (`routes/admin.php`)**
  - Grouped with prefix `/admin`, name prefix `admin.`, middleware `auth`, `verified`, `role:super-admin|content-manager`.
  - Dashboard: `Admin\DashboardController@index`.
  - Services, Projects, Team, Posts, Contacts each expose index (DataTables JSON), store/update/destroy endpoints returning JSON payloads.
  - Media upload endpoints protected by `permission:manage-media`.
  - Profile management routes for password updates using `auth` + `password.confirm`.
- RouteServiceProvider registers separate middleware groups, ensures admin controllers live under `App\Http\Controllers\Admin`.

## 6. Frontend Implementation
- Blade layout hierarchy: `layouts/app.blade.php` (public) and `layouts/admin.blade.php` (admin) with partials for header, footer, nav, CTA banners, modals, DataTable initializers.
- Bootstrap 5 via CDN plus custom utility classes in `public/css/app.css`; color palette aligns with #5AB3F1 accent, neutral backgrounds, and accessible contrast ratios.
- Interactive behaviors executed with vanilla JS modules enhanced by jQuery for DOM querying and AJAX:
  - `public/js/app.js` handles smooth scrolling, mobile navigation toggles, contact form submission feedback, and lazy-loading triggers.
  - `public/js/helpers/request.js` (optional) centralizes AJAX wrapper with CSRF token injection.
  - `public/js/admin.js` initializes module-specific DataTables, modal lifecycle, and WYSIWYG integration.
- No build pipeline: assets referenced directly from Blade via `<link>` and `<script>` tags pointing to CDN or `asset('public/...')`.
- Image optimization executed during upload (Intervention/Image) and served via responsive `<picture>` markup where practical.

## 7. Admin Panel UX Details
- Two-column layout with fixed sidebar navigation (Dashboard, Services, Portfolio, Team, Blog*, Contacts) and breadcrumbed content region.
- Yajra DataTables render searchable, sortable lists; server-side endpoints supply JSON with pagination and filters.
- CRUD actions triggered via modal forms using jQuery AJAX:
  - Create/Edit modals validate inputs client-side (Bootstrap validation) before POST/PATCH calls.
  - Delete actions prompt confirmation and use soft deletes where applicable.
- Real-time validation feedback (inline error states) and toast notifications via Bootstrap alert components.
- Role/permission checks hide unauthorized menu items; Spatie middleware guards endpoints ensuring defense in depth.
- File uploads utilize drag-and-drop components with image preview; upload progress displayed via Bootstrap progress bars.
- Accessibility: focus management returning to triggering element post-modal, keyboard navigable menus.

## 8. Security & Compliance
- CSRF tokens injected into all forms and AJAX headers; Laravel middleware enforces validation.
- Auth uses hashed passwords (Argon2id) with enforced password rules and email verification.
- Input validation/sanitization through Form Requests; HTML content (posts) purified with HTMLPurifier to prevent XSS.
- Role-based access control via Spatie roles/permissions; sensitive routes audited with activity logging.
- reCAPTCHA v3 on contact form plus backend honeypot/rate limiting to mitigate spam.
- HTTPS enforced on staging/production; HSTS headers configured via Forge.
- File uploads scanned for MIME type/size; stored outside public path and served via signed URLs when necessary.
- Privacy: contact data retained per retention policy (e.g., 12 months) with capability to export/delete on request; audit logs protect personally identifiable information.

## 9. Content Management Flow
1. **Services**: Admin selects “Services” → DataTable loads active entries → “Add Service” opens modal → AJAX POST to `/admin/services` with validation → table refreshes; ordering managed via drag-and-drop endpoint.
2. **Portfolio**: Projects list shows status badges (draft/published) and feature toggle; gallery uploads handled asynchronously with queued image optimization; filters generated from `project_categories`.
3. **Team**: Admin updates team ordering via inline controls; social links validated for format; updates push to public page immediately after cache clear.
4. **Blog (Phase 2)**: Rich text editor (Trix/TinyMCE) integrated in modal or dedicated view; slugs auto-generated; preview button renders sanitized HTML.
5. **Contacts**: Inbox view lists unread messages first; viewing a message flips status to read and allows admin notes; archiving removes from default view but retains history.
6. **Global**: All modules broadcast success/error toasts; caches (`cache:forget`) invalidated to sync public pages; activity logs track actor, timestamp, and payload summary.

## 10. Testing Strategy
- **Unit tests**: Model factories validate attribute casting, scopes, and accessors (e.g., featured projects, published services). Permission assignments verified in isolation.
- **Feature tests** (Pest): Cover public endpoints (page loads, SEO metadata, contact form submission success/failure), admin CRUD (create/update/delete) with authorization checks, API JSON responses for DataTables, authentication flows (login, password reset).
- **Integration tests**: Queue/mail assertions for contact notifications, file upload workflows, DataTables JSON pagination, third-party integrations (reCAPTCHA mocked, analytics API stubbed).
- **Browser smoke tests**: Laravel Dusk (optional) for critical admin flows; manual responsive testing using BrowserStack matrix (Chrome, Safari, Firefox, Edge, mobile).
- **CI**: GitHub Actions pipeline running `composer validate`, `phpcs`/`pint`, `phpstan`, `phpunit`; database migrations run against sqlite memory or MySQL service.

## 11. Deployment Checklist
1. Commit merged to `main` triggers Envoyer deployment with zero downtime.
2. On remote host: `composer install --no-dev --optimize-autoloader`.
3. Run `php artisan migrate --force` and seed essential data (roles, admin user).
4. Execute `php artisan config:cache`, `route:cache`, `view:cache`, `event:cache`.
5. Ensure `php artisan storage:link` executed once; sync `public/` assets (CSS/JS/images) via git.
6. Restart services: `php artisan queue:restart`, Supervisor workers, Horizon (if used).
7. Verify permissions on `storage/` and `bootstrap/cache`.
8. Run smoke script (`php artisan health:check`) verifying database, cache, queue, mail.
9. Configure Envoyer/Forge hooks for cache clearing after deployment and schedule tasks (queue workers, nightly backups, log rotation, pruning). No Node/Vite steps required.

## 12. Timeline Alignment
| Phase (PRD) | Duration | Key Implementation Outcomes |
| --- | --- | --- |
| Research & Sitemap | Week 1 | Finalize information architecture, confirm content inventory, capture technical assumptions, set up repository and base Laravel install. |
| Wireframe & UI Design | Weeks 2–3 | Approve responsive wireframes, define design tokens, prepare Blade layout scaffolding, gather imagery/copy. |
| Development | Weeks 4–7 | Implement public pages, build admin modules sequentially (Services → Portfolio → Team → Contacts → Blog*), integrate AJAX/DataTables, configure queues/mail, seed data. |
| Testing & Launch | Week 8 | Execute automated/manual test suite, UAT feedback loop, finalize SEO metadata, prepare deployment artifacts, go-live via Envoyer. |
| Post-Launch Support | Ongoing | Monitor analytics, address bug reports, schedule Phase 2 Blog activation. |
*Blog activation may shift if stakeholders defer Phase 2.

## 13. Open Questions / Decisions Needed
1. Confirm hosting stack (Forge on AWS, DigitalOcean, or existing infrastructure) and SSL certificate management.
2. Define analytics tooling (Google Analytics 4, Plausible) and data retention expectations.
3. Clarify requirement for optional features: testimonials carousel, client logos, downloadable PDFs, or newsletter signup.
4. Decide on asset optimization expectations (e.g., use of WebP conversion, CDN for media).
5. Approve email sending provider (SMTP vs. transactional service like Mailgun) and sender identities.
6. Confirm blog Phase 2 launch timing and necessary content governance workflow.
7. Determine if multilingual support or future localization should influence data model now.
8. Validate privacy policy/legal copy ownership and compliance obligations (GDPR, PDPA, etc.).


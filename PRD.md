# 📘 Product Requirement Document (PRD)

## Devgenfour Company Profile Website

## 1. Background

Devgenfour is a **software house** that has successfully handled various projects for clients across different industries. However, the company currently does not have an **official website** to represent its brand, showcase its work, and attract new clients.

This website will serve as **Devgenfour’s digital identity** — a professional platform that:

* Highlights the company’s expertise and portfolio.
* Builds credibility with potential clients and partners.
* Communicates Devgenfour’s brand values and culture.
* Serves as the main touchpoint for inquiries and collaborations.

---

## 2. Product Goals

1. **Establish credibility** and position Devgenfour as a trusted software development partner.
2. **Generate leads** through clear calls-to-action (CTAs) and contact channels.
3. **Showcase the company’s portfolio** and the range of services offered.
4. **Express the brand identity** through a minimalist, modern, and professional interface.

---

## 3. Target Users

* **Potential clients** — startups, companies, or institutions seeking software development services.
* **Potential hires** — designers, developers, and project managers interested in joining Devgenfour.
* **Business partners or investors** — organizations or individuals exploring collaboration opportunities.

---

## 4. Design & Visual Style

### Visual Direction

* **Design language:** Minimalist, smooth, and clean.
* **Mood:** Professional, modern, elegant.
* **Primary accent color:** Light blue `#5AB3F1`, complemented by white and light gray tones.
* **Typography:**

  * Headings: *Poppins* or *Inter* (bold, clean, modern).
  * Body text: *Open Sans* or *Lato* (legible, neutral).

### Design Principles

* **Simplicity first:** Focus on essential content only.
* **Ample whitespace:** Maintain visual clarity and breathing room.
* **Consistent grid and spacing:** Harmonize layout across breakpoints.
* **Fully responsive:** Designed with a *mobile-first* approach.

---

## 5. Website Structure

### 1. Home / Landing Page

* Hero section with headline:
  *“We build digital products that empower your business.”*
* CTA: **“Let’s Work Together”** → leads to the Contact page.
* Display of 3–4 featured projects.
* Optional client testimonials.
* Core values: Quality, Collaboration, Innovation.

### 2. About Us

* Company background and timeline.
* Leadership or team profiles.
* Core values and mission statement.

### 3. Services

* Overview of main services:

  * **Custom Software Development**
  * **UI/UX Design**
  * **Mobile App Development**
  * **Website Development**
  * **System Integration & Maintenance**
* CTA: **“Discuss Your Project.”**

### 4. Portfolio

* Grid-style project gallery.
* Filter by category (Web, Mobile, Enterprise, etc.).
* Each project page includes:

  * Title, client, technologies, description, and screenshots.

### 5. Contact

* Form (name, email, message).
* Company contact details and embedded Google Maps.
* CTA: **“Schedule a Call.”**

### 6. Blog (Optional — Phase 2)

* Articles about technology, design, and company updates.
* Purpose: SEO and brand authority.

---

## 6. Key Features

| Feature                      | Description                                                              |
| ---------------------------- | ------------------------------------------------------------------------ |
| **Simple Admin Panel**       | Built-in admin interface for managing site content (no third-party CMS). |
| **Responsive Design**        | Optimized for all screen sizes and devices.                              |
| **SEO Optimization**         | Fast loading, proper metadata, and well-structured headings.             |
| **Contact Form Integration** | Laravel-based email system with validation.                              |
| **Image & File Management**  | Efficient upload and display from storage.                               |
| **Security**                 | CSRF protection, authentication, and spam prevention.                    |

---

## 7. Technical Specifications (Laravel Environment) — **Final Version**

| Category                  | Specification                                                                                          |
| ------------------------- | ------------------------------------------------------------------------------------------------------ |
| **Language**              | PHP 8.3+                                                                                               |
| **Framework**             | Laravel 12 (LTS preferred)                                                                             |
| **Frontend**              | **Laravel Blade + Bootstrap 5**                                                                        |
| **UI Interactivity**      | **Vanilla JavaScript** for lightweight DOM manipulation (modals, dropdowns, validation, minor effects) |
| **AJAX CRUD**             | **jQuery AJAX** for CRUD operations without page reload                                                |
| **DataTable Integration** | **Yajra Laravel DataTables** for *server-side processing* (pagination, search, sort)                   |
| **Role Management**       | **Spatie Laravel Permission** for role & permission management via middleware                          |
| **Database**              | MySQL 8+ or PostgreSQL 18+ with Eloquent ORM                                                           |
| **Admin Panel**           | Custom-built using Blade templates (no Filament or Nova)                                               |
| **Authentication**        | Laravel Breeze (login/logout/reset password)                                                           |
| **Caching & Performance** | Redis for caching & queues; route & config cache enabled                                               |
| **Email System**          | Laravel Mail (Mailgun, Postmark, or SES)                                                               |
| **Hosting**               | Nginx + PHP-FPM via Laravel Forge or Envoyer                                                           |
| **Storage**               | Local or AWS S3 via Laravel Filesystem                                                                 |
| **Analytics**             | Google Analytics 4 or Plausible                                                                        |
| **SEO Tools**             | spatie/laravel-seo and spatie/laravel-sitemap                                                          |
| **Security**              | CSRF/XSS protection, hCaptcha or reCAPTCHA for forms                                                   |
| **Testing**               | PestPHP / PHPUnit for unit & feature tests                                                             |
| **Monitoring**            | Laravel Telescope (dev), Sentry (production)                                                           |
| **Performance Goals**     | TTFB < 200ms, LCP < 2.5s, PageSpeed ≥ 90 (desktop)                                                     |

---

### 🔧 Technical Explanation

1. **Bootstrap 5** is used for a modern, elegant, and fast UI development workflow.
2. **Vanilla JS** handles lightweight UI interactions while keeping performance optimal.
3. **jQuery AJAX** enables asynchronous CRUD operations without page reload.
4. **Yajra DataTables** provides *pagination, search, and sort* via server-side rendering — efficient for large datasets.
5. **Spatie Laravel Permission** offers full control over user roles and permissions:

   * Default roles: `Admin`, `Editor`, `Viewer (optional)`
   * Role & permission-based middleware (`role:Admin`, `permission:edit posts`)
   * Built-in tables: `roles`, `permissions`, `model_has_roles`, `role_has_permissions`
   * Route integration example:

     ```php
     Route::group(['middleware' => ['role:Admin']], function() {
         // Routes for admin
     });
     ```
6. The combination of **Spatie + Yajra + jQuery AJAX** results in a **lightweight, modular, and scalable** admin system.

---

## 8. Admin Panel Features

*(Unchanged, now includes Spatie for role-based authorization)*

---

### 8.1 Access & Authentication

* Admin login using email/password
* Forgot password & reset via email
* Session timeout after inactivity
* **Role Management** using Spatie (Admin, Editor, Viewer)

**Route prefix:** `/admin`

---

### 8.2 Dashboard

* Overview of total projects, services, posts, and messages
* Quick links for “Add New” actions
* Dynamic data display using Yajra DataTables
* Optional: Analytics summary from Google Analytics API

---

### 8.3 Content Management Modules

*(All CRUD modules use jQuery AJAX + DataTables)*

#### a. Service Management

* CRUD for services
* Fields: title, short description, detailed description, icon, order
* Actions: Add / Edit / Delete / Reorder

#### b. Portfolio Management

* CRUD for project case studies
* Fields: title, client, category, technology, description, results, cover image, gallery
* Publish / unpublish toggle

#### c. Team Management

* CRUD for team members
* Fields: name, role, bio, photo, social links, order

#### d. Blog Management (Phase 2)

* CRUD for blog posts with WYSIWYG editor (Trix or TinyMCE)
* Fields: title, slug, excerpt, content, cover image, tags, publish toggle

#### e. Contact Messages

* Read-only view of contact form submissions
* Mark as read or archive
* Delete old entries

---

### 8.4 Admin Panel UI & UX

*(Unchanged, now powered by DataTables with AJAX calls)*

* Sidebar navigation: Dashboard, Services, Portfolio, Team, Blog (optional), Contacts
* Table views with pagination, search, and sorting
* Form validation and flash notifications
* File upload with image preview
* CSRF protection and input sanitization
* Clean two-column layout (sidebar + content area)

---

### 8.5 Route Structure

```
/admin
├── /dashboard
├── /services
│   ├── create / edit / delete
├── /portfolio
│   ├── create / edit / delete
├── /team
│   ├── create / edit / delete
├── /blog (optional)
│   ├── create / edit / delete
├── /contacts
│   ├── view / delete
└── /profile
    ├── change-password
```

---

### 8.6 Technical Summary

| Component       | Description                                                          |
| --------------- | -------------------------------------------------------------------- |
| **Views**       | Blade templates in `resources/views/admin`                           |
| **Controllers** | Namespace `app/Http/Controllers/Admin`                               |
| **Models**      | Service, Project, Post, TeamMember, ContactMessage, Role, Permission |
| **Middleware**  | `auth`, `verified`, `role`, `permission`                             |
| **Assets**      | Bootstrap 5, jQuery, Vanilla JS                                      |
| **DataTable**   | Yajra DataTables (server-side rendering)                             |
| **Uploads**     | Stored in `/storage/app/public/uploads`                              |
| **Editor**      | Trix or TinyMCE (local, no third-party integration)                  |

---

## 9. Project Timeline (Estimation)

| Phase                 | Duration | Deliverables                  |
| --------------------- | -------- | ----------------------------- |
| Research & Sitemap    | 1 week   | UX flow and site architecture |
| Wireframe & UI Design | 2 weeks  | High-fidelity mockups         |
| Development (Laravel) | 4 weeks  | Website + Simple Admin Panel  |
| Testing & Launch      | 1 week   | Public release                |

**Total estimated time:** 8 weeks

---

## 10. Success Metrics

* Bounce rate below **40%**
* Average session duration above **1 minute**
* At least **10 inquiries per month** via contact form
* PageSpeed score above **90 (desktop)**
* Admin panel fully functional and easy to use without documentation

---

## 11. Additional Notes

* The main theme remains **light blue (#5AB3F1)** as the primary accent color.
* The visual tone should be **tech-oriented yet human-centered**, avoiding overly corporate aesthetics.
* Include **micro-interactions** for CTAs and transitions.
* Ensure **cross-browser compatibility**.
* Maintain **strict separation between admin and public routes** for security and scalability.

---
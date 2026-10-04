# Migration Roadmap & Strangler Execution Plan

**Project**: GB Technologies Architecture Transformation  
**From**: Monolithic Django 5.0.2 + Bootstrap 5.3  
**To**: Level 3 Decoupled Enterprise Stack (Django REST Framework + Next.js App Router)  
**Strategy**: Incremental Strangler Fig Pattern (Zero-Downtime Migration)

---

## 1. Migration Overview & Strategy

The migration from the monolithic Django codebase to a modern decoupled Next.js + Django API architecture follows the **Strangler Fig Pattern**. Rather than a high-risk "big bang" rewrite, the new system is developed alongside the legacy system. An edge reverse proxy (Cloudflare or Nginx) routes traffic between the legacy Django monolith and the new Next.js front end on a route-by-route basis.

```mermaid
flowchart TD
    subgraph Edge Layer
        User[End User Browser / Client] --> Proxy[Reverse Proxy Gateway (Cloudflare / Nginx)]
    end

    subgraph Phase Progression
        Proxy -->|/api/*| API[Django REST Framework]
        Proxy -->|Migrated Routes: /, /solutions, /portal| Next[Next.js App Router (Vercel / Node)]
        Proxy -->|Legacy Fallback: /admin/*, Unmigrated Routes| Legacy[Legacy Django Monolith]
    end

    API --> DB[(PostgreSQL + Redis)]
    Legacy --> DB
```

---

## 2. Phase-by-Phase Execution Schedule

### Phase 0: Foundations & Project Workspace Setup
* **Objective**: Prepare repository structure, decouple configuration, establish developer environments, and initialize the Next.js workspace.
* **Timeline / Effort**: 1 Week (Sprint 1)
* **Dependencies**: None.
* **Key Deliverables**:
  1. Monorepo structure established (`/backend` containing Django, `/frontend` containing Next.js).
  2. Environment variable decoupling via `django-environ` (`.env.example`, `.env.local`).
  3. Next.js 14/15 App Router initialized with TypeScript strict mode and Tailwind CSS v4.
  4. Docker Compose environment configured for local development (Django, PostgreSQL, Redis, Next.js).
* **Acceptance Criteria**:
  - `docker compose up` spins up backend, database, cache, and frontend without errors.
  - Next.js root layout compiles with TypeScript 0 errors and renders initial blank canvas.

---

### Phase 1: Headless Django API & OpenAPI Engine
* **Objective**: Convert the Django backend into a clean headless REST API with JWT cookie authentication, robust serializers, and auto-generated OpenAPI 3.1 documentation.
* **Timeline / Effort**: 1.5 Weeks (Sprint 2)
* **Dependencies**: Phase 0.
* **Key Deliverables**:
  1. Install & configure `djangorestframework`, `djangorestframework-simplejwt`, `corsheaders`, and `drf-spectacular`.
  2. Implement `/api/v1/auth/` endpoints: `login`, `logout`, `register`, `token/refresh`.
  3. Implement `/api/v1/users/me/` endpoint with `UserProfileSerializer` supporting profile updates and profile image uploads.
  4. Implement `/api/v1/contact/leads/` endpoint for lead capture.
  5. Expose OpenAPI 3.1 schema at `/api/v1/schema/` and interactive Swagger UI at `/api/v1/docs/`.
* **Acceptance Criteria**:
  - `pytest` suite passes with 100% coverage on authentication, user profile mutation, and lead capture.
  - JWT tokens successfully set and cleared in secure HTTP-only cookies.
  - CORS and CSRF correctly block unauthorized cross-origin mutations while permitting frontend origins.

---

### Phase 2: Design System, Tokens & UI Primitives
* **Objective**: Build the accessible, responsive UI foundation in Next.js using OKLCH color tokens, fluid typography, shadcn/ui, and Radix primitives.
* **Timeline / Effort**: 2 Weeks (Sprint 3)
* **Dependencies**: Phase 0.
* **Key Deliverables**:
  1. Configure `globals.css` with OKLCH CSS variables for Light and Dark modes (referencing `color_tokens.md`).
  2. Set up Tailwind v4 theme mapping with fluid type scales (`clamp()`) and 8-point spacing grid (referencing `design_system.md`).
  3. Install and customize core shadcn/ui components: Button, Input, Form, Dialog, Sheet, DropdownMenu, Avatar, Badge, Card, Table.
  4. Implement global components: Navigation Bar (with glassmorphism and mobile drawer), Footer, and Theme Toggle.
  5. Write automated unit tests for components using Vitest and React Testing Library.
* **Acceptance Criteria**:
  - Zero hydration errors or Flash of Wrong Theme (FOWT) on light/dark toggle.
  - Automated Axe accessibility scan reports 0 critical or serious WCAG 2.2 violations on all primitive components.

---

### Phase 3: Public Website & Marketing Experience
* **Objective**: Rebuild marketing landing pages, solutions catalog, interactive calculators, and contact forms on Next.js with sub-second page loads.
* **Timeline / Effort**: 2 Weeks (Sprint 4)
* **Dependencies**: Phase 1 (Lead API), Phase 2 (Design System).
* **Key Deliverables**:
  1. Homepage (`/`): Fluid hero section, interactive capabilities Bento grid, live ROI/metrics simulator, client testimonials carousel, and high-conversion CTA banner.
  2. Solutions & Services pages (`/solutions`, `/services/[slug]`) configured with SSG and ISR.
  3. Contact page (`/contact`) featuring multi-step project scoping form integrated with `/api/v1/contact/leads/`.
  4. SEO optimization: Dynamic OpenGraph images (`@vercel/og`), sitemap generation (`sitemap.ts`), robots.txt, and structured JSON-LD data.
* **Acceptance Criteria**:
  - Google Lighthouse score > 95 across Performance, Accessibility, Best Practices, and SEO.
  - LCP < 1.2s, INP < 100ms, CLS = 0.00 on both mobile and desktop profiles.

---

### Phase 4: Authenticated Portal & Profile Management
* **Objective**: Rebuild the authenticated user experience, dashboard telemetry, and profile management screens with real-time feedback and optimistic mutations.
* **Timeline / Effort**: 2 Weeks (Sprint 5)
* **Dependencies**: Phase 1 (Auth & User API), Phase 2 (Design System).
* **Key Deliverables**:
  1. Auth flows (`/auth/login`, `/auth/register`) with React Hook Form + Zod client validation and server-side error mapping.
  2. Protected dashboard layout (`/portal/layout.tsx`) with collapsible sidebar and route guard middleware.
  3. Portal Dashboard (`/portal/dashboard`): Metric cards with sparklines, recent activity feed, and active engagement overview.
  4. Profile View & Edit (`/portal/profile`, `/portal/profile/edit`): Profile form with phone number validation, country dropdown, and drag-and-drop avatar uploader with optimistic preview.
* **Acceptance Criteria**:
  - Unauthorized users attempting to access `/portal/*` are seamlessly redirected to `/auth/login?returnUrl=...`.
  - Profile image updates reflect immediately in the UI and persist cleanly to backend media storage.

---

### Phase 5: Production Hardening, Reverse Proxy Cutover & Decommissioning
* **Objective**: Configure the production Strangler reverse proxy, execute seamless DNS/traffic cutover, monitor production telemetry, and retire legacy templates.
* **Timeline / Effort**: 1.5 Weeks (Sprint 6)
* **Dependencies**: Phases 1 through 4.
* **Key Deliverables**:
  1. Production Nginx / Cloudflare Worker routing rules deployed (see Section 3).
  2. Complete 301 URL redirect parity map verified.
  3. Datadog / Sentry telemetry and error tracking configured on both frontend and backend.
  4. Full end-to-end regression tests executed via Playwright.
  5. Archive and delete legacy Django templates (`web/templates/web/*.html`) and static vendor scripts (`web/static/assets/vendor/`).
* **Acceptance Criteria**:
  - 100% of public and portal traffic served via Next.js with zero 404s or regression errors.
  - Django server memory footprint reduced by > 40% following template and static asset deprecation.

---

## 3. Strangler Pattern: Reverse Proxy Configuration

Below is the production Nginx configuration snippet illustrating how traffic is split during the Strangler migration:

```nginx
# /etc/nginx/sites-available/gb-technologies.conf

upstream nextjs_upstream {
    server 127.0.0.1:3000;
    keepalive 32;
}

upstream django_upstream {
    server 127.0.0.1:8000;
    keepalive 32;
}

server {
    listen 443 ssl http2;
    server_name gb-technologies.com;

    # SSL Certificates & Security Headers
    ssl_certificate /etc/letsencrypt/live/gb-technologies.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/gb-technologies.com/privkey.pem;
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;

    # 1. API & Backend Services -> Route to Django REST Framework
    location /api/ {
        proxy_pass http://django_upstream;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 2. Django Admin & Internal Tools -> Route to Django
    location /admin/ {
        proxy_pass http://django_upstream;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 3. Media Uploads (User Avatars) -> Route to Django or Object Storage
    location /media/ {
        proxy_pass http://django_upstream;
        proxy_set_header Host $host;
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    # 4. Default / Strangler Front End -> Route to Next.js App Router
    location / {
        proxy_pass http://nextjs_upstream;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 4. URL Parity & 301 Redirect Matrix

To preserve SEO equity, backlinks, and user bookmarks, all legacy routes map cleanly to new target routes:

| Legacy Monolith Route | Target Next.js Route | Status Code | Notes |
| :--- | :--- | :--- | :--- |
| `/` | `/` | 200 OK | Direct replacement with Next.js ISR homepage. |
| `/landing/` | `/` | 301 Moved Permanently | Consolidates legacy alternate homepage into root. |
| `/index/` | `/` | 301 Moved Permanently | Eliminates duplicate URL for homepage. |
| `/login/` | `/auth/login` | 301 Moved Permanently | Canonical authentication route. |
| `/register/` | `/auth/register` | 301 Moved Permanently | Canonical onboarding wizard. |
| `/logout/` | `/auth/logout` | 301 Moved Permanently | Dispatches cookie invalidation and redirects to `/`. |
| `/profile/` | `/portal/profile` | 301 Moved Permanently | Authenticated profile view. |
| `/profile/edit/` | `/portal/profile/edit`| 301 Moved Permanently | Authenticated profile edit form. |
| `/admin/*` | `/admin/*` | 200 OK | Retained on Django backend for staff administration. |
| `/media/profile_images/*` | `/media/profile_images/*` | 200 OK | Preserved path; served from S3/CDN in production. |

---

## 5. Prioritized "First 10 Tasks" Action Checklist

These are the immediate, atomic, first 10 implementation tasks to commence once user approval is granted:

```
[ ] Task 01: Establish Monorepo Workspace Structure
    - Relocate Django files into /backend directory
    - Initialize root README.md and .gitignore
    - Target Files: /backend/*, /frontend/*, /.gitignore
    - Verification: Backend runs from /backend with `python manage.py check` (0 errors)

[ ] Task 02: Decouple Django Configuration with django-environ
    - Add `django-environ` to requirements.txt
    - Extract SECRET_KEY, DEBUG, ALLOWED_HOSTS, DATABASE_URL to .env
    - Target Files: backend/gb_technologies/settings.py, backend/.env.example
    - Verification: Django starts cleanly reading settings from environment variables

[ ] Task 03: Install DRF & Configure OpenAPI (drf-spectacular)
    - Install `djangorestframework`, `drf-spectacular`
    - Configure REST_FRAMEWORK and SPECTACULAR_SETTINGS in settings.py
    - Target Files: backend/gb_technologies/settings.py, backend/gb_technologies/urls.py
    - Verification: Visiting /api/v1/schema/ outputs valid OpenAPI 3.1 YAML

[ ] Task 04: Install & Configure CORS & Cookie-based JWT Authentication
    - Install `django-cors-headers`, `djangorestframework-simplejwt`
    - Configure CORS_ALLOWED_ORIGINS, CSRF_TRUSTED_ORIGINS, and JWT cookie settings
    - Target Files: backend/gb_technologies/settings.py
    - Verification: Test pre-flight OPTIONS request returns correct CORS headers

[ ] Task 05: Build UserProfile & CurrentUser DRF Serializers
    - Create UserProfileSerializer with E.164 phone validator and avatar URL builder
    - Create CurrentUserSerializer with nested UserProfile read/write support
    - Target Files: backend/web/serializers.py, backend/web/tests/test_serializers.py
    - Verification: Serializer unit tests pass for valid and invalid payload validation

[ ] Task 06: Implement Authentication & Profile API Views (/api/v1/)
    - Implement LoginView, LogoutView, RegisterView, and CurrentUserView (GET, PATCH)
    - Wire endpoints in backend/web/urls.py under /api/v1/
    - Target Files: backend/web/views_api.py, backend/web/urls.py
    - Verification: Postman/cURL test returns 200 OK and sets secure HttpOnly auth cookie

[ ] Task 07: Initialize Next.js 14/15 App Router Project
    - Run `create-next-app` in /frontend with TypeScript, ESLint, App Router, and Tailwind
    - Configure tsconfig.json with strict type-checking and path aliases (@/*)
    - Target Files: frontend/package.json, frontend/tsconfig.json
    - Verification: `npm run build` inside /frontend succeeds with zero errors

[ ] Task 08: Implement OKLCH Color Tokens & Fluid Typography
    - Implement variables in frontend/app/globals.css based on color_tokens.md
    - Configure Tailwind v4 CSS theme extensions for fonts, radiuses, and shadows
    - Target Files: frontend/app/globals.css, frontend/tailwind.config.ts
    - Verification: Theme utility classes (.bg-canvas, .text-primary, .border-subtle) compile

[ ] Task 09: Install shadcn/ui & Build Shared Primitives
    - Initialize shadcn/ui CLI and install Button, Input, Form, Card, Avatar, Sheet
    - Build ThemeProvider and accessible ThemeToggle component
    - Target Files: frontend/components/ui/*, frontend/components/shared/theme-toggle.tsx
    - Verification: Visual verification of components in both Light and Dark mode states

[ ] Task 10: Build Client API Gateway & TanStack Query Provider
    - Implement typed Axios/Fetch client with cookie forwarding and 401 token refresh interceptor
    - Create QueryClientProvider wrapper in root frontend/app/layout.tsx
    - Target Files: frontend/lib/api-client.ts, frontend/app/providers.tsx
    - Verification: Client successfully fetches /api/v1/health/ and sets TanStack cache
```

---
*End of Migration Roadmap. All planning artifacts are complete and ready for executive review.*

# GB Cloud — Enterprise Cloud Platform

Enterprise decoupled platform for **GB Cloud**, featuring a high-performance **Next.js App Router** frontend paired with a headless **Django REST Framework** API.

- **Frontend**: Next.js 16 (App Router, React 19, TypeScript, Tailwind CSS v4, OKLCH design system, shadcn/ui primitives, Framer Motion)
- **Backend**: Headless Django 5 REST API (`/api/v1/`), OpenAPI 3.1 schema engine (`drf-spectacular`), secure HTTP-only cookie JWT authentication (`SimpleJWT`), and CORS controls.

---

## Architecture & Directory Structure

```
gbcloud/
├── backend/                  # Django REST API service
│   ├── web/                  # Core application (models, serializers, API views)
│   ├── webint/               # Django project settings & URL configuration
│   ├── manage.py             # Django management CLI
│   └── requirements.txt      # Python dependencies
├── frontend/                 # Next.js App Router front end
│   ├── app/                  # Routes (Marketing, Auth, Portal)
│   ├── components/           # UI primitives & design system components
│   ├── lib/                  # API client & utility functions
│   └── schemas/              # Zod data validation schemas
├── docs/                     # Design system & architecture specifications
│   └── architecture/         # OKLCH tokens, design system, implementation plans
├── .gitignore
└── README.md
```

---

## Quick Start (Local Development)

### 1. Backend Service (Django REST API)
```bash
cd backend
python -m venv venv
# On Windows:
.\venv\Scripts\activate
# On Linux/macOS:
source venv/bin/activate

pip install -r requirements.txt
python manage.py migrate
python manage.py runserver 8000
```
- API Base URL: `http://localhost:8000/api/v1/`
- Interactive Swagger UI: `http://localhost:8000/api/v1/docs/`
- OpenAPI Schema: `http://localhost:8000/api/v1/schema/`
- Health Check: `http://localhost:8000/api/v1/health/`

### 2. Frontend Application (Next.js)
```bash
cd frontend
npm install --legacy-peer-deps
npm run dev
```
- Web Application: `http://localhost:3000`

---

## Design System & Architecture Docs

Complete technical specifications and design guidelines are located in `docs/architecture/`:
- **Color System**: [Color Tokens (`color_tokens.md`)](docs/architecture/color_tokens.md) — OKLCH palette, WCAG 2.2 AA contrast ratios, and FOWT mitigation.
- **Design System**: [Design System (`design_system.md`)](docs/architecture/design_system.md) — Fluid clamp typography, 8-pt spacing grid, 8-state component specs, and responsive test matrices.
- **Implementation Plan**: [Implementation Plan (`implementation_plan.md`)](docs/architecture/implementation_plan.md) — REST API specifications, serializers, and security boundaries.
- **Migration Roadmap**: [Migration Roadmap (`migration_roadmap.md`)](docs/architecture/migration_roadmap.md) — Phased Strangler delivery schedule and proxy setup.

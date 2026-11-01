<!-- ───────────────────────── HERO BANNER ───────────────────────── -->
<div align="center">

<img src="app/static/ZSA.png" alt="Zun Synthesis Academy logo" width="130" />

# 🧬 Zun Synthesis Academy

### A premium ed-tech platform bridging biology and technology — courses, learners, certificates and financial aid in one FastAPI application

*Where life sciences meet code.*

<br/>

[![Language](https://img.shields.io/badge/Python_3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Framework](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Database](https://img.shields.io/badge/SQLite_%2F_PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![ORM](https://img.shields.io/badge/SQLAlchemy_2.0-D71F00?style=for-the-badge&logo=sqlalchemy&logoColor=white)](https://www.sqlalchemy.org/)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge&logo=opensourceinitiative&logoColor=white)](./LICENSE)
[![Status](https://img.shields.io/badge/Status-Active_Development-orange?style=for-the-badge&logo=statuspage&logoColor=white)](#-future-roadmap)
[![Author](https://img.shields.io/badge/Author-Md._Nafizur_Rahman-0F4C81?style=for-the-badge&logo=github&logoColor=white)](https://github.com/mdnafizurrahman8024)

<br/>

🔐 **JWT auth** · 👑 **Role-based admins** · 🎓 **Certificates** · 🎁 **Financial aid** · 🛡️ **Sanitized rich text** · 🔒 **Privacy-first analytics**

</div>

---

## 📖 Overview

**Zun Synthesis Academy** is a full-stack learning platform built on **FastAPI** + **SQLAlchemy 2.0**
that pairs a public marketing site with a complete course-management back office. Instructors publish
courses, learners enrol (or apply for financial aid), progress is tracked, and **certificates are
issued automatically at 100% completion** with a publicly verifiable code.

The stack is deliberately lean and dependency-light: **Jinja2** templates for the public page and
dashboard, a **JSON REST API** for everything programmatic, **PBKDF2** password hashing, **HS256
JWTs**, and **nh3** allowlist sanitisation for author-submitted HTML.

> 🎯 **Design goals** — same-origin by default (no permissive CORS), no card data ever stored, and
> analytics that record *what page* was visited but never *who* visited it.

---

## ✨ Key Features

### 🏛️ Course Management
- **📚 Full course lifecycle** — create, edit, publish and retire courses across three states:
  `upcoming`, `active`, `closed` (enforced by a database `CheckConstraint`).
- **🗂️ Program-linked catalogue** — every course belongs to a `program_slug` with its own syllabus
  JSON, instructor and schedule.
- **💵 Dual pricing** — `regular_price_cents` plus optional `offer_price_cents`, with an automatic
  `discount_badge_text` and `has_offer` flag computed in the API response. Prices are **integer
  minor units** in the API, entered in **major units** in the dashboard.
- **🖼️ Rich-text course editor** — basic formatting, links, uploaded images and
  **YouTube/Vimeo embeds**, sanitised with an [nh3](https://nh3.readthedocs.io/en/stable/index.html)
  allowlist on **both write and read**.

### 🎓 Learner Experience
- **📝 Self-service registration** via the site's **Join Now** form or `POST /api/students/register`.
- **🎫 Token-based enrolment** — `POST /api/courses/{id}/enroll` requires a learner bearer token.
- **🎁 Financial aid workflow** — learners apply, admins approve or reject; approved aid unlocks
  enrolment.
- **📊 Personal dashboard** — `/api/students/me/dashboard` exposes enrolled courses, progress,
  certificates, mock invoice history and pending mock checkouts.
- **🏅 Certificates** — issued the moment progress reaches **100%**, uniquely keyed per enrollment,
  verifiable at `/api/certificates/{code}`, and downloadable from the authenticated dashboard.

### 👑 Administration & Roles
- **Two-tier admin model** — **Super Admin** and staff **admin**.
  - *Super Admin* is the first account created by `python -m app.create_admin`; on an existing
    database, startup promotes the first admin only when no Super Admin exists.
  - Only Super Admin can open **Manage admins** or call `GET/POST /api/admin/users` and
    `PATCH /api/admin/users/{id}`.
  - Staff admins can manage courses and review learners but **cannot create or disable admin
    accounts**.
- **🛡️ Safety rails** — admin passwords must be at least **12 characters**, and the **final active
  Super Admin cannot be disabled** (so the platform can never lock itself out).
- **📈 Analytics dashboard** — rolling hourly/daily aggregates and recent page paths from
  `GET /api/admin/analytics`.

### 🔐 Security & Privacy
- **🔑 PBKDF2-SHA256 password hashing** — 16-byte random salt, **310,000 iterations**, stored as
  `pbkdf2_sha256$iters$salt$digest` and compared with `hmac.compare_digest` (timing-safe).
- **🎫 HS256 JWTs** with a configurable `JWT_EXPIRE_MINUTES` (default **30**), rejected if the
  signing secret is under **32 characters** — the server **refuses to boot** otherwise.
- **🧼 HTML sanitisation on read too** — stored course HTML is re-sanitised on the way out, so a
  tampered database row still cannot inject scripts, event handlers or third-party iframes.
- **🖼️ Hardened image uploads** — re-encoded to **WebP** (quality 86), **8 MB** cap, JPEG/PNG/WebP
  only, and `Image.MAX_IMAGE_PIXELS` bounded at 30M to stop decompression bombs.
- **👁️ Privacy-first analytics** — `visit_events` stores **only `page_path` and a timestamp**: no
  IP addresses, no cookies, no user agents, no account identifiers.
- **🌐 Hardened hosting** — `TrustedHostMiddleware` when `ALLOWED_HOSTS` is explicit, optional
  `HTTPSRedirectMiddleware` via `FORCE_HTTPS`, and **no permissive CORS** (same-origin by design).

### 🖥️ Public Site
- **🧩 API-driven content** — programs and stats are stored as JSON payloads by `kind`
  (`program`, `stat`, …) and rendered dynamically.
- **🪂 Graceful degradation** — the public page keeps its original hand-authored markup if the API
  cannot be reached.

---

## 🛠️ Technology Stack

| Layer | Technology |
| :--- | :--- |
| **Language** | Python 3.10+ |
| **Web Framework** | FastAPI `>=0.115` |
| **ASGI Server** | Uvicorn `[standard]` |
| **ORM** | SQLAlchemy 2.0 (typed `Mapped[...]` models) |
| **Database** | SQLite (default) · PostgreSQL via `psycopg[binary]` 3 |
| **Auth** | PyJWT (HS256) + PBKDF2-SHA256 (`hashlib` + `hmac`) |
| **Templating** | Jinja2 |
| **HTML Sanitisation** | nh3 (allowlist) |
| **Images** | Pillow (WebP re-encoding) |
| **Uploads** | python-multipart |
| **Middleware** | Starlette `TrustedHostMiddleware`, `HTTPSRedirectMiddleware` |

---

## 🏗️ Architecture & Directory Tree

```
Zun-Synthesis-Academy-Website/
├── 🚀 app/
│   ├── main.py                    # FastAPI app · lifespan checks · routers · static mounts
│   ├── settings.py                # Env-driven Settings class (12 configurable values)
│   ├── database.py                # Engine · SessionLocal · init_db() · additive migrations
│   ├── security.py                # require_admin / require_super_admin / require_student
│   ├── create_admin.py            # CLI: create the first (Super) administrator
│   │
│   ├── 🗄️ models/                 # SQLAlchemy 2.0 models
│   │   ├── admin.py               #   AdminUser (role · is_active)
│   │   ├── student.py · profile.py
│   │   ├── course.py              #   Course (status · syllabus · pricing)
│   │   ├── enrollment.py · course_progress.py
│   │   ├── certificate.py         #   Certificate (verification_code)
│   │   ├── financial_aid.py · payment.py
│   │   ├── content.py             #   ContentItem (JSON payloads by kind)
│   │   └── analytics.py           #   VisitEvent (page_path + timestamp only)
│   │
│   ├── 🛣️ routes/                 # API & page routers
│   │   ├── pages.py               #   Public site
│   │   ├── admin.py               #   /admin dashboard page
│   │   ├── api.py                 #   Content & analytics endpoints
│   │   ├── courses.py             #   Course CRUD · enrol · aid · progress
│   │   ├── admin_users.py         #   Super-Admin-only user management
│   │   ├── students.py · student_dashboard.py
│   │   └── media.py               #   Image upload (WebP re-encode)
│   │
│   ├── 🧩 schemas/                # Pydantic request/response models
│   ├── 🧠 services/               # auth · tokens · course_html · default_content
│   ├── 🎨 templates/              # index.html · admin.html
│   └── 📁 static/                 # Logo assets + /uploads media directory
│
├── 📄 index.html                  # Hand-authored landing page (static fallback)
├── 🖼️ demo.png · ZSA.jpg · zun.png
├── 📦 requirements.txt
├── 📜 LICENSE                     # MIT © 2026 Md Nafizur Rahman
└── 📖 README.md
```

### 🔄 Request Flow

```
Browser ──▶ FastAPI (app/main.py)
              ├── TrustedHostMiddleware     (if ALLOWED_HOSTS ≠ "*")
              ├── HTTPSRedirectMiddleware   (if FORCE_HTTPS=true)
              │
              ├── /            → pages.py            (Jinja2 public site)
              ├── /admin       → admin.py            (course dashboard)
              ├── /api/...     → api · courses · students · admin_users
              ├── /uploads/*   → uploaded WebP images (MEDIA_UPLOAD_DIR)
              └── /*           → app/static (logos, assets)
                        │
                SQLite / PostgreSQL (SQLAlchemy 2.0)
```

> ⚙️ **Startup contract** — `lifespan` first validates that `JWT_SECRET` is ≥ 32 chars (else
> `RuntimeError`), then calls `init_db()`, which creates tables, applies **additive column
> migrations**, and promotes the first admin to Super Admin when needed.

---

## 🚀 Local Setup (PowerShell)

### ✅ Prerequisites

- **Python 3.10+**
- Optional: a PostgreSQL server if you do not want SQLite

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

## Create the first administrator

This command prompts for the email and password and stores only a **PBKDF2 password hash**:

```powershell
python -m app.create_admin
```

The account it creates becomes the **Super Admin**.

## Set a private JWT secret, then start FastAPI

```powershell
$env:JWT_SECRET = (py -c "import secrets; print(secrets.token_urlsafe(48))")
uvicorn app.main:app --reload
```

Set `DATABASE_URL` to a PostgreSQL SQLAlchemy URL such as
`postgresql+psycopg://user:password@host/database` to use PostgreSQL instead of the default SQLite
database.

> ⚠️ The generated JWT secret is **specific to that PowerShell process**. Set it again when starting
> a new process, or configure a persistent secret in your deployment environment.

---

## 🌐 Using the Application

### Public site & interactive docs

| URL | Purpose |
| :--- | :--- |
| `http://127.0.0.1:8000` | Public site |
| `http://127.0.0.1:8000/docs` | Interactive OpenAPI documentation |
| `http://127.0.0.1:8000/admin` | Course dashboard (admin login required) |

Log in with `POST /api/admin/login` using JSON `{ "email": "...", "password": "..." }`, then use the
returned **bearer token** for `/api/admin/content` and `/api/admin/analytics` endpoints.

### Course dashboard — `http://127.0.0.1:8000/admin`

It supports course creation, edits, lifecycle status changes, learner progress tracking, certificate
issuance at 100% completion, analytics, and financial aid approval/rejection.

Learner accounts can be created through the site's **Join Now** form or
`POST /api/students/register`. Course enrollment requires a learner token.

> 💳 Paid course enrollment uses a **mock confirmation only** — it is **not** a payment processor and
> stores no card details.

---

## 📡 API Reference

| Method | Endpoint | Access | Purpose |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/admin/login` | Public | Obtain an admin bearer token |
| `POST` | `/api/students/register` | Public | Create a learner account |
| `GET` | `/api/courses` | Public | List courses |
| `GET` | `/api/admin/content` | Admin | Read site content payloads |
| `GET` | `/api/admin/analytics` | Admin | Hourly/daily aggregates + recent paths |
| `GET/POST` | `/api/admin/courses` | Admin | List / create courses |
| `PUT/PATCH/DELETE` | `/api/admin/courses/{id}` | Admin | Update / delete a course |
| `POST` | `/api/admin/media/images` | Admin | Upload image (multipart field `file`) |
| `POST` | `/api/courses/{id}/enroll` | Learner | Enrol in a course |
| `POST` | `/api/courses/{id}/financial-aid` | Learner | Request financial aid |
| `GET` | `/api/admin/financial-aid` | Admin | Review aid requests |
| `PATCH` | `/api/admin/financial-aid/{id}` | Admin | Set `status` to `approved` or `rejected` |
| `GET` | `/api/students/me/dashboard` | Learner | Enrolments, progress, certificates, invoices |
| `PATCH` | `/api/admin/enrollments/{id}/progress` | Admin | Update learner progress |
| `GET` | `/api/certificates/{code}` | Public | Verify a certificate |
| `GET/POST` | `/api/admin/users` | **Super Admin** | List / create admin accounts |
| `PATCH` | `/api/admin/users/{id}` | **Super Admin** | Update or disable an admin |

**Image upload example**

```bash
curl -X POST http://127.0.0.1:8000/api/admin/media/images \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@cover.png"
```

```json
{ "url": "/uploads/9f3c....webp", "content_type": "image/webp" }
```

---

## 📦 Content Model

Content is stored as **JSON payloads by `kind`** (`program`, `stat`, or other content types).

| Kind | Supported fields |
| :--- | :--- |
| `program` | `title`, `description`, `icon`, `accent`, `image_url`, `image_alt`, `video_url`, `external_links` |
| `stat` | `value`, `label` |

The public page reads program and stat content from the API and **retains its original hand-authored
markup if the API cannot be reached**.

---

## 🛡️ Admin Roles, Course Media & Production Configuration

### Admin roles

The first administrator created by `python -m app.create_admin` is the **Super Admin**. On an existing
database, startup promotes the first admin **only when no Super Admin exists**.

- Only that role can open **Manage admins** or use `GET/POST /api/admin/users` and
  `PATCH /api/admin/users/{id}`.
- **Staff admins** can manage courses and review learners but cannot create or disable admin accounts.
- Admin passwords must be at least **12 characters**.
- The final active Super Admin **cannot be disabled**.
- Existing database installations receive **additive columns** for these roles and course pricing
  fields at startup.

### Course media

The course editor supports basic rich text, links, uploaded images, and YouTube/Vimeo embeds.
Course HTML is sanitized on write **and** read; the sanitizer uses an allowlist to remove unsafe
markup and attributes ([nh3 documentation](https://nh3.readthedocs.io/en/stable/index.html)).

Image uploads are re-encoded as **WebP** and saved under `MEDIA_UPLOAD_DIR`
(default `app/static/uploads`), served at `/uploads/`. **Configure this variable to a persistent
volume in production.** Allowed upload formats are **JPEG, PNG, and WebP**, with an **8 MB** upload
limit.

Course prices are **integer minor units** in the API and entered in **major units** in the dashboard;
`price_cents` remains the effective enrollment price for compatibility, while regular and offer
prices are separately available in course responses.

> 🛡️ The editor's formatted descriptions allow safe links, images, and YouTube/Vimeo iframe embeds;
> arbitrary scripts, event handlers, and third-party iframe hosts are **stripped**.

### ⚙️ Environment Variables

| Variable | Default | Purpose |
| :--- | :--- | :--- |
| `JWT_SECRET` | *(required, ≥ 32 chars)* | HS256 signing key — server refuses to start without it |
| `DATABASE_URL` | `sqlite:///academy.db` | SQLAlchemy URL (SQLite or PostgreSQL) |
| `MEDIA_UPLOAD_DIR` | `app/static/uploads` | Where uploaded WebP images are written |
| `ALLOWED_HOSTS` | `*` | Comma-separated hostnames → enables `TrustedHostMiddleware` |
| `FORCE_HTTPS` | `false` | `true` → enables `HTTPSRedirectMiddleware` |
| `JWT_EXPIRE_MINUTES` | `30` | Token lifetime |

### 🔒 Production checklist

For a public deployment:

1. Configure `JWT_SECRET` with a **persistent random secret of at least 32 characters**.
2. Set `DATABASE_URL` to **PostgreSQL**.
3. Set `ALLOWED_HOSTS` to comma-separated public hostnames (e.g.
   `academy.example.com,www.academy.example.com`).
4. Set `MEDIA_UPLOAD_DIR` to **persistent storage**.
5. Set `FORCE_HTTPS=true` when the app directly receives HTTPS **or** your reverse proxy forwards the
   original HTTPS scheme correctly.
6. **Terminate TLS at the edge**, restrict database access, and keep the JWT secret and database
   credentials in the hosting provider's **secret manager**.

> 🌐 The API is **same-origin** with the site and does **not** enable permissive cross-origin access.

---

## 🗺️ Future Roadmap

| Status | Milestone |
| :--- | :--- |
| ✅ Done | FastAPI + SQLAlchemy 2.0 backend with Jinja2 public site |
| ✅ Done | JWT auth with PBKDF2-SHA256 hashing (310k iterations) |
| ✅ Done | Two-tier admin roles with lockout protection |
| ✅ Done | Course CRUD, dual pricing and lifecycle statuses |
| ✅ Done | Enrolment, progress tracking and auto-issued certificates |
| ✅ Done | Financial aid approval workflow |
| ✅ Done | Allowlist HTML sanitisation + hardened WebP uploads |
| ✅ Done | Privacy-first visit analytics |
| 🔄 In progress | Real payment processor (replacing mock checkout & invoices) |
| 🧭 Planned | Certificate PDF rendering with academy branding |
| 🧭 Planned | Email notifications for enrolment, approval and completion |
| 🧭 Planned | Learner discussion threads per lesson |
| 🧭 Planned | Alembic migrations replacing the startup additive-column pass |
| 🧭 Planned | Automated test suite (pytest + httpx) in CI |

---

## 🤝 Contributing

```bash
git checkout -b feature/your-feature-name
git commit -m "feat: describe your change"
git push origin feature/your-feature-name
```

Then open a **Pull Request**. Licensed under **MIT** — see [LICENSE](./LICENSE).

---

<br/>

<!-- ───────────────────────── AUTHOR ───────────────────────── -->
<div align="center">

## 👨‍🔬 Author & Connect

### Md. Nafizur Rahman

**Undergraduate Student** · Faculty of Veterinary and Animal Sciences<br/>
University of Rajshahi, Rajshahi-6205, Bangladesh

<br/>

[![Email](https://img.shields.io/badge/Email-s2210968138%40ru.ac.bd-0F4C81?style=for-the-badge&logo=gmail&logoColor=white)](mailto:s2210968138@ru.ac.bd)
[![WhatsApp](https://img.shields.io/badge/WhatsApp-%2B8801949288024-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)](https://wa.me/8801949288024)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-md--nafizur--rahman--ru-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](http://www.linkedin.com/in/md-nafizur-rahman-ru)
[![GitHub](https://img.shields.io/badge/GitHub-mdnafizurrahman8024-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/mdnafizurrahman8024)

<br/>

[![CV](https://img.shields.io/badge/📄_Curriculum_Vitae-View-1B4D8C?style=for-the-badge&logo=googledrive&logoColor=white)](https://drive.google.com/file/d/106-eTtiqsvXZ9PIY_shcyWJX9kR66uq9/view?usp=drive_link)
[![Portfolio](https://img.shields.io/badge/🌐_Portfolio-Explore-5AA9E6?style=for-the-badge&logo=googlechrome&logoColor=white)](https://sites.google.com/view/md-nafizur-rahman/home)
[![Facebook](https://img.shields.io/badge/Facebook-mdnafizurrahman8024-1877F2?style=for-the-badge&logo=facebook&logoColor=white)](https://www.facebook.com/mdnafizurrahman8024/)

<br/>

**Research Identifiers**

[![ORCID](https://img.shields.io/badge/ORCID-0009--0008--3918--187X-A6CE39?style=for-the-badge&logo=orcid&logoColor=white)](https://orcid.org/0009-0008-3918-187X)
[![Google Scholar](https://img.shields.io/badge/Google_Scholar-bImpEzMAAAAJ-4285F4?style=for-the-badge&logo=googlescholar&logoColor=white)](https://scholar.google.com/citations?user=bImpEzMAAAAJ&hl=en)
[![ResearchGate](https://img.shields.io/badge/ResearchGate-Md--Rahman--1097-00CCBB?style=for-the-badge&logo=researchgate&logoColor=white)](https://www.researchgate.net/profile/Md-Rahman-1097)
[![OSF](https://img.shields.io/badge/OSF-10.17605%2FOSF.IO%2F9AQS3-2F3136?style=for-the-badge&logo=osf&logoColor=white)](https://doi.org/10.17605/OSF.IO/9AQS3)

<br/>

**Contact**

📧 [s2210968138@ru.ac.bd](mailto:s2210968138@ru.ac.bd) &nbsp;·&nbsp;
📞 [+8801949288024](https://wa.me/8801949288024) &nbsp;·&nbsp;
📍 Rajshahi-6205, Bangladesh

</div>

<br/>

<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:1C4C34,50:2E8B57,100:5AA9E6&height=110&section=footer&animation=fadeIn" alt="" />

<div align="center">

<sub>⭐ <i>Where life sciences meet code — learn biology, build with technology.</i></sub>

</div>

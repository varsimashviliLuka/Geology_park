# Geological Museum — Flask App

A bilingual (English / Georgian) museum web app for displaying geological specimens. Each specimen has a stable public URL and a QR code for physical banners in the field.

## Features

### Public site
- **Home page** (`/`) — grid of active specimen cards
- **Banner-style detail pages** (`/1`, `/2`, …) — dark theme with yellow accents, matching physical banner layout
- **Bilingual content** — English and Georgian titles and descriptions
- **Elevation badge** — optional altitude shown on the public page
- **Three photo slots** — rock outcrop, location map, and Cenozoic timescale chart

### Admin panel (`/admin`, login required)
- **Dashboard** — list all specimens with status, QR preview, and actions
- **Create / edit** — titles, descriptions, elevation, visibility, and custom specimen IDs
- **Instant photo management** — upload and remove photos on the edit page without saving the form
- **Hide specimens** — set inactive to remove from the home page while keeping the detail URL
- **Delete specimens** — permanently removes the record, photos, and QR file
- **QR codes** — auto-generated on creation (white modules, transparent background)
- **Regenerate QR codes** — bulk action on the dashboard to rebuild all QR images
- **QR preview popup** — view and download QR codes from the dashboard

### Data & files
- **Optional fields** — photos, titles, and descriptions are all optional
- **Stable IDs** — specimen IDs are used in URLs and QR codes; IDs can be reassigned on edit
- **File cleanup** — deleting a specimen or removing a photo deletes files from disk

## Quick Start

```bash
# 1. Create virtual environment
python3 -m venv .venv
source .venv/bin/activate    # Windows: .venv\Scripts\activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure environment
cp .env.example .env
# Edit .env — set FLASK_SECRET_KEY, ADMIN_PASSWORD, BASE_URL

# 4. Run
python run.py
```

Open http://localhost:5000 for the collection.  
Admin login: http://localhost:5000/auth/login (redirects to `/admin` after login).

On first run, SQLite creates `instance/museum.db` and upload folders under `app/static/uploads/`.

## Project Structure

```
app/
├── __init__.py              # App factory, DB init, lightweight schema migrations
├── config.py                # Settings from .env
├── extensions.py            # db, login_manager, csrf
├── models.py                # Artifact and AdminUser
├── auth/routes.py           # Login / logout
├── main/routes.py           # Home + specimen detail
├── admin/
│   ├── routes.py            # CRUD, photo API, QR regeneration
│   └── forms.py             # WTForms validation
├── services/
│   ├── artifact_service.py  # Delete specimen, change ID
│   ├── file_service.py      # Upload & delete images
│   └── qr_service.py        # QR code generation
├── templates/
│   ├── base.html            # Standard site layout
│   ├── base_banner.html     # Minimal layout for banner detail pages
│   ├── main/                # Public pages
│   ├── admin/               # Admin dashboard and forms
│   └── auth/                # Login
└── static/
    ├── css/style.css
    ├── js/admin.js          # QR preview modal on dashboard
    ├── js/admin-form.js     # Instant photo upload/remove on edit page
    ├── img/                 # Static assets (logo, timescale SVG)
    └── uploads/             # User photos and generated QR codes
        ├── photos/
        └── qrcodes/
instance/museum.db           # SQLite database (auto-created, gitignored)
run.py                       # Entry point
.env.example                 # Environment template
```

## Configuration (.env)

| Variable | Description |
|----------|-------------|
| `FLASK_DEBUG` | `1` for dev auto-reload, `0` for production |
| `FLASK_SECRET_KEY` | Session signing key (use a long random string) |
| `ADMIN_USERNAME` | Admin login username |
| `ADMIN_PASSWORD` | Admin login password |
| `BASE_URL` | Public URL embedded in QR codes (e.g. `https://museum.example.com`) |
| `SITE_FOOTER_LABEL` | Footer text on public pages |
| `SITE_FOOTER_URL` | Footer link URL |
| `DATABASE_URL` | SQLAlchemy URI (optional; defaults to SQLite in `instance/`) |

QR appearance is configured in `app/config.py` (`QR_FILL_COLOR`, `QR_BACK_COLOR`).

## Admin Workflow

1. **Create** a specimen (optionally set a custom ID). You are redirected to the edit page.
2. **Upload photos** on the edit page — each slot uploads immediately when you click **Upload**.
3. **Remove photos** with the **Remove** button (no need to click **Save**).
4. **Save** text fields, elevation, visibility, and specimen ID changes with the main **Save** button.
5. **Preview** the public banner page from the dashboard, or open the QR popup to download the code.

Changing a specimen ID regenerates its QR code. Hidden specimens (`is_active = false`) are excluded from the home page but return 404 on their public URL.

## How to Modify

### Change styling
Edit `app/static/css/style.css`. CSS variables at the top control colors and fonts.

### Change page layout
- Home: `app/templates/main/index.html`
- Specimen banner page: `app/templates/main/artifact.html`
- Admin: `app/templates/admin/`

### Add a new field to specimens
1. Add a column in `app/models.py` (`Artifact` class)
2. Add a form field in `app/admin/forms.py`
3. Handle it in `app/admin/routes.py` (create + edit)
4. Display it in `app/templates/main/artifact.html` and admin templates
5. For simple SQLite column additions, add a migration step in `_ensure_schema_updates()` in `app/__init__.py` (see existing `elevation_m` / `photo3_path` examples). For production, prefer Flask-Migrate.

### Change QR code appearance or target URL
- URL: set `BASE_URL` in `.env`
- Colors: edit `QR_FILL_COLOR` and `QR_BACK_COLOR` in `app/config.py`, then use **Regenerate QR Codes** on the admin dashboard

### Change authentication
Credentials are in `.env` (`ADMIN_USERNAME`, `ADMIN_PASSWORD`).  
Logic is in `app/auth/routes.py` and `app/extensions.py`.

### Switch to PostgreSQL (production)
Set in `.env`:
```
DATABASE_URL=postgresql://user:pass@localhost/museum
```

## Security

- CSRF protection on all forms and AJAX photo endpoints (Flask-WTF)
- Admin routes require login (`@login_required`)
- File uploads restricted by extension and size (10 MB max)
- Secrets in `.env` (never commit `.env`)

## Production Notes

- Set `FLASK_DEBUG=0`
- Use a strong `FLASK_SECRET_KEY` and `ADMIN_PASSWORD`
- Serve with Gunicorn + Nginx (or similar)
- Set `BASE_URL` to your real domain before generating QR codes
- Back up `instance/museum.db` and `app/static/uploads/`

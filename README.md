# FastAPI Blog

A full-stack blogging platform built with **FastAPI**, **SQLAlchemy**, and **Jinja2** templates. Features user authentication, CRUD operations for posts, profile picture uploads to AWS S3, password reset via email, dark/light theme toggling, and paginated feeds — all served through a responsive Bootstrap 5 UI.

🔗 **Live Demo:** [fastapi-blog-beryl.vercel.app](https://fastapi-blog-beryl.vercel.app/)

![FastAPI Blog Homepage](media/screenshot.png)

---

## ✨ Features

- **User Authentication** — Register, login, and manage sessions with JWT-based access tokens (OAuth2 Bearer flow)
- **Blog Posts** — Full CRUD: create, read, update (PUT & PATCH), and delete posts with ownership enforcement
- **Profile Pictures** — Upload, resize, and store profile images on AWS S3 (auto-cropped to 300×300 JPEG via Pillow)
- **Password Management** — Change password from your account, or reset a forgotten password via email with secure tokenized links
- **Pagination** — Server-rendered first page with "Load More" infinite-scroll style loading via the REST API
- **Dark / Light / Auto Theme** — Toggle between themes with `localStorage` persistence and system preference detection
- **Responsive Design** — Mobile-friendly layout powered by Bootstrap 5 with a collapsible navbar
- **Per-User Post Feeds** — View all posts by a specific user on their dedicated profile page
- **Error Handling** — Custom error pages for HTML views and structured JSON error responses for API clients

---

## 🛠️ Tech Stack

| Layer         | Technology                                                                 |
| ------------- | -------------------------------------------------------------------------- |
| **Framework** | [FastAPI](https://fastapi.tiangolo.com/) (async, with Uvicorn)             |
| **Database**  | PostgreSQL (async via [SQLAlchemy 2.0](https://www.sqlalchemy.org/) + asyncpg / psycopg) |
| **ORM**       | SQLAlchemy 2.0 with `Mapped` type annotations                             |
| **Migrations**| [Alembic](https://alembic.sqlalchemy.org/)                                 |
| **Auth**      | JWT tokens via [PyJWT](https://pyjwt.readthedocs.io/), Argon2 password hashing via [pwdlib](https://github.com/AJAnderson/pwdlib) |
| **Templating**| [Jinja2](https://jinja.palletsprojects.com/)                               |
| **Frontend**  | Bootstrap 5, vanilla JavaScript (ES modules)                               |
| **Storage**   | AWS S3 (profile picture uploads via [Boto3](https://boto3.amazonaws.com/)) |
| **Email**     | Async SMTP via [aiosmtplib](https://aiosmtplib.readthedocs.io/)           |
| **Imaging**   | [Pillow](https://pillow.readthedocs.io/) for image processing             |
| **Validation**| [Pydantic v2](https://docs.pydantic.dev/) with `pydantic-settings`        |
| **Testing**   | [pytest](https://docs.pytest.org/) with [moto](https://github.com/getmoto/moto) for S3 mocking |
| **Deployment**| [Vercel](https://vercel.com/)                                              |

---

## 📁 Project Structure

```
fastapi_blog/
├── main.py              # FastAPI app, page routes, exception handlers
├── models.py            # SQLAlchemy models (User, Post, PasswordResetToken)
├── schemas.py           # Pydantic request/response schemas
├── database.py          # Async engine & session setup
├── config.py            # pydantic-settings configuration (loads from .env)
├── auth.py              # JWT creation/verification, password hashing, dependencies
├── image_utils.py       # Profile image processing & S3 upload/delete
├── email_utils.py       # Async email sending (password reset)
├── populate_db.py       # Database seeding script
├── routers/
│   ├── posts.py         # /api/posts — CRUD endpoints for blog posts
│   └── users.py         # /api/users — Registration, login, profile, password reset
├── templates/
│   ├── layout.html      # Base template with navbar, footer, modals
│   ├── home.html        # Homepage with paginated post feed
│   ├── post.html        # Individual post detail page
│   ├── login.html       # Login form
│   ├── register.html    # Registration form
│   ├── account.html     # Account settings (profile pic, username, email, password)
│   ├── user_posts.html  # Posts filtered by author
│   ├── forgot_password.html
│   ├── reset_password.html
│   ├── error.html       # Custom error page
│   └── email/           # HTML email templates
├── static/
│   ├── css/             # Stylesheets
│   ├── js/              # Client-side JavaScript (auth, utils)
│   ├── icons/           # Favicons and PWA icons
│   └── profile_pics/    # Default profile picture
├── alembic/             # Database migration scripts
├── tests/
│   ├── conftest.py      # Fixtures (test DB, S3 mock, test client)
│   ├── test_posts.py    # Post endpoint tests
│   └── test_users.py    # User endpoint tests
├── pyproject.toml       # Project metadata & dependencies
└── requirements.txt     # Pinned dependencies
```

---

## 🚀 Getting Started

### Prerequisites

- **Python 3.13+**
- **PostgreSQL** (or SQLite for local development)
- **AWS S3 bucket** (for profile picture storage)
- An SMTP server (for password reset emails — optional for local dev)

### 1. Clone the Repository

```bash
git clone https://github.com/RonenTGreat/fastapi_blog.git
cd fastapi_blog
```

### 2. Create a Virtual Environment

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS/Linux
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

Or with [uv](https://docs.astral.sh/uv/):

```bash
uv sync
```

### 4. Configure Environment Variables

Create a `.env` file in the project root:

```env
DATABASE_URL=postgresql+psycopg://user:password@localhost:5432/fastapi_blog
SECRET_KEY=your-secret-key-here

# S3 Configuration
S3_BUCKET_NAME=your-bucket-name
S3_REGION=eu-north-1
S3_ACCESS_KEY_ID=your-access-key
S3_SECRET_ACCESS_KEY=your-secret-key

# Email (optional for local dev)
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_FROM=noreply@yourdomain.com
MAIL_USE_TLS=true

FRONTEND_URL=http://localhost:8000
```

### 5. Run Database Migrations

```bash
alembic upgrade head
```

### 6. (Optional) Seed the Database

```bash
python populate_db.py
```

### 7. Start the Development Server

```bash
fastapi dev main.py
```

The app will be available at **http://localhost:8000**.

---

## 📡 API Endpoints

The REST API is served under the `/api` prefix. Interactive API documentation is available at `/docs` (Swagger UI) and `/redocs` (ReDoc).

### Posts (`/api/posts`)

| Method   | Endpoint             | Description              | Auth Required |
| -------- | -------------------- | ------------------------ | :-----------: |
| `GET`    | `/api/posts`         | List posts (paginated)   |      No       |
| `POST`   | `/api/posts`         | Create a new post        |      Yes      |
| `GET`    | `/api/posts/{id}`    | Get a single post        |      No       |
| `PUT`    | `/api/posts/{id}`    | Full update of a post    |      Yes      |
| `PATCH`  | `/api/posts/{id}`    | Partial update of a post |      Yes      |
| `DELETE` | `/api/posts/{id}`    | Delete a post            |      Yes      |

### Users (`/api/users`)

| Method   | Endpoint                    | Description                     | Auth Required |
| -------- | --------------------------- | ------------------------------- | :-----------: |
| `POST`   | `/api/users`                | Register a new user             |      No       |
| `POST`   | `/api/users/token`          | Login (returns JWT)             |      No       |
| `GET`    | `/api/users/me`             | Get current user profile        |      Yes      |
| `GET`    | `/api/users/{id}`           | Get public user profile         |      No       |
| `PATCH`  | `/api/users/{id}`           | Update username/email           |      Yes      |
| `DELETE` | `/api/users/{id}`           | Delete user account             |      Yes      |
| `PATCH`  | `/api/users/{id}/picture`   | Upload profile picture          |      Yes      |
| `DELETE` | `/api/users/{id}/picture`   | Remove profile picture          |      Yes      |
| `GET`    | `/api/users/{id}/posts`     | Get posts by user (paginated)   |      No       |
| `PATCH`  | `/api/users/me/password`    | Change password                 |      Yes      |
| `POST`   | `/api/users/forgot-password`| Request password reset email    |      No       |
| `POST`   | `/api/users/reset-password` | Reset password with token       |      No       |

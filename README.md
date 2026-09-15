# chatbotapp.github.io

A Flask + Socket.IO real-time group chat server with registration, login and an admin page, backed
by PostgreSQL, with Alembic migrations.

> **Two things the name implies but are not true.**
> 1. **This is not a GitHub Pages site.** The `*.github.io` name suggests static hosting, but the
>    repository contains a Flask server that needs Python and PostgreSQL. GitHub Pages serves static
>    files only and cannot run it. The root-level `admin.html`, `chat.html`, `index.html`,
>    `login.html` and `register.html` are copies of the Jinja templates and still contain unrendered
>    `{{ url_for(...) }}` tags, so they do not work as a static site either.
> 2. **There is no chatbot.** No NLP, no AI, no automated replies. Messages are broadcast between
>    human users.

This is one of three near-identical copies of the same project on this account, alongside
[`chatbot`](https://github.com/Haroldke13/chatbot) (minimal) and
[`chatapp`](https://github.com/Haroldke13/chatapp). `app.py` here is byte-identical to `chatapp`'s.
**Consider keeping one and archiving the others.** This copy's distinguishing feature is the
`migrations/` directory.

## What it does

`app.py` (~145 lines):

- `/` — landing page
- `/register` — create an account; bcrypt password hashing; all new users get the role `User`
- `/login`, `/logout` — Flask-Login sessions
- `/chat` — the chat room (login required)
- `/admin` — all users and all chat history, gated by a `role_required('Admin')` decorator

Socket.IO: the browser emits `send_message`, the server stores it in `chathistory` and re-emits
`receive_message` to **every** connected client. One global room; no private messaging, no history
replay on load.

`migrations/versions/e8e70239cc75_initial_migration.py` creates the `user` and `chathistory` tables.

## Tech stack

Python 3.12, Flask, Flask-SocketIO, Flask-SQLAlchemy, Flask-Login, Flask-Bcrypt,
Flask-Migrate/Alembic, `psycopg2` (PostgreSQL), Bootstrap 5 and the Socket.IO 4.0 client from a CDN.

## Setup

```bash
git clone https://github.com/Haroldke13/chatbotapp.github.io.git
cd chatbotapp.github.io
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
```

`requirements.txt` is unpinned and does not list `eventlet` or `gevent`, which Flask-SocketIO
generally needs; you may have to `pip install eventlet` too.

### PostgreSQL is required

`app.py` hardcodes:

```python
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://postgres:password@localhost/chatbot_db'
```

Create a `chatbot_db` database and either match those credentials or edit the line.

```bash
flask db upgrade      # apply the committed migration
python app.py         # → http://127.0.0.1:5000
```

`app.py` also calls `db.create_all()` at startup, so the migration is belt-and-braces.

## Repository hygiene

- **`venv/` is committed** — 2,605 tracked files, ~54 MB. Remove it and add a `.gitignore`; there
  isn't one.
- `__pycache__/*.pyc` and `migrations/**/__pycache__/*.pyc` are committed.
- The five root-level `.html` files duplicate `templates/` and should be deleted.
- `app.py` redefines `User` and `ChatHistory` locally after importing them from `user.py` and
  `db.py`, making those modules dead code.

## Status

**Working prototype.** Last commit December 2024. The chat, auth and admin flows work against a
local PostgreSQL instance. No tests.

## Licence

**Proprietary software — all rights reserved.** Copyright © 2026 Joel Harold Onyango.

This repository is not open source. The full terms are in [LICENSE](LICENSE); in
summary, you may not copy, redistribute, modify, sublicense, publish, re-host or
commercially exploit this software, in whole or in part, without the prior
written permission of the copyright holder. Access to this repository does not
grant any licence beyond reading it.

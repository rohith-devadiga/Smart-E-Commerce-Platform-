# Smartshop: FastAPI (customers) + Django (admin), prices in INR

Both apps share one SQLite database (`db.sqlite3`). Django owns the schema and migrations; FastAPI reads and writes the same tables.


## Windows quick start (PowerShell)
1. Install Python 3.10+ from python.org and tick "Add python.exe to PATH".
2. Open PowerShell in this folder (Shift + right-click > "Open in Terminal").
3. `py -m venv venv`
4. `Set-ExecutionPolicy -Scope Process Bypass` then `.\venv\Scripts\Activate.ps1` (in Command Prompt use `venv\Scripts\activate.bat`)
5. `pip install -r requirements.txt`
6. `copy .env.example .env`
7. `python manage.py makemigrations shop` then `python manage.py migrate` then `python seed.py`
8. Open three PowerShell windows, activate the venv in each, and run:
   - `python manage.py runserver 8001`
   - `uvicorn api.main:app --reload --port 8000`
   - `cd frontend` then `python -m http.server 5500`
9. Open http://localhost:5500 (shop), http://localhost:8001/dashboard/ (dashboard, log in with admin / admin123).

## Setup (step by step)
1. Install Python 3.10+.
2. In a terminal in this folder, create a virtual environment: `python -m venv venv`, then `source venv/bin/activate` (Windows: `venv\Scripts\activate`)
3. `pip install -r requirements.txt`
4. `cp .env.example .env` (Windows: `copy .env.example .env`). Leave Stripe, SMTP and Auth0 blank for demo mode.
5. Create the database: `python manage.py makemigrations shop`, then `python manage.py migrate`
6. Add sample data and the admin login: `python seed.py` (admin / admin123)

## Run (3 terminals, each with the venv active)
- Admin panel (Django): `python manage.py runserver 8001`
  Dashboard: http://localhost:8001/dashboard/  Data admin: http://localhost:8001/admin/
- API (FastAPI): `uvicorn api.main:app --reload --port 8000`  Docs: http://localhost:8000/docs
- Storefront: `cd frontend && python -m http.server 5500`, then open http://localhost:5500

## Try it
1. Sign up in the storefront, add products, click "Pay with Stripe". Demo mode confirms instantly. You get an in-app notification and the email prints in the API terminal.
2. In Django admin > Orders, change an order's status (for example to shipped). The storefront shows the notification within 2 seconds over the WebSocket.
3. Open the dashboard for sales, top sellers, revenue trend and low-stock alerts. Use Export CSV or Export PDF.
4. Roles: in Django admin > Users tick "Staff status" (staff) or "Superuser status" (admin). The API's `/admin/orders` accepts only these roles.

## Real Stripe payments (INR)
1. Put `STRIPE_SECRET_KEY` and `STRIPE_WEBHOOK_SECRET` in `.env` and restart the API.
2. Forward webhooks: `stripe listen --forward-to localhost:8000/payments/webhook`
3. Pay with test card 4242 4242 4242 4242. Your Stripe account must support INR.

## Real email
Fill the `SMTP_*` values in `.env` (a Gmail app password works).

## Google/Facebook login (Auth0)
Create an Auth0 application, enable the Google and Facebook connections, set `AUTH0_DOMAIN` and `AUTH0_AUDIENCE` in `.env`, then send the Auth0 `id_token` to `POST /auth/auth0`. It returns this API's JWT.

## Postman
Import `postman/Smartshop.postman_collection.json`, log in, and paste the `access_token` into the `token` variable.

## Before production
Change `JWT_SECRET` and `DJANGO_SECRET`, set `DEBUG=False`, move to PostgreSQL, restrict CORS, and serve over HTTPS.

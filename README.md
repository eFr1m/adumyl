# Adumyl

**Adumyl** is a full-stack online food ordering platform. Users can discover restaurants, browse menus, place orders, and track deliveries in real time. Restaurant owners can manage their menus and incoming orders, while couriers can register and accept delivery assignments.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Data Models](#data-models)
- [API Endpoints](#api-endpoints)
- [Getting Started](#getting-started)
  - [Backend Setup](#backend-setup)
  - [Frontend Setup](#frontend-setup)
- [Configuration](#configuration)
- [Deployment](#deployment)

---

## Features

### Customer
- **Authentication** – Sign up and sign in with email and password (session-based via Django + CSRF tokens).
- **Restaurant Discovery** – Browse all registered restaurants with pagination (8 per page) and food-category filtering.
- **Menu Browsing** – View a restaurant's full menu with item names, prices, ingredients, preparation time, and weight.
- **Shopping Cart** – Add/remove items and adjust quantities before placing an order.
- **Order Placement** – Choose a delivery address and payment method (Card, Cash, or PayPal).
- **Order History** – View past orders with status and item details.
- **Delivery Tracker** – Follow the real-time status of an active delivery on an interactive Google Maps view.

### Restaurant Owner (`is_owner`)
- **Restaurant Registration** – Register and manage a restaurant profile (name, address, phone, image, operating hours).
- **Menu Management** – Create, update, and delete menu items (name, price, ingredients, prep time, weight, image).
- **Order Management** – View incoming orders and update their status through the pipeline: `pending → accepted → preparing → out_for_delivery → delivered`.
- **Overview Dashboard** – See aggregate statistics (total sales, total orders, average rating).

### Courier (`is_courier`)
- **Courier Registration** – Register as a courier with vehicle type.
- **Delivery Requests** – View and accept orders ready for delivery (`out_for_delivery` with no assigned courier).
- **Delivery Management** – Track current assignments and update delivery stages.

### Profile
- Edit personal information (name, email, phone number, addresses).
- Change password.

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Backend language** | Python 3 |
| **Backend framework** | Django 5.1.3 |
| **REST API** | Django REST Framework 3.15.2 |
| **Authentication** | Django session authentication + CSRF |
| **Database (dev)** | SQLite |
| **Database (prod)** | PostgreSQL (via `psycopg2`) |
| **CORS** | `django-cors-headers` |
| **Static files** | WhiteNoise |
| **WSGI server** | Gunicorn |
| **Frontend language** | JavaScript (ES2022) |
| **Frontend framework** | React 18 |
| **Routing** | React Router v6 |
| **UI components** | Chakra UI v2 (custom green theme, dark mode) |
| **HTTP client** | Axios |
| **Maps** | Google Maps (`@react-google-maps/api`, `@vis.gl/react-google-maps`, `react-geocode`) |
| **Animations** | Framer Motion |
| **Icons** | `@tabler/icons-react`, `react-icons`, `hugeicons-react` |
| **Hosting** | Heroku |

---

## Project Structure

```
adumyl/
├── backend/                        # Django project
│   ├── adumylSite/                 # Project configuration
│   │   ├── settings.py             # Django settings (CORS, auth, DB, etc.)
│   │   ├── urls.py                 # Root URL configuration
│   │   ├── wsgi.py
│   │   └── asgi.py
│   ├── adumyl/                     # Main Django app
│   │   ├── models.py               # Data models (User, Restaurant, MenuItem, Order, …)
│   │   ├── serializers.py          # DRF serializers
│   │   ├── views.py                # API views
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── migrations/
│   ├── manage.py
│   ├── requirements.txt            # Python dependencies
│   ├── Pipfile / Pipfile.lock
│   ├── Procfile                    # Heroku process declaration
│   ├── runtime.txt                 # Python runtime for Heroku
│   └── data.json                   # Fixture / seed data
│
└── frontend/                       # React application
    ├── public/                     # Static HTML, icons, manifest
    ├── src/
    │   ├── App.js                  # Root component and routing
    │   ├── index.js                # React entry point
    │   ├── theme.js                # Chakra UI theme overrides
    │   ├── pages/                  # Top-level route pages
    │   │   ├── LoginPage.js        # Sign up / Sign in
    │   │   ├── HomePage.js         # Restaurant listing
    │   │   ├── MenuPage.js         # Restaurant menu
    │   │   ├── PlaceOrderPage.js   # Address + payment checkout
    │   │   ├── DeliveryTracker.js  # Live delivery map
    │   │   ├── HelpPage.js         # Help / support
    │   │   ├── Profile.js          # Profile shell with side-nav
    │   │   ├── OrderContext.js     # Global order state (React Context)
    │   │   └── profile_pages/
    │   │       ├── ProfileInfo.js      # Edit personal details
    │   │       ├── OrderHistory.js     # Past orders
    │   │       ├── MyRestaurant.js     # Owner: restaurant management
    │   │       └── Delivery.js         # Courier: delivery management
    │   ├── components/             # Reusable UI components
    │   │   ├── NavBar.js
    │   │   ├── SideNav.js
    │   │   ├── RestaurantCard.js
    │   │   ├── MenuItemCard.js
    │   │   ├── ShoppingCart.js
    │   │   ├── FoodCategories.js
    │   │   ├── LastOrders.js
    │   │   ├── Pagination.js
    │   │   ├── OrderAddresForm.js
    │   │   ├── OrderPaymentForm.js
    │   │   └── profile_components/
    │   │       ├── RegisterRestaurantForm.js
    │   │       ├── RegisterDelivery.js
    │   │       ├── RestaurantTabs.js
    │   │       ├── OverviewTab.js
    │   │       ├── MenuTab.js
    │   │       ├── OrdersTab.js
    │   │       └── DeliveryManagement.js
    │   ├── services/               # Axios API wrappers
    │   │   ├── backendAPI.js       # Base axios helper with CSRF handling
    │   │   ├── auth.js             # Login / register
    │   │   ├── user.js             # User profile CRUD
    │   │   ├── restaurant.js       # Restaurant CRUD
    │   │   ├── menuItem.js         # Menu item CRUD
    │   │   ├── orders.js           # Order operations
    │   │   └── delivery.js         # Delivery operations
    │   └── utils/
    │       ├── demos.js            # Demo / seed data helpers
    │       └── geocode.js          # Geocoding utilities
    └── package.json
```

---

## Data Models

### `User`
Extends Django's `AbstractBaseUser`. Authentication is by email (not username).

| Field | Type | Notes |
|---|---|---|
| `email` | EmailField (unique) | Login identifier |
| `first_name` | CharField | |
| `last_name` | CharField | |
| `phone_number` | CharField | Optional |
| `address1/2/3` | CharField | Up to three saved addresses |
| `is_owner` | BooleanField | Grants restaurant-management access |
| `is_courier` | BooleanField | Grants delivery access |
| `is_active` | BooleanField | |
| `is_staff` | BooleanField | Django admin access |

### `Restaurant`
Belongs to a `User` (owner). Tracks cumulative rating with a helper `average_rating` property.

| Field | Type | Notes |
|---|---|---|
| `user` | FK → User | Owner |
| `name` | CharField | |
| `address` | CharField | |
| `phone_number` | CharField | |
| `operating_hours` | TextField | JSON-encoded schedule |
| `days` | TextField | JSON-encoded open days |
| `rating` | PositiveIntegerField | Cumulative total |
| `rating_count` | PositiveIntegerField | Number of ratings |
| `total_sales` | IntegerField | |
| `total_orders` | IntegerField | |
| `image_url` | CharField | |

### `MenuItem`
Belongs to a `Restaurant`.

| Field | Type | Notes |
|---|---|---|
| `restaurant` | FK → Restaurant | |
| `name` | CharField | |
| `price` | DecimalField | Max 999.99 |
| `ingredients` | TextField | Free-text or JSON list |
| `prep_time` | PositiveIntegerField | Minutes |
| `weight` | DecimalField | Grams, optional |
| `image_url` | CharField | Optional |

### `Courier`
Belongs to a `User`.

| Field | Type | Notes |
|---|---|---|
| `user` | FK → User | |
| `rating` | DecimalField | Optional, up to 9.99 |
| `current_location` | CharField | GPS or address string |
| `vehicle` | CharField | Default: "car" |

### `Order`
Links a customer (`User`), a `Restaurant`, and optionally a `Courier`.

| Field | Type | Notes |
|---|---|---|
| `restaurant` | FK → Restaurant | |
| `user` | FK → User | Customer |
| `courier` | FK → Courier | Assigned on acceptance |
| `address` | CharField | Delivery address |
| `total_price` | DecimalField | |
| `payment_method` | CharField | `card` / `cash` / `paypal` |
| `status` | CharField | See pipeline below |
| `created_at` | DateTimeField | |
| `updated_at` | DateTimeField | |

**Order status pipeline:** `pending` → `accepted` → `preparing` → `out_for_delivery` → `delivered` (or `canceled`)

### `OrderItem`
Join table between `Order` and `MenuItem`.

| Field | Type |
|---|---|
| `order` | FK → Order |
| `menu_item` | FK → MenuItem |
| `quantity` | PositiveIntegerField |

### `Delivery`
Tracks the geo-coordinates and stage progress of a delivery.

| Field | Type | Notes |
|---|---|---|
| `order_id` | CharField (unique) | |
| `restaurant_coordinates` | JSONField | `{lat, lng}` |
| `delivery_coordinates` | JSONField | `{lat, lng}` |
| `restaurant_address` | TextField | |
| `delivery_address` | TextField | |
| `delivery_agent` | FK → Courier | |
| `stages` | JSONField | List of stage labels |
| `current_stage` | PositiveIntegerField | Index into `stages` |
| `estimated_time` | CharField | |

---

## API Endpoints

All endpoints are served from the Django backend (default: `http://localhost:8000`). Most require an active session (login first).

| Method | URL | Description |
|---|---|---|
| `POST` | `/auth/login/` | Log in (email + password) |
| `POST` | `/auth/signup/` | Register a new user |
| `GET` | `/user/` | Get current user's profile |
| `PATCH` | `/user/` | Update current user's profile |
| `PUT` | `/change_password/` | Change password |
| `POST` | `/order_history/` | Get current user's order history |
| `GET` | `/restaurants_all/` | List all restaurants |
| `GET` | `/restaurant/` | Get owner's restaurant |
| `POST` | `/restaurant/` | Register a new restaurant |
| `PATCH` | `/restaurant/` | Update owner's restaurant |
| `GET` | `/menu_item/` | List owner's restaurant menu items |
| `POST` | `/menu_item/` | Create a menu item |
| `PATCH` | `/menu_item/<id>/` | Update a menu item |
| `DELETE` | `/menu_item/<id>/` | Delete a menu item |
| `POST` | `/menu-items-all/` | List all items for a given restaurant |
| `GET` | `/orders-all/` | List all orders for the current user |
| `GET` | `/orders/` | List orders for the owner's restaurant |
| `POST` | `/orders/` | Place a new order |
| `POST` | `/orders/<id>/` | Get a single order with restaurant address |
| `PATCH` | `/orders/<id>/` | Update order status (owner) |
| `POST` | `/order-items/` | Add an order item |
| `GET` | `/courier/` | Get current user's courier profile |
| `POST` | `/courier/` | Register as a courier |
| `GET` | `/delivery-requests/` | List unassigned `out_for_delivery` orders |
| `PATCH` | `/delivery-requests/<id>/` | Accept a delivery request |
| `GET` | `/admin/` | Django admin panel |

---

## Getting Started

### Prerequisites
- Python 3.10+
- Node.js 18+
- npm 9+

### Backend Setup

```bash
cd backend

# Create and activate a virtual environment (optional but recommended)
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Apply database migrations
python manage.py migrate

# (Optional) Load seed data
python manage.py loaddata data.json

# Create a superuser for the Django admin
python manage.py createsuperuser

# Start the development server
python manage.py runserver
```

The backend API is available at **http://localhost:8000**.

### Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Start the development server (runs on port 5000)
npm start
```

The frontend is available at **http://localhost:5000**.

> The frontend's `package.json` sets a proxy to `http://127.0.0.1:8000`, so API requests are automatically forwarded to the Django server during development.

---

## Configuration

Key settings in `backend/adumylSite/settings.py`:

| Setting | Value | Notes |
|---|---|---|
| `AUTH_USER_MODEL` | `adumyl.User` | Custom email-based user model |
| `CORS_ORIGIN_WHITELIST` | `http://localhost:5000` | Adjust for your frontend URL |
| `CSRF_TRUSTED_ORIGINS` | `http://localhost:5000` | Required for CSRF-protected requests |
| `DATABASES` | SQLite (dev) / PostgreSQL (prod) | Configured via `django-heroku` in production |
| `DEBUG` | `True` | Set to `False` in production |

---

## Deployment

The backend is configured for deployment to **Heroku**:

- **`Procfile`** – Declares the release (migrations) and web (Gunicorn) processes.
- **`runtime.txt`** – Specifies the Python version.
- **`django-heroku`** – Automatically configures `DATABASE_URL`, static files, and `ALLOWED_HOSTS` from Heroku environment variables.
- **WhiteNoise** – Serves static files directly from Django in production.
- **PostgreSQL** – `psycopg2` / `psycopg2-binary` are included for the Heroku Postgres add-on.

To deploy:
```bash
heroku create <app-name>
heroku addons:create heroku-postgresql:essential-0
git push heroku main
```

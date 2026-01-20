# Deal-Maniac – Smart Product Price Tracker

Track product prices across e-commerce sites and get alerts on price drops. Built with Next.js, Firecrawl, and Supabase.

---

## 🎯 Features

* 🔍 **Track Any Product** – Works with Amazon, Zara, Walmart, and more
* 📊 **Price History Charts** – Interactive graphs showing price trends over time
* 🔐 **Google Authentication** – Secure sign-in with Google OAuth
* 🔄 **Automated Daily Checks** – Scheduled cron jobs check prices automatically
* 📧 **Email Alerts** – Get notified when prices drop via Resend

---

## 🛠️ Tech Stack

* **Next.js 16** – React framework with App Router
* **Firecrawl** – Web data extraction API

  * Handles JavaScript rendering
  * Rotating proxies & anti-bot bypass
  * Structured data extraction with AI
  * Works across different e-commerce sites
* **Supabase** – Backend platform

  * PostgreSQL Database
  * Google Authentication
  * Row Level Security (RLS)
  * pg_cron for scheduled jobs
* **Resend** – Transactional emails
* **shadcn/ui** – UI component library
* **Recharts** – Interactive charts
* **Tailwind CSS** – Styling

---

## 📋 Prerequisites

Before you begin, ensure you have:

* Node.js 18+ installed
* A Supabase account
* A Firecrawl account
* A Resend account
* Google OAuth credentials from Google Cloud Console

---

## 🚀 Setup Instructions

### 1. Clone and Install

```bash
git clone https://github.com/Yuvrajsinghko/Deal-Maniac.git
cd Deal-Maniac
npm install
```

---

### 2. Supabase Setup

#### Create Project

1. Create a new project at supabase.com
2. Wait for the project to be ready

---

#### Run Database Migrations

Go to **SQL Editor** in your Supabase dashboard and run these migrations.

---

### Migration 1: Database Schema (`supabase/migrations/001_schema.sql`)

```sql
create extension if not exists "uuid-ossp";

create table products (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references auth.users(id) on delete cascade not null,
  url text not null,
  name text not null,
  current_price numeric(10,2) not null,
  currency text not null default 'USD',
  image_url text,
  created_at timestamp with time zone default now(),
  updated_at timestamp with time zone default now()
);

create table price_history (
  id uuid primary key default uuid_generate_v4(),
  product_id uuid references products(id) on delete cascade not null,
  price numeric(10,2) not null,
  currency text not null,
  checked_at timestamp with time zone default now()
);

ALTER TABLE products
ADD CONSTRAINT products_user_url_unique UNIQUE (user_id, url);

alter table products enable row level security;
alter table price_history enable row level security;

create policy "Users can view their own products"
on products for select
using (auth.uid() = user_id);

create policy "Users can insert their own products"
on products for insert
with check (auth.uid() = user_id);

create policy "Users can update their own products"
on products for update
using (auth.uid() = user_id);

create policy "Users can delete their own products"
on products for delete
using (auth.uid() = user_id);

create policy "Users can view price history for their products"
on price_history for select
using (
  exists (
    select 1 from products
    where products.id = price_history.product_id
    and products.user_id = auth.uid()
  )
);

create index products_user_id_idx on products(user_id);
create index price_history_product_id_idx on price_history(product_id);
create index price_history_checked_at_idx on price_history(checked_at desc);
```

---

### Migration 2: Setup Cron Job (`supabase/migrations/002_setup_cron.sql`)

```sql
CREATE EXTENSION IF NOT EXISTS pg_cron;
CREATE EXTENSION IF NOT EXISTS pg_net;

CREATE OR REPLACE FUNCTION trigger_price_check()
RETURNS void
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  PERFORM net.http_post(
    url := 'https://your-app-url.vercel.app/api/cron/check-prices',
    headers := jsonb_build_object(
      'Content-Type', 'application/json',
      'Authorization', 'Bearer YOUR_CRON_SECRET_HERE'
    )
  );
END;
$$;

SELECT cron.schedule(
  'daily-price-check',
  '0 9 * * *',
  'SELECT trigger_price_check();'
);
```

---

## 🔍 How It Works

### User Flow

1. User adds a product URL
2. Firecrawl extracts product name, price, currency, and image
3. Data is stored securely in Supabase
4. Users view current price and price history chart

---

### Automated Price Checking

1. Supabase `pg_cron` runs daily
2. Secure API endpoint is triggered
3. Firecrawl re-scrapes product prices
4. Database updates price history
5. Email alerts are sent if prices drop

---

### Why Firecrawl?

* JavaScript rendering support
* Anti-bot & proxy handling
* AI-powered structured extraction
* Multi-site compatibility
* Production-ready reliability

---

## 📁 Project Structure

```
deal-maniac/
├── app/
│   ├── page.js
│   ├── actions.js
│   ├── auth/callback/route.js
│   └── api/cron/check-prices/route.js
├── components/
│   ├── ui/
│   ├── AddProductForm.js
│   ├── ProductCard.js
│   ├── PriceChart.js
│   └── AuthModal.js
├── lib/
│   ├── firecrawl.js
│   ├── email.js
│   └── utils.js
├── utils/supabase/
│   ├── client.js
│   ├── server.js
│   └── middleware.js
├── supabase/migrations/
│   ├── 001_schema.sql
│   └── 002_setup_cron.sql
└── .env.local
```

---

## 🧪 Testing

```bash
curl -X POST https://your-app.vercel.app/api/cron/check-prices \
  -H "Authorization: Bearer your_cron_secret" \
  -H "Content-Type: application/json"
```

---

## 🐛 Troubleshooting

* Ensure `SUPABASE_SERVICE_ROLE_KEY` is set
* Verify Firecrawl API logs
* Confirm cron job exists in Supabase
* Check Resend dashboard for email delivery

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Commit changes
4. Push and open a Pull Request

---

Built with ❤️ using **Next.js, Firecrawl, and Supabase**

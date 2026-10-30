# StockWise — Multi-User Authentication & Cloud Portfolio Setup Guide

This guide explains how to configure and deploy the **Supabase Authentication & Private Cloud Portfolio Backend** for StockWise.

---

## 1. Architecture Overview

- **Authentication**: Supabase Auth (Email & Password, sessions persisted in `localStorage`).
- **Database**: PostgreSQL on Supabase.
- **Access Control**: Row Level Security (RLS) enabled on all tables (`portfolios`, `holdings`, `transactions`, `watchlist`, `profiles`).
- **Trade Consistency**: Atomic financial execution via server-side stored procedures:
  - `execute_buy_trade`: Checks cash balance, updates cash, recalculates weighted-average cost basis, and writes to transaction ledger in a single transaction.
  - `execute_sell_trade`: Verifies share ownership, calculates net proceeds and realized profit/loss, updates holding quantity/deletes liquidated positions, and writes to transaction ledger in a single transaction.
- **Resilient Fallback Mode**: If Supabase credentials are not provided or in placeholder format, StockWise smoothly runs in demo evaluation mode without breaking.

---

## 2. Supabase Project Setup (Manual Steps)

1. Create a free account at [supabase.com](https://supabase.com).
2. Click **New Project** and name it `stockwise` (choose the region closest to you, e.g. South Asia / Mumbai).
3. Once the database is provisioned, go to **Project Settings** → **API**.
4. Note your:
   - **Project URL** (`https://<project-ref>.supabase.co`)
   - **anon / public key** (`eyJhbGci...`)

---

## 3. Database Schema & Migration Execution

1. In your Supabase Dashboard, open the **SQL Editor** from the left navigation bar.
2. Open [`supabase/migrations/20261029000000_stockwise_schema.sql`](./supabase/migrations/20261029000000_stockwise_schema.sql) in this repository.
3. Copy the entire contents into the Supabase SQL Editor and click **Run**.
4. The script will create:
   - `profiles` with RLS
   - `portfolios` with RLS (with unique constraint per user)
   - `holdings` with RLS (with unique constraint per portfolio and symbol)
   - `transactions` with RLS
   - `watchlist` with RLS
   - Trigger `on_auth_user_created` (automatically allocates ₹100,000 virtual cash and starter watchlist to any new registered user)
   - Atomic RPC functions `execute_buy_trade` and `execute_sell_trade`

---

## 4. Local Environment Configuration

1. Create a `.env` file in the root directory by duplicating `.env.example`:
   ```bash
   cp .env.example .env
   ```
2. Populate the environment variables:
   ```env
   VITE_SUPABASE_URL=https://your-project.supabase.co
   VITE_SUPABASE_ANON_KEY=your-actual-anon-key
   ```
3. Restart your Vite development server:
   ```bash
   npm run dev
   ```

---

## 5. Verification Checklist

- [x] **Registration**: Register a new user with email and password.
- [x] **Auto Provisioning**: Trigger creates an initial private portfolio with ₹100,000 cash balance.
- [x] **Authentication Protection**: Unauthenticated users only see the Sign In / Sign Up views.
- [x] **Trading Operations**: Buying and selling Indian equities atomically updates cash, average cost basis, and records the trade in the transactions audit ledger.
- [x] **Multi-Device Sync**: Sign in with the same account on another browser or device — private portfolio records reload seamlessly.
- [x] **Data Isolation**: Row Level Security ensures User A can never query, read, or modify User B's portfolio or transactions.
- [x] **Local Migration**: Existing browser simulator data can be imported into the cloud account via **Settings → Import Browser Portfolio to Cloud**.

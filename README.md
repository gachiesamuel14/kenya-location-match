# Karibu Mtaani — Location-Based Dating for Kenya

A modern location-based matchmaking website for people in Kenya. Match by county, distance, interests, and more.

**Stack:** Next.js 16 (App Router) + Tailwind CSS + Supabase (planned) + Netlify

## Live / Deploy

- GitHub: https://github.com/gachiesamuel14/kenya-location-match
- Deploy to Netlify by connecting this repo (build command: `npm run build`, publish directory: `.next` or use Netlify Next.js runtime)

## Features (MVP roadmap)

1. **Landing page** — Kenya-focused branding, how-it-works, safety
2. **Auth** — Email/password + optional Google (via Supabase Auth)
3. **Profiles** — Photos, bio, age, county, interests, tribe, religion, lifestyle
4. **Location** — County selection + optional GPS (approximate distance only)
5. **Discovery** — Filter by distance / county / age / interests
6. **Matching** — Like / pass → mutual match
7. **Chat** — Realtime messaging after match (Supabase Realtime or Stream)
8. **Safety** — Report, block, guidelines
9. **Premium** (optional) — Boosts, more likes, M-Pesa payments

## Quick start (local)

```bash
npm install
npm run dev
```

Open http://localhost:3000

## Next steps to make it fully functional

1. Create a free project at [supabase.com](https://supabase.com)
2. Enable Auth (Email + Google if desired)
3. Create tables (see schema below)
4. Add environment variables to Netlify / local `.env.local`:

```
NEXT_PUBLIC_SUPABASE_URL=your-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

5. Install packages:

```bash
npm install @supabase/supabase-js @supabase/ssr
```

6. Add middleware + client helpers for auth (standard Supabase + Next.js App Router pattern)

### Suggested database schema (PostgreSQL / Supabase)

```sql
-- profiles (extends auth.users)
create table public.profiles (
  id uuid references auth.users on delete cascade primary key,
  full_name text,
  username text unique,
  bio text,
  birthdate date,
  gender text,
  looking_for text[],
  county text,
  town text,
  lat double precision,
  lng double precision,
  interests text[],
  tribe text,
  religion text,
  lifestyle text,
  avatar_url text,
  photos text[],
  is_verified boolean default false,
  last_active timestamptz default now(),
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- likes / matches
create table public.likes (
  id uuid default gen_random_uuid() primary key,
  from_user uuid references public.profiles(id) on delete cascade,
  to_user uuid references public.profiles(id) on delete cascade,
  created_at timestamptz default now(),
  unique(from_user, to_user)
);

-- reports
create table public.reports (
  id uuid default gen_random_uuid() primary key,
  reporter uuid references public.profiles(id),
  reported uuid references public.profiles(id),
  reason text,
  details text,
  created_at timestamptz default now()
);

-- Enable RLS on all tables
alter table public.profiles enable row level security;
-- Add policies: users can read public profile fields, update only own row, etc.
```

Distance matching can use a simple Haversine formula in a Postgres function or client-side for MVP, or enable the PostGIS extension later.

## Kenya localization ideas

- County list (all 47)
- Sheng / Swahili UI option
- Student / Church / Professional modes
- M-Pesa for premium (Daraja API later)

## Safety notes

Never expose exact coordinates to other users. Store them server-side and return only approximate distance or county.

---

Built for Kenyans who want to meet people near them.

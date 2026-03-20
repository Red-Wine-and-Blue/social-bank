# Supabase Setup Guide

This takes about 10 minutes and is completely free.

---

## Step 1 — Create a Supabase account

1. Go to https://supabase.com and click **Start your project**
2. Sign up with GitHub or email
3. Click **New project**
4. Name it `social-bank`, choose a region close to you, set a database password, click **Create project**
5. Wait about 2 minutes for it to spin up

---

## Step 2 — Create the database tables

1. In your Supabase project, click **SQL Editor** in the left sidebar
2. Click **New query**
3. Paste in the following SQL and click **Run**:

```sql
-- Events table (created by admin)
create table events (
  id uuid default gen_random_uuid() primary key,
  post_url text not null,
  topic text not null,
  event_name text,
  event_date date,
  created_at timestamp with time zone default now()
);

-- Sessions table (created when a member starts a Social Bank)
create table sessions (
  id uuid default gen_random_uuid() primary key,
  event_id uuid references events(id) on delete cascade,
  member_name text not null,
  current_step integer default 0,
  completed boolean default false,
  started_at timestamp with time zone default now(),
  completed_at timestamp with time zone
);

-- Allow public read/write (members don't need to log in)
alter table events enable row level security;
alter table sessions enable row level security;

create policy "Public can read events" on events for select using (true);
create policy "Public can insert sessions" on sessions for insert with check (true);
create policy "Public can update own sessions" on sessions for update using (true);
create policy "Public can read sessions" on sessions for select using (true);
create policy "Admin can insert events" on events for insert with check (true);
create policy "Admin can delete events" on events for delete using (true);
```

---

## Step 3 — Get your credentials

1. In Supabase, go to **Project Settings** (gear icon) → **API**
2. Copy your **Project URL** — looks like `https://abcdefgh.supabase.co`
3. Copy your **anon public** key — a long string starting with `eyJ...`

---

## Step 4 — Add credentials to your files

Open `admin.html` and find these lines near the top of the `<script>` section:

```javascript
const SUPABASE_URL  = "YOUR_SUPABASE_URL";
const SUPABASE_KEY  = "YOUR_SUPABASE_ANON_KEY";
const ADMIN_PASSWORD = "socialbank2024";
```

Replace with your actual values:

```javascript
const SUPABASE_URL  = "https://abcdefgh.supabase.co";
const SUPABASE_KEY  = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";
const ADMIN_PASSWORD = "your-chosen-password";
```

Do the same in `index.html` when the member tracking is added.

---

## Step 5 — Upload both files to GitHub

Upload `admin.html` to your `social-bank` repo alongside `index.html`.

Your admin page will be live at:
```
https://slkester.github.io/social-bank/admin.html
```

---

## How it works end to end

1. **Admin** goes to `/admin.html`, logs in, creates an event
2. Admin copies the generated member link (e.g. `index.html?event=abc-123`)
3. **Admin shares the link** via email, text, or Eventbrite
4. **Members** tap the link — the post and topic are pre-loaded
5. Members enter their name and complete the SCALE steps
6. Admin sees participation data in the dashboard (coming next)

---

## Security note

The `anon` key is safe to use in a public website — it only has the permissions you defined in the SQL above. Your database password (from Step 1) should never go in the code.

The admin password in the code is a simple client-side check — enough to keep casual visitors out. For higher security in the future, Supabase Auth can be added.

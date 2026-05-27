# Vend prototype — security setup (Option C)

Real auth via Supabase + Google OAuth. Only `@finn.no`, `@m10s.io`, and `@vend.com`
accounts can sign in and read/write team data. Set up takes ~30 minutes once IT
approval is in hand.

---

## 1. Enable Google OAuth in Supabase

1. Supabase dashboard → **Authentication** → **Providers** → **Google** → toggle on.
2. You'll need a Google Cloud OAuth Client ID + Secret. Steps in
   [Google's OAuth docs](https://support.google.com/cloud/answer/6158849).
   - **Authorized JavaScript origins:** `https://leef337.github.io` (and any
     dev origin, e.g. `http://localhost:8765`).
   - **Authorized redirect URIs:** Copy the **Callback URL** Supabase shows
     under the Google provider settings (looks like
     `https://<your-project>.supabase.co/auth/v1/callback`).
3. Paste the Client ID + Secret back into Supabase's Google provider settings.
4. Save.

**For Schibsted IT:** the OAuth app needs to be registered under the
Schibsted Google Workspace so `@finn.no` and `@schibsted.com` accounts can
sign in. The ask is: *"Approve a new Google OAuth client for an internal
strategy prototype hosted on GitHub Pages. Redirect target is
`<supabase-project>.supabase.co/auth/v1/callback`. Restrict the OAuth app
to internal users."* Usually a same-day or next-day ticket.

---

## 2. Add Supabase URLs to the redirect allowlist

Supabase dashboard → **Authentication** → **URL Configuration**:

- **Site URL:** `https://leef337.github.io/apply-flow-v1/`
- **Redirect URLs:** add the same URL (and any local dev URLs you use).

Without this, Supabase will reject the OAuth callback after Google
authenticates and you'll see an opaque error on the gate.

---

## 3. Run the RLS policies SQL

Supabase dashboard → **SQL Editor** → paste the block below → Run.

This:
- Creates a `is_team_member()` SQL helper that reads `auth.jwt() ->> 'email'`
  and checks it against the three allowed domains.
- Enables RLS on every prototype_* table.
- Adds `SELECT` + `ALL` policies that allow only team-member emails to read
  or write.

```sql
-- ─────────────────────────────────────────────────────────────────────
-- Vend prototype — RLS policies enforcing @finn.no / @m10s.io / @vend.com
-- access on every team-data table. Paste-and-run in Supabase SQL editor.
-- ─────────────────────────────────────────────────────────────────────

-- Allowlist check, called from every RLS policy below. SECURITY DEFINER
-- so it runs with the function owner's privileges (i.e. has access to
-- auth.jwt()) even if the caller's role can't read auth.users directly.
CREATE OR REPLACE FUNCTION public.is_team_member()
RETURNS BOOLEAN
LANGUAGE SQL STABLE SECURITY DEFINER
SET search_path = public
AS $$
  SELECT COALESCE(
    (auth.jwt() ->> 'email') ~* '@(finn\.no|m10s\.io|vend\.com)$',
    false
  );
$$;

-- Per-table: enable RLS and add read+write policies. Adjust table names
-- if your prototype_* tables are named differently in your Supabase
-- project (this matches the names in `cloudStore` in index.html).

ALTER TABLE public.prototype_state    ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.prototype_events   ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.prototype_ac       ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.prototype_readiness ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.prototype_comments ENABLE ROW LEVEL SECURITY;

-- Drop any previous policies so re-running this block is idempotent.
DROP POLICY IF EXISTS "team reads state"   ON public.prototype_state;
DROP POLICY IF EXISTS "team writes state"  ON public.prototype_state;
DROP POLICY IF EXISTS "team reads events"  ON public.prototype_events;
DROP POLICY IF EXISTS "team writes events" ON public.prototype_events;
DROP POLICY IF EXISTS "team reads ac"      ON public.prototype_ac;
DROP POLICY IF EXISTS "team writes ac"     ON public.prototype_ac;
DROP POLICY IF EXISTS "team reads readiness"  ON public.prototype_readiness;
DROP POLICY IF EXISTS "team writes readiness" ON public.prototype_readiness;
DROP POLICY IF EXISTS "team reads comments"   ON public.prototype_comments;
DROP POLICY IF EXISTS "team writes comments"  ON public.prototype_comments;

-- State (storymap / matrix / tree) ----------------------------------
CREATE POLICY "team reads state" ON public.prototype_state
  FOR SELECT USING (public.is_team_member());
CREATE POLICY "team writes state" ON public.prototype_state
  FOR ALL USING (public.is_team_member())
           WITH CHECK (public.is_team_member());

-- Events (activity log) ---------------------------------------------
CREATE POLICY "team reads events" ON public.prototype_events
  FOR SELECT USING (public.is_team_member());
CREATE POLICY "team writes events" ON public.prototype_events
  FOR ALL USING (public.is_team_member())
           WITH CHECK (public.is_team_member());

-- Acceptance criteria -----------------------------------------------
CREATE POLICY "team reads ac" ON public.prototype_ac
  FOR SELECT USING (public.is_team_member());
CREATE POLICY "team writes ac" ON public.prototype_ac
  FOR ALL USING (public.is_team_member())
           WITH CHECK (public.is_team_member());

-- Readiness ---------------------------------------------------------
CREATE POLICY "team reads readiness" ON public.prototype_readiness
  FOR SELECT USING (public.is_team_member());
CREATE POLICY "team writes readiness" ON public.prototype_readiness
  FOR ALL USING (public.is_team_member())
           WITH CHECK (public.is_team_member());

-- Comments (pin-on-screen feedback) ---------------------------------
CREATE POLICY "team reads comments" ON public.prototype_comments
  FOR SELECT USING (public.is_team_member());
CREATE POLICY "team writes comments" ON public.prototype_comments
  FOR ALL USING (public.is_team_member())
           WITH CHECK (public.is_team_member());

-- Optional: revoke direct public access at the role level. Supabase's
-- default anon role is restricted via RLS above, so this is belt-and-
-- braces. Skip if you're not sure.
-- REVOKE ALL ON public.prototype_state FROM anon;
-- REVOKE ALL ON public.prototype_events FROM anon;
-- REVOKE ALL ON public.prototype_ac FROM anon;
-- REVOKE ALL ON public.prototype_readiness FROM anon;
-- REVOKE ALL ON public.prototype_comments FROM anon;
```

After running, test by signing out, signing back in with a non-allowlisted
account (e.g. a personal Gmail), and verifying you can't write to any
table.

---

## 4. Flip the GitHub repo private

Even with auth, the prototype source is currently public on GitHub at
[leef337/apply-flow-v1](https://github.com/leef337/apply-flow-v1).

The source contains the Supabase URL and anon key. That's *normal* — the
anon key is meant to be public; security comes from RLS. But it also means
a sophisticated visitor could query Supabase directly without going
through the prototype UI. The RLS policies above block that, but if the
strategy data is sensitive enough to warrant auth, you probably also want
the repo private.

**To flip private:**

1. GitHub → repo Settings → General → Danger Zone → **Change visibility**.
2. Confirm.
3. GitHub Pages will keep working for the same `leef337.github.io/apply-flow-v1/`
   URL — Pages on private repos requires GitHub Pro/Team for organisations,
   or is free on personal accounts.
4. If you don't want to upgrade to Pro, alternative: deploy to Netlify or
   Vercel from a private GitHub repo — both free for private deployments.

---

## 5. Smoke test

1. Open the prototype URL in an incognito window.
2. You should see the **Sign in with Google** screen.
3. Sign in with your `@m10s.io` (or whichever allowlisted) account → you
   should land in the prototype.
4. Sign out from the dev panel → you should bounce back to the sign-in screen.
5. In another browser, sign in with a non-allowlisted Google account
   (e.g. a personal Gmail) → you should hit the "this account isn't on the
   access list" screen with a Sign out button.
6. Drag a story on the storymap, then check Supabase → `prototype_state`
   table — the row should have your edit. Then try the same from a
   non-authenticated client (e.g. via `curl` against the Supabase REST
   endpoint with no token) → should return 401/403.

---

## Notes

- **Allowlist domains** live in `index.html` as `ALLOWED_DOMAINS` (front-end UX guard)
  AND in the `is_team_member()` SQL function (real boundary). If you add a
  domain, change both — the SQL is authoritative.
- **Sessions persist** via Supabase's localStorage default. Sign-out clears
  the session everywhere.
- **The fullscreen QR mode** (`?fullscreen=1`) still bypasses the gate — it
  only renders the prototype's demo screens which carry no team data. If
  you want that gated too, remove the `fullscreen` early-return in `App()`.
- **`actorName` is auto-populated** from the Google profile's display name
  (first token) → falls back to email local-part. The `NamePrompt` modal
  is still wired up for cases where neither field is populated.

# Roll Streak

BJJ and workout tracker with streaks, a login page, and one-way sync of BJJ sessions to a Notion database.

- Frontend: `index.html` + `config.js` (plain HTML/JS, no build step)
- Login and database: Supabase (free tier)
- Notion sync: a Supabase Edge Function (`supabase/functions/notion-sync`)

Only mat sessions (Fundamentals, Advanced, Open Mat, Competition Class, Private Lesson) sync to Notion. Workouts stay in the app. Sessions from the first offline version of the app don't carry over.

## 1. Supabase (login + database)
1. Create a project at supabase.com.
2. SQL Editor > New query > paste `supabase/schema.sql` > Run.
3. Authentication > Providers: keep Email enabled. "Confirm email" is on by default, which is good for real users.
4. Project Settings > API: copy the Project URL and the `anon public` key into `config.js`.

## 2. Notion
1. Go to notion.so/my-integrations > New integration > copy the secret (starts with `ntn_` or `secret_`).
2. Open "🥋 BJJ Daily Class Tracker" > `...` > Connections > add your integration. Notion won't write to a database until you do this.
3. The database ID is `6f95441fa8b54a72b676b1d032a97761`.

## 3. Deploy the sync function
```
npm install -g supabase
supabase login
supabase link --project-ref YOUR-PROJECT-REF
supabase secrets set NOTION_TOKEN=YOUR_NOTION_SECRET NOTION_DATABASE_ID=6f95441fa8b54a72b676b1d032a97761
supabase functions deploy notion-sync
```

## 4. Put it on GitHub and host it
```
git init
git add .
git commit -m "Roll Streak v1"
git branch -M main
git remote add origin https://github.com/YOUR-USER/roll-streak.git
git push -u origin main
```
Then go to Settings > Pages > Deploy from a branch > `main` / `/ (root)`. GitHub Pages on a private repo needs a paid GitHub plan. Otherwise, make the repo public (safe, since no secrets are in it) or host the folder on Netlify or Cloudflare Pages.

In Supabase, go to Authentication > URL Configuration and set the Site URL to your Pages address.

## Security rules
- Never commit the Notion secret. Keep it only in `supabase secrets`.
- `config.js` holds the public anon key, which is safe. Each person can only read and write their own rows.
- Deleting a session in the app does not delete its Notion page. Sync is one-way.

## Changing things
- Notion property names must match the database exactly. If you rename a column in Notion, update `props` in `notion-sync/index.ts`.
- To add a new mat class type, add it to `MAT` in `notion-sync/index.ts` and to the Mat optgroup in `index.html`. The Notion database needs a matching option.

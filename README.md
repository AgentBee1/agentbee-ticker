# AgentBee — Updates & Alerts widget

Self-contained iframe widget showing institution-specific updates and alerts,
embedded in Circle.so client spaces.

---

## First-time setup

### 1. Add your credentials

Open `config.js` and replace the two placeholder values:

```js
const SUPABASE_URL  = 'https://YOUR_PROJECT_REF.supabase.co';
const SUPABASE_ANON = 'YOUR_ANON_KEY';
```

Find both in Supabase: **Project Settings → API**
- SUPABASE_URL  → "Project URL"
- SUPABASE_ANON → "anon / public" key

`config.js` is gitignored — it will never be committed to GitHub or
overwritten by a future zip from Claude. You only do this once.

### 2. Deploy to Vercel

- Push this folder to a new GitHub repo named `agentbee-ticker`
- Go to Vercel → Add New Project → Import from GitHub
- Select the repo → Deploy (no build settings needed)
- Copy the assigned URL e.g. `https://agentbee-ticker.vercel.app`

### 3. Set environment variables on Vercel

In Vercel: **Project Settings → Environment Variables**, add:

| Name           | Value                              |
|----------------|------------------------------------|
| SUPABASE_URL   | https://your-ref.supabase.co       |
| SUPABASE_ANON  | your-anon-key                      |

This means credentials are also safe on Vercel's side.

### 4. Embed in Circle.so

In each institution's Circle space, add a custom HTML embed:

```html
<iframe
  src="https://agentbee-ticker.vercel.app?institution_id=INSTITUTION_UUID&member_id=CIRCLE_MEMBER_ID"
  width="100%"
  height="600"
  frameborder="0"
  scrolling="no"
  style="border:none;">
</iframe>
```

Replace:
- `INSTITUTION_UUID`  — the institution's `id` from the `institutions` table
- `CIRCLE_MEMBER_ID` — the Circle member's `community_member_id` from the SSO session

---

## Updating the widget (future zips from Claude)

1. Download the new zip
2. Replace `index.html`, `vercel.json`, `README.md` in your GitHub repo
3. Leave `config.js` and `.gitignore` untouched
4. Vercel redeploys automatically on push — done

---

## Adding events to the feed

Insert rows into `notification_events` in Supabase:

```sql
insert into notification_events
  (institution_id, event_type, title, action_label, action_url, severity, icon)
values (
  'institution-uuid-here',
  'incident',
  'Incident report filed for Global Education Partners — Fraud allegation',
  'View report',
  'https://agentbee.com/reports/123',
  'high',
  'ti-alert-triangle'
);
```

### Event types
| event_type  | Use for                                    |
|-------------|--------------------------------------------|
| incident    | New or updated incident reports            |
| review      | Student reviews added                      |
| agent       | Agent added or removed from network        |
| risk        | Agent risk classification changes          |
| information | Knowledge base articles, community posts   |

### Severity values
| severity  | Displays as      | Colour |
|-----------|------------------|--------|
| high      | High severity    | Red    |
| med       | Medium severity  | Amber  |
| low       | Low severity     | Green  |
| risk-up   | Increased risk   | Red    |
| risk-down | Risk reduced     | Green  |
| (null)    | No pill shown    | —      |

### Icon values
Any Tabler icon name e.g. `ti-alert-triangle`, `ti-star`, `ti-user-plus`,
`ti-user-minus`, `ti-trending-up`, `ti-trending-down`, `ti-bulb`, `ti-users`

---

## Scheduled cleanup

To expire read notifications older than 90 days, enable `pg_cron` in
Supabase (**Database → Extensions**) then run once:

```sql
select cron.schedule(
  'expire-read-notifications',
  '0 3 * * *',
  'select public.cleanup_expired_notification_reads()'
);
```

# One-time Google Sign-In setup for the CSE Metrics Dashboard

The dashboard (`cse-metrics-dashboard.html`) is ready to fetch live data from the shared Google Drive folder, but it needs one thing before that switch turns on: a Google OAuth Client ID. This is a one-time setup, not something anyone repeats.

## Why this is needed
The Drive folder is shared as "Anyone at contentstack.com," which only unlocks for a request carrying a real signed-in Google identity — not a bare API key or link. So each viewer signs in once with their own Contentstack Google account, and their identity (already covered by the folder's existing sharing) is what grants read access.

## Steps (5–10 minutes, done once, by whoever has Google Cloud Console access for the org — likely IT/Workspace admin)

1. Go to console.cloud.google.com and select or create a project (any existing internal project works fine).
2. Enable the **Google Drive API** for that project (APIs & Services → Library → search "Google Drive API" → Enable).
3. Go to APIs & Services → Credentials → Create Credentials → OAuth client ID.
   - Application type: **Web application**
   - Authorized JavaScript origins: add the URL where the dashboard will be hosted (e.g. its Contentstack Launch URL). `file://` local previews cannot be authorized — this step only takes effect once the dashboard has a real hosted URL.
4. If prompted to configure the OAuth consent screen first: set User type to **Internal** (restricts sign-in to contentstack.com accounts only — this is the important setting) and fill in the required app name/support email fields.
5. Copy the generated **Client ID** (looks like `xxxxxxxxxx-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.apps.googleusercontent.com`).
6. Open `cse-metrics-dashboard.html`, find this line near the top of the `<script>` block:
   ```js
   const CONFIG = {
     GOOGLE_CLIENT_ID: '',
   ```
   Paste the Client ID between the quotes.
7. Re-host/redeploy the file. That's it — the "Sign in with Google" button in the dashboard header will start working, and once someone signs in, the page auto-refreshes from the live Drive snapshot every 5 minutes.

## What's already automated (no setup needed)
A Cowork scheduled task (`cse-dashboard-drive-snapshot`) already pulls all 5 metrics from Salesforce and writes a fresh JSON snapshot to the shared Drive folder every 5 minutes. Nothing needs to be triggered manually for the data side — only the sign-in wiring above is pending.

## Known limitation to be aware of
The Drive connector used for the write side has no delete/overwrite tool, so a new snapshot file is created every run and old ones aren't cleaned up. The dashboard always reads the newest file by creation time, so this doesn't affect correctness — but the folder will accumulate roughly 288 small JSON files/day. Worth revisiting later (either a periodic manual cleanup, or widening the refresh interval) if that folder needs to stay tidy.

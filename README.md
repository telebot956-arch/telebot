<<<<<<< HEAD
# Telegram Reposter (Cloud Run + Firestore)

This service pulls new posts from **public Telegram channels** and republishes them to **your channel** with a footer appended:

```
Limited Time Opportunities.
Share with your friends, colleagues, and college groups!

Thank You ❤️
```

> Sources: `@gocareers`, `@jobhuntcamp`, `@hiringdaily`  
> Target: `@careersin`

## ⚠️ Important limitations & compliance
- **Bots cannot read other people’s channels** unless they are members of those channels. To collect posts from public channels you don't control, you must use the **Telegram client API (MTProto)** with a **user account session** (here we use **Telethon**).  
- **Posting** to your channel is done via the **Bot API**. Add your bot as an **admin** in your target channel with “Post Messages” permission.  
- Only repost content you have rights to share. Avoid violating copyright or spamming. See Telegram’s Terms of Service.  

## Overview
- Stateless HTTP service (FastAPI) designed for **Cloud Run**.
- Triggered by **Cloud Scheduler** every minute (or your preferred interval).
- Keeps per-channel checkpoints in **Firestore** to avoid duplicates.
- Supports **text + photo/document**. (Albums & other media can be added later.)

---

## 0) One-time Telegram setup

### 0.1 Create a bot
1. Talk to **@BotFather** → `/newbot`. Save the **BOT_TOKEN**.
2. Add this bot as **Admin** to your channel `@careersin` with permission to **Post Messages**.

### 0.2 Get API ID/HASH and a StringSession for a user account
We use a **user account** (one of yours) to read the source channels.

1. Visit https://my.telegram.org → **API Development Tools** → create an app → note **api_id** and **api_hash**.
2. Generate a **STRING_SESSION** locally by running the helper below (once):
   ```bash
   python3 generate_string_session.py
   ```
   Follow the prompts (phone number, OTP, 2FA password if any). Copy the printed string safely.

> Tip: Open Telegram app and **join** `@gocareers`, `@jobhuntcamp`, `@hiringdaily` with the same account used to generate the StringSession.

---

## 1) Google Cloud + Firebase (use free credits)

> New accounts get **90-day $300** free credits. You’ll need to **enable billing** to access Cloud Run & Scheduler.

### 1.1 Create/Select project & link Firebase
```bash
gcloud auth login
gcloud auth application-default login
gcloud projects create <YOUR_PROJECT_ID>
gcloud config set project <YOUR_PROJECT_ID>
firebase login
firebase projects:addfirebase <YOUR_PROJECT_ID>
```

### 1.2 Enable APIs
```bash
gcloud services enable run.googleapis.com \
  cloudbuild.googleapis.com artifactregistry.googleapis.com \
  secretmanager.googleapis.com firestore.googleapis.com \
  cloudscheduler.googleapis.com
```

### 1.3 Initialize Firestore (Native mode)
In the Firebase Console → **Firestore Database** → **Create database** → **Native Mode** in a region near you.

---

## 2) Secrets & configuration

We’ll store sensitive values in **Secret Manager** and pass them into Cloud Run as env vars.

```bash
# Replace values
echo -n '<YOUR_BOT_TOKEN>'        | gcloud secrets create BOT_TOKEN --data-file=-
echo -n '<YOUR_TG_API_ID>'        | gcloud secrets create TG_API_ID --data-file=-
echo -n '<YOUR_TG_API_HASH>'      | gcloud secrets create TG_API_HASH --data-file=-
echo -n '<YOUR_STRING_SESSION>'   | gcloud secrets create TG_STRING_SESSION --data-file=-
```

You can later update a secret with:
```bash
echo -n '<NEW_VALUE>' | gcloud secrets versions add BOT_TOKEN --data-file=-
```

---

## 3) Build & deploy to Cloud Run

### 3.1 Build container
```bash
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/telegram-reposter:v1
```

### 3.2 Deploy service
```bash
gcloud run deploy telegram-reposter \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/telegram-reposter:v1 \
  --platform managed --region us-central1 \
  --allow-unauthenticated \
  --set-secrets BOT_TOKEN=BOT_TOKEN:latest,TG_API_ID=TG_API_ID:latest,TG_API_HASH=TG_API_HASH:latest,TG_STRING_SESSION=TG_STRING_SESSION:latest \
  --set-env-vars TARGET_CHANNEL=@careersin,SOURCE_CHANNELS=@gocareers,@jobhuntcamp,@hiringdaily \
  --set-env-vars FIRESTORE_COLLECTION=tg_reposter_state,FIRESTORE_DOC=offsets_v1,MAX_FETCH=50
```

> You may restrict invocation to **authenticated** only and use Cloud Scheduler with OIDC. If you keep `--allow-unauthenticated`, set a random path (e.g. `https://.../sync?key=<longsecret>`) and use it in the scheduler.

---

## 4) Schedule the sync

Find the service URL:
```bash
gcloud run services describe telegram-reposter --region us-central1 --format='value(status.url)'
```

Create a scheduler job to call `/sync` every minute:
```bash
gcloud scheduler jobs create http tg-reposter-sync \
  --schedule="* * * * *" \
  --http-method=GET \
  --uri="https://<SERVICE_URL>/sync" \
  --time-zone="Asia/Kolkata"
```

> If you disabled public access, add:  
> `--oidc-service-account-email=<PROJECT_NUMBER>-compute@developer.gserviceaccount.com`

---

## 5) Test it

1. Run the job once:
   ```bash
   gcloud scheduler jobs run tg-reposter-sync
   ```
2. Check Cloud Run logs (Logs Explorer) to verify messages were sent.
3. Confirm posts appear in `@careersin` with the footer.

---

## 6) Customization

- Change footer via env var:
  ```bash
  gcloud run services update telegram-reposter \
    --set-env-vars "FOOTER_TEXT=Limited Time Opportunities.\nShare with your friends, colleagues, and college groups!\n\nThank You ❤️"
  ```
- Add/remove source channels by updating `SOURCE_CHANNELS` env var (comma-separated).
- Increase `MAX_FETCH` to backfill more messages per run.

---

## Troubleshooting

- **403 posting to channel**: Ensure the bot is **Admin** in `@careersin` with post rights.
- **Telethon not authorized**: Regenerate `TG_STRING_SESSION` with `generate_string_session.py`.
- **No new messages**: Make sure the account (StringSession) has **joined** the source channels.
- **Rate limits**: The app handles Bot API `RetryAfter` and Telethon `FloodWaitError` with sleeps.
- **Region quotas/cost**: Cloud Run + Scheduler + Firestore typically fit in GCP free credits. Monitor in Billing.

---

## Notes on content usage
Reposting content from third-party channels may be subject to **copyright** and **platform rules**. Use attribution/permissions where required and avoid reposting gated or paid content. You are responsible for compliance.

Good luck! 🚀
=======
# telebot
>>>>>>> 5a677ac212edc9d3468cd6dea659dd7f0751dacd

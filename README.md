# SafePing MVP

Consent-based emergency location sharing triggered by repeated missed calls from a **verified trusted contact**.
Java + Android + Firebase (Auth, Firestore, Cloud Functions, FCM).

## Flow
1. Both people install SafePing and register.
2. Owner invites a contact by phone number -> contact accepts ("Trusted contacts" -> "Check/accept invites").
3. Owner turns **Emergency Mode** ON (explicit opt-in).
4. Trusted contact calls 2+ times within 10 min and is unanswered -> full-screen "Are you safe?" (30 s countdown).
5. "I'm safe" -> nothing is shared. No response -> Cloud Function `startShare` re-checks consent, stores a
   30-minute share, logs it, and pushes a link to ONLY that contact.
6. Link (`viewShare`) works only until expiry; `purgeExpired` deletes data every 10 minutes.
7. "Sharing history" screen shows who received your location and when it expires.

## Setup
1. Android Studio: open the `SafePing` folder.
2. Firebase console: create project, add Android app `com.example.safeping`,
   download `google-services.json` into `app/`.
3. Enable Authentication (Email/Password), Firestore, Cloud Messaging, and Cloud Functions (Blaze plan required).
4. `npm i -g firebase-tools && firebase login && cd functions && npm install && cd .. && firebase deploy`
   (create an empty `public/` folder first for hosting rewrites).
5. Run on two physical devices with SIM/phone numbers (the receiver needs a real incoming call).
   Grant Phone, Call log, Location, Notification permissions.

## Known MVP limits / next steps
- Missed-call detection uses the PHONE_STATE broadcast; on Android 9+ the incoming number needs READ_CALL_LOG
  (granted). Detection only works while the app has been run recently; harden with a foreground service or
  CallScreeningService for production.
- Emergency Mode is stored on-device; mirror it server-side in `startShare` for stronger enforcement.
- Location is captured once at timeout (foreground-capable). Live updates would need background location
  (`ACCESS_BACKGROUND_LOCATION`) plus Google Play declaration, disclosure and demo video.
- Phone ownership isn't verified (email login). Replace with Firebase phone-number OTP auth.
- Add SMS fallback and a demo/admin dashboard (consent status, alerts, expiry) as a web page.
- Add proper `res/` icons/layouts if desired; UI here is built in code for brevity.

## Safety rules enforced
- Never shares to arbitrary callers; contact must be invited + accepted.
- Emergency Mode must be on; user gets a chance to cancel.
- Temporary (max 60 min) and auto-expiring; every share is logged and visible to the owner.

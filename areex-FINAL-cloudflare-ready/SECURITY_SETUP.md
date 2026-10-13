# Security setup before deployment

## 1. Set server environment variables

Do this in your hosting provider's **server environment variables / secrets** screen, not in frontend code and not in GitHub. Copy the names from `.env.example`, but replace every `REPLACE_...` / `GENERATE_...` value.

Required in production:
- `SESSION_SECRET`: a unique random value, at least 32 characters.
- `AES_MASTER_KEY`: a different unique random value, at least 32 characters. Changing this key may make previously encrypted data unreadable, so back up and plan the rotation.
- `ADMIN_PASSWORD_1` and `ADMIN_PASSWORD_2`: two different, unique passwords, at least 16 characters each.
- `SMTP_EMAIL` and `SMTP_APP_PASSWORD`: a Gmail address plus a 16-character Gmail App Password (from <https://myaccount.google.com/apppasswords>). The admin login emails a one-time 6-digit code from this account.
- `ALLOWED_ORIGINS`: comma-separated frontend origins allowed to call this API when the site is hosted separately (for example on Cloudflare).

Never share these secrets in a chat, commit them to Git, or put them in client-side React code.

## 2. Admin login flow (email one-time code)

The admin login at `/rtx5090` is two-step: enter email + admin password, then the server emails a fresh 6-digit code to that admin's Gmail inbox. The code expires in 5 minutes, allows 5 guesses, and works once. No authenticator app is needed.

## 3. Customer 2FA

Customer accounts must enroll in an authenticator app before a session is issued. New signups are prompted to set it up immediately. Existing users are prompted on their next login; their previous sessions are rejected until MFA is enabled.

The customer setup screen displays the secret and `otpauth://` URI. Add the secret manually to an authenticator app, then enter the current six-digit code to complete enrollment.

## 4. Audit log and saved changes

- Open `/rtx5090`, log in, then choose **Audit Log**.
- The server appends API request records to `.areex-audit.jsonl`; the admin screen shows up to the latest 1,000 records.
- Logs contain timestamps, actor/attempted login identity, route, method, HTTP status, IP, user-agent, field names, and safe short values where available. Passwords, OTP codes, tokens, and other secret fields are excluded.
- Admin edits to plans, coupons, discount settings, support tickets, and site config are stored in server-side runtime files. Keep `.areex-store.json`, `.areex-users.json`, and `.areex-audit.jsonl` on persistent writable storage and back them up. If your hosting filesystem is ephemeral, use a persistent volume/database before relying on these records.

## 5. Before sending real traffic

1. Confirm the host serves the site over HTTPS.
2. Add all required environment variables before restarting production. The server intentionally refuses to start in production if critical security variables are missing or malformed.
3. Test a customer signup and login with an authenticator app, and test admin login for both admins.
4. Verify the audit log records a test admin change.
5. Keep `.env` private and rotate any credentials that have ever been hardcoded into source code.

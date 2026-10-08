# Vault

## Bootstrap (only use of env vars)

- `GOOGLE_APPLICATION_CREDENTIALS` — path to the GCP JSON key, or ADC when it is unset.
- `GCP_PROJECT_ID`.

Nothing else. `SATORU_ADMIN_EMAIL`, `JWT_SECRET`, and `JWT_TTL_SECS` are vault documents, not env vars.

## Secret document

Path:

    {APP_NAME}/{CONTEXT}/vault/{SECRET_NAME}

```json
{
  "name": "JWT_SECRET",
  "value": "super-secret-string"
}
```

`name` matches the document id. `value` is a string.

## Documents this layer reads

| Document id | Value |
|-------------|--------|
| JWT_SECRET | HS256 secret |
| JWT_TTL_SECS | optional. Positive integer overrides the default. Missing or invalid means 604800 |
| SATORU_ADMIN_EMAIL | email compared lowercase on signup |

## Rules

- Read through the `secret_provider` port. Handlers do not query `vault/` themselves.
- Cache at most 5 minutes.
- Never commit a secret. Never log `value`.
- Context of the secret follows the request context. Local work reads vault under `"0"` only.

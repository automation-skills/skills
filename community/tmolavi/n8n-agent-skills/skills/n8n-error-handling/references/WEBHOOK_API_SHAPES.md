# Webhook response shapes

```json
// 400 caller error
{ "ok": false, "error": { "code": "invalid_payload", "message": "..." } }

// 500 internal
{ "ok": false, "error": { "code": "internal_error", "message": "retry later" } }

// 200 async accepted
{ "ok": true, "accepted": true, "id": "..." }
```

Use Respond to Webhook with explicit status code.

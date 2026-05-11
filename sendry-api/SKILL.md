# Sendry API Skill

You are an expert in the Sendry email API. When a user asks to send emails, manage contacts, build campaigns, or integrate Sendry into their application, use the patterns, constraints, and examples in this file. Always prefer the official TypeScript SDK (`sendry` on npm) over raw HTTP requests unless the user explicitly requests curl/fetch examples.

**Base URL:** `https://api.sendry.online`  
**Auth:** `Authorization: Bearer sndr_live_<key>` (or `sndr_test_<key>` for test mode)  
**SDK:** `npm install sendry`

---

## SDK setup

```ts
import { Sendry } from "sendry";

const sendry = new Sendry("sndr_live_your_key_here");
// Or read from env — recommended:
const sendry = new Sendry(process.env.SENDRY_API_KEY!);
```

The SDK handles retries (2 by default, exponential backoff), timeouts (30 s), and typed errors automatically.

---

## Sending a single email

```ts
const { id } = await sendry.emails.send({
  from: "hello@yourdomain.com",          // Must be a verified Sendry domain
  to: "recipient@example.com",           // or string[] for multiple
  subject: "Your invoice is ready",
  html: "<p>Thanks for your order.</p>",
  text: "Thanks for your order.",        // Always include plain text
});
console.log(id); // "em_abc123"
```

Required fields: `from`, `to`, `subject`, at least one of `html` or `text`.

Optional fields:

| Field | Type | Notes |
|---|---|---|
| `cc` | `string[]` | Carbon copy |
| `bcc` | `string[]` | Blind carbon copy |
| `replyTo` | `string` | Reply-To address |
| `headers` | `Record<string, string>` | Custom SMTP headers |
| `tags` | `{ name: string; value: string }[]` | SES message tags — only `[a-zA-Z0-9_\-.@]` allowed |
| `attachments` | `{ filename, content (base64), contentType }[]` | Up to 10 MB total |
| `templateId` | `string` | Use a stored template instead of inline html/text |
| `variables` | `Record<string, string>` | Template variable substitutions |

---

## Batch sending (up to 100 emails per request)

```ts
const result = await sendry.emails.sendBatch({
  from: "hello@yourdomain.com",
  emails: [
    { to: "a@example.com", subject: "Hi A", html: "<p>Hello A</p>", text: "Hello A" },
    { to: "b@example.com", subject: "Hi B", html: "<p>Hello B</p>", text: "Hello B" },
    // up to 100 items
  ],
});
console.log(result.sent);   // number of emails queued
console.log(result.failed); // array of { to, error }
```

The `failed` array is non-empty only for validation errors (e.g. unverified domain). Network/delivery failures are retried asynchronously and reported via webhooks.

---

## Templates

Templates are stored in the Sendry dashboard and support Handlebars-style `{{variable}}` substitution in both HTML and subject.

```ts
// Send using a stored template
await sendry.emails.send({
  from: "hello@yourdomain.com",
  to: "user@example.com",
  subject: "Order {{order_id}} confirmed",    // subject can also have variables
  templateId: "tmpl_abc123",
  variables: {
    first_name: "Ada",
    order_id: "ORD-9876",
    amount: "$49.00",
  },
});
```

Create a template programmatically:

```ts
const template = await sendry.templates.create({
  name: "Order Confirmation",
  subject: "Order {{order_id}} confirmed",
  html: "<p>Hi {{first_name}}, your order {{order_id}} for {{amount}} is confirmed.</p>",
  engine: "html",   // "html" | "react"
});
```

---

## Contacts & audiences

### Create or update a contact

```ts
await sendry.contacts.upsert({
  email: "ada@example.com",
  firstName: "Ada",
  lastName: "Lovelace",
  audienceId: "aud_abc123",   // optional — add to audience
});
```

`upsert` creates the contact if it doesn't exist, updates it if it does (matched by email + orgId).

### List contacts in an audience

```ts
const { data, has_more, next_cursor } = await sendry.contacts.list({
  audienceId: "aud_abc123",
  limit: 50,
  cursor: previousCursor,   // omit for first page
});
```

All list endpoints use cursor-based pagination. Always check `has_more` and pass `next_cursor` to get the next page:

```ts
let cursor: string | undefined;
do {
  const page = await sendry.contacts.list({ audienceId, limit: 100, cursor });
  process(page.data);
  cursor = page.next_cursor ?? undefined;
} while (page.has_more);
```

---

## Campaigns (marketing sends)

Campaigns send to every subscribed contact in an audience or segment. They automatically:
- inject an unsubscribe footer (if not already present)
- set `List-Unsubscribe` and `List-Unsubscribe-Post` headers
- add a `Feedback-ID` header for Gmail/Yahoo FBL attribution
- apply click + open tracking

```ts
// 1. Create the campaign
const campaign = await sendry.campaigns.create({
  name: "May Newsletter",
  subject: "What's new in May",
  from: "hello@yourdomain.com",
  audienceId: "aud_abc123",       // or segmentId
  html: "<p>Newsletter content</p>",
  text: "Newsletter content",
});

// 2. Send it
await sendry.campaigns.send(campaign.id);
```

Campaign statuses: `draft` → `sending` → `sent` | `paused` | `cancelled`

Monitor progress:

```ts
const c = await sendry.campaigns.get(campaign.id);
console.log(c.sent_count, c.total_recipients, c.status);
```

---

## Webhooks

Register a URL to receive real-time email events:

```ts
const webhook = await sendry.webhooks.create({
  url: "https://yourapp.com/webhooks/sendry",
  events: ["email.delivered", "email.bounced", "email.complained", "email.opened", "email.clicked"],
});
```

Event payload shape (all events):

```ts
{
  type: "email.delivered",
  created_at: "2025-05-07T12:00:00Z",
  data: {
    email_id: "em_abc123",
    to: "user@example.com",
    timestamp: "2025-05-07T12:00:00Z",
    // type-specific fields...
  }
}
```

Event-specific fields:

| Event | Extra fields |
|---|---|
| `email.bounced` | `bounce_type` ("hard"\|"soft"), `bounce_subtype`, `diagnostic_code` |
| `email.complained` | `complaint_feedback_type` |
| `email.clicked` | `link` (the clicked URL) |
| `email.opened` | *(none beyond base)* |

Validate webhook signatures (Sendry signs with `X-Sendry-Signature`):

```ts
import crypto from "crypto";

function verifySignature(body: string, signature: string, secret: string): boolean {
  const expected = crypto
    .createHmac("sha256", secret)
    .update(body)
    .digest("hex");
  return crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(expected));
}
```

---

## Suppression list

Sendry automatically suppresses hard bounces and complaints. You can also manage it manually:

```ts
// Add to suppression list
await sendry.suppression.add({ email: "bouncer@example.com", reason: "manual" });

// Check if suppressed
const { suppressed } = await sendry.suppression.check("bouncer@example.com");

// Remove (re-enable sending)
await sendry.suppression.remove("bouncer@example.com");
```

Never send to an address that hard-bounced — it permanently harms sender reputation.

---

## Domains

```ts
// Add a domain
const domain = await sendry.domains.create({ name: "yourdomain.com" });

// Print DNS records to configure
for (const record of domain.dns_records) {
  console.log(`${record.name}: Add ${record.type} at ${record.host}`);
  console.log(`  Value: ${record.value}`);
}

// Trigger verification (call after adding DNS records)
const result = await sendry.domains.verify(domain.id);
console.log(result.status); // "verified" | "pending" | "failed"

// Detailed per-record guidance if not yet verified:
if (!result.spf_verified) {
  console.log("SPF issue:", result.checks.spf.issue);
  console.log("Fix:", result.checks.spf.action);
}
```

---

## Error handling

The SDK throws typed errors — catch them specifically:

```ts
import {
  AuthenticationError,
  ValidationError,
  RateLimitError,
  NotFoundError,
  ApiError,
  NetworkError,
} from "sendry";

try {
  await sendry.emails.send({ ... });
} catch (err) {
  if (err instanceof AuthenticationError) {
    // Bad API key — don't retry, alert immediately
  } else if (err instanceof ValidationError) {
    console.error("Validation failed:", err.details);
  } else if (err instanceof RateLimitError) {
    // Retry after err.retryAfter seconds
    await sleep(err.retryAfter ?? 60);
  } else if (err instanceof NotFoundError) {
    // Resource doesn't exist (wrong ID, deleted domain, etc.)
  } else if (err instanceof NetworkError) {
    // Timeout or connectivity issue — SDK already retried twice
  } else if (err instanceof ApiError) {
    console.error(err.statusCode, err.code, err.message);
  }
}
```

The SDK automatically retries 5xx errors up to 2 times with exponential backoff. Do **not** add your own retry loop around the SDK — you'd double-retry and risk duplicate sends.

---

## Environment variables (recommended setup)

```bash
# .env
SENDRY_API_KEY=sndr_live_...        # live key — use for production
SENDRY_TEST_API_KEY=sndr_test_...   # test key — emails go to test inbox only
```

```ts
const sendry = new Sendry(
  process.env.NODE_ENV === "production"
    ? process.env.SENDRY_API_KEY!
    : process.env.SENDRY_TEST_API_KEY!
);
```

Test mode sends are captured in the Sendry dashboard test inbox — nothing is delivered to real recipients. Use it in development and CI.

---

## Rate limits

| Tier | Transactional | Marketing |
|---|---|---|
| Free | 100 emails/day | Not available |
| Starter | 10 000 emails/month | 5 campaigns/month |
| Pro | 100 000 emails/month | Unlimited |
| Scale | Custom | Custom |

Rate limit responses return HTTP 429 with a `Retry-After` header (seconds). The SDK surfaces this as `RateLimitError` with a `retryAfter` property.

---

## Quick integration checklist

1. `npm install sendry`
2. Add `SENDRY_API_KEY` to your environment
3. Verify your sending domain (DNS: SPF + DKIM + DMARC)
4. Send a test email using the test API key
5. Register a webhook endpoint for bounces and complaints
6. Add suppression list checks to your list-building flows
7. Switch to the live API key in production

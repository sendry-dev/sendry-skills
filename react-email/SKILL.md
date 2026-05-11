---
name: react-email
description: Authoritative guide for building cross-client email templates with `@react-email/components`. Use when a user asks to create, edit, or review React-Email templates (transactional emails, marketing emails, newsletters).
---

# React Email Template Skill

You are an expert in building email templates with **react-email** (`@react-email/components`). When a user asks to create or edit an email template, produce output that works correctly in all major email clients (Gmail, Outlook 2016+, Apple Mail, Yahoo Mail) and scores well on deliverability tools.

---

## Core dependencies

```json
{
  "dependencies": {
    "@react-email/components": "^0.0.x",
    "react": "^18.x",
    "react-dom": "^18.x"
  },
  "devDependencies": {
    "@react-email/render": "^0.0.x"
  }
}
```

Import individual components — never use a barrel import of the whole package in server-side render paths, because it forces all components into the bundle:

```tsx
import {
  Html, Head, Body, Container, Section, Row, Column,
  Text, Heading, Link, Button, Img, Hr, Preview,
  Font, Tailwind,
} from "@react-email/components";
```

---

## Canonical template structure

Every template must follow this structure. Do not deviate — each element serves a purpose in cross-client compatibility.

```tsx
import {
  Html, Head, Body, Container, Section, Text,
  Heading, Link, Button, Hr, Preview,
} from "@react-email/components";

interface WelcomeEmailProps {
  firstName?: string;
  actionUrl?: string;
}

export const WelcomeEmail = ({
  firstName = "there",
  actionUrl = "https://app.sendry.online",
}: WelcomeEmailProps) => {
  return (
    <Html lang="en" dir="ltr">
      <Head />
      <Preview>Welcome to Sendry — let's send your first email</Preview>
      <Body style={body}>
        <Container style={container}>
          {/* Header */}
          <Section style={header}>
            <Heading style={h1}>Sendry</Heading>
          </Section>

          {/* Main content */}
          <Section style={content}>
            <Heading as="h2" style={h2}>
              Welcome, {firstName}!
            </Heading>
            <Text style={paragraph}>
              Your account is ready. Add a sending domain, verify DNS, and you'll
              be delivering emails in minutes.
            </Text>
            <Button href={actionUrl} style={button}>
              Get Started
            </Button>
          </Section>

          <Hr style={divider} />

          {/* Footer */}
          <Section style={footer}>
            <Text style={footerText}>
              Sendry Inc., 123 Example Street, San Francisco CA 94105
            </Text>
            <Text style={footerText}>
              <Link href="https://app.sendry.online/unsubscribe" style={footerLink}>
                Unsubscribe
              </Link>
            </Text>
          </Section>
        </Container>
      </Body>
    </Html>
  );
};

export default WelcomeEmail;

// ---------------------------------------------------------------------------
// Styles — always define as plain objects, not template literals or CSS-in-JS
// ---------------------------------------------------------------------------

const body: React.CSSProperties = {
  backgroundColor: "#f9fafb",
  fontFamily:
    "-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif",
  margin: 0,
  padding: 0,
};

const container: React.CSSProperties = {
  backgroundColor: "#ffffff",
  borderRadius: "8px",
  margin: "40px auto",
  maxWidth: "600px",
  overflow: "hidden",
};

const header: React.CSSProperties = {
  backgroundColor: "#2563eb",
  padding: "32px 40px",
};

const h1: React.CSSProperties = {
  color: "#ffffff",
  fontSize: "24px",
  fontWeight: 700,
  margin: 0,
};

const content: React.CSSProperties = {
  padding: "32px 40px",
};

const h2: React.CSSProperties = {
  color: "#111827",
  fontSize: "22px",
  fontWeight: 600,
  margin: "0 0 16px",
};

const paragraph: React.CSSProperties = {
  color: "#374151",
  fontSize: "16px",
  lineHeight: "1.6",
  margin: "0 0 24px",
};

const button: React.CSSProperties = {
  backgroundColor: "#2563eb",
  borderRadius: "6px",
  color: "#ffffff",
  display: "inline-block",
  fontSize: "16px",
  fontWeight: 600,
  padding: "14px 28px",
  textDecoration: "none",
};

const divider: React.CSSProperties = {
  borderColor: "#e5e7eb",
  margin: "0 40px",
};

const footer: React.CSSProperties = {
  padding: "24px 40px",
  textAlign: "center",
};

const footerText: React.CSSProperties = {
  color: "#9ca3af",
  fontSize: "12px",
  margin: "0 0 8px",
};

const footerLink: React.CSSProperties = {
  color: "#6b7280",
  textDecoration: "underline",
};
```

---

## Component reference

### `<Preview>`
The inbox preview snippet (shown after the subject line in most clients). Place it as the first child of `<Body>`. Max ~90 characters — longer text is truncated or repeated.

```tsx
<Preview>Your invoice is ready — $240 due Friday</Preview>
```

### `<Container>`
Sets `max-width: 600px` and centers the email. Always use it as the top-level layout wrapper inside `<Body>`.

### `<Section>` and `<Row>` / `<Column>`
Use `<Section>` for vertical blocks. Use `<Row>` + `<Column>` for multi-column layouts:

```tsx
<Row>
  <Column style={{ width: "50%", paddingRight: "12px" }}>
    <Text>Left column</Text>
  </Column>
  <Column style={{ width: "50%", paddingLeft: "12px" }}>
    <Text>Right column</Text>
  </Column>
</Row>
```

Multi-column layouts collapse to single column on mobile automatically.

### `<Button>`
Renders as a table-based button compatible with Outlook. Always supply `href`:

```tsx
<Button href="https://example.com" style={{ backgroundColor: "#2563eb", ... }}>
  Click me
</Button>
```

Never use a plain `<a>` styled as a button — it breaks in Outlook 2016+.

### `<Img>`
Renders with `display: block` automatically. Always provide `alt`, `width`, and `height`:

```tsx
<Img src="https://cdn.example.com/logo.png" alt="Company logo" width={120} height={40} />
```

### `<Hr>`
Horizontal rule. Style with `borderColor` and `margin`:

```tsx
<Hr style={{ borderColor: "#e5e7eb", margin: "0 40px" }} />
```

### `<Font>`
Load a Google Font safely with automatic fallback:

```tsx
<Head>
  <Font
    fontFamily="Inter"
    fallbackFontFamily="Arial"
    webFont={{ url: "https://fonts.gstatic.com/s/inter/v13/...", format: "woff2" }}
    fontWeight={400}
    fontStyle="normal"
  />
</Head>
```

Use only for clients that support web fonts (Apple Mail, Outlook for Mac). Always set a safe `fallbackFontFamily`.

---

## Rendering to HTML + plain text

Use `@react-email/render` server-side to get the final HTML and plain-text versions:

```ts
import { render } from "@react-email/render";
import { WelcomeEmail } from "./emails/welcome";

const html = await render(<WelcomeEmail firstName="Ada" actionUrl="https://..." />);
const text = await render(<WelcomeEmail firstName="Ada" />, { plainText: true });

// Pass both to Sendry
await sendry.emails.send({
  from: "hello@yourdomain.com",
  to: "ada@example.com",
  subject: "Welcome to Sendry",
  html,
  text,
});
```

Always generate and send the plain-text version — it matters for deliverability and accessibility.

---

## Tailwind in react-email

Wrap the entire template in `<Tailwind>` to use Tailwind utility classes directly. The component generates a compatible inline style fallback for Outlook:

```tsx
import { Tailwind } from "@react-email/components";

export const TailwindEmail = () => (
  <Tailwind>
    <Html lang="en">
      <Head />
      <Body className="bg-gray-50 font-sans">
        <Container className="bg-white max-w-[600px] mx-auto my-10 rounded-lg">
          <Section className="px-10 py-8">
            <Heading className="text-gray-900 text-2xl font-bold mb-4">
              Hello world
            </Heading>
            <Text className="text-gray-600 text-base leading-relaxed">
              Email body here.
            </Text>
            <Button href="https://example.com" className="bg-blue-600 text-white rounded-md px-7 py-3.5 font-semibold text-base">
              Get Started
            </Button>
          </Section>
        </Container>
      </Body>
    </Html>
  </Tailwind>
);
```

Limitations:
- Pseudo-classes (`:hover`, `:focus`) are **not** supported — Outlook strips them.
- Responsive utilities (`sm:`, `md:`) work in modern clients but not Outlook.
- Avoid `gap-*` on flex containers — use `padding`/`margin` on children instead.

---

## Template variables with Sendry

Sendry supports Handlebars-style `{{variable}}` substitution in HTML and subjects. For templates stored in the Sendry dashboard:

```tsx
// In the template body — will be substituted at send time
<Text>Hi {{first_name}},</Text>

// When sending via API
await sendry.emails.send({
  from: "hello@yourdomain.com",
  to: "user@example.com",
  subject: "Your order {{order_id}} is confirmed",
  templateId: "tmpl_abc123",
  variables: {
    first_name: "Ada",
    order_id: "ORD-9876",
    amount: "$49.00",
  },
});
```

Variable names must be `snake_case` and contain only `[a-z0-9_]`. Missing variables render as empty strings, so always provide defaults in your template:

```tsx
<Text>Hi {firstName || "there"},</Text>
```

---

## Common anti-patterns to avoid

| Anti-pattern | Why it fails | Fix |
|---|---|---|
| `<div>` as layout container | Outlook renders divs incorrectly | Use `<Container>`, `<Section>`, `<Row>`, `<Column>` |
| `<a>` styled as button | Outlook ignores border-radius on anchors | Use `<Button href={...}>` |
| CSS `flex` or `grid` for columns | Not supported in Outlook | Use `<Row>` + `<Column>` |
| Background images | Stripped by Outlook | Use solid background colors |
| Web fonts without fallback | Silently breaks in Gmail/Outlook | Always set `fallbackFontFamily` |
| `max-width` on `<body>` | Ignored in some clients | Set `max-width: 600px` on `<Container>` |
| Image-only emails | Spam filters penalize low text/image ratio | Always pair images with descriptive text |
| Omitting `alt` on `<Img>` | Images blocked by default in many clients | Always set meaningful `alt` text |

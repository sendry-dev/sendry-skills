# Email Best Practices Skill

You are an expert in email deliverability, HTML email development, and email marketing. When helping users write or review email content, apply all of the guidelines below. Never skip a section — each one materially affects inbox placement or engagement.

---

## 1. Authentication — always verify first

Before sending anything, confirm the domain has all three records in place:

- **SPF** — `v=spf1 include:amazonses.com ~all` on the root domain. Only one SPF record is allowed; merge with any existing one.
- **DKIM** — a 2048-bit RSA key published at `<selector>._domainkey.<domain>`. Long keys must be split across multiple strings (255-char limit per string) and will be reassembled by receivers.
- **DMARC** — at minimum `v=DMARC1; p=none;` at `_dmarc.<domain>`. Graduate to `p=quarantine` then `p=reject` once `ruf`/`rua` reports confirm alignment.

No authentication = high spam rate regardless of content quality.

---

## 2. From address

- Use a **real, monitored mailbox** for the local part (e.g. `hello@`, `team@`, not `noreply@`). Unmanned `noreply` addresses increase complaints because recipients can't reply to opt out gracefully.
- Keep the **friendly name** consistent across sends: "Sendry" not "Sendry Team" one week and "The Sendry Team" the next. Inconsistency erodes recognition.
- The From domain must match (or be aligned with) the DKIM-signing domain for DMARC to pass.

---

## 3. Subject lines

- **40–60 characters** — most mobile clients truncate beyond that.
- Lead with the most important word. "Invoice ready: $240 due Friday" beats "Regarding your account — invoice".
- Avoid spam-trigger patterns: ALL CAPS, excessive punctuation (`!!!`, `???`), misleading "Re:" or "Fwd:" prefixes, and urgency words like "FREE", "ACT NOW", "WINNER".
- Use the **preheader** (the text after the subject in inbox preview) as a second sentence, not a repetition of the subject. Set it as a hidden `<span>` in the email `<body>` before any visible content.

```html
<!-- Preheader pattern — hidden, but visible in inbox preview -->
<span style="display:none;max-height:0;overflow:hidden;">
  Your March invoice is ready — due in 5 days.
</span>
```

---

## 4. HTML structure

Every marketing or transactional HTML email must follow this skeleton exactly:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge" />
  <title>Email subject here</title>
  <!--[if mso]>
  <noscript><xml><o:OfficeDocumentSettings>
    <o:PixelsPerInch>96</o:PixelsPerInch>
  </o:OfficeDocumentSettings></xml></noscript>
  <![endif]-->
</head>
<body style="margin:0;padding:0;background-color:#f9fafb;">
  <!-- Hidden preheader -->
  <span style="display:none;max-height:0;overflow:hidden;">Preview text here.</span>

  <!-- Wrapper table -->
  <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="100%"
         style="background-color:#f9fafb;">
    <tr>
      <td align="center" style="padding:40px 16px;">
        <!-- Content table: max 600px -->
        <table role="presentation" cellspacing="0" cellpadding="0" border="0" width="600"
               style="max-width:600px;background:#ffffff;border-radius:8px;">
          <!-- Header -->
          <tr>
            <td style="padding:32px 40px 24px;border-bottom:1px solid #e5e7eb;">
              <!-- Logo / brand -->
            </td>
          </tr>
          <!-- Body -->
          <tr>
            <td style="padding:32px 40px;">
              <!-- Main content -->
            </td>
          </tr>
          <!-- Footer -->
          <tr>
            <td style="padding:24px 40px;border-top:1px solid #e5e7eb;text-align:center;
                       font-size:12px;color:#9ca3af;font-family:-apple-system,BlinkMacSystemFont,
                       'Segoe UI',Roboto,sans-serif;">
              <!-- Address, unsubscribe link -->
            </td>
          </tr>
        </table>
      </td>
    </tr>
  </table>
</body>
</html>
```

Key rules:
- Use `role="presentation"` on all layout tables so screen readers ignore them.
- **Max width 600px** — the safe zone for all major clients.
- Inline all CSS. `<style>` blocks are stripped by Gmail and some Outlook versions.
- Use `border="0"`, `cellpadding="0"`, `cellspacing="0"` on every table.
- Set `font-family` on every element — email clients do not inherit from `<body>`.

---

## 5. Typography

- **Body text**: 16px minimum, 1.5–1.6 line height. Use a system font stack:
  `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif`
- **Headlines**: 24–32px, font-weight 600–700.
- Avoid web fonts via `@import` in the `<head>` — they fail silently in Outlook and slow render in Gmail. Stick to the system stack or use a Google Fonts `<link>` with a safe fallback.
- Line length: 50–70 characters per line for optimal readability. The 600px wrapper enforces this at normal font sizes.

---

## 6. Calls to action

- One primary CTA per email. Multiple competing CTAs split attention and reduce clicks.
- Style the CTA as a **table-based button**, not a `<div>` or `<a>` alone — Outlook doesn't support `border-radius` on `<a>` elements:

```html
<table role="presentation" cellspacing="0" cellpadding="0" border="0">
  <tr>
    <td style="border-radius:6px;background:#2563eb;">
      <a href="https://your-link.com"
         style="display:inline-block;padding:14px 28px;color:#ffffff;
                font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,sans-serif;
                font-size:16px;font-weight:600;text-decoration:none;border-radius:6px;">
        Get Started
      </a>
    </td>
  </tr>
</table>
```

- Minimum tap target: **44×44px**. Pad generously.
- Contrast ratio ≥ 4.5:1 between button text and background (WCAG AA).

---

## 7. Images

- Always set `alt` text. Many clients block images by default — the alt text is the first thing the recipient sees.
- Use `display:block` on `<img>` to prevent the 1–4px gap that table cells add beneath images.
- Host images on a reliable CDN, not as attachments. Attachments increase spam scores dramatically.
- Never rely on images alone to convey critical information (price, CTA label, deadline).
- Limit total image payload to **< 200KB** for the whole email.

---

## 8. Plain-text version

Always send both HTML and a plain-text version. Rules:
- Every link must appear as a full URL in the plain text (not anchor text).
- Maintain the same hierarchy: headline → body → CTA → footer.
- Maximum line length: 72 characters (use soft line wraps).
- The plain-text version is also read by spam filters — make it coherent, not a dumbed-down version of the HTML.

---

## 9. Footer & legal requirements

Under CAN-SPAM, GDPR, and CASL, every commercial email must include:

1. **Physical mailing address** — a real street address or a registered PO box.
2. **Unsubscribe mechanism** — a one-click link that processes the request within 10 business days (CAN-SPAM) or immediately (GDPR). Sendry injects this automatically for marketing sends.
3. **Honest From/Subject** — no deceptive headers.

Failure to include these is a legal liability, not just a deliverability issue.

---

## 10. Deliverability checklist (run before every send)

| Check | Pass condition |
|-------|---------------|
| SPF | Record exists, includes sending provider |
| DKIM | Key published, selector matches From domain |
| DMARC | Policy exists; `rua` report address set |
| Spam score | < 3.0 on Mail-Tester or GlockApps |
| Text/image ratio | At least 60% text by content area |
| No URL shorteners | Full destination URLs only |
| List hygiene | Hard bounces removed, suppression list applied |
| Unsubscribe link | Present and functional in every marketing send |
| Physical address | In footer |
| Reply-To | Points to a monitored inbox |

---

## 11. List hygiene

- Remove hard bounces **immediately** after they occur. Sending to hard bounces again permanently harms your sender reputation.
- Soft bounces: suppress after 3–5 consecutive failures.
- Implement a **sunset policy**: stop sending to subscribers who haven't opened or clicked in 6 months. Send a re-engagement email first, then suppress.
- Never purchase email lists. Every address on a purchased list is a potential spam trap.
- Use **double opt-in** for marketing lists — the confirmation email proves consent and dramatically improves list quality.

---

## 12. Send timing

- Transactional emails: send immediately, no delay.
- Marketing campaigns: Tuesday–Thursday, 9am–11am or 1pm–3pm in the recipient's local timezone typically outperform.
- Avoid sending large volumes in a single burst if your IP is new — warm up gradually (start at 200/day, double weekly).
- Space campaign sends by at least 3–5 days to avoid inbox fatigue.

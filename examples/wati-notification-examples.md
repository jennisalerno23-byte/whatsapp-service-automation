# WATI Notification Examples

## Overview

Freshservice uses the WATI REST API to send automated WhatsApp notifications after processing tickets created through the chatbot.

These examples are sanitized representations of the production requests.

No production API tokens, internal URLs, customer phone numbers, or company identifiers are included.

---

## Incident Number Notification

After a WhatsApp-originated ticket is created, Freshservice sends the incident number back to the customer.

### Request

```http
POST https://<wati-server>/api/v1/sendSessionMessage/<customer-mobile-number>
Authorization: Bearer <WATI_API_TOKEN>
Content-Type: application/x-www-form-urlencoded
```

### Message Body

```text
messageText=Dear customer, your request has been registered with incident number {{ticket.id}}. Our technical team will review your case shortly.
```

The `{{ticket.id}}` value is dynamically obtained from the Freshservice ticket.

---

## Engineer Assignment Notification

After the On-Call engineer is assigned, Freshservice sends a second WhatsApp notification.

### Request

```http
POST https://<wati-server>/api/v1/sendSessionMessage/<customer-mobile-number>
Authorization: Bearer <WATI_API_TOKEN>
Content-Type: application/x-www-form-urlencoded
```

### Message Body

```text
messageText=Update for case {{ticket.id}}: your request has been assigned to {{ticket.assigned_agent.name}}, who will be responsible for handling your case. Further follow-up will be provided via email.
```

Freshservice dynamically provides:

* `{{ticket.id}}` — incident identifier
* `{{ticket.assigned_agent.name}}` — assigned engineer

The destination mobile number comes from the ticket information originally received from WATI.

---

## Message Sequence

The intended customer communication sequence is:

```text
Ticket created
      |
      v
Incident number notification
      |
      v
Engineer assignment
      |
      v
60-second workflow delay
      |
      v
Engineer assignment notification
```

The workflow delay prevents the engineer-assignment message from reaching the customer before the incident-number notification.

---

## Security

The production implementation uses a WATI API credential for authentication.

The following information must never be committed to this repository:

* WATI API keys
* Bearer tokens
* Production WATI endpoint identifiers
* Customer mobile numbers
* Production ticket information

Placeholders are used in all public examples.

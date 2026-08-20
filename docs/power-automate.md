# Power Automate Integration

## Overview

Microsoft Power Automate is used to process after-hours requests for the Sales and Administration departments.

WATI collects customer information through the WhatsApp chatbot and sends the data to Power Automate through an HTTP POST request.

Power Automate then converts the incoming request into a structured email notification for the appropriate internal team.

Two separate flows are used:

* Sales email notification
* Administration email notification

Both flows follow the same technical pattern.

---

## Integration Flow

```text
Customer
   |
   v
WhatsApp
   |
   v
WATI Chatbot
   |
   | Collect customer information
   v
Customer Confirmation
   |
   v
HTTP POST
   |
   v
Power Automate
   |
   v
HTTP Request Trigger
   |
   v
Send Email (V2)
   |
   v
Internal Department
```

---

## Incoming Request

WATI sends the collected information to a dedicated Power Automate HTTP endpoint.

A sanitized request example is:

```json
{
  "nombre": "@nombre",
  "telefono": "{{phone}}",
  "correo": "@correo",
  "detalle": "@descripcion"
}
```

The values are generated dynamically from the information collected during the WhatsApp conversation.

The mobile number is obtained directly from the WhatsApp session.

---

## HTTP Trigger

Each Power Automate flow begins with an HTTP request trigger.

The trigger accepts a JSON object containing the following properties:

```json
{
  "type": "object",
  "properties": {
    "nombre": {
      "type": "string"
    },
    "telefono": {
      "type": "string"
    },
    "correo": {
      "type": "string"
    },
    "detalle": {
      "type": "string"
    }
  }
}
```

This schema allows Power Automate to expose the incoming values as dynamic content for later actions.

---

# Sales Flow

## Flow Name

```text
WATI - Envío Correo Ventas
```

The production implementation uses a dedicated HTTP endpoint for the Sales workflow.

After receiving the request, Power Automate executes a **Send an email (V2)** action.

The email subject follows this pattern:

```text
[WHATSAPP SALES] New request from: <customer name>
```

The message includes:

* Customer name
* Mobile number
* Email address
* Request details

It also includes a WhatsApp contact link generated from the customer's mobile number.

Conceptually:

```text
https://wa.me/<mobile-number>
```

This allows the receiving team to initiate follow-up without manually searching for or re-entering the customer's number.

---

# Administration Flow

## Flow Name

```text
WATI - Envío Correo Administración
```

The Administration flow uses the same technical structure as the Sales flow.

It includes:

```text
HTTP Request Trigger
        |
        v
Send Email (V2)
```

The JSON schema is equivalent to the Sales implementation.

The main differences are:

* Dedicated HTTP endpoint
* Administration email recipient
* Department-specific message content

Maintaining separate flows allows the requests to be routed independently while reusing the same integration pattern.

---

## Dynamic Content Mapping

The values received from WATI are mapped directly into Power Automate dynamic content.

Conceptually:

```text
WATI                  Power Automate
----------------------------------------
@nombre            -> nombre
{{phone}}          -> telefono
@correo            -> correo
@descripcion       -> detalle
```

These values are then inserted into the generated email.

---

## Why Power Automate Was Used

The after-hours workflow required WATI to forward collected customer information to internal Sales and Administration teams.

Direct email delivery from the chatbot was not used in the implemented solution.

Power Automate therefore acts as the integration layer between WATI and the corporate email service.

This keeps the chatbot focused on customer interaction while Power Automate handles internal notification processing.

---

## Separation from Technical Support

The Technical Support workflow uses a different backend process.

```text
Technical Support
      |
      v
Freshservice REST API
      |
      v
Ticket + Workflow Automation
```

Sales and Administration do not require incident creation and instead use:

```text
Sales / Administration
      |
      v
Power Automate
      |
      v
Internal Email Notification
```

This allows each department to use a workflow aligned with its operational requirements.

---

## Security Considerations

Power Automate generates an HTTP endpoint used by WATI to trigger each flow.

Production endpoint values must not be stored in a public repository.

The complete trigger URL may contain sensitive authorization information and should be treated as a secret.

For this reason, this repository does not include:

* Production Power Automate trigger URLs
* Signature or authorization values
* Corporate email addresses
* Customer information
* Internal environment identifiers

Sanitized placeholders are used throughout the documentation.

---

## Current Limitations

The current implementation is intentionally simple and focused on reliable request forwarding.

Potential limitations include:

* Separate flows must be maintained for Sales and Administration.
* The HTTP endpoint is directly exposed for external invocation.
* There is no centralized integration error dashboard.
* Failed email delivery handling can be improved.
* Retry and exception handling can be expanded.
* Request information is currently delivered primarily through email.

---

## Potential Improvements

Possible future enhancements include:

* Additional endpoint authentication controls
* Centralized reusable flow components
* Error logging and alerting
* Automated retries
* Request persistence before email delivery
* Integration with Microsoft Lists, Dataverse, or another request-tracking system
* Shared notification logic for multiple departments
* Additional validation before processing incoming requests

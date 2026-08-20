# Chatbot Flow

## Overview

The chatbot is implemented in WATI and acts as the main interaction layer between customers and the service workflow.

It provides different behavior depending on configured business hours and routes the customer according to the selected department.

The main goals of the chatbot are to:

* Provide an official WhatsApp service channel
* Route requests to the appropriate department
* Allow live-agent interaction when available
* Collect structured customer information
* Validate user input before processing
* Create technical support tickets automatically
* Forward Sales and Administration requests outside business hours
* Allow customers to navigate between menus

## Business-Hours Logic

Business hours are configured directly in WATI.

WATI automatically selects the corresponding chatbot flow depending on whether the interaction occurs during business hours or outside the configured schedule.

The implementation uses separate chatbot flows for:

* Business hours
* After-hours periods, including lunch breaks, holidays, and other non-working periods

This allows the same WhatsApp number to provide different routing behavior without requiring the customer to select a schedule manually.

## Main Menu

The customer is welcomed and asked to select one of three service areas:

```text
Technical Support
Sales
Administration
```

If the customer provides an invalid option, the chatbot displays an error message and asks the customer to select one of the available options.

## Technical Support Flow

After selecting Technical Support, the chatbot provides the following options:

```text
Open ticket
Talk to an agent
Previous menu
```

### Talk to an Agent

During business hours, the conversation can be transferred to the ROC support team.

Outside business hours, the conversation can be transferred to the configured On-Call support process.

WATI agent accounts are associated with corporate user accounts used for live customer support.

### Open Ticket

When the customer chooses to create a support ticket, the chatbot starts a structured data-collection process.

The customer is informed that some basic information is required.

The chatbot collects:

1. Customer name
2. Email address
3. Incident description
4. Province
5. Mobile number

The mobile number is obtained directly from the WhatsApp session and does not need to be entered manually by the customer.

## Email Validation

The email address is validated in WATI before the flow can continue.

The validation uses the following regular expression:

```regex
^[^\s@]+@[^\s@]+\.[^\s@]+$
```

If the value does not match the expected email format, the chatbot asks the customer to enter the address again.

The customer is allowed up to three attempts.

This validation helps reduce invalid requester information before the data is sent to Freshservice.

## Province Selection

Province is collected using a predefined selection list rather than free-text input.

The available options include:

```text
Bocas del Toro
Coclé
Colón
Chiriquí
Darién
Herrera
Los Santos
Panamá
Veraguas
Panamá Oeste
```

Using a controlled list reduces typing mistakes and prevents multiple variations of the same value from entering the ticketing system.

## Data Confirmation

Before the ticket is created, WATI displays the collected information back to the customer.

The confirmation screen includes:

```text
Name
Email
Province
Incident description
```

The customer can then choose one of the following options:

```text
Confirm
Previous menu
Main menu
```

The Freshservice integration is triggered only after the customer confirms the information.

This confirmation step reduces the possibility of creating incidents with incorrect or incomplete information.

## Freshservice Ticket Creation

After confirmation, WATI sends an HTTP POST request directly to the Freshservice REST API.

The production endpoint follows this pattern:

```text
https://<freshservice-domain>/api/v2/tickets
```

Authentication is handled using Freshservice API credentials.

The request body is generated dynamically using the information captured by the chatbot.

A sanitized example of the request structure is:

```json
{
  "email": "@email",
  "subject": "WhatsApp Support: @name",
  "description": "<b>New Incident:</b><br><br><b>User:</b> @name<br><b>Province:</b> @province<br><b>Phone:</b> {{phone}}<br><br><b>Details:</b><br>@description",
  "phone": "{{phone}}",
  "priority": 1,
  "status": 2,
  "custom_fields": {
    "province": "@province",
    "mobile_number": "{{phone}}"
  }
}
```

The exact production field names, internal URLs, and credentials are not included in this repository.

## Sales Flow

During business hours, customers selecting Sales can be transferred directly to an available agent.

Outside business hours, the chatbot collects the information required for follow-up.

The collected information includes:

```text
Customer name
Email address
Request description
Mobile number
```

The mobile number is obtained from the WhatsApp session.

Before sending the information, the customer is asked to review and confirm the captured data.

After confirmation, WATI sends the information through an HTTP POST request to a Microsoft Power Automate HTTP trigger.

Power Automate then generates a structured email notification for the Sales team.

## Administration Flow

Administration follows the same logic as Sales.

During business hours, the conversation can be transferred to an available Administration agent.

Outside business hours, WATI collects customer contact information and the request description.

After confirmation, the information is sent through a dedicated Power Automate HTTP endpoint.

A separate Power Automate flow generates the internal notification for the Administration team.

## Sales and Administration Request Structure

Both after-hours flows use the same request structure.

A sanitized example is:

```json
{
  "nombre": "@nombre",
  "telefono": "{{phone}}",
  "correo": "@correo",
  "detalle": "@descripcion"
}
```

The Power Automate endpoint is different for each department, while the data structure remains equivalent.

## Navigation and Error Handling

The chatbot includes navigation controls that allow customers to return to:

* The previous menu
* The main menu

Invalid selections are handled through default branches.

When the customer selects an unavailable or unsupported option, WATI displays an explanatory message and asks the customer to select one of the available alternatives.

Input validation is also applied where appropriate, such as the email-address validation used during ticket creation.

## Design Considerations

Several design decisions were introduced to improve reliability and data quality:

* Business-hour routing is handled directly by WATI.
* Department selection uses predefined buttons.
* Province selection uses a predefined list instead of free text.
* Email format is validated before ticket creation.
* Customers confirm their information before external integrations are triggered.
* Mobile numbers are obtained directly from the WhatsApp session.
* Menu navigation allows customers to correct their path without restarting the conversation.
* Technical Support, Sales, and Administration use different backend integrations depending on operational requirements.

## Current Improvement Opportunities

The chatbot can be enhanced further in future iterations.

Potential improvements include:

* Adding an explicit conversation-exit option
* Expanding validation rules for additional fields
* Improving retry and fallback behavior
* Adding centralized error logging for failed integrations
* Improving chatbot messages and contextual greetings
* Reviewing greeting text dynamically instead of using time-specific wording
* Adding more advanced session and conversation-state management

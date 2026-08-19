# Solution Architecture

## Overview

The solution uses WhatsApp as the main customer-facing communication channel and WATI as the chatbot and messaging platform.

WATI routes requests according to business hours and the department selected by the customer.

The architecture integrates three main platforms:

* **WATI** for chatbot interactions, routing, data collection, and WhatsApp messaging.
* **Freshservice** for automated technical support ticket creation and workflow processing.
* **Microsoft Power Automate** for forwarding Sales and Administration requests received outside business hours.

## Main Flow

```text
Customer
   |
   v
WhatsApp
   |
   v
WATI
   |
   +-----------------------+-----------------------+
   |                       |                       |
   v                       v                       v
Technical Support         Sales               Administration
   |                       |                       |
   |                       +-----------+-----------+
   |                                   |
   v                                   v
Freshservice                     Power Automate
   |                                   |
   v                                   v
Ticket Workflows                 Email Notification
   |
   v
WATI REST API
   |
   v
WhatsApp Notifications
```

## Business Hours Routing

Business hours are configured directly in WATI.

Two main chatbot flows are used:

* Business-hours chatbot
* After-hours chatbot

WATI automatically selects the appropriate chatbot depending on the configured working schedule.

### During Business Hours

Customers can select one of three departments:

* Technical Support
* Sales
* Administration

Sales and Administration conversations can be transferred directly to available agents.

Technical Support customers can either interact with a support agent or create a support ticket.

### Outside Business Hours

Sales and Administration customers can leave their contact information and request details.

The collected information is sent through an HTTP webhook to Microsoft Power Automate, which generates an email notification for the corresponding department.

Technical Support remains available through the On-Call process and also allows automated ticket creation.

## Technical Support Integration

When a customer chooses to create a support ticket, WATI collects:

* Name
* Email address
* Incident description
* Province
* Mobile number

The mobile number is obtained from the WhatsApp session.

The email address is validated before continuing, and the customer is asked to review and confirm the information before ticket creation.

Province selection uses a predefined list to reduce data-entry inconsistencies.

After confirmation, WATI sends an HTTP POST request to the Freshservice REST API.

```text
POST /api/v2/tickets
```

The request contains the customer information and the required ticket attributes.

Freshservice then processes the ticket using Workflow Automator.

## Freshservice to WATI Integration

Freshservice uses outbound webhooks to communicate back with WATI.

After a WhatsApp-originated ticket is created, a Freshservice workflow verifies that:

* The ticket subject identifies it as a WhatsApp request.
* A mobile number is available.

The workflow then sends an HTTP POST request to the WATI REST API.

This allows the customer to receive the newly created incident number directly through WhatsApp.

A private note is also added to the Freshservice ticket to provide operational traceability.

## On-Call Assignment

Outside business hours, Freshservice automatically assigns the configured On-Call engineer.

The assignment workflow checks that:

* The ticket was created outside business hours.
* No engineer has already been assigned.
* The ticket originated from the WhatsApp process.

After the engineer is assigned, the workflow waits 60 seconds before sending the assignment notification.

The delay was introduced because the incident-number workflow and assignment workflow execute almost simultaneously.

Without the delay, the engineer-assignment notification could arrive before the incident-number notification.

The minimum available timer value was used to maintain the intended customer message sequence:

```text
1. Incident created
2. Incident number notification
3. Engineer assignment
4. Engineer assignment notification
```

The assignment notification dynamically includes both the ticket ID and assigned engineer name.

## Sales and Administration Integration

Sales and Administration use equivalent after-hours workflows.

WATI collects:

* Customer name
* Mobile number
* Email address
* Request details

The information is sent as JSON to a Power Automate HTTP trigger.

Power Automate then generates a structured email notification for the appropriate internal team.

The email also includes a WhatsApp contact link based on the customer's mobile number so the receiving team can follow up directly.

## Security Considerations

The production implementation uses authentication credentials including:

* Freshservice API credentials
* WATI API credentials
* Power Automate HTTP trigger URLs

Production credentials and internal endpoint values are intentionally excluded from this repository.

All examples included in this repository must use sanitized placeholders.

## Current Limitations

The current implementation includes some operational limitations:

* The On-Call engineer rotation is updated manually.
* The engineer must currently be updated across three equivalent Freshservice workflows.
* Multiple workflows are required because of scheduling constraints in the current Freshservice configuration.
* The support ticket structure can be further enriched with additional ITSM fields and classification.
* Message sequencing currently relies on a fixed workflow timer.

## Potential Improvements

Future versions could include:

* Automated On-Call rotation
* Centralized scheduling logic
* Dynamic engineer assignment from an external roster
* Improved webhook authentication
* Retry and error-handling mechanisms
* More advanced Freshservice categorization
* Automated incident classification
* Removal of the fixed sequencing timer through event-based processing

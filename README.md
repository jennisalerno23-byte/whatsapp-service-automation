# WhatsApp Service Automation

A service automation solution that integrates **WhatsApp, WATI, Freshservice, Power Automate, and REST APIs** to streamline customer requests, incident creation, routing, and automated notifications.

## Overview

This project was designed to provide an official WhatsApp-based service channel where customers can interact with different business areas and submit requests without depending exclusively on live-agent availability.

The solution supports three main service areas:

* Technical Support
* Sales
* Administration

Depending on the selected department and business hours, the chatbot can transfer the conversation to an available agent, collect customer information for later follow-up, or automatically create a support ticket.

## Main Technologies

* WATI
* WhatsApp Business
* Freshservice
* Microsoft Power Automate
* REST APIs
* JSON
* HTTP Webhooks

## Key Features

* Business-hours and after-hours chatbot flows
* Department-based request routing
* Input validation and structured data collection
* Automated Freshservice ticket creation
* Automatic incident-number notification through WhatsApp
* On-call engineer assignment outside business hours
* Automated WhatsApp notification after engineer assignment
* Sales and Administration request forwarding through Power Automate
* Operational traceability using Freshservice private notes
* Navigation and error handling within chatbot flows

## High-Level Architecture

```text
Customer
   |
   v
WhatsApp
   |
   v
WATI Chatbot
   |
   +--------------------+--------------------+
   |                    |                    |
   v                    v                    v
Technical Support      Sales            Administration
   |                    |                    |
   v                    +---------+----------+
Freshservice                      |
REST API                          v
   |                       Power Automate
   v                              |
Ticket + Workflows                v
   |                         Email Notification
   v
WATI REST API
   |
   v
WhatsApp Notifications
```

## Repository Purpose

This repository documents the architecture, integration logic, implementation decisions, and sanitized examples of the solution.

It does **not** contain production credentials, API keys, customer information, internal URLs, or other confidential company data.

Detailed technical documentation and architecture diagrams will be added progressively.

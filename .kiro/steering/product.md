---
inclusion: always
---

# Product: TechStore SupportBot

## Overview

TechStore SupportBot is a customer support chatbot that answers frequently asked questions about the TechStore e-commerce platform. When it cannot answer a question, it collects the customer's name, email, and query to open a support ticket.

## Core Capabilities

- Answer FAQs covering: delivery deadlines, product exchanges/returns, and payment methods
- Detect when a question falls outside the FAQ scope
- Collect customer info (name, email, question) and open a support ticket for unanswered queries
- Provide clear, friendly responses in Portuguese (pt-BR)

## Domain Concepts

- **FAQ**: A predefined question-answer pair covering known support topics
- **Ticket**: A support request created when the bot cannot resolve the customer's query; contains customer name, email, and the original question
- **Intent**: The classification of a customer message (e.g., FAQ match vs. unknown)

## Business Rules

- The bot must always attempt to match a customer message to a known FAQ before escalating
- A ticket must only be created when no FAQ match is found
- Ticket creation requires all three fields: name, email, and question — none are optional
- Responses must be concise and helpful; avoid technical jargon in customer-facing messages

## Out of Scope

- Authentication or user account management
- Order tracking or real-time inventory queries
- Proactive outreach or notifications

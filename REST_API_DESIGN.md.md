# HadatHub – REST API Architecture Design

Project: Regional Event & Ticketing System

---

## Executive Summary

HadatHub’s REST API is designed as a clean, scalable blueprint for managing regional events and ticketing across East Africa. The architecture models real-world entities such as events, venues, organizers, users, attendees, and tickets using RESTful resources with predictable CRUD operations. Emphasis is placed on correctness, scalability, and developer usability through proper normalization, stateless design, and consistent error handling.

---

## Domain Analysis (Business Analysis)

HadatHub is replacing a manual, error-prone workflow (Excel sheets and email confirmations) with a centralized digital system capable of supporting event discovery, ticket sales, and attendee validation across multiple East African cities such as Nairobi, Kigali, Dar es Salaam, and Kampala.

In this ecosystem, ticketing challenges include overselling venues, unclear attendee identity (buyer vs attendee), event cancellations, multi-currency pricing, and fraud at event entry points. A robust API design must therefore model more than just “Events” and reflect the operational realities faced by organizers and venue staff.

### Primary Business Entities

1. **Venue**  
   Venues define physical constraints such as capacity and location. Capacity directly limits ticket inventory and prevents overselling.

2. **Event**  
   Events are the core product listing users browse and attend. Events must support scheduling, categorization, pricing, and cancellation.

3. **Organizer**  
   Organizers are responsible for creating, managing, and canceling events. Modeling organizers explicitly enables accountability and permission boundaries.

4. **User**  
   Users represent authenticated platform accounts that purchase tickets or perform staff actions such as check-in.

5. **Attendee**  
   In East African contexts, tickets are often purchased for friends or family. Separating attendee from buyer ensures accurate attendance tracking.

6. **Ticket**  
   Tickets represent the right to attend an event and serve as the join between events and attendees. They support fraud prevention via unique codes and one-time check-in.

### Relationships

- One Venue → Many Events
- One Organizer → Many Events
- One Event → Many Tickets
- One User (buyer) → Many Tickets
- One User → Many Attendees
- Many Events ↔ Many Attendees (implemented via Tickets)

### Core Operations

- Discover events by city, date, and category
- Purchase tickets and assign them to attendees
- Prevent overselling and invalid ticket sales
- Validate tickets at entry
- Cancel events and block further sales

---

## Resource Specifications

### Data Type Conventions

- UUID: String
- DateTime: ISO 8601 string
- Money: Decimal string (e.g., `"30.00"`)
- Enum: String with predefined values

---

### Venue

- id: UUID
- name: String
- address: String
- city: String
- country: String
- max_capacity: Integer (>0)
- timezone: String
- created_at: ISO 8601
- updated_at: ISO 8601

**Rules**

- Capacity must be greater than zero
- Optional: no overlapping events at the same venue

---

### Organizer

- id: UUID
- name: String
- contact_email: String (unique)
- contact_phone: String
- country: String
- created_at: ISO 8601
- updated_at: ISO 8601

---

### Event

- id: UUID
- organizer_id: UUID
- venue_id: UUID
- title: String
- description: String
- category: Enum (`conference`, `concert`, `festival`, `workshop`)
- start_time: ISO 8601
- end_time: ISO 8601
- status: Enum (`scheduled`, `canceled`, `completed`)
- ticket_price: Decimal string
- currency: String
- created_at: ISO 8601
- updated_at: ISO 8601

**Rules**

- end_time must be after start_time
- No ticket sales if canceled or completed
- Ticket count ≤ venue capacity

---

### User

- id: UUID
- full_name: String
- email: String (unique)
- phone: String
- role: Enum (`buyer`, `staff`, `admin`)
- status: Enum (`active`, `suspended`)
- created_at: ISO 8601
- updated_at: ISO 8601

---

### Attendee

- id: UUID
- full_name: String
- email: String (optional)
- phone: String (optional)
- created_by_user_id: UUID
- created_at: ISO 8601
- updated_at: ISO 8601

---

### Ticket

- id: UUID
- ticket_code: String (unique)
- event_id: UUID
- buyer_user_id: UUID
- attendee_id: UUID
- price_paid: Decimal string
- currency: String
- status: Enum (`valid`, `canceled`, `checked_in`)
- checked_in_at: ISO 8601 (nullable)
- created_at: ISO 8601

**Rules**

- No sales if event is canceled or sold out
- Ticket can be checked in only once

---

## Endpoint Documentation

### Base URL

`/api/v1`

### Common Status Codes

- 200 OK
- 201 Created
- 204 No Content
- 400 Bad Request
- 403 Forbidden
- 404 Not Found
- 409 Conflict

---

### Example: Tickets (CRUD + Domain Actions)

| Action          | Method | URI                    |
| --------------- | ------ | ---------------------- |
| Purchase Ticket | POST   | /tickets               |
| Get Ticket      | GET    | /tickets/{id}          |
| Cancel Ticket   | PATCH  | /tickets/{id}          |
| Delete Ticket   | DELETE | /tickets/{id}          |
| Check-in Ticket | POST   | /tickets/{id}/check-in |

**Purchase Ticket – Request**

```json
{
  "event_id": "EVENT_UUID",
  "buyer_user_id": "USER_UUID",
  "attendee_id": "ATTENDEE_UUID"
}
```

## AI Tool Usage Disclosure

An AI tool (ChatGPT) was used to assist with structuring and refining this document. All design decisions and technical content were reviewed and finalized by the author.

# Craftlane

Craftlane is a modern freelance marketplace built for project-based work.

It connects clients with independent professionals and provides the infrastructure needed to manage the entire lifecycle of a freelance project — from publishing a job and selecting a freelancer to contracts, milestones, communication, delivery, payments, and reviews.

The platform is designed around trust, transparency, and long-term professional relationships rather than simple hourly hiring.

## Core idea

Traditional freelance platforms often focus on profiles, hourly rates, and large lists of freelancers.

Craftlane takes a project-first approach.

A client creates a project with a defined scope, budget, deadline, and expected deliverables. Freelancers can apply to projects that match their skills and availability. Once a freelancer is selected, both parties work through a structured project workflow with milestones, communication, deliverables, and payments.

The goal is to make freelance work feel less like an open marketplace and more like a reliable professional workspace.

## Key principles

- Project-first instead of profile-first
- Outcome-oriented work instead of hours
- Transparent contracts and milestones
- Secure payments between clients and freelancers
- Clear responsibilities for both parties
- Reputation based on completed work
- Real-time communication
- Simple and predictable workflows
- Strong separation between business logic and presentation
- Privacy and security by default

## Main features

### Client accounts

Clients can:

- Create and manage projects
- Define project requirements
- Set budgets and deadlines
- Receive and review applications
- Compare freelancers
- Communicate with applicants
- Select a freelancer
- Create contracts
- Divide projects into milestones
- Approve delivered work
- Make payments
- Leave reviews

### Freelancer accounts

Freelancers can:

- Create professional profiles
- Define skills and expertise
- Add portfolio projects
- Set availability
- Browse available projects
- Submit applications
- Communicate with clients
- Manage active contracts
- Deliver work
- Track milestones
- Receive payments
- Build a professional reputation

### Projects

Each project has a structured lifecycle:

    Draft
      ↓
    Published
      ↓
    Applications
      ↓
    Freelancer selected
      ↓
    Contract active
      ↓
    In progress
      ↓
    Review
      ↓
    Completed

Projects can contain multiple milestones, each with its own description, deadline, amount, and delivery status.

### Contracts

Contracts define the relationship between a client and a freelancer.

A contract can contain:

- Scope of work
- Deliverables
- Budget
- Payment terms
- Milestones
- Deadlines
- Revision terms
- Cancellation conditions
- Contract status

### Milestones

Large projects can be divided into smaller milestones.

Each milestone can have:

- Title
- Description
- Amount
- Deadline
- Deliverables
- Status
- Submission
- Client approval

This provides a clear progression of work and reduces ambiguity between both parties.

### Messaging

Craftlane provides real-time communication between clients and freelancers.

Messaging supports:

- Direct conversations
- Project conversations
- Message history
- Delivery notifications
- Online presence
- Read status
- System events

Project-related communication remains associated with the relevant project or contract.

### Payments

Payments are tied to contracts and milestones rather than arbitrary transfers.

The payment lifecycle is designed around:

    Payment initiated
          ↓
    Payment authorized
          ↓
    Work delivered
          ↓
    Client approval
          ↓
    Freelancer payout

The platform itself should not implement card processing or financial infrastructure from scratch. A dedicated marketplace payment provider should handle payment processing, identity verification, payouts, refunds, and related financial operations.

### Reviews and reputation

After completing a project, both parties can review each other.

Reputation can include:

- Overall rating
- Completed projects
- Successful completion rate
- On-time delivery
- Client reviews
- Freelancer reviews
- Professional skills
- Portfolio history

Reputation is intended to reflect actual completed work rather than simply the amount of activity on the platform.

### Disputes

Projects may enter a dispute when the client and freelancer cannot agree on delivery, scope, payment, or contract conditions.

The dispute system should provide:

- Dispute creation
- Evidence and attachments
- Project history
- Contract information
- Milestone information
- Communication history
- Administrative review
- Resolution
- Refund or payout actions

## Architecture

Craftlane is designed as a modular monolith.

The initial system intentionally avoids premature microservices and keeps the business domain inside a single Phoenix application.

    ┌──────────────────────────────────┐
    │            Phoenix               │
    │                                  │
    │  LiveView      JSON API          │
    │      │             │             │
    │      └──────┬──────┘             │
    │             │                    │
    │        Domain Contexts           │
    │             │                    │
    └─────────────┼────────────────────┘
                  │
        ┌─────────┼──────────┐
        │         │          │
        ▼         ▼          ▼
    PostgreSQL   Oban     Object Storage
       Ecto                 S3 / R2

The architecture can later evolve into separate services if a specific domain requires independent scaling.

## Technology stack

### Backend

- Elixir
- Phoenix
- Phoenix LiveView
- Ecto
- PostgreSQL
- OTP
- Oban

### Frontend

- Phoenix LiveView
- HEEx
- Tailwind CSS
- JavaScript/TypeScript only where necessary

### Infrastructure

- PostgreSQL
- S3-compatible object storage
- Background jobs through Oban
- WebSockets / Phoenix Channels where appropriate
- Containerized deployment
- GitHub Actions for CI/CD

## Domain structure

The application is organized around business domains rather than technical layers.

Possible contexts include:

    Accounts
    Profiles
    Skills
    Projects
    Applications
    Contracts
    Milestones
    Messaging
    Payments
    Reviews
    Disputes
    Notifications
    Files
    Organizations

Each context owns its business rules and persistence logic.

For example:

    Projects
      ├── project creation
      ├── publishing
      ├── project status
      └── project requirements

    Contracts
      ├── contract creation
      ├── terms
      ├── acceptance
      └── lifecycle

    Milestones
      ├── creation
      ├── delivery
      ├── approval
      └── completion

This keeps the business domain explicit and makes the application easier to maintain as the product grows.

## Data model

The initial domain model is centered around:

    User
      ├── Freelancer Profile
      ├── Client Organization
      └── Authentication

    Project
      ├── Applications
      ├── Contract
      ├── Milestones
      ├── Messages
      ├── Files
      └── Reviews

    Contract
      ├── Client
      ├── Freelancer
      ├── Milestones
      ├── Payments
      └── Disputes

Financial records should use an immutable transaction/ledger model rather than relying on a mutable account balance.

## Security

Security is a core requirement of the platform.

Important areas include:

- Secure authentication
- Authorization by role and resource ownership
- Organization-level access control
- Input validation
- CSRF protection
- Rate limiting
- Secure file uploads
- Audit logs
- Payment webhook verification
- Protection against unauthorized contract and payment changes
- Secure handling of personal information

Every sensitive action should be authorized on the server side.

The client should never be trusted to determine permissions, ownership, payment state, or contract state.

## Realtime architecture

Phoenix PubSub and Channels can be used for realtime functionality such as:

- Messaging
- Notifications
- Presence
- Project status changes
- Application updates
- Milestone events
- Payment status updates

LiveView can consume these events and update the interface without requiring a traditional frontend/backend synchronization layer.

## Background processing

Long-running and asynchronous operations should be handled outside the request lifecycle.

Oban can process:

- Emails
- Notifications
- Payment reconciliation
- File processing
- Scheduled reminders
- Deadline notifications
- Cleanup jobs
- Webhook processing
- Periodic maintenance

Jobs should be designed to be retryable and idempotent wherever possible.

## Payment architecture

Craftlane should integrate with a dedicated payment provider designed for marketplaces.

The platform should maintain its own internal representation of financial events while delegating sensitive payment processing and payouts to the external provider.

Financial operations should be auditable.

Example:

    Client payment
        ↓
    Payment provider
        ↓
    Marketplace transaction
        ↓
    Platform fee
        ↓
    Freelancer payout

Money-related state changes should be represented as explicit transactions rather than hidden mutations.

## Repository structure

A possible Phoenix project structure:

    craftlane/
    ├── assets/
    ├── config/
    ├── lib/
    │   ├── craftlane/
    │   │   ├── accounts/
    │   │   ├── profiles/
    │   │   ├── projects/
    │   │   ├── applications/
    │   │   ├── contracts/
    │   │   ├── milestones/
    │   │   ├── messaging/
    │   │   ├── payments/
    │   │   ├── reviews/
    │   │   ├── disputes/
    │   │   └── notifications/
    │   │
    │   └── craftlane_web/
    │       ├── components/
    │       ├── controllers/
    │       ├── live/
    │       └── channels/
    │
    ├── priv/
    │   ├── repo/
    │   └── static/
    │
    ├── test/
    ├── mix.exs
    └── README.md

The exact structure may evolve as domain boundaries become clearer.

## Development philosophy

Craftlane follows a few important engineering principles:

### Keep the domain explicit

Business rules should live in domain contexts rather than controllers or LiveView modules.

### Prefer simple architecture

Start with a modular monolith.

Introduce additional infrastructure only when there is a concrete reason to do so.

### PostgreSQL first

Use PostgreSQL as the primary source of truth for transactional business data.

### Server-side authorization

All important permissions must be enforced on the server.

### Immutable financial history

Financial events should be auditable and traceable.

### Realtime where it matters

Use realtime functionality for communication and state that benefits from immediate updates, rather than making every interaction realtime unnecessarily.

### Test business rules

The most important tests should cover:

- Project lifecycle
- Applications
- Contract transitions
- Milestones
- Payments
- Permissions
- Disputes
- Reviews

## MVP scope

The first release should focus on the smallest complete marketplace workflow:

    Client
      ↓
    Create project
      ↓
    Publish project
      ↓
    Receive applications
      ↓
    Select freelancer
      ↓
    Create contract
      ↓
    Create milestone
      ↓
    Freelancer delivers
      ↓
    Client approves
      ↓
    Payment released
      ↓
    Both parties review each other

Everything outside this workflow should be considered secondary until the core marketplace loop is reliable.

## Long-term direction

Craftlane is intended to become a professional infrastructure layer for independent work.

Potential future areas include:

- Teams
- Agencies
- Recurring contracts
- Subscription-based services
- Enterprise accounts
- Multi-currency support
- Advanced reporting
- Time tracking
- Invoicing
- Tax documentation
- Public portfolios
- Professional verification
- Escrow workflows
- Project templates
- External integrations
- Mobile applications

These features should be added without compromising the core project lifecycle or introducing unnecessary complexity.

## Status

Craftlane is currently in development.

The initial goal is to build a reliable, maintainable marketplace foundation before expanding the feature set.

## License

License to be determined.

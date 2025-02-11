# The Sysops Squad Architectural Kata by O'Reilly, May 2021

## Team Seven

**Pavel, Suheyl, Nikita, Hassan**

## Contents
<img align="right" width="210" height="368" src="images/badge.png">

- [Welcome](#welcome)  
    - [About the name](#about-the-name)
- [Business Case](#business-case)  
- [System Requirements](#system-requirements)  
    - [Functional Requirements](#functional-requirements)
    - [Architecture Characteristics Requirements](#architecture-characteristics-requirements)
    - [Constraints](#constraints)
    - [Assumptions](#assumptions)
- [Baseline Architecture](#baseline-architecture)  
- [Target Architecture](#target-architecture)  
    - [Use Case Model](#use-case-model)  
    - [System Context](#system-context)  
    - [Containers](#containers)  
    - [Process Views](#process-views)  
    - [Deployment](#deployment)  
- [Transition Architecture](#transition-architecture)
- [Architecture Decision Records](#architecture-decision-records)

## Welcome
> *Everything in software architecture is a trade-off.  
First Law of Software Architecture*

Welcome to the Sysops Squad Architectural Kata run by O'Reilly in May 2021.

This page is architectural documentation for the solution proposal by **Team Seven**.

### About the name
<img align="right" width="200" height="183" src="images/jersey.jpg">
Magic number Seven... This is not a random number in our name. This number joined our team together. One of us has a birthday at 07/07 thus joined the Team Seven. Somebody had a successful career in a football team under number 7 thus joined the Team Seven. Somebody believes that this is his lucky number throughout the whole life.  
And of course, the average number of services in Service-based architecture that we defined as our initial candidate, is about 7.
That's why we decided to make this number a symbol of our team.

## Business Case

Penultimate Electronics is a large electronics giant that has numerous retail stores throughout the country. When customers buy computers, TV's, stereos, and other electronic equipment, they can choose to purchase a support plan. Customer-facing technology experts (the "Sysops Squad") will then come to the customers residence (or work office) to fix problems with the electronic device.

The current trouble ticket system is a large monolithic application that was developed many years ago. Customers are complaining that consultants are never showing up due to lost tickets, and often times the wrong consultant shows up to fix something they know nothing about. Customers and call-center staff have been complaining that the system is not always available for web-based or call-based problem ticket entry. Change is difficult and risky in this large monolith - whenever a change is made, it takes too long and something else usually breaks. Due to reliability issues, the monolithic system frequently "freezes up" or crashes - they think it's mostly due a spike in usage and the number of customers using the system. If something isn't done soon, Penultimate Electronics will be forced to abandon this very lucrative business line and fire all of the experts.

The desired solution from the functional perspective is represented at the following marketecture diagram.

![Marketecture](images/marketecture.jpg)

**Business Drivers**

What business drivers can we learn from the situation:

 * Poorly performing ticket tracking system.
 * The whole business line can be abandoned.
 * Many people are at risk of losing their jobs.

**Business Goals**

The company establishes the following business goal to help the situation:

 * Develop a new ticket tracking system that will satisfy the required qualities.

The company suffers from a poorly performing customer support system that can wind up the business line. They want to develop a new robust and highly performing system that will allow them to stay in business and enable future growth.

## System Requirements

### Stakeholders

This section describes key stakeholders of the system and their architectural concerns.

* **SH-1**: **Administrator** (security)
    - security is the second name for an administrator; these people deal with user accounts and the billing system.

* **SH-2**: **Customer** (availability, performance, scalability, robustness)
    - customers want the system they're accessing is available anytime they want to use it, and that it responses quickly to their actions;
    - they also don't want their tickets to be precessed and never lost.

* **SH-3**: **Expert** (availability, performance)
    - whenever an expert works on the field, the success of the problem resolution may depend on can they access the knowledge base the ticket system;
    - a poorly performing system may have a high impact on the problem resolution and wastes customer's and expert's time.

* **SH-4**: **Manager** (reportability)
    - these people need to pay careful attention on what's going on: are there unsatisfied customer, are there tickets that won't have an expert assigned too long, are there any billing issues, etc.

* **SH-5**: **Helpdesk** (availability, performance)
    - this is the first line of support for the customers, so they need to have access to the troubleshooting articles and customer tickets when customer are calling for that;
    - they provide direct person-to-person phone support, so their answers will have to be found as quickly as possible.

* **SH-6**: **Development team** (extensibility)
    - these guys are having hard time deploying changes to production because of high coupling and poor modularity of the current system.

### Functional Requirements

* **UC-1**: **User maintenance**:
    - administrator maintains internal user accounts (SH-1);
    - administrator maintains expert skillset, location, and availability (SH-1);

* **UC-2**: **Customer registration**:
    - customers register their profile, credit card and support plan (SH-2);

* **UC-3**: **Ticket workflow**:
    - customers submit tickets via web or by phone call (SH-2, SH-5);
    - experts use mobile app to read ticket and change ticket status (SH-3);
    - experts can search knowledge base via mobile app (SH-3);

* **UC-4**: **Survey submission**:
    - customers fill out and submit satisfaction surveys (SH-2);

* **UC-5**: **Knowledge base maintenance**:
    - experts update knowledge base (SH-3);

* **UC-6**: **Reporting**:
    - managers track ticket operations (SH-4);
    - managers generate reports: financial, expert performance, ticketing (SH-4);

* **UC-7**: **Billing**:
    - customers are billed automatically on monthly basis (SH-2);
    - customers can view their billing history and statements (SH-2);
    - administrator manages billing processing for customers (SH-1);

* **UC-8**: **Notification**:
    - customers receive SMS or email about expert assignment (SH-2);
    - customers receive email with a link to survey web form (SH-2);
    - experts receive SMS about ticket assignment (SH-3);

* **UC-9**: **Ticket search**:
    - helpdesk users need access to the ticket base to clarify ticket status (SH-5);

### Architecture Characteristics Requirements

* **QA-1**: **scalability** (UC-3)
    - country scale geography (USA?);
    - number of customers - millions;
    - number of tickets per customer <= 100 (let's assume something crazy);

* **QA-2**: **availability** (UC-2, UC-3, UC-4)
    - customer-facing services and KB must be highly available because outages will make a negative impact on business;
    - 99.9% seems reasonable here;

* **QA-3**: **performance** (UC-2, UC-3, UC-6)
    - response time < 2s for page load;
    - knowledge search time several seconds;
    - reports generation should not take an excessive amount of time;

* **QA-4**: **robustness** (UC-3)
    - lost tickets or wrong experts may lead to the business closure;

* **QA-5**: **security** (UC-2, UC-7)
    - customer personal information and credit cards should be stored in secure and comply to PCI requirements;

* **QA-6**: **extensibility** (all use cases, SH-6)
    - one of the critical concerns regarding the current system is that whenever a change is made, it takes too long and something else usually breaks. This makes us think about improved modularity of the new system.

### Constraints
* **CON-1**: Integration? Cloud/on-prem? Technology stak?

### Assumptions
* **ASM-1**: The mobile app is a part of the system and can be changed.
* **ASM-2**: Helpdesk assistants (aka Call Center) need access to the ticket sub-system and some customer information (contacts, maybe support plan). These are also users of the system although are not listed in the "Main Four Users" of the original requirements.
* **ASM-3**: Call tracking system is out of scope of Sysops Squad system.
* **ASM-4**: The company stores customer credit card information locally and does not interoperate with a 3-party authority (like authorize.net in the US).

## Baseline Architecture
This section describes the architecture of the current ticket system.

Please note that all views are documented in [C4 model](https://c4model.com) style, although only System Context, Container and dynamic views are presented. The most diagrams use informal notation style. All diagrams are supplied with a key explaining meaning of each shape on the diagram.

The current ticket system demonstrates very poor characteristics of availability, maintainability, deployability and performance. Our goal is to design a new system that solves aforementioned problems.

The following diagram depicts the containers diagram of the current ticket system:

![Baseline Architecture](images/baseline.jpg "Baseline Architecture")

## Target Architecture
This section describes the target software architecture.

Please note that all views are documented in [C4 model](https://c4model.com) style, although only System Context, Container and dynamic views are presented. The most diagrams use informal notation style. All diagrams are supplied with a key explaining meaning of each shape on the diagram.

### Use Case Model

The following diagram shows mapping of architecture characteristics requirements on the key use cases based on discovered [requirements](Requirements.md):

![Use Case Model](images/use-case-model.jpg "Use Case Model")


### System Context

The system context diagram below depicted key users of the system and its external dependencies:

![System Context](images/system-context.jpg "System Context")

### Containers

The containers diagram that follows shows the high-level shape of the software architecture and how responsibilities are distributed across containers. It also shows the major technology choices and how the containers communicate with one another.

The architecture is build around four main domains that have been discovered during the problem analysis:
 - customer-facing services, such as ticket submission, customer profiles, survey submission etc;
 - expert services, such as ticket acceptance and knowledge base search;
 - administration services, such as reporting, survey analysis, ticket tracking etc;
 - billing service, which require high attention to security.

The architectural style used here as the bases is Service-based architecture (see [ADR-1](ADR/ADR-1-service-based.md) for details).

![Containers](images/containers.jpg "Containers")

### Process Views

This section explains some key use cases to demonstrate how corresponding workflows pass through containers.

#### UC-2: Customer registration
The following sequence diagram highlights some key requests that the customer performs during registration in the system.
One worth paying attention is registration of a credit card. In the customer database we store only some minimal credit card data to let the customer possibility identify which card do they have already registered. All the details of the credit card are encrypted and securely passed to the billing system (see [ADR-4](ADR/ADR-4-extract-billing-quanta.md)).

![UC-2: Customer registration](images/customer-registration.jpg "Customer Registration")

#### UC-3: Ticket submission
The following diagram illustrates the process of a ticket registration by the customer.

![UC-3: Ticket submission](images/ticket-submission.jpg "Ticket Submission")

Important thing to note is that the requests succeeds after the ticket is saved in the customer database and the corresponding event is fired for the ticket processing area. This way the customer will be able to see the new ticket immediately after the page refresh and will not have to wait on any further actions on the ticket.

#### UC-3: Ticket assignment
The diagram below explains how the system processes a new ticket and assigns it an expert.

![UC-3: Ticket Created](images/ticket-assignment.jpg)

Since Ticket Process is a job that runs periodically, tickets that cannot be assigned at the given moment will never be lost, they we bill processed next time the job will run.

Also, notice that an assignment is a separate entity. This way we can store a history of assignments.

#### UC-3: Ticket acceptance
This diagram continues the ticket workflow and shows how the Ticket Assigned event is processed by the Sysops Expert user.

![UC-3: Ticket Assigned](images/ticket-acceptance.jpg)

The experts operation succeeds as soon as the ticket status is saved in the database. And in case of acceptance the corresponding even is fired to the customer area.

#### UC-3: Ticket in-progress
This diagram demonstrates how the customer is notified when the Sysops Expert accepted the ticket.

![UC-3: Ticket In-Progress](images/ticket-inprogress.jpg)

Important to notice that the ticket is saved in the customer database prior to the notification event so that the customer will see the actual ticket status upon the notification receive.

#### UC-3: Ticket completion
This diagram explains the process when the Sysops Expert solved the problem and marks the ticket as completed.

![UC-3: Ticket Completed](images/ticket-completion.jpg)

#### UC-3: Ticket Resolved
This diagram illustrates how the customer receives a notification about the ticket resolution and link to the survey form.

![UC-3: Ticket Resolved](images/ticket-resolved.jpg)

First, the ticket status has to be updated in the customer database, so that upon receiving any notifications the customer will see the actual ticket status on the Customer Portal.

#### UC-4: Survey Submission

And finally the last step in the ticket resolution flow is survey submission by the customer.

![UC-4: Survey Submission](images/survey-submission.jpg)

From the customer perspective this is a fire-and-forget even so the operation succeeds as soon as the "Submit" button is clicked.

Analytics API can perform some preliminary processing of the survey if necessary or simply store it in the database for the reporting.

#### UC-7: Monthly billing
The diagram illustrates the monthly billing workflow.

![UC-7: Monthly billing](images/billing-sequence.jpg "Monthly Billing")

### Deployment

The deployment diagram illustrates how the system containers are mapped to the infrastructure:

![Deployment](images/deployment.jpg "Deployment")
Note the colors have not special meaning, they are just to distinguish thing from one another.

The deployment strategy here is cloud-agnostic, assuming you can use any cloud provider of your choice or stay totally on-prem. An exception is the billing stuff, which is recommended to remain on-prem anyway for security considerations.

## Transition Architecture

The solution proposed in the [Target Architecture](#containers) section is the final ambition that solves most of the problems and risks, but can require significant development efforts because of the database split required. Thus we can divide the whole work into two phases:

1. Solve critical problems an stick with a monolithic database until it becomes a bottleneck.
2. Migrate further to the target architecture to deal with all remaining risks.

Here is the transitional architecture proposal that solves critical problems but leaves some risks (analysis follows).
Note that we still leverage asynchronous messaging for ticket processing here to enable independent scalability and availability for different parts of the system. In this case, messages can contain mach less information because all the details can be taken by the receiver from the database.

![Transition Architecture](images/transition.jpg)

Since we have a single monolithic database we can save some efforts on additional messaging and replication.

### Risk Analysis
These are the possible high risks of the transition architecture.

#### Performance
Because this is a monolithic database it can become a performance bottleneck. The same concern regarding the single API Gateway - if not scaled properly may also become a bottleneck.

#### Availability
A single API Gateway may introduce a single point of failure for the whole system (see [ADR-12](ADR/ADR-12-gateways.md)).

#### Security
There is a risk that admin staff can get access to the customer credit card data. We certainly want to prevent that by extracting billing into a separate architectural quantum (see [ADR-4](ADR/ADR-4-extract-billing-quanta.md)) and isolating it in a separate network zone with strict access permissions.

The same concern is regarding the customer services - we don't want to allow an attacker to get access to the reset of the system. A significant security improvement would be to migrate customer services and data in a separate quantum in isolate it in a separate network zone (see [ADR-5](ADR/ADR-5-extract-customer-quantum.md)).

#### Other
Additional concerns regarding the API Gateway:

* Adds coupling between the gateway and the internal service.
* If developed by a single development team, may become a development bottleneck.

## Architecture Decision Records

> *Why is more important than how.  
Second Law of Software Architecture*

 - [ADR-1](ADR/ADR-1-service-based.md) Use Service-based architectural style as the basic style.
 - [ADR-2](ADR/ADR-2-event-driven-broker.md) Use message queues with guaranteed delivery for ticket workflow.
 - [ADR-3](ADR/ADR-3-search-expert.md) Extract ticket assignment into a separate batch processing job.
 - [ADR-4](ADR/ADR-4-extract-billing-quanta.md) Extract billing architectural quantum.
 - [ADR-5](ADR/ADR-5-extract-customer-quantum.md) Extract customer architectural quantum.
 - [ADR-6](ADR/ADR-6-separate-customer-db.md) Use separate customer database.
 - [ADR-7](ADR/ADR-7-separate-reporting-db.md) Separate analytics and reporting database.
 - [ADR-9](ADR/ADR-9-notification-service.md) Extract notification service.
 - [ADR-10](ADR/ADR-10-modular-services.md) Use sub-domain partitioning for service design.
 - [ADR-11](ADR/ADR-11-extract-payment-job.md) Extract payment processing into a separate component (Payment Job).
 - [ADR-12](ADR/ADR-12-gateways.md) Offload operational concerns into API Gateways.

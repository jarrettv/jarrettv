---
layout: "../../layouts/BlogPost.astro"
title: "Implementation Plan for Acme Insurance"
description: "Fictional implementation plan for client project"
pubDate: "November 17 2022"
updatedDate: "November 18 2022"
heroImage: "/placeholder-hero.jpg"
---
## Overview
<div style="font-weight:bold;line-height:0.7rem">Build a modern webapp for Order Exports and Employee Profiles</div>

This plan covers cloud first development of a modular solution targeting AWS. The following features are in scope for the first iteration of this project.

<div style="border:1px solid #222a;border-radius: 6px;">

| Use Case          | In Scope | Description
| ------------------| ---------| -----------------------------------------------
| Order Export      | Yes      | Display and generate PDF by extracting OMS data
| My Profile        | Yes      | Employee directory and profile customization

</div>

The following diagram shows a high level design of the solution. This plan will further detail each part shown below:

<img src="/overview.svg" style="max-height:420px"/>

## Infrastructure

This plan targets AWS for new infrastructure to support our solution. 

<div style="border:1px solid #222a;border-radius: 6px;">

| Infrastructure    | Description                                                 | Alternates
| ------------------| ------------------------------------------------------------| ------------
| AWS Fargate       | Serverless compute for containers with automatic scaling    | EKS
| AWS ECS           | Elastic container service to support service modules        | EKS
| AWS CloudWatch    | Observability, debugging, monitoring                        | DataDog
| AWS Aurora        | PostgreSQL relational database for storing records          | DynamoDB
| AWS DynamoDB      | NoSQL database for fast record lookup times                 | Aurora
| AWS S3            | File storage for temporarily storing PDF files              | 

</div>

## Security

This plan recommends a combination of role-based access security (RBAC) and attribute-based access security (ABAC). User's will login through Open ID. The IMS will include user claims upon successful authentication. The claims will be utilized for authorization.

This plan recommends the following permissions:

| Permission             | RBAC   | ABAC    | Description
| -----------------------| -------|---------|------------------------------------------
| `emp-profile-view`     | ✅     |        | View and lookup employee profiles
| `emp-profile-my-edit`  | ✅     | ✅     | Edit my own profile
| `emp-profile-any-edit` | ✅     |        | Edit any employee's profile (HR only)
| `order-export`         | ✅     |        | View and export an order

## Modules

This plan recommends building multiple business and infrastructure modules to support our use cases. The goal
of this architecture is to provide the ground-work for supporting additional features in future interations.

<div style="border: 1px solid #f2db00; border-radius:6px; background:#fffde5;padding:1rem;display:flex;align-content:top">
<div>
<svg xmlns="http://www.w3.org/2000/svg" style="width:1.4rem;color:#0092ff;margin:0.3rem 0.8rem" preserveAspectRatio="xMidYMid meet" viewBox="0 0 256 256"><path fill="currentColor" d="M224 128a96 96 0 1 1-96-96a96 96 0 0 1 96 96Z" opacity=".2"/><path fill="currentColor" d="M128 24a104 104 0 1 0 104 104A104.1 104.1 0 0 0 128 24Zm0 192a88 88 0 1 1 88-88a88.1 88.1 0 0 1-88 88Zm16-40a8 8 0 0 1-8 8h-8a8 8 0 0 1-8-8v-48a8 8 0 0 1 0-16h8a8 8 0 0 1 8 8v48a8 8 0 0 1 8 8Zm-30-92a12 12 0 1 1 12 12a12 12 0 0 1-12-12Z"/></svg></div>
  For simplicity, the team may consider a single service that combines the Web App, Order Service, and
  Profile Service.
</div>

### Web App

The web app is a front-end ASP.NET MVC infrastructure service utilizing server side rendering (SSR). This plan recommends adopting a common CSS framework to keep styling consistent with existing systems and client designs. It may be useful to consider a modern SPA framework if we anticipate complex UX use cases.

* Auth using cookies and OpenID with existing identity management system (IMS)
* OpenTelemetry instrumentation with a CloudWatch adapter
* Call Profile Service and Order Service
* Utilize Vertical Slice project layout

### Profile Service

The profile service is a business module utilizing .NET microservice pattern and minimal APIs. This plan recommends the service store custom  profile information in AuroraDB for the following reasons:

* Well known profile structure
* Ad-hoc queries find other employees

For simplicity, this service will retrieve in real-time the "read-only" data from the IMS as needed. If necessary, this plan recommends caching to reduce latency.

<div style="border: 1px solid #f2db00; border-radius:6px; background:#fffde5;padding:1rem;display:flex;align-content:top">
<div>
<svg xmlns="http://www.w3.org/2000/svg" style="width:1.4rem;color:#0092ff;margin:0 0.8rem" preserveAspectRatio="xMidYMid meet" viewBox="0 0 24 24"><path fill="currentColor" d="M6 6.39v4.7c0 4 2.55 7.7 6 8.83c3.45-1.13 6-4.82 6-8.83v-4.7l-6-2.25l-6 2.25z" opacity=".3"/><path fill="currentColor" d="M12 2L4 5v6.09c0 5.05 3.41 9.76 8 10.91c4.59-1.15 8-5.86 8-10.91V5l-8-3zm6 9.09c0 4-2.55 7.7-6 8.83c-3.45-1.13-6-4.82-6-8.83v-4.7l6-2.25l6 2.25v4.7z"/></svg></div>
This service will protect private employee profile information by checking claims on the JWT token.
</div>

This service will contain the following endpoints as defined by Open API version 3 specification:

| Method | Endpoint         | Request                             | Response
| -------| -----------------| ------------------------------------| ------------
| `GET`  | /employees       | Search criteria* paging querystring | JSON paged list of profiles
| `GET`  | /employees/{id}  | Load full employee profile          | JSON employee profile
| `POST` | /employees/{id}  | JSON employee profile               | JSON employee profile

This plan also recommends the following design choices:

* Auth to utilize JWT token with user claims
* OpenTelemetry instrumentation with a CloudWatch adapter

### Order Service

The order service is a business module utilizing .NET microservice pattern and minimal APIs. This plan recommends the service store all order events in a record-per-order document in DynamoDB for the follwing reasons:

* Wide range of order event schemas
* Low-latency lookups by known partition and sort key

The `Orders` table in DynamoDB will need to be populated from the existing OMS system via an ETL job. Although this job is only meant to run once, we may need to run it multiple times as part of reconciliation process. See `Sync Job` for more details.

The service will accept streaming order events from the OMS and update order records in the `Orders` table.

The following sequence diagram shows the flow of communication for order exports:

<img src="/sequence.svg" style="max-height:620px"/>

This following endpoints will be provided as defined by Open API version 3 specification:

| Method | Endpoint             | Request                              | Response
| -------| ---------------------| -------------------------------------| ------------
| `GET`  | /orders              | Search criteria* paging querystring  | JSON paged list of orders
| `GET`  | /orders/export/{id}  | Order id in route                    | JSON order
| `POST` | /orders/export       | JSON with order id and template id   | JSON PDF download link

This plan also recommends the following design choices:

* Auth to utilize JWT token with user claims
* OpenTelemetry instrumentation with a CloudWatch adapter

### PDF Service

The PDF service is an infrastructure service that utilizes a headless chromium solution for rendering PDFs from HTML. This service will accept a URL or raw HTML for rendering. Any assets required for rendering must be publically available. This plan recommends embedding assets into HTML whenever possible.

Generating PDFs from HTML is a computationally expensive process. This design recommends storing PDFs in S3 for the following reasons:

* Generated PDFs can be large binary files
* S3 supports object expiration

| Method | Endpoint             | Request                              | Response
| -------| ---------------------| -------------------------------------| ------------
| `POST` | /generate            | JSON with HTML or URL to render      | JSON with unique hash ID download URL

### Sync Job

The sync job is responsible for loading all order events into combined order records in the `Orders` table.

More information TBD


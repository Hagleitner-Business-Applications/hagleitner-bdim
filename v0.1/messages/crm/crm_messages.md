# Dynamics 365 CE Messages
This document lists all messages of the event pipeline that need to be part of the BDIM. The interface must handle all these messages to ensure that any relevant event or data change is communicated to the BDIM.
[Event Framework](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/event-framework)

## Datamodel
This section provides additional information about the CRM data model.
TODO: insert relevant part of the crm datamodel here

## Tables and Events
Every table supports events, and events are produced at different stages of the event pipeline.

A common table relevant for the BDIM is the **Account** table. An Account is an entity containing customer-related information such as correspondence details, postal addresses, billing information, and more.

Typical crm events that should be considered for the BDIM are:

 [Create](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.messages.createrequest?view=dataverse-sdk-latest
 [Update](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.messages.updaterequest?view=dataverse-sdk-latest)
 [Delete](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.messages.deleterequest?view=dataverse-sdk-latest)
 [Assign](https://learn.microsoft.com/en-us/dotnet/api/microsoft.crm.sdk.messages.assignrequest?view=dataverse-sdk-latest)
 [Associate](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.messages.associaterequest?view=dataverse-sdk-latest)
 [Dissacoiate](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.messages.disassociaterequest?view=dataverse-sdk-latest). 
 
These messages normally support both a **Pre Operation** and **Post Operation** stage.

Pre Operation events are executed before data is written.
Post Operation events are executed after data has been written.

The BDIM should use Post Operation events to ensure that only successfully committed data is communicated.

### CRM Events vs BDIM Events
CRM Events are not the same as BDIM Events. All CRM Events are mapped to three basic BDIM Events that are: Create, Update, Delete.

CRM Event Create maps to BDIM Event Create
CRM Event Update maps to BDIM Event Update
CRM Event Delete maps to BDIM Event Delete
CRM Event Assign maps to BDIM Event Update
CRM Event Associate maps to BDIM Event Update
CRM Event Dissacociate maps to BDIM Event Update

### customer

Draft/ TODO:
A customer is changed in CRM, and the change must be communicated to the Service Bus.
We distinguish between two categories: events and requests/responses.


### customerPriceGroup
Draft/ TODO:
### employee
Draft/ TODO:
### responsibilityCenter
Draft/ TODO:


# Examples

## Events

### A customer is created in CRM

- CRM sends a message to the Topic **sbt_crm** ([Schema](./events/customer/crm.event.customer.created.schema.json))

### A customer is updated in CRM

- CRM sends a message to the Topic **sbt_crm** ([Schema](./events/customer/crm.event.customer.updated.schema.json))

### A customer is deleted in CRM

- CRM sends a message to the Topic **sbt_crm** ([Schema](./events/customer/crm.event.customer.deleted.schema.json))

> Event operations use the past tense: **created**, **updated**, **deleted**.

---

## Requests / Responses

### A customer has to be synchronized from CRM to ERP

1. CRM sends a request message to the queue **sbq_erp** ([Schema](./requests/customer/crm.request.customer.create.schema.json)).
2. ERP reads the message from **sbq_erp** and processes the request.
3. ERP sends the response message to the queue **sbq_crm** ([Schema](./responses/customer/crm.response.customer.create.schema.json)).
4. The response contains either:
   - the ERP payload (for example, the ERP customer number), or
   - an error message.
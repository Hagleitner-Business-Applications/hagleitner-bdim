# Dynamics 365 CE Messages
This document lists all messages of the event pipeline that need to be part of the BDIM. The interface must handle all these messages to ensure that any relevant event or data change is communicated to the BDIM.
[Event Framework](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/event-framework)


## Service Bus
The Service Bus is the place where applications send messages to communicate events and data changes.

We distinguish between two categories: **events** and **requests/responses**.

If an application needs to communicate an event, it uses its application-owned topic on the Service Bus. For example, **sbt_crm** is the Service Bus topic owned by the CRM application. The CRM application sends events to that topic for interested subscribers.

requests/responses, on the other hand, are used to communicate an action or task to another explicitly addressed application. For example, the action "Create Customer in ERP" is used by CRM (Dynamics 365 CE) to instruct ERP (Dynamics 365 Business Central) to create a customer.

The action is sent to the application-owned queue **sbq_erp**, where the ERP system picks up the action and creates the customer in its database.

The ERP systems sends a response to the request to the application-owned queue **sbq_crm** of the CRM system. The response queue is send as part of the request message. The field is called **replyTo**. In case of an error, the response contains details of the error and additional informations as payload. In case of success, the payload contains most importantly, the ERP UUID and ERP Number.

A request and the corresponding response shares the same correlationid even that they are in differant queues.

### AMQP Properties

Following AMQP Properties have to be used and set by the applications to make the correct processing of messages possible:

| AMQP Property    | Purpose                                                                                                                            |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `message-id`     | Unique identifier of the message.                                                                                                  |
| `correlation-id` | Correlates a response with its request.                                                                                            |
| `reply-to`       | Queue where the receiver should send the response (requests only).                                                                 |
| `to`             | Destination logical endpoint (e.g. `ERP`, `CRM`) if needed.                                                                        |
| `content-type`   | Usually `application/json`.                                                                                                        |
| `creation-time`  | Transport timestamp when the message was created.                                                                                  |

### Business Fields

| Business Field   | Purpose                                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------------------------------- |
| `messageVersion` | Version of the BDIM message contract.                                                                         |
| `sourceSystem`   | Identifies the application that created the business message (e.g. CRM, ERP).                                 |
| `targetSystem`   | Identifies the intended receiving application. (requests/responses only)                                      |
| `objectType`     | Specifies the business object contained in the payload (e.g. `customer`, `employee`, `responsibilityCenter`). |
| `operation`      | Specifies the business operation (e.g. `created`, `updated`, `deleted`, `create`).                            |
| `occurredAt`     | Timestamp when the business event occurred in the source system.  (events only)                               |
| `payload`        | Contains the business object or response data.                                                                |
| `error`          | Contains error information if the request could not be processed. (responses on failure only)                 |



### Service Bus Topics and Queues
This section describes how an application participates in the BDIM.

**Example: Application CRM**

**Topic:** sbt_crm

Used to publish events for interested subscribers. This is a fire-and-forget mechanism. CRM does not expect or process responses from subscribers.

**Queue:** sbq_crm

Used to receive requests/responses for CRM. An application publishing messages to this queue instructs CRM to perform a specific action.

The action must contain control information that allows CRM to publish a success or failure message to the originating application's queue, for example sbq_erp.

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


## Examples

A customer is created in CRM
    CRM sends a message to the Topic **sbt_crm** ([Schema](./events/customer/crm.event.customer.created.schema.json))

A customer is changed in CRM
    CRM sends a message to the Topic **sbt_crm** ([Schema](./events/customer/crm.event.customer.updated.schema.json))

A customer is deleted in CRM
    CRM sends a message to the Topic **sbt_crm** ([Schema](./events/customer/crm.event.customer.deleted.schema.json))

Events are in past tense: created, updated, deleted

A customer has to be synced from CRM to ERP
    CRM sends a message to the queue **sbq_erp** ([Schema](./requests/customer/crm.request.customer.create.schema.json))

    ERP reads from **sbq_erp** and tries to process the request.
    ERP sends the response as message to queue **sbq_crm** ([Schema](./responses/customer/crm.response.customer.create.schema.json))
    The response contains either the ERP payload (ERP Customer Number) or an error message



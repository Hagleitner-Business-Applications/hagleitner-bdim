# Dynamics 365 CE Messages
This document lists all messages of the event pipeline that need to be part of the BDMI. The interface must handle all these messages to ensure that any relevant event or data change is communicated to the BDMI.
[Event Framework](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/event-framework)


## Service Bus
The Service Bus is the place where applications send messages to communicate events and data changes.

We distinguish between two categories: **events** and **actions**.

If an application needs to communicate an event, it uses its application-owned topic on the Service Bus. For example, sbt_crm is the Service Bus topic owned by the CRM application. The CRM application sends events to that topic for interested subscribers.

actions, on the other hand, are used to communicate an action or task to another explicitly addressed application. For example, the action "Create Customer in ERP" is used by CRM (Dynamics 365 CE) to instruct ERP (Dynamics 365 Business Central) to create a customer that already exists in CRM.

The action is sent to the application-owned queue **sbq_bc**, where the ERP system picks up the action and creates the customer in its database.

In case of success, the ERP system sends an erp-response.customer-create-successful message to the application-owned queue **sbq_crm**, containing, most importantly, the ERP UUID and ERP Number.

In case of an error, an erp-response.customer-create-failed message is sent to the application-owned queue.

TODO: Do we need a dedicated error queue?

### Service Bus Topics and Queues
This section describes how an application participates in the BDMI.

**Example: Application CRM**

**Topic:** sbt_crm

Used to publish events for interested subscribers. This is a fire-and-forget mechanism. CRM does not expect or process responses from subscribers.

**Queue:** sbq_crm

Used to receive actions for CRM. An application publishing messages to this queue instructs CRM to perform a specific action.

The action must contain control information that allows CRM to publish a success or failure message to the originating application's queue, for example sbq_erp.

## Datamodel
This section provides additional information about the CRM data model.

## Tables and Events
Every table supports events, and events are produced at different stages of the event pipeline.

A common table relevant for the BDMI is the **Account** table. An Account is an entity containing customer-related information such as correspondence details, postal addresses, delivery addresses, invoice addresses, billing information, and more.

Typical events that should be considered for the BDMI are:

 [Create](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.messages.createrequest?view=dataverse-sdk-latest
 [Update](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.messages.updaterequest?view=dataverse-sdk-latest)
 [Delete](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.messages.deleterequest?view=dataverse-sdk-latest)
 [Assign](https://learn.microsoft.com/en-us/dotnet/api/microsoft.crm.sdk.messages.assignrequest?view=dataverse-sdk-latest)
 [Associate](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.messages.associaterequest?view=dataverse-sdk-latest)
 [Dissacoiate](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.messages.disassociaterequest?view=dataverse-sdk-latest). 
 
These messages normally support both a **Pre Operation** and **Post Operation** stage.

Pre Operation events are executed before data is written.
Post Operation events are executed after data has been written.

The BDMI should use Post Operation events to ensure that only successfully committed data is communicated.

### Account

Draft:

An Account is changed in CRM, and the change must be communicated to the Service Bus.

We distinguish between two categories: events and actions.
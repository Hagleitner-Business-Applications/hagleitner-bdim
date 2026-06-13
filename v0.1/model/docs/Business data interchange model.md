# Hagleitner business data interchange model
The Business Data Inter Model (BDIM) defines the common contract for exchanging business data between participating systems and services. It provides a shared, service-independent model of exchanged entities, their structure, validation rules, and general exchange conventions, so that systems such as CRM, ERP, IoT, and other backend services can communicate consistently over interfaces such as REST APIs and Azure Service Bus messaging.

The Business Data Interchange Model defines a common, service-independent model for the business entities exchanged between participating systems. This common model establishes the shared structure, validation rules, and semantics of the exchanged data. Where required, it is complemented by more explicit message-type-specific contracts, such as create, update, response, or event messages, often represented by dedicated JSON Schemas. This is necessary because a general entity model alone is often not precise enough to express the rules of a specific interaction. Explicit message types make such rules visible at contract level, improve validation, and reduce ambiguity for all participating systems.

> [!NOTE]
> The common entity model is not a message contract; message-specific schemas must define the rules of each interaction.

## Hashing
The owner of an entity provides a `contentHash` that represents the entire entity payload, including nested objects such as address data. The hash is used to detect changes and validate that two parties see the same content.

Guidelines:
- The hash must be computed over the canonical JSON of the entity.
- The `contentHash` field itself is excluded from the hash input.
- All properties present in the payload, including nested objects and arrays, are included.
- A stable, deterministic serialization is required (sorted object keys, consistent number and date formats, no insignificant whitespace).
- Use a SHA-256 as single algorithm across systems

## Strings and nulls
String properties follow one of two nullability conventions, whether defined via a shared base type (e.g. singleLineText, singleLineTextOrNull) or declared inline in the schema (e.g. "type": ["string", "null"]). 

Required properties use a non-nullable type and must always carry a valid, non-empty value. 

Optional properties use a nullable type and can appear in one of three states: omitted (key not present), set to a valid non-empty string, or explicitly set to null. 

Sending null is the canonical way for a client to clear a previously set value; empty strings are never permitted, as all string types enforce a minimum length or pattern constraints that reject them. This ensures a clear semantic distinction between "no value" (null or key absent) and "has a value" (always a valid, non-empty string).
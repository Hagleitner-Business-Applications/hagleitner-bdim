# Customer

> [!NOTE]
        "customerRelations",
> This common entity model is not a message contract; message-specific schemas must define the rules of each interaction.

## Property documentation, validation rules.

### `id`
We use a UUID since the ownership of the customer entity will change in future, and in the transition phase multiple systems will provide customers (namely BC and HsM), therfore a UUID is favored over a simple sequential integer IDs. Additionally BC already has a UUID for each customer (and for each tenant).

### `erpTenantId`
Given to make the life of the ERP easier, i.e. an erp attribute stored in the BDIM.

When customers are created in the CRM the CRM must implement a logic that determines the correct tenat. Could be the 1st level reseller on the reseller relationship path.
 
### `name1`, `name2`, `name3`
Name of the customer, split in up to 3 parts for printing postal addresses.
`name4` is intentionally not implemented as this is just an OEM specific addon in HsM.

### `phoneNumber1`, `phoneNumber2`
Phone numbers of the customer
(Business) Phone number validation should be performed by the owner

### `emailAddress`
We use an object containing multiple email address properties, each of them having a defined semantic.

### `blockedFor`
Enumeration

### ``blockedForReason``
Separate entity

*Warning*: Single language only

### `languageTag`
BCP47 language tag, it is also used to derive the locale or culture with formatting conventions. 

### ``
Only canonical IANA s are allowed

### `taxNumbers`


### `roles[]`
Instead of using individual flags like `isReseller`, roles provide a more abstract and extensible model for associating customers with one or more roles. Roles might also have role-spcific data models, which—if necessary—are modelled as separate entities.

##### Role changes
* Reseller role cannot be removed the customer has linked sub-customers

#### TODOS
* (CRM) Define the actual roles
* (CRM) Discuss using roles as the necessary flag to allow adding descendants for specific customer relationship types

### `customerRelations`

#### TODOS
Customer relations between customers under different resellers need a more deatilled discussion.

#### `reseller`
* Can only point to customers, that have the `reseller` role
* Must not be null except for one customer (the root customer)
* Must not form loops
* Must not be modified after creation

#### `createdAt`
This is the max  date and time when any of the properties of this entity was created

#### `updatedAt`
This is the max  date and time when any of the properties of this entity was last changed



## Properties not yet in the schema
### `updatedBy`
Would be string only, therfore troublesome with GPDR

### `customerCode`
E.g. HHAT. A code uniquely identifying a customer. 

#### TODO
* (CRM) Is this needed
* May also be limited to resellers only, but then would be part of the customer/reseller profile
* May be a uniqueCustomerAlias, theoretically available for all customers
* May be a problem with delete/undelete

* `customerDisplayName` Preferred human-readable customer name for display in user interfaces; may be derived from name1, name2, and name3 if not explicitly maintained.

## General questions and todos

* The `customer` in our model is a very generic entity. Spezializations of a customer might need additional properties. In that case we use role profiles e.g. `customerResellerProfile` to communicate additional properties for that role.
* `geolocationQuality`: Probably not relevant in the CRM, keep it in the FLS if possible (like name4 in hsm)

## Sequence
* Customer creation shall be done without any relations other than the reseller relationship, all other relationships are created using a separate entity `customerRelation`
* Traverse customers recursively with a "preorder depth-first traversal" strategy. 
* On each depth-level sort customers by [Name1, Name2, Name3, id]

# Aditional fields
* homepage: URI
* currencyCode: ISO 4217, default = EUR
* shippingInstruction: Volllieferung Teillieferung (noch unklar) enum
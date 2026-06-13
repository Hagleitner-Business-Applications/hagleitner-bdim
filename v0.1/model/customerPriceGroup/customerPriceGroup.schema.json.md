# customerPriceGroup.schema.json

#### `id`
ID of the customer price group

#### `code`
Hash of the customer content for change detection.

##### Todos:
* Code seems to be in German (in BC), is this intended to stay german or is it also a multilanguage text?
* Is the code unique within a tenant, or a across tenants?

#### `description`
##### TODOs:
* Is it a good idea to call this description, would description not imply some longer, probably multiline text, or is `name`the better wording here

#### `erpTenantIds`
BC will restructure data in a way that customerPriceGroups are global entities with reference to tenatns.
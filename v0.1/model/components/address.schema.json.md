# Address

## Properties

### `address1`, `address2`

### `postalCode`

#### Decisions
Rationale:
* We use postal code instead of zipCode since zipCode is US-specific, most international schemas (OASIS, schema.ort etc.) use postal code
* The postal code is not required since there are countries that have no postal code system.

#### Todo
* is this a reference or a string

### `city`
Rationale: maxLength 30 is historical for Hagleitner, we keep this restriction

#### Todo
* is this a reference or a string

## `stateorProvince`
Many countries require a state or province. 

### `countryCode`

### `Latitude`, `Longitude`
* Structure: probably a separate Coordinate object.
* (Business) Who is owner of the coordinates FLS/ERP or CRM.
* (Business) What about elvation? Could be interesting for Dosing systems.
Rationale:
* It seems adviseable to use lat/long because: precision (no prjection needed), commonly used (at least chatGpt says so), CRM seems to use it (at least this was their suggestion).
* FLS is currently _the_ souce for geo-location FLS seems to use a projected coordinate system (but not sure which)
* On-the fly conversion when exchanging data (project the FLS coordinates to a lat/long pair - might be trorublesome due to rounding)
* CRM creates coordinates from the given address


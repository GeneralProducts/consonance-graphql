---
description: Select basic identifying details for an organisation.
---

# Select organisation details

## Explanation

Organisation IDs are retrieved for a single organisation along with relevant identifying information.

```
query GetOrgDetails($organisationId: Int!) {
  organisation(id: $organisationId) {
    id
    corporateName
    dunsNumber
    globalLocationNumber
    inhouseCustomerId
    inhouseSupplierId
    organisationCountry
    isni
    ringgoldId
    standardAddressNumber
    vatNumber
  }
}

```

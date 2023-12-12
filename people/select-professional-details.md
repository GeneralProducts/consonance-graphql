---
description: Select professional details for a person.
---

# Select professional details

## Explanation

Person IDs are retrieved for a single contact along with their relevant professional information.

```
query GetProfessionalDetails($personId: Int!) {
  person(id: $personId) {
    id
    professionalAffiliations {
      affiliation
    }
    qualificationsAndHonoursAfterNames
    roleFulfillments {
      role
      organization
    }
  }
}
```

Define your person contact ID variable:

```
{
  "personId": YOUR_PERSON_ID_HERE
}
```

---
description: Select person details and associated works.
---

# Select person details and contributing works

## Explanation

Person IDs are retrieved for a single contact along with their relevant details and any relevant associated works.

```
query GetPersonProfile($personId: Int!) {
  person(id: $personId) {
    id
    name
    biographicalNote
    contributions {
      workTitle
      role
    }
    notes {
      content
    }
    addresses {
      street
      city
      country
    }
    gender
    genderClarification
  }
}
```

Define your person contact ID variable:

```
{
  "personId": YOUR_PERSON_ID_HERE
}
```

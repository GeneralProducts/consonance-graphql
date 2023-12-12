---
description: Select basic identifying contact details for a person.
---

# Select basic contact details

## Explanation

Person IDs are retrieved for a single contact along with their relevant identifying information.

```
query GetPersonDetails($personId: Int!) {
  person(id: $personId) {
    id
    name
    biographicalNote
    gender
    titlesBeforeName
    keyNames
    namesAfterKeyNames
    personNameInverted
  }
}

```

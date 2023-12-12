---
description: Select basic identifying details for a person.
---

# Select basic person details

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

Define your person contact ID variable:

```
{
  "personId": YOUR_PERSON_ID_HERE
}
```

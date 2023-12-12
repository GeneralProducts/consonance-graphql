---
description: Select basic contact details for a person.
---

# Select person contact details

## Explanation

Person IDs are retrieved for a single contact along with their relevant contact information.

```
query GetPersonContactInfo($personId: Int!) {
  person(id: $personId) {
    id
    emails {
      email
    }
    phones {
      number
    }
    webLinks {
      url
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

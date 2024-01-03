---
description: Select basic identifying details for a person.
---

# Select basic person details

## Explanation

Relevant identifying information is retrieved for a single contact, filtered by their name.

```
query GetPersonDetails {
  contacts(contactSearch: {personNameCont: "Cate"}) {
    id
    name
    keyNames
    gender
    addresses {
      addressLine1
      addressLine2
      addressLine3
      building
      careOf
      country {
        description
        value
      }
      departmentName
      organisationName
      postOfficeBox
      postcode
      subBuilding
    }
    emails {
      email
    }
    phones {
      phoneNumber
      phoneType
    }
    isni
    notes {
      noteText
    }
  }
}
```


---
description: Select basic contact details for a person.
---

# Select person contact details

## Explanation

Relevant contact information is retrieived for a person, filtered by name.

```
query GetPersonContactInfo {
  contacts(contactSearch: {personNameCont: "Cate"}) {
    id
    name
    gender
    keyNames
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
  }
}
```


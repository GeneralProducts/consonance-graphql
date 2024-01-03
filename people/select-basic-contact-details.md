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
    isni
    notes {
      noteText
    }
  }
}
```


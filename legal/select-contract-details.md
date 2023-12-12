---
description: Select basic contract details.
---

# Select contract details

## Explanation

Contract IDs are retrieved for a single contract along with relevant information including works and dates.

```
query GetContractDetails($contractId: Int!) {
  contract(id: $contractId) {
    id
    contractName
    description
    client {
      name
    }
    work {
      title
    }
    signedDate
    term
    terminatedDate
    terminationReason
    deliveryDates {
      date
      type
    }
    gratisCopies {
      quantity
      reason
    }
  }
}
```

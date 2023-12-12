---
description: Select delivery dates and associated rights included in contract.
---

# Select contract dates and rights

## Explanation

Contract IDs are retrieved for a single contract along with delivery dates and licensed rights details.

```
query GetContractDatesAndRights($contractId: Int!) {
  contract(id: $contractId) {
    id
    msDeliveryDate
    revisedMsDeliveryDate
    actualMsDeliveryDate
    msProofsDate
    pubBeforeDate
    licensedRights {
      right
      territories
    }
  }
}

```

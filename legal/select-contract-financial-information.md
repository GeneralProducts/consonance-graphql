---
description: Select financial information included in contract details.
---

# Select contract financial information

## Explanation

Contract IDs are retrieved for a single contract along with financial information.

```
query GetContractFinancialDetails($contractId: Int!) {
  contract(contractSearch: {idEq: $contractId}) {
    id
    flatFeeValue
    guaranteedRoyaltyValue
    authorDiscount
    remainderRate
    remainderRateOverCost
  }
}
```

Define your contract ID variable:

```
{
  "contractId": YOUR_CONTRACT_ID_HERE
}
```

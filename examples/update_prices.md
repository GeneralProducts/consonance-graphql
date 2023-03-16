# Description

Update prices on a product

## Explanation

This mutation updates all the prices based on a Work ID and a pricing scope.

```gql
mutation UpdateWorkPrices($id: ID!, $pricingScope: String!, $productInputs: [UpdateProductInput!]!) {
  updateWork(id: $id, productInputs: $productInputs, scope: $pricingScope) {
    work {
      id
      title
      prices {
        id
        priceAmount
      }
    }
    errors {
      field
      message
    }
  }
}
```

Then add the variables to the "Query Variables" section.

```gql
{
  "id": "1",
  "pricingScope": "01-05-02-05",
  "productInputs": [
    {
      "id": "1",
      "price": 9.99
    },
    {
      "id": "2",
      "price": 12.99
    }
  ]
}
```

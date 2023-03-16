# Description

Update the publication status of a product

## Explanation

This mutation updates the publication status of a product, specified by its product ID. The results are then displayed as part of the query.

```gql
mutation UpdateProductOnixPublishingStatusCode($input: UpdateProductInput!) {
  updateProduct(input: $input) {
    product {
      id
      isbn
      title
      onixPublishingStatusCode
    }
    errors {
      field
      messages
    }
  }
}

```

Then add the variables to the "Query Variables" section.

```gql
{
  "input": {
    "id": 123,
    "onixPublishingStatusCode": "04"
  }
}

```

# Description

Update the publication date of a product

## Explanation

This mutation updates the publication date of a product, specified by its product ID. The results are then displayed as part of the query.

```gql
mutation UpdateProductPublicationDate($input: UpdateProductInput!) {
  updateProduct(input: $input) {
    product {
      id
      isbn
      title
      publicationDate
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
    "publicationDate": "2023-04-01"
  }
}
```

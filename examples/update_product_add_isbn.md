# Description

Update a product to assign an ISBN

## Explanation

This mutation will update the product's isbn field and return the updated product record and any validation errors that occurred during the update process.

```gql
mutation UpdateProduct($id: ID!, $isbn: String!) {
  updateProduct(input: {
    id: $id,
    isbn: $isbn
  }) {
    product {
      id
      isbn
    }
    errors {
      attribute
      message
    }
  }
}
```

Then add the two variables to the "Query Variables" section.

{
  "id": PRODUCT_ID_HERE,
  "isbn": "NEW_ISBN_HERE"
}


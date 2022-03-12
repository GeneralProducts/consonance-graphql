```gql
{
  works(productSearch: { publicationDateGt: "2000-01-01" }) {
    id
    title
    products {
      publicationDate
    }
  }
}
```

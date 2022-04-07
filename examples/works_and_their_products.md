# Description

Select works and their products

## Explanation

Works are retrieved along with their products. This is structured so that the products are naturally grouped under their works, so less data needs to be processed.

```gql
{
  works {
    id
    title
    lcSubjectHeading
    products {
      id
      publicationDate
      isbn {
        isbn13
      }
    }
  }
}
```

## Description

Select works and their products

## Explanation

Works are retrieved along with their products. Compare with TODO, which gives the same information but is structured
so that the products are naturally grouped under their works. This means that less data needs to be processed.

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

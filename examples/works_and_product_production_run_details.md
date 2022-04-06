# Description

Select works and production run information for their products

## Explanation

All products are retrieved for a single work along with their relevant production run information.

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

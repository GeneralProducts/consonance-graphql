## Description

Select products and their works

## Explanation

Products are retrieved along with their work and the Library of Congress subject heading, which is held against the work only

```gql
{
  products {
    id
    publicationDate
    isbn {
      isbn13
    }
    work {
      id
      title
      lcSubjectHeading
    }
  }
}
```

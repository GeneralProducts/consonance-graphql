# Description

Select products and their works

## Explanation

Products are retrieved along with their work. In this example the Library of Congress subject heading, which is held against the work only, is also retrieved.

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

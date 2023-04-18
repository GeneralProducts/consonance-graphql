# Description

Select products and some of their attributes

## Explanation

All products are retrieved, with their ID, publication date, and ISBN.

The ISBN is a composite type which can return different versions of the value, and here we have selected the ISBN-13 and ISBN-10, possibly for indexing purposes on a website, and the ISBN-13 formatted with dashes, possibly for display purposes.

```gql
{
  products {
    id
    publicationDate
    isbn {
      isbn13
      isbn10
      isbnWithDashes
    }
  }
}
```

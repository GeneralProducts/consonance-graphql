# Description

Select products with particular product forms, and their work

## Explanation

Particular products are retrieved along with their work, filtered by work ID. In this example only print products are returned.

```gql
{
  products(productSearch: { onix21ProductFormCodeIn: [BB, BC] }) {
    id
    onix21ProductFormCode
    onix21ProductForm {
      code
      description
    }
    work {
      id
    }
  }
}
```

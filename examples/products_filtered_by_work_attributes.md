# Description

Filter products by their work attributes

## Explanation

Products are retrieved and filtered by their work attributes. In this example only print products are retrieved with a title that contains the search criteria "phys".

```gql
{
  products(
    productSearch: { onix21ProductFormCodeIn: [BB, BC] }
    workSearch: { titleCont: "phys" }
  ) {
    id
    onix21ProductFormCode
    onix21ProductForm {
      code
      description
    }
    work {
      title
    }
  }
}
```

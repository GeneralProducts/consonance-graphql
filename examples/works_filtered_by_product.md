# Description

Select works that have any products published after 2000-01-01.

## Explanation

When retrieving products for many works it is more efficient and intuitive to get a result that lists the works and then the products that belong to them, rather than listing the products and the work for each.

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

## Description

Select works that have any products published after 2000-01-01, and their products that were published after 2000-01-01

## Explanation

When retrieving products for many works it is more efficient and intuitive
to get a result that lists the works and then the products that belong to them,
rather than listing the products and the work for each.
However, when you only want details for a particular set of products,
selecting all works and then just the required products will retrieve a lot
of works without any products at all.
Therefore the product search is also applied to the work, so only works that
have one of the required products are retrieved.

```gql
{
  works(productSearch: { publicationDateGt: "2000-01-01" }) {
    id
    title
    products(productSearch: { publicationDateGt: "2000-01-01" }) {
      publicationDate
    }
  }
}
```

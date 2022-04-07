# Description

Filter products by price and format

## Explanation

Only products in a specified format with a price equal to or less than the specified value are retrieved, with their ID, ISBN, title and publishing status.

In this example, only eBooks priced at 0.99 or lower are retrieved.

This could help with identifying price errors or confirming all relevant products have been included in a price promotion.

```gql
{
 {
    products(
        priceSearch: {amountLteq: 0.99}
        productSearch: {onix21ProductFormCodeIn: [DG]}
    )
    {
    id
        isbn {isbn13}
        onix21ProductFormCode
        onix21ProductForm {
            code
            description
        }
        onixPublishingStatusCode
        onixPublishingStatus {
            code
            description
        }
        work {
            title
        }
  }
}
```

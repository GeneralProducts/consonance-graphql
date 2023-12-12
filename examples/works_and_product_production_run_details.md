# Select works and their product production run details

Select works and production run information for their products

## Explanation

Product IDs are retrieved for a single work along with their relevant production run information.

```
query GetWorkById($workId: Int!) {
  work(workSearch: {idEq: $workId}) {
    id
    inHouseWorkReference
    title
    subtitle
    workType
    productionRuns {
      id
      name
      deliveryDate
      productIds
      quantities {
        id
        quantity
        name
        description
        receivedQuantity
        productId
      }
      bindings {
        id
        widthOfBoard
        pageBindingMethod
        trimmingMethod
        endpapers
        caseSpineStyle
        description
        bindingInstructions
        productIds
        products {
          id
        }
      }
      covers {
        id
        paper
        coverType
        coverColoursOutside
        coverColoursOutsideDescription
        coverNotes
        productIds
        products {
          id
        }
      }
      embellishments {
        id
        name
        description
        productIds
        products {
          id
        }
      }
      finishes {
        id
        name
        description
        productIds
        products {
          id
        }
      }
      interiors {
        id
        paper
        textColour
        textColourDescription
        textPrintInstructions
        fsc
        productIds
        products {
          id
        }
      }
    }
    projectStage
    season
  }
}
```

Define your Work ID variable:

```
{
  "workId": YOUR_ID_HERE
}

```

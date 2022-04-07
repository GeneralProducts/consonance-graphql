# Description

Select works and production run information for their products

## Explanation

All product IDs are retrieved for a single work along with their relevant production run information.

```gql
query WorkType {
  work(workSearch: {idEq: WORKID}) {
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

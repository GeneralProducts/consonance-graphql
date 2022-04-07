# Description

Select works and production run information for their products

## Explanation

<<<<<<< HEAD
All product IDs are retrieved for a single work along with their relevant production run information.

```gql
query WorkType {
  work(workSearch: {idEq: 91804}) {
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


=======
All products are retrieved for a single work along with their relevant production run information.

```gql
{
  works {
    id
    title
    lcSubjectHeading
    products {
      id
      publicationDate
      isbn {
        isbn13
      }
    }
  }
}
>>>>>>> fb0535f69352f5a76ddcab35ba2cb75511cd0fdf
```

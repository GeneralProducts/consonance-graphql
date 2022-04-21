# Description

Generate a report in Google Sheets with customisable columns

## Explanation

Using a combination of GraphQL and Javascript in Google Sheets and Apps Script you can generate a custom report with your choice of column headings.

Create a new sheet in [Google Sheets](https://sheets.google.com/) and navigate to **Extensions > Apps Script** from the main navigation bar to open the Apps Script editor.

Paste the code below into this screen, replacing any existing text.

Replace YOUR_API_KEY in the code with the API key provided by Consonance support.

This script will generate a report of every field currently available in Consonance's GraphQL schema. If you would like to edit or delete any columns to produce a smaller report you can refer to the inline notes for help doing that.

Choose *Run* from the main navigation bar to generate your report. You may be prompted to authorize your script by signing into your Google account and choosing *allow*. This grants permission for the script to run.

Go back to your sheet and this should now be populated with the columns of data you've specified.

```gql
  const main = () => {
    const products = queryResponse().data.products
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName("sheet1");
    sheet.appendRow(header())
    const dataArray = dataToArray(products);
    ss.setNamedRange("product_data", sheet.getRange(2, 1, dataArray.length, dataArray[0].length))
    ss.getRange("product_data").setValues(dataArray)
    sheet.autoResizeColumns(1, dataArray[0].length)
  }
  
  function dataToArray(products) {
    return products.map(productToArray)
  }
  
  function queryResponse() {
    const queryString = `
      query {
        products {
        id
        __typename
        isbn {isbn10 isbn13 isbnWithDashes}
        shortDescription: marketingTexts(variantIn: [SHORT_DESCRIPTION]) {
          externalText
        }
        gbp_consumer_price: prices(priceSearch:{currencyCodeIn: [GBP], onixPriceTypeCodeIn: [_02], onixPriceTypeQualifierCodeIn: [_05]}) {
          amount
        }
        fullTitle
        titleWithoutPrefix
        titlePrefix
        subtitle
        authorshipDescription
        productAuthorshipDescription
        onix21ProductForm{ code description notes isDeprecated}
        onix21ProductFormCode
        publicationDate
        publicationDateString: publicationDateString(directives: "%-d %B %Y")
        publicationYear: publicationDateString(directives: "%Y")
        plannedPublicationDate
        plannedPublicationDateString: plannedPublicationDateString(directives: "%-d %B %Y")
        work {
          id
        }
        onixPublishingStatusCode
        onixPublishingStatus {
          code
          description
          notes
          isDeprecated
        }
          ... on PhysicalBookProduct {
            productHeightMm: productHeight
            productWidthMm: productWidth
            productHeightCm: productHeight(unit: CM)
            productWidthCm: productWidth(unit: CM)
            productHeightIn: productHeight(unit: IN)
            productWidthIn: productWidth(unit: IN)
            approximatePageCount
            productionPageCount
            mainContentPageCount
            frontMatterPageCount
            backMatterPageCount
            totalNumberedPages
            contentPageCount
            totalUnnumberedInsertPageCount
            wordCount
          }
        prices(priceSearch: {currencyCodeIn: [GBP, AUD]}) {
          id
        }
        inHouseFormat{
          name
          code
          productsCount
          inHouseEditions {code}
        }
        inHouseEdition{
          name
          code
          inHouseFormats {code}
        }
        forSaleCountryCodes
        notForSaleCountryCodes
        salesRightsDescription
        isForSaleWorldwide
        webLinks {
          id
          link
          role
          note
        }
      }
    }` 
    
    const payload = JSON.stringify({query: queryString})
    const response = UrlFetchApp.fetch("https://web.consonance.app/graphql", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        authorization: "Bearer YOUR_API_KEY",
      },
      payload: payload,
    });
    const stringResponse = response.getContentText()
    return JSON.parse(stringResponse)
  }
  
  // The function below generates your column headings. Each string of text in quotes refers to
  // a new column and you can name these however is most helpful to you. The order of this list
  // is the order your column headings will appear, so you can rearrange them as you please, but
  // you'll need to remember to also rearrange the equivalent rows in the second function below
  // so that they match up. We have kept the names here matching the fields in Consonance's
  // GraphQL schema for clarity and have included all fields currently available in the schema.

  function header() {
    const headRow = [
      "id",
      "isbn13", 
      "isbn10", 
      "isbn with dashes",
      "fullTitle",
      "titlePrefix" ,
      "titleWithoutPrefix", 
      "subtitle", 
      "gbp_consumer_price", 
      "shortDescription[0] externalText", 
      "authorshipDescription", 
      "productAuthorshipDescription", 
      "onix21ProductForm code", 
      "onix21ProductForm description", 
      "onix21ProductForm notes", 
      "onix21ProductForm isDeprecated", 
      "onix21ProductFormCode", 
      "publicationDate", 
      "publicationDateString", 
      "publicationYear", 
      "plannedPublicationDate", 
      "plannedPublicationDateString", 
      "work id", 
      "onixPublishingStatusCode", 
      "onixPublishingStatus code", 
      "onixPublishingStatus description", 
      "onixPublishingStatus notes", 
      "onixPublishingStatus isDeprecated", 
      "productHeightMm", 
      "productWidthMm", 
      "productHeightCm", 
      "productWidthCm", 
      "approximatePageCount", 
      "productionPageCount", 
      "mainContentPageCount", 
      "frontMatterPageCount", 
      "backMatterPageCount", 
      "totalNumberedPages", 
      "contentPageCount", 
      "totalUnnumberedInsertPageCount", 
      "wordCount", 
      "inHouseFormat name", 
      "inHouseFormat code", 
      "inHouseFormat productsCount", 
      "inHouseFormat inHouseEditions[0] code", 
      "inHouseEdition name", 
      "inHouseEdition code", 
      "inHouseEdition inHouseFormats[0] code", 
      "forSaleCountryCodes", 
      "notForSaleCountryCodes", 
      "salesRightsDescription", 
      "isForSaleWorldwide", 
    ]
  
    return headRow
  }
  
  // The function below populates the rows with your data using the column headings you've
  // defined above. If you have added, rearranged or deleted any headings in the above function
  // you will also need to add, rearrange or delete the equivalent row.push line here or your
  // data won't appear in the correct columns. The field names here must match how they are
  // named in the GraphQL schema in order for them to retrieve the correct data.

  function productToArray(product) {
    const row = []
  
    // Amend these lines using dot notation to get the right data into the right column
    row.push(product.id)
    row.push(product.isbn.isbn13)
    row.push(product.isbn.isbn10)
    row.push(product.isbn.isbnWithDashes)
    row.push(product.fullTitle)
    row.push(product.titlePrefix)
    row.push(product.titleWithoutPrefix)
    row.push(product.subtitle)
    row.push(product.gbp_consumer_price)
    row.push(product.shortDescription[0]?.externalText)
    row.push(product.authorshipDescription)
    row.push(product.productAuthorshipDescription)
    row.push(product.onix21ProductForm.code)
    row.push(product.onix21ProductForm.description)
    row.push(product.onix21ProductForm.notes)
    row.push(product.onix21ProductForm.isDeprecated)
    row.push(product.onix21ProductFormCode)
    row.push(product.publicationDate)
    row.push(product.publicationDateString)
    row.push(product.publicationYear)
    row.push(product.plannedPublicationDate)
    row.push(product.plannedPublicationDateString)
    row.push(product.work.id)
    row.push(product.onixPublishingStatusCode)
    row.push(product.onixPublishingStatus.code)
    row.push(product.onixPublishingStatus.description)
    row.push(product.onixPublishingStatus.notes)
    row.push(product.onixPublishingStatus.isDeprecated)
    row.push(product.productHeightMm)
    row.push(product.productWidthMm)
    row.push(product.productHeightCm)
    row.push(product.productWidthCm)
    row.push(product.approximatePageCount)
    row.push(product.productionPageCount)
    row.push(product.mainContentPageCount)
    row.push(product.frontMatterPageCount)
    row.push(product.backMatterPageCount)
    row.push(product.totalNumberedPages)
    row.push(product.contentPageCount)
    row.push(product.totalUnnumberedInsertPageCount)
    row.push(product.wordCount)
    row.push(product.inHouseFormat?.name)
    row.push(product.inHouseFormat?.code)
    row.push(product.inHouseFormat?.productsCount)
    row.push(product.inHouseFormat?.inHouseEditions[0]?.code)
    row.push(product.inHouseEdition?.name)
    row.push(product.inHouseEdition?.code)
    row.push(product.inHouseEdition?.inHouseFormats[0]?.code)
    row.push(product.forSaleCountryCodes)
    row.push(product.notForSaleCountryCodes)
    row.push(product.salesRightsDescription)
    row.push(product.isForSaleWorldwide)
    return row
  }
  
  
  
  
  
```

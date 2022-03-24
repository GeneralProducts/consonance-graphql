## Description

Populate a dashboard of charts and graphs using Google Data Studio

## Explanation

Using Google Sheets and Google Data Studio you can populate charts and graphs to create a live dashboard of your data.

```gql
function onOpen() {
    SpreadsheetApp.getUi() 
      .createMenu('Consonance')
      .addItem('Show options', 'showSidebar')
      .addToUi();
  }
  
  function showSidebar() {
    var html = HtmlService.createHtmlOutputFromFile('sidebar')
      .setTitle('Consonance');
    SpreadsheetApp.getUi() 
      .showSidebar(html);
  }
  
  const main = () => {
    const products = queryResponse().data.products
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName("sheet1");
    sheet.appendRow(header())
    
    const dataArray = dataToArray(products);
    ss.getRange("product_data").clearContent()
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
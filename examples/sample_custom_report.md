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
              identifyingDoi
              inHouseWorkReference
              title
              titleInOriginalLanguage
              subtitle
              subtitleInOriginalLanguage
              titleStatement
              abbreviatedTitle
              contributorPrettyList
              workType
              yearOfAnnual
              acronym
              alternativeTitles
              authorshipDescription
              projectStage
              publishersUrl
              salesforceUid
              supportingResources {
                url
                caption
              }
              projectStage
              season
              audience {
                description
                onixAudiences {code}
                contentWarnings {code}
                contentWarnings {description}
              }
              ebookLccn
              editionNumber
              editionStatement
              lcChildrensSubjectHeading
              lcFictionGenreHeading
              lcSubjectHeading
              lcSubjectHeadingRegion
              lccn
              seriesMemberships{
                yearOfAnnual
                partNumber
                id
              series{
                name
                titleWithSubtitle
                subtitle
                id
                printIssn
                onlineIssn
              }
            }
contributions {
    id
    sequenceNumber
    productIds
    onixContributorRoleCode
    onixContributorRole {description}
    contributor {
      id
      name
      ... on Person {
          keyNames
          personName
          personNameInverted
        }
        ... on Organisation {
            organisationName
        }
    }
}
            similarProducts {
              authorshipDescription
              id
              isbn{isbn13}
              fullTitle
              inHouseEdition{id}
              contributions {
                contributor{name}
                onixContributorRole{description}
              }
            }
            mainBicCode: subjectCodes(schemesIn: [BIC], mainOnly: true) {
           code
           description
          }
            mainThema: subjectCodes(schemesIn: [THEMA], mainOnly: true) {
              code
              description
              ... on Thema {
                parentCode
              }
            }
            mainBisac: subjectCodes(schemesIn: [BISAC], mainOnly: true) {
              code
              description
            }
            contributions {
              id
              sequenceNumber
              productIds
              onixContributorRoleCode
              onixContributorRole {description}
              contributor {
                id
                name
                ... on Person {
                keyNames
                personName
                personNameInverted
              }
              ... on Organisation {
              organisationName
                }
              }
            }
            prizes {
              id
              prizeName
              onixPrizeAchievementCode
              prizeJury
              prizeYear
              prizeStatement
              onixPrizeAchievement{
                value
                code
                description
              }
            }
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
      "isbn10", 
      "isbn13", 
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
      "Title of Work",
      "Identifying DOI of Work",
      "In House Work Reference",
      "Title of Work In Original Language",
      "Subtitle of Work",
      "Subtitle of Work in Original Language",
      "Title Statement of Work",
      "Abbreviated Title of Work",
       "Work Type",
      "Year of Annual",
      "Acronym",
      "Alternative Titles",
      "Authorship Description",
      "Concatenated Contributors",
      "Contributions ID",
      "Contributions Sequence Number",
      "Contributions Product IDs",
      "Contributions ONIX Contributor Role Code",
      "Contributions ONIX Contributor Role Description",
      "Work Contributor ID",
      "Work Contributor Name",
      "Work Contributor Key Names",
      "Work Contributor Person Name",
      "Work Contributor Person Name Inverted",
      "Work Contributor Organisation Name",
      "Prize ID",
      "Prize Name",
      "ONIX Prize Achievement Code",
      "Prize Jury",
      "Prize Year",
      "Prize Statement",
      "ONIX Prize Achievement Value",
      "ONIX Prize Achievement Description",
      "Work Project Stage",
      "Work Season",
      "Publisher's URL",
      "Salesforce UID",
      "Work Supporting Resources URL",
      "Work Supporting Resources Caption",
      "Work Audience Description",
      "Work ONIX Audiences Code",
      "Work Audience Content Warnings Code",
      "Work Audience Content Warnings Description",
      "Work eBook LCCN",
      "Work Edition Number",
      "Work Edition Statement",
      "Work LC Childrens Subject Heading",
      "Work LC Fiction Genre Heading",
      "Work LC Subject Heading",
      "Work LC Subject Heading Region",
      "Work LCCN",
      "Work Main BIC Code",
      "Work Main BIC Code Description",
      "Work Main THEMA Code",
      "Work Main THEMA Code Description",
      "Work Main THEMA Parent Code",
      "Work Main BISAC Code",
      "Work Main BISAC Description",
      "Work Series Memberships Part Number",
      "Work Series Memberships ID",
      "Work Series Name",
      "Work Series Title with Subtitle",
      "Work Series Subtitle",
      "Work Series ID",
      "Work Series Print ISSN",
      "Work Series Online ISSN",
      "Work Similar Product Authorship Description",
      "Work Similar Product ID",
      "Work Similar Product ISBN",
      "Work Similar Product Full Title",
      "Work Similar Product In House Edition ID",
      "Work Similar Product Contributor Name",
      "Work Similar Product ONIX Contributor Role Description"
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
    row.push(product.isbn.isbn10)
    row.push(product.isbn.isbn13)
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
    row.push(product.work.title)
    row.push(product.work.identifyingDoi)
    row.push(product.work.inHouseWorkReference)
    row.push(product.work.titleInOriginalLanguage)
    row.push(product.work.subtitle)
    row.push(product.work.subtitleInOriginalLanguage)
    row.push(product.work.titleStatement)
    row.push(product.work.abbreviatedTitle)
    row.push(product.work.workType)
    row.push(product.work.yearOfAnnual)
    row.push(product.work.acronym)
    row.push(product.work.alternativeTitles)
    row.push(product.work.authorshipDescription)
    row.push(product.work.contributorPrettyList)
    row.push(product.work.contributions[0]?.id)
    row.push(product.work.contributions[0]?.sequenceNumber)
    row.push(product.work.contributions[0]?.productIds)
    row.push(product.work.contributions[0]?.onixContributorRoleCode)
    row.push(product.work.contributions[0]?.onixContributorRole?.description)
    row.push(product.work.contributions[0]?.contributor?.id)
    row.push(product.work.contributions[0]?.contributor?.name)
    row.push(product.work.contributions[0]?.contributor?.keyNames)
    row.push(product.work.contributions[0]?.contributor?.personName)
    row.push(product.work.contributions[0]?.contributor?.personNameInverted)
    row.push(product.work.contributions[0]?.contributor?.organisationName)
    row.push(product.work.prizes[0]?.id)
    row.push(product.work.prizes[0]?.prizeName)
    row.push(product.work.prizes[0]?.onixPrizeAchievementCode)
    row.push(product.work.prizes[0]?.prizeJury)
    row.push(product.work.prizes[0]?.prizeYear)
    row.push(product.work.prizes[0]?.prizeStatement)
    row.push(product.work.prizes[0]?.onixPrizeAchievement[0]?.value)
    row.push(product.work.prizes[0]?.onixPrizeAchievement[0]?.description)
    row.push(product.work.projectStage)
    row.push(product.work.season)
    row.push(product.work.publishersUrl)
    row.push(product.work.salesforceUid)
    row.push(product.work.supportingResources[0]?.url)
    row.push(product.work.supportingResources[0]?.caption)
    row.push(product.work.audience?.description)
    row.push(product.work.audience?.onixAudiences[0]?.code)
    row.push(product.work.audience?.contentWarnings[0]?.code)
    row.push(product.work.audience?.contentWarnings[0]?.description)
    row.push(product.work.ebookLccn)
    row.push(product.work.editionNumber)
    row.push(product.work.editionStatement)
    row.push(product.work.lcChildrensSubjectHeading)
    row.push(product.work.lcFictionGenreHeading)
    row.push(product.work.lcSubjectHeading)
    row.push(product.work.lcSubjectHeadingRegion)
    row.push(product.work.lccn)
    row.push(product.work.mainBicCode[0]?.code)
    row.push(product.work.mainBicCode[0]?.description)
    row.push(product.work.mainThema[0]?.code)
    row.push(product.work.mainThema[0]?.description)
    row.push(product.work.mainThema[0]?.parentCode)
    row.push(product.work.mainBisac[0]?.code)
    row.push(product.work.mainBisac[0]?.description)
    row.push(product.work.seriesMemberships[0]?.partNumber)
    row.push(product.work.seriesMemberships[0]?.id)
    row.push(product.work.seriesMemberships[0]?.series?.name)
    row.push(product.work.seriesMemberships[0]?.series?.titleWithSubtitle)
    row.push(product.work.seriesMemberships[0]?.series?.subtitle)
    row.push(product.work.seriesMemberships[0]?.series?.id)
    row.push(product.work.seriesMemberships[0]?.series?.printIssn)
    row.push(product.work.seriesMemberships[0]?.series?.onlineIssn)
    row.push(product.work.similarProducts[0]?.authorshipDescription)
    row.push(product.work.similarProducts[0]?.id)
    row.push(product.work.similarProducts[0]?.isbn?.isbn13)
    row.push(product.work.similarProducts[0]?.fullTitle)
    row.push(product.work.similarProducts[0]?.inHouseEdition?.id)
    row.push(product.work.similarProducts[0]?.contributions[0]?.contributor?.name)
    row.push(product.work.similarProducts[0]?.contributions[0]?.onixContributorRole?.description)
    return row
  }
  
```
Here is an example of how your report could look.

![Google Sheets screenshot showing a populated spreadsheet with metadata retrieved from Consonance.](/images/samplecustomreport.png)

# Description

Generate a report in Google Sheets with customisable columns

## Explanation

Using a combination of GraphQL and Javascript in Google Sheets and Apps Script you can generate a custom report with your choice of column headings.

Create a new sheet in [Google Sheets](https://sheets.google.com/) and navigate to **Extensions > Apps Script** from the main navigation bar to open the Apps Script editor.

Paste the code below into this screen, replacing any existing text, and add your API key.

This script will generate a report of every field currently available in Consonance's GraphQL schema. If you would like to edit or delete any columns to produce a smaller report you can refer to the inline notes for help.

Choose *Run* from the main navigation bar to generate your report. You may be prompted to authorize your script by signing into your Google account and choosing *allow*. This grants permission for the script to run.

Go back to your sheet and this should now be populated with the columns of data you've specified.

```gql
// Replace the text YOUR_API_KEY below with the API key provided by Consonance support.
const API_KEY = 'YOUR_API_KEY';

const FIELD_TYPE = Object.freeze({
  HEADERS: 'headers',
  PRODUCT_ROWS: 'product_row',
});

const main = () => {
  const products = queryResponse().data.products;
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName("sheet1");
  sheet.appendRow(getFieldData(FIELD_TYPE.HEADERS));
  
  const dataArray = dataToArray(products);
   ss.getRange("product_data").clearContent();
  ss.setNamedRange("product_data", sheet.getRange(2, 1, dataArray.length, dataArray[0].length));
  ss.getRange("product_data").setValues(dataArray);
  sheet.autoResizeColumns(1, dataArray[0].length);
}

function dataToArray(products) {
  return products.map(productToArray);
}

// This function contains all queries currently available from our GraphQL schema.
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
    }`;
  
  const payload = JSON.stringify({query: queryString});
  const response = UrlFetchApp.fetch("https://web.consonance.app/graphql", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      authorization: `Bearer ${API_KEY}`,
    },
    payload,
  });
  const stringResponse = response.getContentText();
  try {
    return JSON.parse(stringResponse);
  }
  catch (err) {
    return null;
  }
}

function productToArray(product) {
  return getFieldData(FIELD_TYPE.PRODUCT_ROWS, product);
}

// The function below generates both your column headings and the rows with your data. 
// Each row represents a column in your worksheet. The order of this list is the order 
// your columns will appear, so you can rearrange them as you please.

// We have kept the column header names here matching the fields in Consonance's
// GraphQL schema for clarity but you can name the columns anything you like
// by amending the text inside the quotation marks, as long as the column names
// are unique from one another.

// The field name after each column heading is the data that will be used in that column 
// and must match how it is named the GraphQL schema to retrieve the correct data.

// It may be easier to first delete columns you do not need before reordering the remaining
// columns into your desired order. Alternatively you can add a double forward slash (//) to
// the start of a particular row, and when the script runs that line of code will be ignored.

function getFieldData(fieldType, product = null) {
  const fields = new Map([
    ["id", product?.id],
    ["isbn10",  product?.isbn.isbn10],
    ["isbn13",  product?.isbn.isbn13],
    ["isbn with dashes", product?.isbn.isbnWithDashes],
    ["fullTitle", product?.fullTitle],
    ["titlePrefix" , product?.titlePrefix],
    ["titleWithoutPrefix",  product?.titleWithoutPrefix],
    ["subtitle",  product?.subtitle],
    ["gbp_consumer_price",  product?.gbp_consumer_price],
    ["shortDescription[0] externalText",  product?.shortDescription[0]?.externalText],
    ["authorshipDescription",  product?.authorshipDescription],
    ["productAuthorshipDescription",  product?.productAuthorshipDescription],
    ["onix21ProductForm code",  product?.onix21ProductForm.code],
    ["onix21ProductForm description",  product?.onix21ProductForm.description],
    ["onix21ProductForm notes",  product?.onix21ProductForm.notes],
    ["onix21ProductForm isDeprecated",  product?.onix21ProductForm.isDeprecated],
    ["onix21ProductFormCode",  product?.onix21ProductFormCode],
    ["publicationDate",  product?.publicationDate],
    ["publicationDateString",  product?.publicationDateString],
    ["publicationYear",  product?.publicationYear],
    ["plannedPublicationDate",  product?.plannedPublicationDate],
    ["plannedPublicationDateString",  product?.plannedPublicationDateString],
    ["work id",  product?.work.id],
    ["onixPublishingStatusCode",  product?.onixPublishingStatusCode],
    ["onixPublishingStatus code",  product?.onixPublishingStatus.code],
    ["onixPublishingStatus description",  product?.onixPublishingStatus.description],
    ["onixPublishingStatus notes",  product?.onixPublishingStatus.notes],
    ["onixPublishingStatus isDeprecated",  product?.onixPublishingStatus.isDeprecated],
    ["productHeightMm",  product?.productHeightMm],
    ["productWidthMm",  product?.productWidthMm],
    ["productHeightCm",  product?.productHeightCm],
    ["productWidthCm",  product?.productWidthCm],
    ["approximatePageCount",  product?.approximatePageCount],
    ["productionPageCount",  product?.productionPageCount],
    ["mainContentPageCount",  product?.mainContentPageCount],
    ["frontMatterPageCount",  product?.frontMatterPageCount],
    ["backMatterPageCount",  product?.backMatterPageCount],
    ["totalNumberedPages",  product?.totalNumberedPages],
    ["contentPageCount",  product?.contentPageCount],
    ["totalUnnumberedInsertPageCount",  product?.totalUnnumberedInsertPageCount],
    ["wordCount",  product?.wordCount],
    ["inHouseFormat name",  product?.inHouseFormat?.name],
    ["inHouseFormat code",  product?.inHouseFormat?.code],
    ["inHouseFormat productsCount",  product?.inHouseFormat?.productsCount],
    ["inHouseFormat inHouseEditions[0] code",  product?.inHouseFormat?.inHouseEditions[0]?.code],
    ["inHouseEdition name",  product?.inHouseEdition?.name],
    ["inHouseEdition code",  product?.inHouseEdition?.code],
    ["inHouseEdition inHouseFormats[0] code",  product?.inHouseEdition?.inHouseFormats[0]?.code],
    ["forSaleCountryCodes",  product?.forSaleCountryCodes],
    ["notForSaleCountryCodes",  product?.notForSaleCountryCodes],
    ["salesRightsDescription",  product?.salesRightsDescription],
    ["isForSaleWorldwide", product?.isForSaleWorldwide],
    ["Title of Work", product?.work.title],
    ["Identifying DOI of Work", product?.work.identifyingDoi],
    ["In House Work Reference", product?.work.inHouseWorkReference],
    ["Title of Work In Original Language", product?.work.titleInOriginalLanguage],
    ["Subtitle of Work", product?.work.subtitle],
    ["Subtitle of Work in Original Language", product?.work.subtitleInOriginalLanguage],
    ["Title Statement of Work", product?.work.titleStatement],
    ["Abbreviated Title of Work", product?.work.abbreviatedTitle],
    ["Work Type", product?.work.workType],
    ["Year of Annual", product?.work.yearOfAnnual],
    ["Acronym", product?.work.acronym],
    ["Alternative Titles", product?.work.alternativeTitles],
    ["Authorship Description", product?.work.authorshipDescription],
    ["Concatenated Contributors", product?.work.contributorPrettyList],
    ["Contributions ID", product?.work.contributions[0]?.id],
    ["Contributions Sequence Number", product?.work.contributions[0]?.sequenceNumber],
    ["Contributions Product IDs", product?.work.contributions[0]?.productIds],
    ["Contributions ONIX Contributor Role Code", product?.work.contributions[0]?.onixContributorRoleCode],
    ["Contributions ONIX Contributor Role Description", product?.work.contributions[0]?.onixContributorRole?.description],
    ["Work Contributor ID", product?.work.contributions[0]?.contributor?.id],
    ["Work Contributor Name", product?.work.contributions[0]?.contributor?.name],
    ["Work Contributor Key Names", product?.work.contributions[0]?.contributor?.keyNames],
    ["Work Contributor Person Name", product?.work.contributions[0]?.contributor?.personName],
    ["Work Contributor Person Name Inverted", product?.work.contributions[0]?.contributor?.personNameInverted],
    ["Work Contributor Organisation Name", product?.work.contributions[0]?.contributor?.organisationName],
    ["Prize ID", product?.work.prizes[0]?.id],
    ["Prize Name", product?.work.prizes[0]?.prizeName],
    ["ONIX Prize Achievement Code", product?.work.prizes[0]?.onixPrizeAchievementCode],
    ["Prize Jury", product?.work.prizes[0]?.prizeJury],
    ["Prize Year", product?.work.prizes[0]?.prizeYear],
    ["Prize Statement", product?.work.prizes[0]?.prizeStatement],
    ["ONIX Prize Achievement Value", product?.work.prizes[0]?.onixPrizeAchievement[0]?.value],
    ["ONIX Prize Achievement Description", product?.work.prizes[0]?.onixPrizeAchievement[0]?.description],
    ["Work Project Stage", product?.work.projectStage],
    ["Work Season", product?.work.season],
    ["Publisher's URL", product?.work.publishersUrl],
    ["Salesforce UID", product?.work.salesforceUid],
    ["Work Supporting Resources URL", product?.work.supportingResources[0]?.url],
    ["Work Supporting Resources Caption", product?.work.supportingResources[0]?.caption],
    ["Work Audience Description", product?.work.audience?.description],
    ["Work ONIX Audiences Code", product?.work.audience?.onixAudiences[0]?.code],
    ["Work Audience Content Warnings Code", product?.work.audience?.contentWarnings[0]?.code],
    ["Work Audience Content Warnings Description", product?.work.audience?.contentWarnings[0]?.description],
    ["Work eBook LCCN", product?.work.ebookLccn],
    ["Work Edition Number", product?.work.editionNumber],
    ["Work Edition Statement", product?.work.editionStatement],
    ["Work LC Childrens Subject Heading", product?.work.lcChildrensSubjectHeading],
    ["Work LC Fiction Genre Heading", product?.work.lcFictionGenreHeading],
    ["Work LC Subject Heading", product?.work.lcSubjectHeading],
    ["Work LC Subject Heading Region", product?.work.lcSubjectHeadingRegion],
    ["Work LCCN", product?.work.lccn],
    ["Work Main BIC Code", product?.work.mainBicCode[0]?.code],
    ["Work Main BIC Code Description", product?.work.mainBicCode[0]?.description],
    ["Work Main THEMA Code", product?.work.mainThema[0]?.code],
    ["Work Main THEMA Code Description", product?.work.mainThema[0]?.description],
    ["Work Main THEMA Parent Code", product?.work.mainThema[0]?.parentCode],
    ["Work Main BISAC Code", product?.work.mainBisac[0]?.code],
    ["Work Main BISAC Description", product?.work.mainBisac[0]?.description],
    ["Work Series Memberships Part Number", product?.work.seriesMemberships[0]?.partNumber],
    ["Work Series Memberships ID", product?.work.seriesMemberships[0]?.id],
    ["Work Series Name", product?.work.seriesMemberships[0]?.series?.name],
    ["Work Series Title with Subtitle", product?.work.seriesMemberships[0]?.series?.titleWithSubtitle],
    ["Work Series Subtitle", product?.work.seriesMemberships[0]?.series?.subtitle],
    ["Work Series ID", product?.work.seriesMemberships[0]?.series?.id],
    ["Work Series Print ISSN", product?.work.seriesMemberships[0]?.series?.printIssn],
    ["Work Series Online ISSN", product?.work.seriesMemberships[0]?.series?.onlineIssn],
    ["Work Similar Product Authorship Description", product?.work.similarProducts[0]?.authorshipDescription],
    ["Work Similar Product ID", product?.work.similarProducts[0]?.id],
    ["Work Similar Product ISBN", product?.work.similarProducts[0]?.isbn?.isbn13],
    ["Work Similar Product Full Title", product?.work.similarProducts[0]?.fullTitle],
    ["Work Similar Product In House Edition ID", product?.work.similarProducts[0]?.inHouseEdition?.id],
    ["Work Similar Product Contributor Name", product?.work.similarProducts[0]?.contributions[0]?.contributor?.name],
    ["Work Similar Product ONIX Contributor Role Description", product?.work.similarProducts[0]?.contributions[0]?.onixContributorRole?.description],
  ]);
  if (
    !Object.values(FIELD_TYPE).includes(fieldType) || 
    fieldType === FIELD_TYPE.PRODUCT_ROWS && product === null
  ) {
    return [];
  }
  switch (fieldType) {
    case FIELD_TYPE.HEADERS:
      return Array.from(fields.keys());
    case FIELD_TYPE.PRODUCT_ROWS:
      return Array.from(fields.values());
  }
}
```

Here is an example of how your report could look.

![Google Sheets screenshot showing a populated spreadsheet with metadata retrieved from Consonance.](/images/samplecustomreport.png)

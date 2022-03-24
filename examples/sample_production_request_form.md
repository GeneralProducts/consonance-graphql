# Description

Generate a production request form for a particular product

## Explanation

Using a combination of GraphQL and Javascript in Google Docs and Apps Script you can populate fields for a production request form.

```gql
function onOpen() {
  DocumentApp.getUi()
    .createMenu('Consonance')
    .addItem('Show sidebar', 'showSidebar')
    .addToUi();
}

function showSidebar() {
  const html = HtmlService.createHtmlOutputFromFile('sidebar')
    .setTitle('Choose product');
  DocumentApp.getUi()
    .showSidebar(html)
}

function queryResponse(isbn, key, url) {
  const queryString = `
    query {
      product(productSearch: {isbn13Eq: "${isbn}"}) {
        id
        fullTitle
        subtitle
        publicationDate
        plannedPublicationDate
        isbn {isbn10 isbn13}
        marketingTexts {
          externalText
        }
      inHouseEdition{
        name
        code
        inHouseFormats {code}
      }
      supportingResources {
        url
        caption
      }
      contributions {
        id
        sequenceNumber
        productIds
        contributor {
          id
          __typename
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
    }
  }
` 
  const payload = JSON.stringify({query: queryString})
  const response = UrlFetchApp.fetch(`${url}/graphql`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      authorization: `Bearer ${key}`,
    },
    payload: payload,
  });
  const stringResponse = response.getContentText()
  return JSON.parse(stringResponse)
}
// This sets default values if "form" is not there - i.e. if you're running this from the script editor
const main = (form = {isbn13:"YOUR ISBN", key:"YOUR_API_KEY", url:"https://web.consonance.app"}) => {
  const doc = DocumentApp.getActiveDocument()
  const body = doc.getBody();
  const product = queryResponse(form.isbn13, form.key, form.url).data.product
  console.log(product)

  body.replaceText("^.*Title_*$", product.fullTitle);
  body.replaceText("^.*Blurb_*$", product.marketingTexts[0]?.externalText || "");
  body.replaceText("^.*Subtitle_*$", product.subtitle || "");
  body.replaceText("^.*contributors_*$", product.contributions[0].contributor?.name || "");
  body.replaceText("^.*isbn_13_*$", product.isbn.isbn13 || "-");
  body.replaceText("^.*pub_date_*$", product.publicationDate || "-");
  body.replaceText("^.*planned_pub_*$", product.plannedPublicationDate || "-");
  body.replaceText("^.*inHouseEditionName_*$", product.inHouseEdition.name || "-");
  body.replaceText("^.*inHouseEditionCode_*$", product.inHouseEdition.code || "-");

}





```

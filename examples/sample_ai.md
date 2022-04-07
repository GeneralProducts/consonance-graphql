# Description

Generate an AI sheet from a particular work

## Explanation

Using a combination of GraphQL and Javascript in Google Docs and Apps Script you can populate fields for an AI or tip sheet using Product level fields.

Create a new document in Google Docs and navigate to **Tools > Script Editor**

From the Script Editor screen select the plus icon next to **Files** and create a new HTML file. The code below will generate interactive sidebar that can be accessed from the main navigation of your document with a form that will allow you to populate a new AI sheet by inputting an ISBN. It can be customised as with a typical HTML document using CSS.

```gql
<!DOCTYPE html>
<html>
  <head>
    <base target="_top">
        <!--link href='https://fonts.googleapis.com/css?family=Open+Sans:400,600,300' rel='stylesheet' type='text/css'-->
    <link href='open-sans.css' rel='stylesheet' type='text/css'>
    <style type="text/css">
        * {
            font-family: 'Open Sans';
            font-style: normal;
            font-size: 13px;
        }
        .light { 
            font-weight: 300;
        }
        .regular { 
            font-weight: 400;
        }
        .semibold { 
            font-weight: 600;
        }
    </style>
  </head>
  <body>
    <p>
      Please type in the URL, Key, and ISBN13 that you want, and choose Find.
    <ul>
      <li>URL is usually https://web.consonance.app</li>
      <li>Key will be your API that you get by emailing support@consonance.app</li>
      <li>ISBN is something like 9781905005123</li>
    </ul>
    </p>
    <form id="isbnGetter">
      <p class="semibold">
        <label>Consonance URL</label><br />
        <input type="text" value="https://web.consonance.app" id="url" name="url">
      </p>
      <p class="semibold">
        <label>Key</label>
        <br/>
        <input type="text" value="" id="key" name="key">
      </p>
      <p class="semibold">
        <label>ISBN</label>
        <br/>
        <input type="text" value="" id="isbn13" name="isbn13">
       </p>
      <input type="button" value="Find" onclick="google.script.run.main(isbnGetter)">
    </form>
  </body>
</html>
```

Still on the Script Editor screen, select Code.gs from the file section. Paste the code below, which will be your main script.

Return to your main Google Doc and there should now be a **Consonance** option in your main navigation bar. You can select this to open the sidebar and populate the form.

You can style your AI sheet to your requirements. To add data to your sheet you need to include keyword text that the script can find and replace with the values retrieved from Consonance. If you wanted to include the product's title, you would need to include **Title_** where you would like it to appear and once the script runs it would replace this with your title, retaining any styling or formatting you have applied. Some examples are included here, but you can add and remove fields by referring to the GraphQL schema and following the structure in the script for the existing fields.

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


//Here you can insert or remove GraphQL fields to customise 
// what you would like to appear in your AI sheet. Currently this retrieves
// a product's title, pub date, contributors, marketing texts and front cover.

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
// This sets default values if "form" is not there - i.e. if you're running this from the script editor. Paste your API key here and a default ISBN.
const main = (form = {isbn13:"YOUR ISBN", key:"YOUR_API_KEY", url:"https://web.consonance.app"}) => {
  const doc = DocumentApp.getActiveDocument()
  const body = doc.getBody();
  const product = queryResponse(form.isbn13, form.key, form.url).data.product
  console.log(product)

//This section is how the Google doc is populated, replacing keywords with the data pulled
// from Consonance. You can add or delete these as you require. For the data to appear in
// your doc you will need to put the value in quotations into your doc like this **Blurb_** 
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

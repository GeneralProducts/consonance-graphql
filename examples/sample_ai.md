# Sample AI/Tip Sheet

Generate an AI sheet from a particular work

## Explanation

Using a combination of GraphQL and Javascript in Google Docs and Apps Script you can populate fields for an AI or tip sheet using Product level fields.

**Setting Up the Sidebar in Google Docs:**

* Open a new Google Docs document.
* Navigate to `Extensions > Apps Script`.
* In the Apps Script Editor, click the plus icon (+) next to "Files" to create a new HTML file named 'sidebar'.
* Use the provided HTML code to create an interactive sidebar. This sidebar, accessible from the main navigation of your document, includes a form for inputting an ISBN, URL, and API key. The sidebar can be customised with CSS.

```
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
        <label>Consonance URL</label>
        <br />
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

**Creating the Main Script:**

* In the Script Editor, select `Code.gs` from the file section.
* Copy and paste the provided JavaScript code into this file. This code includes functions for opening the sidebar, sending GraphQL queries, and populating your Google Doc with the fetched data.
* Replace placeholders like `YOUR_API_KEY` with actual values provided by Consonance support.

<pre><code>function onOpen() {
<strong>    DocumentApp.getUi()
</strong>    .createMenu('Consonance')
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
    query ProductsByISBN{
      product(productSearch: {isbn13Eq: "${isbn}"}) {
        id
        fullTitle
        subtitle
        publicationDate
        plannedPublicationDate
        isbn {isbn10 isbn13 isbnWithDashes}
        gbp_consumer_price: prices(priceSearch:{currencyCodeIn: [GBP], onixPriceTypeCodeIn: [_02], onixPriceTypeQualifierCodeIn: [_05]}) {
          amount
        } 
        shortDescription: marketingTexts(variantIn: [SHORT_DESCRIPTION]) {
          externalText
        }       
        marketingTexts {
          externalText
        }
        inHouseEdition{
          name
          code
          inHouseFormats {code}
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
        work {
          contributorPrettyList
          editionNumber
          seriesMemberships{
            series{
              name
              titleWithSubtitle
              subtitle
              id
              printIssn
              onlineIssn
            }
          }
          supportingResources {
                url
                caption
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
const main = (form = {isbn13:"YOUR_ISBN", key:"YOUR_API_KEY", url:"https://web.consonance.app"}) => {
  const doc = DocumentApp.getActiveDocument()
  const body = doc.getBody();
  const product = queryResponse(form.isbn13, form.key, form.url).data.product

  // This function generates your product cover image  
  function insertImage(doc, url) {
    var resp = UrlFetchApp.fetch(url);
    var image = resp.getBlob();
    var body = doc.getBody();
    var oImg = body.findText("coverImage").getElement().getParent().asParagraph();
    oImg.clear();
    oImg = oImg.appendInlineImage(image);
    oImg.setWidth(200);
    oImg.setHeight(280);
  }

// This section controls which of your metadata fields appear on the page.
  body.replaceText("^.*Title_*$", product.fullTitle || "");
  body.replaceText("^.*contributorNames_*$", product.work.contributorPrettyList || "");
  body.replaceText("^.*Blurb_*$", product.shortDescription[0]?.externalText || "");
  body.replaceText("^.*marketingText_*$", product.marketingTexts[0]?.externalText || "");
  body.replaceText("^.*pubDate_*$", product.publicationDate || "-");
  body.replaceText("^.*Subtitle_*$", product.subtitle || "");
  body.replaceText("^.*ISBN13_*$", product.isbn.isbnWithDashes || "-");
  body.replaceText("^.*editionNumber_*$", product.work.editionNumber || "-");
  body.replaceText("^.*priceGBP_*$", product.gbp_consumer_price[0]?.amount || "-");
  body.replaceText("^.*pageCount_*", product.approximatePageCount || "-");
  body.replaceText("^.*heightMm_", product.productHeightMm || "-");
  body.replaceText("^.*widthMm_", product.productWidthMm || "-");
  body.replaceText("^.*marketName_*$", product.work.mainBisac[0]?.description || "-");
  body.replaceText("^.*BISACcode_*$", product.work.mainBisac[1]?.code || "-");
  body.replaceText("^.*BISACdescription_*$", product.work.mainBisac[1]?.description || "-");
  body.replaceText("^.*BICcode_*$", product.work.mainBicCode[0]?.code || "-");
  body.replaceText("^.*BICdescription_*$", product.work.mainBicCode[0]?.description || "-");
  body.replaceText("^.*THEMAcode_*$", product.work.mainThema[0]?.code || "-");
  body.replaceText("^.*THEMAdescription_*$", product.work.mainThema[0]?.description || "-");
  body.replaceText("^.*seriesName_*$", product.work.seriesMemberships[0]?.series?.name || "-");
  insertImage(doc, product.work.supportingResources[0]?.url);
}
</code></pre>

**Using the Sidebar:**

* Back in your Google Docs document, a new menu item "Consonance" should appear.
* Click on it and select "Show sidebar" to open the newly created sidebar.
* Enter the required information (URL, API Key, ISBN) and click "Find" to populate the AI sheet.

**Customising Your AI Sheet:**

* In your Google Doc, use specific keywords as placeholders (e.g., `Title_`) where you want product information to appear.
* The script will replace these placeholders with actual product data, retaining any formatting or styling.
* Refer to the GraphQL schema to add or remove fields as needed.

**Resetting the Document:**

* To generate a new AI sheet for a different product, reset the document to its template form before running the script again. Failure to do so may result in incorrect or incomplete data population.

Here is an example of how your AI could look before it is populated.

![Google Docs screenshot showing an AI sheet containing templated text.](../images/sampleAI.png)

This is how your AI could look once you have run the script.

[An AI Sheet populated using GraphQL](../images/sampleAI.pdf)

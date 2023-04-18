# Description

Generate a production request form for a particular product using Google Docs

## Explanation

Using a combination of GraphQL and Javascript in Google Docs and Apps Script you can populate fields for a production request form using fields that retrieve Work and Product level information. Note that some of these fields will require a production run to exist in Consonance in order to populate.

Create a new document in Google Docs and navigate to **Tools > Script Editor**

From the Script Editor screen select the plus icon next to **Files** and create a new HTML file. The code below will generate interactive sidebar that can be accessed from the main navigation of your document with a form that will allow you to populate a production request form by inputting an ISBN. It can be customised as with a typical HTML document using CSS. We have included some basic font styles, but these can be changed to suit your preferences.

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

Still on the Script Editor screen, select Code.gs from the file section. Paste the code below, which will be your main script. Replace YOUR_API_KEY with the API key supplied by Consonance support and choose an ISBN to use as a default. You will be able to enter any ISBN using the sidebar form, but the script will need a default value to use first.

Return to your main Google Doc and there should now be a **Consonance** option in your main navigation bar. You can select this to open the sidebar and populate the form.

You can style your form to your requirements. To add data to your sheet you need to include keyword text that the script can find and replace with the values retrieved from Consonance. If you wanted to include the product's ISBN, you would need to include **ISBN13_** where you would like it to appear and once the script runs it would replace this with your title, retaining any styling or formatting you have applied. Some examples are included here, but you can add and remove fields by referring to the GraphQL schema and following the structure in the script for the existing fields.

You will need to undo the document back to its templated values each time the script is run. If you run the script with the form already populated it will not work correctly.

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
        publicationDateString: publicationDateString(directives: "%B %Y")
        plannedPublicationDate
        isbn {isbn10 isbn13 isbnWithDashes}
        gbp_consumer_price: prices(priceSearch:{currencyCodeIn: [GBP], onixPriceTypeCodeIn: [_02], onixPriceTypeQualifierCodeIn: [_05]}) {
          amount
        }
      onix21ProductForm{ code description notes isDeprecated}
      onix21ProductFormCode
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
          imprint {
          id 
          name
          }
productionRuns {
        id
        name
        deliveryDate
        quantities {
          id
          name
          description
          quantity
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
  // Looks for a YYYY-MM-DD date pattern and replaces with today's date
  const today = Utilities.formatDate(new Date(), "GMT", "yyyy-MM-dd");
  const pattern = "\\d{2,4}-\\d{1,2}-\\d{1,2}";
  const todayDate = DocumentApp.getActiveDocument().getBody();
  body.editAsText().replaceText(pattern, today);

  body.replaceText("^.*QUOTEREF_*$", product.id || "-");
  body.replaceText("^.*ISBN13_*$", product.isbn.isbnWithDashes || "-");
  body.replaceText("^.*PUBDATE_*$", product.publicationDateString || "-");
  body.replaceText("^.*PUBDATE_*$", product.publicationDateString || "-");
  body.replaceText("^.*JOBNO_*$", product.work[0]?.productionRuns[0]?.id || "-");
  body.replaceText("^.*IMPRINTNAME_*$", product.work[0]?.imprint.name || "-");
  body.replaceText("^.*GBPRRP_*$", product.gbp_consumer_price[0]?.amount || "-");
  body.replaceText("^.*FORMATTYPE_*$", product.onix21ProductForm.description || "-");
  body.replaceText("^.*TRIMSIZE_*$", product.work[0]?.productionRuns[0]?.bindings[0]?.trimmingMethod || "-");
  body.replaceText("^.*BINDINGTYPE_*$", product.work[0]?.productionRuns[0]?.bindings[0]?.description || "-");
  body.replaceText("^.*PAGEEXTENT_*", product.productionPageCount || "-");
  body.replaceText("^.*PLATEEXT_*", product.totalUnnumberedInsertPageCount || "-");
  body.replaceText("^.*PLATEPOS_*", product.work[0]?.productionRuns[0]?.bindings[0]?.endpapers || "-");
  body.replaceText("^.*SPINEWIDTH_*", product.work[0]?.productionRuns[0]?.bindings[0]?.caseSpineStyle || "-");
  body.replaceText("^.*BOARDWIDTH_*", product.work[0]?.productionRuns[0]?.bindings[0]?.widthOfBoard || "-");
  body.replaceText("^.*TEXTCOL_*", product.work[0]?.productionRuns[0]?.interiors[0]?.textColour || "-");
  body.replaceText("^.*SECTCOL_*", product.work[0]?.productionRuns[0]?.interiors[0]?.textColourDescription || "-");
  body.replaceText("^.*SECTCOVER_*", product.work[0]?.productionRuns[0]?.covers[0]?.coverType || "-");
  body.replaceText("^.*PRINTFINISH_*", product.work[0]?.productionRuns[0]?.finishes[0]?.description || "-");
  body.replaceText("^.*TEXTPAPER_*", product.work[0]?.productionRuns[0]?.interiors[0]?.paper || "-");
  body.replaceText("^.*SECTPAPER_*", product.work[0]?.productionRuns[0]?.interiors[0]?.paper || "-");
  body.replaceText("^.*JACKETMAT_*", product.work[0]?.productionRuns[0]?.covers[0]?.paper || "-");
  body.replaceText("^.*CLOTHCOL_*", product.work[0]?.productionRuns[0]?.covers[0]?.coverColoursOutside || "-");
  body.replaceText("^.*ENDPAPERMAT_*", product.work[0]?.productionRuns[0]?.bindings[0]?.endpapers || "-");
  body.replaceText("^.*MATEXTRAS_*", product.work[0]?.productionRuns[0]?.embellishments[0]?.description || "-");
  body.replaceText("^.*EXPDELIVERY_*", product.plannedPublicationDate || "-");
  body.replaceText("^.*DELQUANTITY_*", product.work[0]?.productionRuns[0]?.quantities[0]?.receivedQuantity || "-");
}




```

Here is an example of how your Google doc might look before the script runs.

![Google Docs screenshot. The document has the heading Consonance Production Request Form and has many fields that are unpopulated.](/images/sampleprodreqformbefore.png)

This is how it might look after choosing the 'Find' button.

![Google Docs screenshot. The previously unpopulated fields now have relevant information retrieved from Consonance.](/images/sampleprodreqformafter.png)

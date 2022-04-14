# Description

Generate a production request form for a particular product

## Explanation

Using a combination of GraphQL and Javascript in Google Docs and Apps Script you can populate fields for a production request form using fields that retrieve Work and Product level information. Note that some of these fields will require a production run to exist in Consonance.

Create a new document in Google Docs and navigate to **Tools > Script Editor**

From the Script Editor screen select the plus icon next to **Files** and create a new HTML file. The code below will generate interactive sidebar that can be accessed from the main navigation of your document with a form that will allow you to populate a production request form by inputting a Work ID. It can be customised as with a typical HTML document using CSS. We have included some basic font styles, but these can be changed to suit your preferences.

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

You can style your AI sheet to your requirements. To add data to your sheet, include keyword text that the script can find and replace with the values retrieved from Consonance. For example, to include the product's title, add **Title_** where you would like it to appear. Once the script runs it would replace this with your title, retaining any styling or formatting you have applied. Some examples are included here, but you can add and remove fields by referring to the GraphQL schema and following the structure in the script for the existing fields.

```gql
function onOpen() {
  DocumentApp.getUi()
    .createMenu('Consonance')
    .addItem('Show sidebar', 'showSidebar')
    .addToUi();
}

function showSidebar() {
  const html = HtmlService.createHtmlOutputFromFile('sidebar')
    .setTitle('Choose work');
  DocumentApp.getUi()
    .showSidebar(html)
}

// This section dictates which GraphQL fields are being queried. There are many other fields available but these are most relevant to production or print runs.
function queryResponse(id, key, url) {
  const queryString = `
    query {
  works(workSearch: {idEq: "${id}"}) {
    clientId
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
    imprint {
      id 
      name
    }
    products {
    id
    isbn {isbn10 isbn13 isbnWithDashes}
      publicationDate
      shortDescription: marketingTexts(variantIn: [SHORT_DESCRIPTION]) {
        externalText
      }
      gbp_consumer_price: prices(priceSearch:{currencyCodeIn: [GBP], onixPriceTypeCodeIn: [_02], onixPriceTypeQualifierCodeIn: [_05]}){
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
    projectStage
    season
  }
}`
  const date = Utilities.formatDate(new Date(), "GMT", "MM-dd-yyyy"); 
  const pattern = "\\d{1,2}-\\d{1,2}-\\d{2,4}";
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
const main = (form = {id:"91804", key:"56ff84fdac5d4bf985f8f0def461e4a9", url:"https://web.consonance.app"}) => {
const doc = DocumentApp.getActiveDocument()
const body = doc.getBody();
const work = queryResponse(form.id, form.key, form.url).data.works

// This looks for a YYYY-MM-DD date pattern and replaces with today's date. CAUTION: If you
// select the Find button more than once, this may overwrite any dates that have already been
// generated, e.g. pub date. If you want to re-generate your form, reset the default values first.

const today = Utilities.formatDate(new Date(), "GMT", "yyyy-MM-dd");
const pattern = "\\d{2,4}-\\d{1,2}-\\d{1,2}";
const todayDate = DocumentApp.getActiveDocument().getBody();
body.editAsText().replaceText(pattern, today);

// This section controls which fields will appear in your form. You can place these in any order
// in your script, add new ones using the pattern shown here, or delete unused fields. The script is 
// currently set to retrieve the first value it finds that meets the criteria for that field.

  body.replaceText("^.*Title_*$", work[0]?.title);
  body.replaceText("^.*imprintName_*$", work[0]?.imprint.name);
  body.replaceText("^.*productID_*$", work[0]?.products[0]?.id);
  body.replaceText("^.*ISBN13_*$", work[0]?.products[0]?.isbn.isbn13 || "-");
  body.replaceText("^.*contributorName_*$", work[0]?.contributions[0]?.contributor?.name || "")
  body.replaceText("^.*GBPconsumer_*$", work[0]?.products[0]?.gbp_consumer_price || "")
  body.replaceText("^.*pubDate_*$", work[0]?.products[0]?.publicationDate || "-");
  body.replaceText("^.*deliveryDate_*$", work[0]?.productionRuns[0]?.deliveryDate || "-");
  body.replaceText("^.*productionRunID_*$", work[0]?.productionRuns[0]?.id || "-");
  body.replaceText("^.*widthOfBoard_*$", work[0]?.productionRuns[0]?.bindings[0]?.widthOfBoard || "-");
  body.replaceText("^.*bindingDescription_*$", work[0]?.productionRuns[0]?.bindings[0]?.description || "-");
  body.replaceText("^.*pageBindingMethod_*$", work[0]?.productionRuns[0]?.bindings[0]?.pageBindingMethod || "-");
  body.replaceText("^.*trimmingMethod_*$", work[0]?.productionRuns[0]?.bindings[0]?.trimmingMethod || "-");
  body.replaceText("^.*productionPageCount_*$", work[0]?.products[0]?.productionPageCount || "-");
  body.replaceText("^.*productFormat_*$", work[0]?.products[0]?.onix21ProductForm.description || "-");
  body.replaceText("^.*textColour_*$", work[0]?.productionRuns[0]?.interiors[0]?.textColour  || "-");
  body.replaceText("^.*textColourDescription_*$", work[0]?.productionRuns[0]?.interiors[0]?.textColourDescription  || "-");
  body.replaceText("^.*isFSC_*$", work[0]?.productionRuns[0]?.interiors[0]?.fsc  || "-");
  body.replaceText("^.*textPaper_*$", work[0]?.productionRuns[0]?.interiors[0]?.paper  || "-");
  body.replaceText("^.*coverPaper_*$", work[0]?.productionRuns[0]?.covers[0]?.paper || "-")
  body.replaceText("^.*coverType_*$", work[0]?.productionRuns[0]?.covers[0]?.coverType || "-")
  body.replaceText("^.*coverColours_*$", work[0]?.productionRuns[0]?.covers[0]?.coverColoursOutsideDescription || "-")
  body.replaceText("^.*embellishmentDescription_*$", work[0]?.productionRuns[0]?.embellishments[0]?.description || "-")
  body.replaceText("^.*finishesDescription_*$", work[0]?.productionRuns[0]?.finishes[0]?.description || "-")

  }
}

```

Here is an example of how your Google doc might look before the script runs.

![Google Docs screenshot. The document has the heading Consonance Production Request Form and has many fields that are unpopulated.](/images/sampleprodreqformbefore.png)

This is how it might look after choosing the 'Find' button.

![Google Docs screenshot. The previously unpopulated fields now have relevant information retrieved from Consonance.](/images/sampleprodreqformafter.png)

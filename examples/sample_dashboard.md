# Description

Populate a dashboard of charts and graphs using Google Data Studio

## Explanation

Using Google Sheets and Google Data Studio you can populate charts and graphs to create a dashboard of product level data.

First, you'll need to build a [custom report](/examples/sample\_custom\_report.md) that contains all of the fields you need and run this custom report at least once.

Create a new report on [Google Data Studio](https://datastudio.google.com/) and select **Google Sheets** from the selection of Google Connectors.

Link your previously created custom report created in Google Sheets as a data source by selecting the spreadsheet. There will be a pop-up warning you that you are about to add data to your report; choose **Add to Report** to continue.

Use the options in the main navigation or sidebar to add a chart using the fields from your custom report.

Refresh the data in your report with choosing **View > Refresh Data** from the main navigation bar, or with the keyboard shortcut **Ctrl+Shift+E**

### Examples

Total products filtered by format, publication status, sales territories or publication date

![Google Data Studio screenshot. The dashboard shows a pie chart, a line graph, a bar chart and a pivot table.](/images/sampledashboard.png)

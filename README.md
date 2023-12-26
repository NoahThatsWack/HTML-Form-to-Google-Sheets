# How to Send Data From an HTML Form to Google Sheets

*Updated for Google Script Editor 2022 Version.*

The following will teach you how to send data from an HTML form to a Google Sheet using only HTML and JavaScript.

This example shows how to set up a mailing list/newsletter form that sends data from an HTML form to Google Sheets, but you can use it for any sort of data.

### 1. Set up a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com) and create a new sheet. This is where we'll store the form data.
2. Set the headers in the first row to whatever data you are collecting, _they must match the input name you are using in your HTML form_:

Example:

|   |     A     |   B   | C | ... |
|---|:---------:|:-----:|:-:|:---:|
| 1 | Date | firstANDlast | Email   |     |


### 2. Create a Google Apps Script

<img src="https://gyazo.com/0be3aa60627deb228133de1d6d2d4560.gif" width="600" />

Click on `Extensions -> Apps Script`. This will open new Google Script. Rename it to something like _"Newsletter"_.

<img src="https://gyazo.com/acc0402a87460de94d61388b9377308a.gif" width="600" />

Replace `function myFunction() { ...` with the following code:

```js
// Original code from https://github.com/jamiewilson/form-to-google-sheets
// Updated for 2021 and ES6 standards

const sheetName = 'Sheet1'
const scriptProp = PropertiesService.getScriptProperties()

function initialSetup () {
  const activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet()
  scriptProp.setProperty('key', activeSpreadsheet.getId())
}

function doPost (e) {
  const lock = LockService.getScriptLock()
  lock.tryLock(10000)

  try {
    const doc = SpreadsheetApp.openById(scriptProp.getProperty('key'))
    const sheet = doc.getSheetByName(sheetName)

    const headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0]
    const nextRow = sheet.getLastRow() + 1

    const newRow = headers.map(function(header) {
      return header === 'Date' ? new Date() : e.parameter[header]
    })

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow])

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  catch (e) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': e }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  finally {
    lock.releaseLock()
  }
}
```

Save the project `ctrl+s` before moving on to the next step.

### 3. Run the initialSetup function

<img src="https://gyazo.com/e5f165d74d02d1c85c1c5a1c73bedcd3.gif" width="600" />

You should see a modal/popup asking for permissions. Click `Review permissions` and continue to the next screen.

<img src="https://gyazo.com/5615474e9c767680710f8dff6e360d12.png" width="600" />

Because this script has not been reviewed by Google, it will generate a warning before you can continue. You must click the "Go to Newsletter (Unsafe)" for the script to have the correct permissions to update your form.

<img src="https://gyazo.com/1ac09434dc808a6a367c8a0de99a56f7.png" width="450" />

Once you click the "Go to Newsletter (Unsafe)" click "Allow".

<img src="https://gyazo.com/60b1003931896faafda89220a3398526.png" width="450" />

After giving the script the correct permissions, run the script again, you should see the following output in the script editor console:

<img src="https://gyazo.com/835e7398ee684999920e68159e90e9a6.png" width="450" />

Now your script has the correct permissions to continue to the next step.

### 4. Add a trigger for the script

<img src="https://gyazo.com/2e9ed96ba8ce5fa19b7c3084d0b52853.gif" width="600" />

Select the project "Triggers" from the sidebar and then click the `Add Trigger` button.

In the window that appears, select the following options:

- Choose which function to run: `doPost`
- Choose which deployment should run: `Head`
- Select event source: `From spreadsheet`
- Select event type: `On form submit`

<img src="https://gyazo.com/3fcf4414069df204e629eac9e2660a9c.gif" width="600" />

Then select "Save". It will ask you to review permissions *again*, click "Allow".

### 5. Publish the project

Now your project is ready to publish. Select the `Deploy` button and `New Deployment` from the drop-down.

<img src="https://gyazo.com/920ebb0b86b9b8cbc506008c216351b2.gif" width="600" />

Click the "Select type" icon and select `Web App`. 

In the form that appears, select the following options:

- Description: `Newsletter blah blah blah` (This can be anything that you want. Just make it descriptive.)
- Web app → Execute As: `Me`
- Web app → Who has access: `Anyone`

Then click `Deploy`.

**Important:** Copy and save the web app URL before moving on to the next step.

<img src="https://gyazo.com/145988a4dd7a73672a3eb3ccfd703c0b.gif" width="600" />

### 6. Configure your HTML form

Configure your HTML form like the following, replacing `WEBAPP_URL` with the web app URL you saved from the previous step.

**Make sure you update the the code with the proper information.**

```html
<form id="FORM_ID" method="POST" action="WEBAPP_URL">
  <input name="Email" type="email" placeholder="Email" required>
  <input name="Name" type="text" placeholder="Name" required>
  <button type="submit">Send</button>
</form>
```
---

**OPTIONAL:** To create a custom thank you page copy the javascript below. Replace `FORM_ID` in the HTML and JavaScript with the proper ID and `https://www.YOUR_WEBSITE.com/thanks` with the proper url for your thank you or home page.

```html
<script type = "text/javascript" >
    window.addEventListener("DOMContentLoaded", function() {
        const yourForm = document.getElementById('FORM_ID');
        yourForm.addEventListener("submit", function(e) {
            e.preventDefault();
            const data = new FormData(yourForm);
            const action = e.target.action;
            fetch(action, {
                method: 'POST',
                body: data,
            }).then(() => {
                window.location.replace('https://www.YOUR_WEBSITE.com/thanks')
            })
        })
    }); 
</script>
```
---

*Without this javascript code you will be redirected to a page that looks similar to the following.*

<img src="https://gyazo.com/dd29750aea025a73326bf51cb7a492e6.png" width="600" />

_**Note:** You will need to create a thank you page on your website. However a thank you page is not required, you could always redirect someone to your website home page, or previous page._

---

Now when you submit this form from any location, the data will be saved in your specified Google Sheet.

### 7. Please Note!

The input names on the google sheet are _case sensitive_. They **MUST** match the same casing as the form script. 

For example:

If your input name is _"firstANDlast"_,
```html
<input name="firstANDlast">
```
`A1` on your form would look like the following:

|   |     A     |   B   | C | ... |
|---|:---------:|:-----:|:-:|:---:|
| 1 | firstANDlast | ... | ...   |     |
| 2 | John Smith | ... | ...   |     |

THIS WILL NOT WORK:

|   |     A     |   B   | C | ... |
|---|:---------:|:-----:|:-:|:---:|
| 1 | FiRstaNdLast | ... | ...   |     |
| 2 |   | ... | ...   |     |

Additionally, to have the date included on your form you must use "Date", this is also case sensitive. A form with the date included would look similar to the following:

|   |     A     |   B   | C | ... |
|---|:---------:|:-----:|:-:|:---:|
| 1 | firstANDlast | Date | ...   |     |
| 2 | John Smith | 1/23/2023 | ...   |     |

## Thanks

Thanks to the following articles, projects, and people that inspired this guide;

- https://medium.com/@dmccoy/how-to-submit-an-html-form-to-google-sheets-without-google-forms-b833952cc175
- https://github.com/jamiewilson/form-to-google-sheets
- https://github.com/levinunnink/html-form-to-google-sheet

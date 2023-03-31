# Html-form-upload-to-google-sheets. Including files
This code helps to submit an HTML form and log all the datas (Including Files) to google sheet and get email notification.

# Workflow
1. While submiting the form, the file is uploaded to a drive folder and the link of the file is logged to out google sheet.
2. All the other text datas will be logged to the same row of the sheet in their respictive columns.
3. An email alert will be sent to us.

Use the code below and follow the instructions to achive this.

# HTML Code
Paste a the html form code in your file. Make sure you don't change the name attribute

    <form method="post" name="form-submit">
        
        <label for="name">Name</label>
        <input type="text" name="name" id="name" placeholder="Enter your Name here">
        
        <label for="mail">Email</label>
        <input type="email" name="email" id="mail" placeholder="Enter your email here">

        <label for="upload">Upload your file</label>
        <input type="file" name="upload" id="upload">

        <input type="submit" value="Place my order" id="submit">

    </form>
    
# JavaScript Code    
Paste the Javescript code in your file.


        
        const scriptURL = 'Paste URL'; //url of the app script project(google form)---------------------------
        const url = "Paste URL"; //url of the seperate app script project---------------------------

        let form = document.forms['form-submit'];
        let file = document.querySelector("#upload");
        let submitBtn = document.querySelector("input[type='submit']");

        form.addEventListener('submit', (e) => {
            e.preventDefault();

            submitBtn.value = "Submiting"; //the text to be displayed while submiting---------------------------
            submitBtn.style.backgroundColor = 'grey';

            //uploading image
            if (file.files.length > 0) {
                let fr = new FileReader();
                fr.addEventListener('loadend', () => {
                let res = fr.result;
                let spt = res.split("base64,")[1];
                let obj = {
                    base64: spt,
                    type: file.files[0].type,
                    name: file.files[0].name
                };
                fetch(url, {
                    method: "POST",
                    body: JSON.stringify(obj)
                })
                    .then(r => r.text())
                    .then(data => console.log(data));
                });
                fr.readAsDataURL(file.files[0]);
            }

            //uploading text
            fetch(scriptURL, { method: 'POST', body: new FormData(form) })
                .then(response => {
                console.log('Success!', response);
                //action to be done---------------------------
                // window.open('new.html', '_self');
                })
                .catch(error => console.error('Error!', error.message))
        });
    
# App Script Code 1

1. Open a google sheet
2. Enter the value of name attribute of the input HTML tag on the 1st row of the sheet.
3. Select Extensions form the menu bar, then App script.
4. Paste the below code.
5. Save and run, authorise the project.
6. Once the file is executed, Select deploy -> New deployment.
7. Select type as web app, give access to anyone, Deploy.
8. Once you get the web app url, paste it in the JS code on scriptURL variable.

        var sheetName = 'Sheet1' //-------------------------
        var scriptProp = PropertiesService.getScriptProperties()

        function intialSetup () {
          var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet()
          scriptProp.setProperty('key', activeSpreadsheet.getId())
        }

        function doPost (e) {
          var lock = LockService.getScriptLock()
          lock.tryLock(10000)

          try {
            var doc = SpreadsheetApp.openById(scriptProp.getProperty('key'))
            var sheet = doc.getSheetByName(sheetName)

            var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0]
            var nextRow = sheet.getLastRow() + 1

            var newRow = headers.map(function(header) {
              return header === 'timestamp' ? new Date().toLocaleString() : e.parameter[header]
            })

            sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow])

            // Send email notification
            var emailSubject = "type email subject"; //-------------------------
            var emailBody = "type body";  //-------------------------
            var email = "enter email"; //-------------------------
            MailApp.sendEmail(email, emailSubject, emailBody);

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
        
        
# App Script code 2
1. Open this url, https://script.google.com/home and create a new App Script project.
2. Paste the below code.
3. Go to the google sheet you created and add "upload" on the first row in which you'll get the link of the uploaded file.
4. Copy the url of that page from browser url bar.
5. Paste the url on the app variable.
6. change the folder name in folder variable.
7. Save and run, authorise the project.
8. Once the file is executed, Select deploy -> New deployment.
9. Select type as web app, give access to anyone, Deploy.
10. Once you get the web app url, paste it in the JS code url variable.

        let app = SpreadsheetApp.openByUrl("Paste google sheet URL"); //-------------------------
        let sheet = app.getSheetByName("Sheet1"); //-------------------------

        function doPost(e) {
          try {
            let obj = JSON.parse(e.postData.contents);
            let dcode = Utilities.base64Decode(obj.base64);
            let blob = Utilities.newBlob(dcode, obj.type, obj.name);

            // Replace "FolderName" with the name of your destination folder
            let folder = DriveApp.getFoldersByName("Folder name").next(); //-------------------------
            let newFile = folder.createFile(blob);

            let link = newFile.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW).getDownloadUrl();
            let lr = sheet.getLastRow();

            // find the column number of the header "upload"
            let headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
            let uploadCol = headers.indexOf("upload") + 1; //-------------------------

            // set the value of 'link' in the same row under the header "upload"
            sheet.getRange(lr, uploadCol).setValue(link);

            return ContentService.createTextOutput("image uploaded");
          } catch (err) {
            return ContentService.createTextOutput(err);
          }
        }

# GoogleSheet_FirebaseRealtimeDatabase
Google Sheet &amp; Firebase Realtime Database SYNC

appscript.json:

{
  "timeZone": "Asia/Dhaka",
  "dependencies": {
    "libraries": [
      {
        "userSymbol": "FirebaseApp",
        "version": "30",
        "libraryId": "1hguuh4Zx72XVC1Zldm_vTtcUUKUA6iBUOoGnJUWLfqDWx5WlOJHqYkrt",
        "developmentMode": true
      }
    ]
  },
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",

  "oauthScopes": [
    "https://www.googleapis.com/auth/firebase.database", 
    "https://www.googleapis.com/auth/userinfo.email", 
    "https://www.googleapis.com/auth/spreadsheets", 
    "https://www.googleapis.com/auth/script.scriptapp", 
    "https://www.googleapis.com/auth/script.external_request",
    "https://www.googleapis.com/auth/gmail.send",
    "https://www.googleapis.com/auth/forms"],

    "executionApi": {
    "access": "DOMAIN"
  }
}

Form.gs:

function onFormSubmit(e) {
  var prefix = 'EC';
  var startNumber = 0;
  var numDigits = 6;
  var increment = 1;
  var sheetName = 'DRS'; // Enter the sheet name
  var idColumn = 'ECID';
  var emailColumn = 'Email';
  var emailSubject = 'Registration Confirmation for Recruitment V23.2';
  var emailBody = 'You have successfully registered for the interview of Recruitment V23.2 by the Earth Club of North South University. Our Club Room is located at NAC111, beside the recreation hall and at the entrance from GATE 8. Visit our RECRUITMENT BOOTH at lower plaza area for further assistance. Remember your registration ID while attending the interview. Your registration ID: ';

  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(sheetName);
  var lastRow = sheet.getLastRow();
  var generatedId = generateId(prefix, startNumber, numDigits, increment, lastRow);

  sheet.getRange(lastRow, getColumnIndex(sheet, idColumn)).setValue(generatedId);

  var email = e.namedValues[emailColumn][0];
  var emailText = emailBody + generatedId;

  GmailApp.sendEmail(email, emailSubject, emailText);
}

function generateId(prefix, startNumber, numDigits, increment, lastRow) {
  var id = prefix + '232' + padNumber(startNumber + increment * (lastRow - 1), numDigits - 3);
  return id;
}

function padNumber(number, numDigits) {
  var paddedNumber = String(number).padStart(numDigits, '0');
  return paddedNumber;
}

function getColumnIndex(sheet, columnName) {
  var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
  var columnIndex = headers.indexOf(columnName) + 1;
  return columnIndex;
}

Gsheet_Database.gs:

/**
* Copyright 2019 Google LLC.
* SPDX-License-Identifier: Apache-2.0
*/

function getEnvironment() {
 var environment = {
   spreadsheetID: "1mOGY7ZJQaBuHCETTxkQr93lO96WTIhjPlSAV5Nd16UU",
   firebaseUrl: "https://drsystem-139f0-default-rtdb.asia-southeast1.firebasedatabase.app/"
 };
 return environment;
}

// Creates a Google Sheets on change trigger for the specific sheet
function createSpreadsheetEditTrigger(sheetID) {
 var triggers = ScriptApp.getProjectTriggers();
 var triggerExists = false;
 for (var i = 0; i < triggers.length; i++) {
   if (triggers[i].getTriggerSourceId() == sheetID) {
     triggerExists = true;
     break;
   }
 }

 if (!triggerExists) {
   var spreadsheet = SpreadsheetApp.openById(sheetID);
   ScriptApp.newTrigger("importSheet")
     .forSpreadsheet(spreadsheet)
     .onChange()
     .create();
 }
}

// Creates a form submission trigger for the specific form
function createFormSubmissionTrigger(formID) {
 var triggers = ScriptApp.getProjectTriggers();
 var triggerExists = false;
 for (var i = 0; i < triggers.length; i++) {
   if (triggers[i].getTriggerSourceId() == formID) {
     triggerExists = true;
     break;
   }
 }

 if (!triggerExists) {
   ScriptApp.newTrigger("importSheet")
     .forForm(formID)
     .onFormSubmit()
     .create();
 }
}

// Delete all the existing triggers for the project
function deleteTriggers() {
 var triggers = ScriptApp.getProjectTriggers();
 for (var i = 0; i < triggers.length; i++) {
   ScriptApp.deleteTrigger(triggers[i]);
 }
}

// Initialize
function initialize(e) {
 writeDataToFirebase(getEnvironment().spreadsheetID);
}

// Write the data to the Firebase URL
function writeDataToFirebase(sheetID) {
 var ss = SpreadsheetApp.openById(sheetID);
 SpreadsheetApp.setActiveSpreadsheet(ss);
 createSpreadsheetEditTrigger(sheetID);
 var sheets = ss.getSheets();
 for (var i = 0; i < sheets.length; i++) {
   importSheet(sheets[i]);
   SpreadsheetApp.setActiveSheet(sheets[i]);
 }
}

// A utility function to generate nested object when
// given a keys in array format
function assign(obj, keyPath, value) {
 lastKeyIndex = keyPath.length - 1;
 for (var i = 0; i < lastKeyIndex; ++i) {
   key = keyPath[i];
   if (!(key in obj)) obj[key] = {};
   obj = obj[key];
 }
 obj[keyPath[lastKeyIndex]] = value;
}

// Import each sheet when there is a change
function importSheet() {
 var sheet = SpreadsheetApp.getActiveSheet();
 var name = sheet.getName();
 var data = sheet.getDataRange().getValues();

 var dataToImport = {};

 for (var i = 1; i < data.length; i++) {
   dataToImport[data[i][0]] = {};
   for (var j = 0; j < data[0].length; j++) {
     assign(dataToImport[data[i][0]], data[0][j].split("__"), data[i][j]);
   }
 }

 var token = ScriptApp.getOAuthToken();

 var firebaseUrl =
   getEnvironment().firebaseUrl + sheet.getParent().getId() + "/" + name;
 var base = FirebaseApp.getDatabaseByUrl(firebaseUrl, token);
 base.setData("", dataToImport);
}

// Set up the form submission trigger
createFormSubmissionTrigger("1QVWn_aE1E6BBQQZ7mJ6pixm5g46KQshZeTg0EB38ebM");

Database_Gsheet.gs:



<a href="index">
<img src="images/back.png" alt="Back" height="35" width="35">
</a>

## Batch Record Manager

##### for Windows
---
### Background:
When the corporate heads at my company decided to change the software that was used in our lab, the Quality Assurance department was left with the task of receiving and organizing new records that didn't contain any easily-fileable information included in the record packets. My task was to devise a way to easily and accurately add these records to a database, assign them a new internal number, and make it easy for the computer-illiterate chemists at the laboratory to look up old records by the new software's automatically-assigned reference values. The laboratory kept most of its logs in Excel sheets, so in order to keep the learning curve to a minimum, it was decided that an Excel sheet would be the database of choice. Python3 and [openpyxl](https://openpyxl.readthedocs.io/en/stable/) were used behind Tkinter to create a GUI to manage these records within the QA department.

#### Assign Tab:
Management and ISO regulations required that the QA department enter and verify each record into the database. This, and the problems that come with having multiple people using the same spreadsheet at the same time, means that enabling each analyst to assign a batch record number individually was not an option. 

The new lab software assigned a number to each batch in the following format: ASSAY_TYPE-DATE-NUMBER. The solution for QA to manage these non-sequential records was to assign each record a new "BR number." In order to efficiently and accurately log these, a user interface was developed which allowed one to quickly choose an assay type, date, and number from drop-down menus, and save it to the Excel log with as few clicks as possible.

<img src="images/BR1.PNG">

#### Search Tab:
The search tab provided a quick way to search by either BR number or Batch Record ID. This was used to reference the folder's position in the filing area, so it could be quickly found. The program contained a local python dictionary of all of the records within the log. When typing into the search box, the program live-filters results that match the query, and populates the results box.

<img src="images/BR2.PNG">


#### Labels Tab:
The labels tab allows the user to generate sequential BR number labels to affix to the folder after the number is assigned to it. The program uses [python-docx](https://python-docx.readthedocs.io/en/latest/) to create a word doc formatted to Avery 8167 labels. If python-docx is not installed, the UI directs the user on how to install the PyPi module.

<img src="images/BR3.PNG">

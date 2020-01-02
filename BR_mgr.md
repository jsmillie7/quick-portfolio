<a href="index">
<img src="images/back.png" alt="Back" height="35" width="35">
</a>

## Batch Record Manager

##### for Windows
---
### Background:
When the corporate heads at my company decided to change the software that was used in our lab, the Quality Assurance department was left with the task of receiving and organizing new records that didn't contain any easily-fileable information included in the record packets. My task was to devise a way to easily and accurately add these records to a database, assign them a new internal number, and make it easy for the computer-illiterate chemists at the laboratory to look up old records by the new software's automatically-assigned reference values. The laboratory kept most of its logs in Excel sheets, so in order to keep the learning curve to a minimum, it was decided that an excel sheet would be the database of choice. Python3 and [openpyxl](https://openpyxl.readthedocs.io/en/stable/) were used behind Tkinter to create a GUI to manage these records within the QA department.

#### Assign Tab:
Management and ISO regulations required that the QA department enter and verify each record into the database. This, and the problems that come with having multiple people using the same spreadsheet at the same time, means that enabling each analyst to assign a batch record number individually was not an option. 

The software assigned a number to each batch in the following format: ASSAY_TYPE-DATE-NUMBER. The solution was to create a user interface that allowed one to quickly choose an assay type, date, and number from drop-down menus, and save it to the Excel log with as few clicks and typing as possible. Labels created using the __Labels__ tab were then affixed to each folder and readied to be filed in an efficient way.

<img src="images/BR1.PNG">

#### Search Tab:

<img src="images/BR2.PNG">


#### Labels Tab:

<img src="images/BR3.PNG">

<a href="index">
<img src="images/back.png" alt="Back" height="35" width="35">
</a>

## Instrument Maintenance Manager

##### for Windows
---
### Background:
Working in an analytical chemistry lab with an ISO certification, it's imperitive that instrument maintenance be performed and documented punctually. It was the responsibility of the Quality Assurance department to make sure that this was done, and I was tasked with developing a method to achieve this. Previously, several different programs had to be used together to achieve the desired results, so the goal was to develop software which fulfilled the following tasks:
* Create a data structure that allows for ease of access to data records, secure storage of records, as well as track and easily update individual instrument maintenance.
* Send out monthly reminder emails for upcoming maintenance
* Interface with a shared Microsoft Outlook calendar to sync maintenance events
* Print out ISO-mandated maintenance stickers for instruments

This program is one of the most complex pieces of software that I have individually developed. I was able to simplify the overall complexity by taking advantage of python's object-oriented programming language and modularizing the overall GUI into individual frames, which made upgrades and changes much easier. The entire code is long, messy, and a bit hard to read, so for this portfolio entry, I will go over some key functions implemented in the backend without going in depth about Tkinter and the rest of the front-end.


#### Step 1: Data Structure:
The first step was to create an easily-indexable dictionary of the instruments within the laboratory. Each instrument was created as a new instance of a class called "Equipment", which allowed the user to create an object with individual attributes for each HPLC, balance, and all of the other various equipment types.

```python
class Equipment:
    def __init__(self):
        self.equpiment_number = None
        self.hidden = False
        self.in_service = True
        self.history = {
            'OQ': history(),
            'Annual PM': history(),
            'Biannual PM': history(),
            'Calibration':history()
            }
        self.notes = ''
```
The instances were added to a dictionary in the following format,
```python
instruments = {
   'E00001': <class obj: Equipment>,
   'E00002': <class obj: Equipment>,
   ...
}
```
where they could be easily referenced by their offical equipment number (E#). Additional attributes, such as equipment type, serial numbers, and location within the building, could be optionally added to each instrument. Within the self.history attribute, an instance of each of the 4 possible maintenance types was created as the history() class, which contained the information of that instrument's maintenance history.
```python
class history:
    def __init__(self):
        self.date = None # the date of the maintenance in datetime.date format
        self.expiration = None # when the next maintenance item is due (datetime.date)
        self.analyst = None # string of who performed the maintenance
        self.number = None # each mainenance event is assigned a unique number.
```
The heart of the instrument maintenance history record information is contained in the instruments dictionary. It was serialized and pickled to a .pkl file, which was loaded each time the program opened, and saved when the user chose, and at program exit. Once this was complete, the other outlined requirements above could be accomplished.

#### Step 2: Monthly Email Reminder
With a solid data structure in hand, and the manual labor behind creating instances for each of the hundreds of pieces of equipment within the lab was complete, it was time to start automating some of the tasks that this software really aimed at simplifying. The first task I chose to tackle was simple: generate and send a monthly email reminder to a list of email addresses that contained all upcoming and overdue equipment items through a shared Outlook email account. With Outlook 2017 and Windows, this was a fairly straight-forward task. 

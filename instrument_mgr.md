<a href="index">
<img src="images/back.png" alt="Back" height="35" width="35">
</a>

## Instrument Maintenance Manager

##### for Windows
---
### Background:
Working in an analytical chemistry lab with an ISO certification, it's imperitive that instrument maintenance be performed and documented punctually. It was the responsibility of the Quality Assurance department to make sure that this was done, and I was tasked with developing a method to achieve this. Previously, several different programs had to be used together to achieve the desired results, so the goal was to develop software which fulfilled the following tasks:
* Create a data structure that allows for ease of access to data records, and allows for secure storage of records.
### update below
* Send out monthly reminder emails for upcoming maintenance
    * this was a manual task which required the sender to create a list of upcoming equipment by scrolling through a log and adding each item, which was tedious and error-prone. Python can do this faster without error.
* Interface with a shared Microsoft Outlook calendar to sync maintenance events
    * this was also a manual task wich required each of the 250+ instruments to be added to an Outlook calendar. This process was easily done with python as well.
* Print out ISO-mandated maintenance stickers for instruments
    * each time a maintenance event occurred, a new sticker was required on the instrument. Manually, not a difficult task, but with Python and a networked label printer, this task became completely automated.
* Track and easily update individual instrument maintenance
    * This was previously not done, which led to some instruments missing a calibration date, so implementing this was very important for the Quality system. This ties the first 3 requirements together into one place.

This program is one of the most complex pieces of software that I have individually developed.  I was able to simplify the overall complexity by taking advantage of python's object-oriented programming language, and "chunking" the overall GUI into individual modules. This made upgrades and changes much easier.
### to here

#### Step 1: Data Structure:
The first step was to create an easily-indexable dictionary of the instruments within the laboratory. Each instrument was created as a new instance of a class called "Equipment", which allowed me to created an object with individual attributes for each HPLC, balance, and all of the other various equipment types.

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
The instances were added to a dictionary in the following format:
```python
instruments = {
   'E00001': <class obj: Equipment>,
   'E00002': <class obj: Equipment>,
   ...
}
```
Where they could be easily referenced by their offical equipment number (E#). Additional attributes, such as equipment type, serial numbers, and location within the building, could be optionally added to each instrument. Within the self.history attribute, an instance of each of the 4 possible maintenance types was created as the history() class, which contained the information of that instrument's maintenance history.
```python
class history:
    def __init__(self):
        self.date = None # the date of the maintenance in datetime.date format
        self.expiration = None # when the next maintenance item is due (datetime.date)
        self.analyst = None # string of who performed the maintenance
        self.number = None # each mainenance event is assigned a unique number.
```
The heart of the instrument maintenance history record information is contained here. The dictionary was serialized and pickled to a .pkl file, which was loaded every time the program loaded, and saved when the user chose, and at program exit. 

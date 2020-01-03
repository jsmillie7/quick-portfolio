<a href="index">
<img src="images/back.png" alt="Back" height="35" width="35">
</a>

## Instrument Maintenance Manager

##### for Windows
---
### Background:
Working in an analytical chemistry lab with an ISO certification, it's imperitive that instrument maintenance be performed and documented punctually. It was the responsibility of the Quality Assurance department to make sure that this was done, and I was tasked with developing a method to achieve this. Previously, several difference programs had to be used together to achieve the desired results, so the goal was to develop a user interface with Tkinter that allowed a user to achieve the following required goals:
* Send out monthly reminder emails for upcoming maintenance
    * this was a manual task which required the sender to create a list of upcoming equipment by scrolling through a log and adding each item, which was tedious and error-prone. Python can do this faster without error.
* Interface with a shared Microsoft Outlook calendar to sync maintenance events
    * this was also a manual task wich required each of the 250+ instruments to be added to an Outlook calendar. This process was easily done with python as well.
* Print out ISO-mandated maintenance stickers for instruments
    * each time a maintenance event occurred, a new sticker was required on the instrument. Manually, not a difficult task, but with Python and a networked label printer, this task became completely automated.
* Track and easily update individual instrument maintenance
    * This was previously not done, which led to some instruments missing a calibration date, so implementing this was very important for the Quality system. This ties the first 3 requirements together into one place.


This program is one of the most complex pieces of software that I have individually developed.  I was able to simplify the overall complexity by taking advantage of python's object-oriented programming language, and "chunking" the overall GUI into individual modules. This made upgrades and changes much easier.

The purpose of this software was to establish a method to track and maintain instrument maintenance records in an analytical chemistry laboratory to ensure compliance with ISO:17025's instrument maintenance requirements.

---
### Data Structure:

The heart of the data structure revolves around the "Equipment" class:


```python
class Equipment:
    def __init__(self):
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

Each instrument and its attributes were added to a new instance of this class, and added to a dictionary. This dictionary was serialized using the built-in Pickle module, which was written to a file and saved. 

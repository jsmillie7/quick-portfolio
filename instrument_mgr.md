<a href="index">
<img src="images/back.png" alt="Back" height="35" width="35">
</a>

## Instrument Maintenance Manager

##### for Windows
---
### Background:
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

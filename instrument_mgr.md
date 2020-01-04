<a href="index">
<img src="images/back.png" alt="Back" height="35" width="35">
</a>

## Instrument Maintenance Manager

##### for Windows

<img src="images/InstrumentMaintenance.PNG">

---
### Background:
Working in an analytical chemistry lab with an ISO certification, it's imperitive that instrument maintenance be performed and documented punctually. It was the responsibility of the Quality Assurance department to make sure that this was done, and I was tasked with developing a method to achieve this. Previously, several different programs had to be used together to achieve the desired results, so the goal was to develop software which could simplify the work. The following were the goals of this project:
* Create a [data structure](#step-1-data-structure) that allows for ease of access to data records, secure storage of records, as well as track and easily update individual instrument maintenance.
* Send out [monthly reminder emails](#step-2-monthly-email-reminder) for upcoming maintenance
* [Interface with a shared Microsoft Outlook calendar](#step-3-outlook-calendar-interface) to sync maintenance events
* Print out ISO-mandated [maintenance stickers](#step-4-equipment-maintenance-sticker-generation) for instruments

This program is one of the most complex pieces of software that I have individually developed. I was able to simplify the overall complexity by taking advantage of python's object-oriented programming language and modularizing the overall GUI into individual frames, which made upgrades and changes much easier. The entire code is long, messy, and a bit hard to read, so for this portfolio entry, I will go over some key functions implemented in the backend without going in depth about Tkinter and the rest of the front-end.

### Development:

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
With a solid data structure in hand, and the manual labor behind creating instances for each of the hundreds of pieces of equipment within the lab was complete, it was time to start automating some of the tasks that this software really aimed at simplifying. The first task I chose to tackle was simple: generate and send a monthly email reminder to a list of email addresses that contained all upcoming and overdue equipment items through a shared Outlook email account. With Outlook 2017 and Windows, this was a fairly straight-forward task. A class called Emailer was created to facilitate this event, which itself got pretty long in the code. The initialization of the class became:
```python
class Emailer:
    def __init__(self,instuments, send=False): 
        '''
        Arguments in __init__:
        instruments is a dictionary of instrument data made in step 1
        send is a boolean, true means send immediately, false means open the message and send manually
        
        The following sets up the email message
        '''
        self.send = send
        self.data = instruments
        self.date = datetime.date.today() + relativedelta(months=1, day=31) # end of next month
        self.subject = 'SUBJECT TEXT'
        self.get_overdue()
        self.email_body()
        self.emailer(Send=self.send)
        settings.last_email = str(datetime.datetime.today())
        settings.update()
```
Within init, several other definitions are called, the first being self.get_overdue(), which as its name implies, it searches the provided data for overdue items, and creates a dictionary sorted by duedate and type

```python
    def get_overdue(self):
        '''
        Returns self.overdue with the following data structure:
        self.overdue = {
            datetime.date:{
                object: [list of maintenance items due that duedate],
                object: [list of maintenance items due that duedate],
                },
            datetime.date:{
                object: [list of maintenance items due that duedate]
                }
            }
        '''
        
        self.overdue = {}
        for obj in self.data.values():
            if obj.in_service and not obj.hidden:
                for typ,d in obj.history.items():
                    date = d.expiration
                    if isinstance(date, datetime.date) and date <= self.date:
                        ### check if the date and type exist yet:
                        self.overdue.setdefault(datetime.date(date.year, date.month, 1),{})
                        self.overdue[datetime.date(date.year, date.month, 1)].setdefault(obj,[])
                        ### add item to overdue dict
                        self.overdue[datetime.date(date.year, date.month, 1)][obj].append(typ)
        self.overdue = dict(sorted(self.overdue.items())) #organize dict chronologically
```
Next, init calls self.email_body(), which creates the body text that will be displayed in the email by iterating through each overdue item and listing its attributes and due dates. It returns a string called self.body that will be used for the body text in the next function.


Finally, init calls self.emailer(), which constructs the email in Outlook and either sends it or opens the composition depending on your choice of the attribute 'Send'. It takes advantage of the windows-only module [win32com.client](https://pypi.org/project/pywin32/), and works effortlessly. 
```python
    def emailer(self):
        outlook = win32.Dispatch('outlook.application')
        mail = outlook.CreateItem(0)
        mail.To = settings.email_contacts # string of email contacts saved in settings file
        mail.CC = '' # optional email CC contact
        mail.Subject = self.subject
        mail.HtmlBody = self.body
        mail.Importance = 2
        if self.send == True:
            mail.send
        else:
            mail.Display(True)
```
The rest of the functions called in init are used to update usage information within the settings file, which is beyond the scope of this. 

#### Step 3: Outlook calendar interface
Another important feature of this software was the ability to add maintenance due dates to an Outlook calendar for the entire laboratory to reference. Again using the win32com.client module, a new class called Calendar was devised:

```python
class Calendar:
    def __init__(self):
        self()
    
    def __call__(self):
        self.read_cal()
        self.parse_cal()
        
    def read_cal(self):
        self.outlook = win32com.client.Dispatch("Outlook.Application")
        self.namespace = self.outlook.GetNamespace("MAPI")
        self.recipient = self.namespace.Folders(settings.calendar_account)
        self.sharedCalendar = self.recipient.Folders('Calendar')
        self.appointments = self.sharedCalendar.Items
   
    def parse_cal(self):
        self.parsed_cal = {}
        for apt in self.appointments:
            m_type, e_num = apt.Subject.split(' - ')
            self.parsed_cal.setdefault(e_num,{})
            self.parsed_cal[e_num][m_type] = apt
```
Any time an instance of this class is created or called, 2 functions run: self.read_cal() opens the Outlook calendar and reads the appointment items from it. self.parse_cal() creates a dictionary of all appointments by equipment number. This dictionary can thus be live updated whenver an item is added or removed from the calendar. 

I added several ancillary functions to the class to perform various actions, the most important of which allowed adding and deleting appointments to the calendar. With this, the calendar can be built and maintained with just a few clicks.
```python    
    def add_appointment(self, obj, m_type):
        new_apt = self.sharedCalendar.Items.Add(1)
        new_apt.Start = str(obj.history[m_type].expiration)
        new_apt.Subject = 'SUBJECT'
        new_apt.Body = 'BODY TEXT'
        new_apt.AllDayEvent = 1
        new_apt.Importance = 2
        new_apt.BusyStatus = 0
        new_apt.ReminderOverrideDefault = True
        new_apt.ForceUpdateToAllAttendees = 1
        new_apt.ReminderSet = True
        new_apt.RequiredAttendees = 'EMAIL ADDRESSES' # email addresses of required contacts 
        new_apt.ResponseRequested = 0
        new_apt.Location = obj.location
        new_apt.ReminderMinutesBeforeStart = 1440
        new_apt.MeetingStatus = 1
        new_apt.Send()
        new_apt.Save()
        self()
    
    def delete_appointment(self, e_num, m_type):
        event = self.parsed_cal[e_num][m_type]
        event.Delete()
        self()
```
Ultimately, the Calendar is built from the Equipment.history[xx].expiration class variables. Keeping two points of reference allows for rebuilding the calendar if something happens to the public Outlook calendar.

#### Step 4: Equipment Maintenance Sticker Generation
The last task I am going to highlight from this software development was to find a way to generate a few different types of stickers that could be formatted to fit on a 4"x2" label that could be automatically printed from our networked label printer. Maintenance stickers were historically handwritten by members of QA with varying levels of legibility and accuracy. Since accurate maintenance information is required to be on the instruments per ISO 17025:2017, an incorrect label could jeopardize the laboratory accredation. Automating the label creation would eliminate any human-error. Using the pypi module [ReportLab](https://pypi.org/project/reportlab/), the program generats a pdf sticker.

<p align="center">
    <img src="images/sticker.png" height="150" width="295" title="Example of a Python Generated Maintenance Sticker">
</p>
A class named Sticker is used to generate the stickers dynamically. The arguments required to create a Sticker object are the Equipment class object and a filename, which was normally left to its default filename.

```python
class Sticker:
    def __init__(self, obj, filename='temp.pdf'):
        self.height = 144
        self.width = 288
        self.offset = 10
        self.obj = obj
        self.font = 'Helvetica'
        self.filename = filename
        self.c = canvas.Canvas(self.filename, pagesize=(self.width,self.height))
        
        # run class functions functions
        self.extract()
        self.outline()
        self.labels()
        self.fill_info()
        self.c.save()
```
Several class functions are called to format the sticker. First, self.extract() reads the data stored in self.obj, and assigns the values to variables to be added to the sticker:
```python
    def extract(self):
        # Extract needed info from self.obj
        self.PM = self.obj.history['Annual PM'].number
        self.PM_date = self.obj.history['Annual PM'].date
        self.PM_expiration = self.obj.history['Annual PM'].expiration
        self.biannual_due = self.obj.history['Biannual PM'].expiration
        self.OQ = self.obj.history['OQ'].number
        self.OQ_date = self.obj.history['OQ'].date
        self.OQ_expiration = self.obj.history['OQ'].expiration
        self.e_number = self.obj.equipment_number
        self.completed_by = self.obj.history['Annual PM'].analyst
```
Next, self.outline() draws all of the lines one the sticker. self.labels() draws all of the non-dynamic text that is constant on each sticker.
```python
    def outline(self):
        #Formats the lines on the label
        self.c.setLineWidth(1)
        self.c.line(0,int(self.height*0.85),self.width,int(self.height*0.85))
        self.c.line(0,int(self.height*0.85)-2,self.width//2-1,int(self.height*0.85)-2)
        self.c.line(self.width//2+1,int(self.height*0.85)-2,self.width,int(self.height*0.85)-2)
        self.c.line(self.width//2-1,0, self.width//2-1, int(self.height*0.85)-2)
        self.c.line(self.width//2+1,int(self.height*0.20)+2, self.width//2+1, int(self.height*0.85)-2)
        self.c.line(self.width//2+1,int(self.height*0.20), self.width, int(self.height*0.20))
        self.c.line(self.width//2+1,int(self.height*0.20)+2, self.width, int(self.height*0.20)+2)
        self.c.line(self.width//2+1,0, self.width//2+1, int(self.height*0.20))
        
    def labels(self):
        # Formats non-dynamic text on label
        self.c.setFont(self.font, 20)
        self.c.drawString(self.offset,self.height-18,'Annual Maintenance')
        self.c.setFont('Helvetica', 8)
        self.c.drawString(self.offset,80,'PM Completion Date:')
        self.c.drawString(self.offset,50,'Annual PM Expiration:')
        self.c.drawString(self.offset,20,'Biannual PM Due:')
        self.c.drawString(self.width//2 + self.offset,80,'OQ Completion Date:')
        self.c.drawString(self.width//2 + self.offset,50,'OQ Expiration:')
        self.c.drawString(self.width//2 + self.offset,20,'Performed By:')
```
Finally, the program will fill in the dynamic information with self.fill_info() and self.formatter('xxx') is used to format the datetime.date variables and strings into the desired format. 
```python
    def fill_info(self):
        self.c.setFont(self.font, 30)
        if self.PM is None:
            self.PM = 'PM- N/A'
        self.c.drawString(self.offset,92,self.PM)
        if self.OQ is None:
            self.OQ = 'OQ- N/A'
        self.c.drawString(self.width//2 + self.offset,92,self.OQ)
        self.c.setFont(self.font, 16)
        self.c.drawString(self.offset,65,self.formatter(self.PM_date))
        self.c.drawString(self.offset,35,self.formatter(self.PM_expiration))
        self.c.drawString(self.offset,5,self.formatter(self.biannual_due))
        self.c.drawString(self.width//2 + self.offset,65,self.formatter(self.OQ_date))
        self.c.drawString(self.width//2 + self.offset,35,self.formatter(self.OQ_expiration))
        self.c.drawString(self.width//2 + self.offset,5,self.formatter(self.completed_by))
        if self.e_number is not None:
            self.c.setFont(self.font, 14)
            self.c.drawString(4*self.width/5,self.height-18,self.e_number)
            
    def formatter(self, value):
        if isinstance(value, datetime.date):
            return '{:02d}-{}-{}'.format(value.day,calendar.month_abbr[value.month],str(value.year))#[2:])
        elif isinstance(value,str):
            if len(value)>16:
                size = 12
            else:
                size = 16
            self.c.setFont(self.font, size)
            return value
        else:
            return 'N/A'
```
And with the addition of this script, the program has achieved the 4 goals set out for it. The software includes several more features, like the ability to add and delete instruments, place them in or out of service, update instrument information, generate logs of specific equipment (overdue, all HPLCs, every item in a specific lab area, etc.), and many more, but this entry is long enough as it is. 

[return to top](#instrument-maintenance-manager)

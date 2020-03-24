<a href="/index">
<img src="/images/back.png" alt="Back" height="35" width="35">
</a>

## Modeling the Earth

##### Jupyter Notebook
---
### Background:

<p align="center">
  <img src="images/header.png" width="100%">
</p>


Growing up in Colorado, the mountains are a way of life. I spent many of my formative years in and around Steamboat Springs, which is home to world-class skiing at Steamboat Resort. As an adult, the mountains are just a daydream on the horizon most days. I have always been fascinated by cartography and topography, but I had never applied what I knew about maps to any situation outside of real-world navigation. I wanted to find a way to combine my computer science and mathematics experience with real-world data, and thus, an idea was born: I wanted to make a physical representation of one of my favorite places on Earth utilizing just the resources I had at hand: an internet connection, python and a [CNC laser cutter](/laser). 

---

### Resource Acquisition:

#### Outlining the Area

The first step was to get a polygon of the coordinates of the desired area. The easiest way to do this was to use Google Earth Pro to draw a path and export it as a .KMZ file. The resulting path for Mt. Werner can be seen below.

<p align="center">
  <img src="images/path.png" width="100%">
</p>


#### Acquiring the Elevation Data

The US Geological Survey is an excellent resource for free Earth-relevant data. Using [EarthExplorer](https://earthexplorer.usgs.gov), I navigated to Mt. Werner data sets, chose the Digital Elevation branch, chose SRTM 1 Arc-Second Global data, and downloaded the result as a [GeoTIFF file](files/n40_w107_1arc_v3.tif).

<p align="center">
  <img src="images/usgs.png" width="100%">
</p>

---

### Data Processing:

#### Parsing the KMZ file

A KMZ file is a special zipped XML file that contains all of the data surrounding the path, and most importantly for this project, a list of coordinates for each corner of the outlined polygon. [This](http://programmingadvent.blogspot.com/2013/06/kmzkml-file-parsing-with-python.html) source was a big help in figuring out how to get the coordinate data out of the KMZ file. I used the PlacemarkHandler class from this source verbatin, since the tedious work was already done. I created another class called KMZ to wrap all of the actions surrounding the KMZ data into one easy to use package:


```python
class KMZ:
    def __init__(self, kmz_file):
        self.file = kmz_file

        with ZipFile(self.file, 'r') as kmz:
            self.kml = kmz.open('doc.kml', 'r')
            parser = xml.sax.make_parser()
            self.handler = PlacemarkHandler()
            parser.setContentHandler(self.handler)
            parser.parse(self.kml)
        
        # build geofence coordinates
        self.get_coords()
        
        ### The name of the GeoTIFF file that include the desired area:
        self.filename = self.get_filename()
            
    def get_coords(self):
        ### This function builds the geofence polygon from the KMZ file data
        self.fence = [coord(lat=float(coordinate.split(',')[1]), lon=float(coordinate.split(',')[0])) \
                      for coordinate in self.handler.mapping[list(self.handler.mapping)[0]] \
                      ['coordinates'].split()]
        
        ### Prevent the closed loop divide by zero error
        if self.fence[0] == self.fence[-1]:
            del self.fence[-1]
            
    def get_filename(self):
        ### This function will find the name of the GeoTIFF file that contains the required data
        ### using the file naming convention that the USGS defaults to
        ### i.e. 'n40_w107_1arc_v3.tif'
        ### This only works in Northern and Western hemispheres
        
        fname = f'n{math.floor(self.fence[0].lat)}_w{abs(math.floor(self.fence[0].lon))}_1arc_v3.tif'
        if path.exists(fname):
            return fname
        else:
            print(f'File not found. Download {fname} from USGS EarthExplorer!')
            return None
        
    def plot_fence(self):
        ### A quick function to plot the geofence coordinates
        ### Mostly for debugging
        
        plt.plot([i.lon for i in self.fence], [i.lat for i in self.fence])
        plt.show()
```



---

### Results

Running the new g-code file on my laser cutter, I was pleasantly surprised with the results: a 90 layer sphere made out just math and paper!

<p align="center">
  <img src="result.png" width="100%">
</p>

This program is remarkably useful for simple 3D objects! The [Jupyter Notebook](Cura2Laser.ipynb) is available here.

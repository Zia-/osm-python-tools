[back](../../../)

# Data Tools

The module contains tools for easily collecting, mining, and drawing data from OSM. It is meant to be used in combination with the other modules.

Imagine the following example: we try to understand how the number of roads has developed over time in different towns. We are not only interested in the number of roads in general, but also to different kind of roads. We first fomulate different "dimensions", for example, the temporal dimension, the dimension of different towns, and the dimension of different roads (this example is [part of the repository](https://github.com/mocnik-science/osm-python-tools/blob/master/examples/example.py)):
```python
from OSMPythonTools.nominatim import Nominatim
from OSMPythonTools.overpass import Overpass, overpassQueryBuilder
from OSMPythonTools.data import Data, dictRangeYears, ALL

from collections import OrderedDict

dimensions = OrderedDict([
  ('year', dictRangeYears(2013, 2017.5, 1)),
  ('town', OrderedDict({
    'heidelberg', 'Heidelberg, Germany',
    'manhattan': 'Manhattan, New York',
    'vienna': 'Vienna, Austria',
  })),
  ('typeOfRoad', OrderedDict({
    'primary': 'primary',
    'secondary': 'secondary',
    'tertiary': 'tertiary',
  })),
])
```
Having defined the requirements for the dimensions, we can now mine the data (mining the data takes a while due to limited resources of the endpoint):
```python
nominatim = Nominatim()
overpass = Overpass()

def fetch(year, town, typeOfRoad):
    areaId = nominatim.query(town).getAreaId()
    query = overpassQueryBuilder(area=areaId, elementType='way', selector='"highway"="' + typeOfRoad + '"', out='count')
    return overpass.query(query, date=year, timeout=60).countElements()

data = Data(fetch, dimensions)
```
The object `data` contains the results of the queries and can be imagined as being a table (this is similar to the package [xarray](http://xarray.pydata.org), which is also internally used):
```
                              value
town       typeOfRoad year
heidelberg primary    2013.0    281
                      2014.0    403
                      ...
           secondary  2013.0    282
                      2014.0    371
                      ...
           tertiary   2013.0    229
                      2014.0    285
                      ...
manhattan  primary    2013.0    346
                      2014.0    407
                      ...
vienna     primary    2013.0    998
                      2014.0   1284
                      ...
```
As it has a representation as a string, it can even be printed in the interactive python interpreter in a human readable way. The data can also be restricted to one value for some dimension, for example, to the town Vienna:
```python
data.select(town='vienna')
```
This yields a table only containing the values for Vienna:
```
                   value
typeOfRoad year
primary    2013.0    998
           2014.0   1284
           2015.0   1462
           2016.0   1619
           2017.0   1719
secondary  2013.0   1381
           2014.0   1560
           2015.0   1713
           2016.0   1892
           2017.0   2022
tertiary   2013.0   1235
           2014.0   1420
           2015.0   1540
           2016.0   1706
           2017.0   1802
```
Also the number of secondary roads in Vienna in 2014 can be accessed:
```python
data.select(town='vienna', year=2014, typeOfRoad='secondary')
# 1560
```
Instead of providing explicit values for a dimension, we can also provide the value `ALL`. The corresponding dimension is not restricted, but the values are relocated to columns:
```python
data.select(typeOfRoad=ALL, town='vienna')
```
This yields the following table:
```
        primary  secondary  tertiary
year
2013.0      998       1381      1235
2014.0     1284       1560      1420
2015.0     1462       1713      1540
2016.0     1619       1892      1706
2017.0     1719       2022      1802
```
Assume that only primary and secondary roads are of interest. In this case, we write:
```python
data.select(typeOfRoad=['primary', 'secondary'], town='vienna')
```
This yields the following table:
```
        primary  secondary
year
2013.0      998       1381
2014.0     1284       1560
2015.0     1462       1713
2016.0     1619       1892
2017.0     1719       2022
```
The data can be analyzed (number of values, mean value, standard derivation, etc.):
```python
data.describe(typeOfRoad=ALL, town='vienna')
```
This yields:
```
           primary    secondary     tertiary
count     5.000000     5.000000     5.000000
mean   1416.400000  1713.600000  1540.600000
std     286.042479   255.515753   225.623137
min     998.000000  1381.000000  1235.000000
25%    1284.000000  1560.000000  1420.000000
50%    1462.000000  1713.000000  1540.000000
75%    1619.000000  1892.000000  1706.000000
max    1719.000000  2022.000000  1802.000000
```

Instead of computing table representations, the data can also be plotted by using the same syntax to restrict the data:
```python
data.plot(town='manhattan', typeOfRoad=ALL)
```
![data.plot(town='manhattan', typeOfRoad=ALL)](https://github.com/mocnik-science/osm-python-tools/blob/master/examples/plot-manhattan.png)

Also the primary roads from different towns can be compared:
```python
data.plot(town=ALL, typeOfRoad='primary')
```
![data.plot(town=ALL, typeOfRoad='primary')](https://github.com/mocnik-science/osm-python-tools/blob/master/examples/plot-primary.png)

The rows correspond to the x axis, and the columsn to the y axis. It is thus important to restrict the number of row dimensions until only one row dimension is left. The rows should only contain numerical values, when being plotted. If the values are not numerical, a bar plot can be used:
```python
data.plotBar(town='manhattan', year=ALL)
```
![data.plotBar(town='manhattan', year=ALL)](https://github.com/mocnik-science/osm-python-tools/blob/master/examples/plotbar-manhattan.png)

When two values shall be compared, a scatter plot can be used. The following plot compares the number of primary roads in Vienna (x axis) to the number of primary roads in Manhattan (y axis):
```python
data.plotScatter('vienna', 'manhattan', town=['vienna', 'manhattan'], typeOfRoad='primary')
```
![data.plotScatter('vienna', 'manhattan', town=['vienna', 'manhattan'], typeOfRoad='primary')](https://github.com/mocnik-science/osm-python-tools/blob/master/examples/plotscatter-primary.png)

When the plots (`plot`, `plotBar`, and `plotScatter`) are generated, they are (on most systems) shown in a graphical user interface. If the parameter `filename` is added, the data is instead saved to the corresponding file:
```python
data.plot(town='manhattan', typeOfRoad=ALL, filename='manhattan.pdf')
```

The data is encapsulated inside an object. It can, however, be accessed in different formats:
```python
data.getDataFrame()    # as a pandas DataFrame
data.getDataset()      # as a xarray Dataset
data.getDict()         # as a python dictionary
data.getCSV()          # as comma separated values
data.excelClipboard()  # Excel format, copied to clipboard
```
Information about the packages [xarray](http://xarray.pydata.org) and [pandas](http://pandas.pydata.org) can be found on their websites.

The following commands are not documented:
* `drop`: drop a row
* `apply`: apply a function to the data
* `toColumn`: apply `select` and produce a column from the resulting data
* `renameColumns`: rename a column
* `selectColumns`: select a number of columns
* `showPlot`: different plot can be combined (by using `showPlot=False`); the function `showPlot` is then called to show the plot.
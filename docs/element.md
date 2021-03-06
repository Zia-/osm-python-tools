[back to readme](../../../)

# Element

Several services, amongst them the [OSMPythonTools.**api**](api.md) and [OSMPythonTools.**overpass**](overpass.md) return OSM objects, which are represented by an instance of the class `Element`, which provides functions to access its properties.

## Accessing general properties

Consider the following example of a query to the OSM Api:
```python
from OSMPythonTools.api import Api
api = Api()
busStop = api.query('node/42467507')
```
The result, that is, `busStop`, is an element, whose properties can easily be access as follows:
```python
busStop.id()
# 42467507
busStop.type()
# 'node'
busStop.lat()
# 40.701424
busStop.lon()
# -73.943064
```

## Accessing tags

The tags can be accessed as a python dictionary:
```python
busStop.tags()
# {'asset_ref': '2030', 'highway': 'bus_stop', 'location': 'Broadway / Thornton Street', 'name': 'Broadway / Thornton Street', 'route_ref': 'B46;B46-LTD'}
```
The value of a single tag can be accessed like follows:
```python
busStop.tag('highway')
# 'bus_stop'
```

## Accessing nodes and members

Ways incorporate nodes, and relations other elements, called members. Nodes of a way can easily be accessed:
```python
way = api.query('way/108402486')
way.nodes()
# [<OSMPythonTools.api.ApiResult object at 0x104c06940>, <OSMPythonTools.api.ApiResult object at 0x104c2d8d0>, ...
way.nodes()[0].id()
# 1243967857
```
The members of a relation can also easily be accessed:
```python
relation = api.query('relation/1539714')
relation.members()
# [<OSMPythonTools.api.ApiResult object at 0x104c20240>, <OSMPythonTools.api.ApiResult object at 0x104c2db38>, ...
relation.members()[0].id()
# 108402486
```

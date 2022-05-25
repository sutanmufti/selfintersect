# self intersect
jupyter notebook that explain self intersecting line algorithm. By Sutan Mufti (@urban_planning)

---



 1. read the file

```python
import geopandas as gpd

# before
gdf = gpd.read_file('selfintersects.geojson')
gdf.plot()
```

 2. let's check the endpoints

```python
def get_endpoints(gdf):
    from shapely.geometry import Point
    startpoint = gdf.geometry.apply(lambda x: x.coords[0])
    endpoint = gdf.geometry.apply(lambda x: x.coords[-1])

    startpoints = [Point(i) for i in startpoint]
    endpoints = [Point(i) for i in endpoint]

    return startpoints, endpoints

def create_endpoints(startpoints, endpoints):
    geom = []
    for a,b in zip(startpoints, endpoints):
        from shapely.geometry import Point
        geom.append(a)
        geom.append(b)

    endpoints = gpd.GeoDataFrame({'id': range(0, len(geom))}, crs=gdf.crs, geometry=geom)
    return endpoints

startpoints, endpoints = get_endpoints(gdf)
endpoints = create_endpoints(startpoints, endpoints)
import matplotlib.pyplot as plt

fig, ax = plt.subplots()
gdf.plot(ax=ax)
endpoints.plot(ax=ax)

```

 3. union it to merge all lines into one geometry. **Note**: unary_union will take time if your data is large!

```python
union_geom = gdf.unary_union
union = gpd.GeoDataFrame({'id':[0]}, crs=gdf.crs, geometry=[gdf.unary_union])
union.plot()
```

 4. and then explode it!
```python
from shapely.ops import linemerge

lm = gpd.GeoDataFrame({'id':[0]}, crs=gdf.crs, geometry=[linemerge(union_geom)]).explode().reset_index(drop=True)
lm.plot()
```

 5. let's check the endpoint of the exploded union.
```python
startpoints, endpoints = get_endpoints(lm)
endpoints = create_endpoints(startpoints, endpoints)

# cleansing with snap
from shapely.ops import snap
endpoints['geometry'] = endpoints.geometry.apply(lambda x: snap(x, union_geom, 0.00001))

fig, ax = plt.subplots()
gdf.plot(ax=ax)
endpoints.plot(ax=ax)
```

 6. filter out the dangles

```python
sjoin = endpoints.sjoin(gdf, how='left')

fig, ax = plt.subplots()
gdf.plot(ax=ax)
sjoin[sjoin['index_right'].isna()].plot(ax=ax)

```



There you go! now we have the points.

---

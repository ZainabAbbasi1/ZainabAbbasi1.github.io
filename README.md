```python
# imports the various library for the lab
import numpy as np
import pandas as pd
import matplotlib
import matplotlib.pyplot as plt
import osmnx as ox # this line imports osmnx
import networkx as nx # this line imports networkx
import matplotlib.cm as cm
import matplotlib.colors as colors
#from IPython.display import IFrame
#ox.config(log_console=True, use_cache=True)

import sys
print (f'current environment: {sys.prefix}')

if ox.__version__=='2.0.1':
    #prints OSMNx version 
    print (f'current osmnx version: {ox.__version__}') 
else:
    #recommends student to upgrade to newer osmnx version.
    print (f'current osmnx version: {ox.__version__}. student might need to upgrade to osmnx=2.0.1 for the notebook to work')
```

    current environment: /opt/anaconda3/envs/envGEOG0051
    current osmnx version: 2.0.1



```python
# Select a neighbourhood of London you know well and get the graph in osmnx with a radius of 2000m

G=ox.graph_from_address('Walthamstow, London', dist=2000)

#This retrieves a street network as a MultiDiGraph (a directed graph that allows multiple edges between nodes) using OSMnx.
```


```python

```


```python
# This converts G (a MultiDiGraph) into a DiGraph (a directed graph without multiple edges between nodes).
DG = ox.convert.to_digraph(G)
```


```python
# Run a street network analysis for a neighbourhood in London: closeness centrality

edge_bc = nx.closeness_centrality(nx.line_graph(DG))
nx.set_edge_attributes(DG, edge_bc,'cc')
```


```python
edge_dc = nx.degree_centrality(nx.line_graph(DG))
nx.set_edge_attributes(DG, edge_dc,'dc')

```


```python
#Run a street network analysis for a neighbourhood in London: betweeness centrality - taking too long to run

#edge_bc = nx.betweenness_centrality(nx.line_graph(DG))
#nx.set_edge_attributes(DG, edge_bc,'bc')


```


```python
#turn it into a multi-graph
G1 = nx.MultiGraph(DG)
```


```python
# convert graph to geopandas dataframe
gdf_edges = ox.graph_to_gdfs(G1,nodes=False,fill_edge_geometry=True)

# set crs to 3857 (needed for contextily)
gdf_edges = gdf_edges.to_crs(epsg=3857) # setting crs to 3857

# plot edges according to degree centrality
ax=gdf_edges.plot('cc',cmap='plasma',figsize=(10,10),legend=True,
                     legend_kwds={'label': "Closeness Centrality", 'orientation': "vertical"})

# add a basemap using contextilly
import contextily as ctx
ctx.add_basemap(ax,source=ctx.providers.CartoDB.Positron)
plt.axis('off')
plt.show()
```


    
![png](formative_files/formative_8_0.png)
    



```python
# convert graph to geopandas dataframe
gdf_edges = ox.graph_to_gdfs(G1,nodes=False,fill_edge_geometry=True)

# set crs to 3857 (needed for contextily)
gdf_edges = gdf_edges.to_crs(epsg=3857) # setting crs to 3857

# plot edges according to degree centrality
ax=gdf_edges.plot('dc',cmap='plasma',figsize=(10,10),legend=True,
                     legend_kwds={'label': "Degree Centrality", 'orientation': "vertical"})

# add a basemap using contextilly
import contextily as ctx
ctx.add_basemap(ax,source=ctx.providers.CartoDB.Positron)
plt.axis('off')
plt.show()
```


    
![png](formative_files/formative_9_0.png)
    



```python
#Run a street network analysis for a neighbourhood in London: land use
```


```python
ox.io.save_graph_geopackage(G)
```


```python
# you can get the geometries of a place similar to getting a graph 
tags= tags={'amenity': True, 'highway':True, 'landuse':True, 'building':True, 'waterway': True, 'railway': True}
all_geom=ox.features_from_address('Walthamstow, London', tags, dist=1000)
all_geom = all_geom.to_crs(epsg=3857)
```


```python
fig,ax = plt.subplots(figsize=(10,10))
all_geom[all_geom['landuse'].notna()].plot(ax=ax,color='black')
import contextily as ctx
ctx.add_basemap(ax,source=ctx.providers.CartoDB.Positron)
plt.axis('off')
plt.show()
```


    
![png](formative_files/formative_13_0.png)
    



```python
fig,ax = plt.subplots(figsize=(10,10))
all_geom[all_geom['building'].notna()].plot('building',
                                            ax=ax,
                                            categorical=True,
                                            legend=True)
import contextily as ctx
ctx.add_basemap(ax,source=ctx.providers.CartoDB.Positron)
plt.axis('off')
plt.show()


```


    
![png](formative_files/formative_14_0.png)
    



```python
fig, ax = plt.subplots(figsize=(10, 10))

# Filter only university buildings
universities = all_geom[(all_geom['building'].notna()) & (all_geom['building'].str.contains('retail', case=False, na=False))]

# Plot universities
universities.plot(ax=ax, color='red', edgecolor='black', alpha=0.7, legend=True)

# Add basemap
ctx.add_basemap(ax, source=ctx.providers.CartoDB.Positron)

plt.axis('off')
plt.title("University Buildings")
plt.show()
```


    
![png](formative_files/formative_15_0.png)
    



```python
#Run a street network analysis for a neighbourhood in London: shortest path analysis

```


```python
walthamstow = ox.geocode("Walthamstow, London")

print(walthamstow)
```

    (51.5815237, -0.0237594)



```python
tottenham = ox.geocode("Tottenham, London")

print(tottenham)
```

    (51.5881223, -0.0599366)



```python

origin_point = [51.5815237, -0.0237594] #trafalgar square
destination_point = [51.5881223,-0.0599366] #covent garden
origin_node = ox.nearest_nodes(G, origin_point[1],origin_point[0])
destination_node = ox.nearest_nodes(G, destination_point[1],destination_point[0])
origin_node, destination_node 



# find the shortest path between origin and destination nodes
route = nx.shortest_path(G, origin_node, destination_node, weight='length')
str(route)

# plot the route showing origin/destination lat-long points in red
fig,ax = ox.plot_graph_route(G, route )
```


    
![png](formative_files/formative_19_0.png)
    



```python
# Load street network
place = "Walthamstow, London"
G = ox.graph_from_place(place, network_type="walk")  # or 'drive' for cars

# Convert to DiGraph for centrality measures
G = ox.convert.to_digraph(G)

# Compute centrality (e.g., betweenness)
betweenness = nx.betweenness_centrality(G, weight="length")
nx.set_edge_attributes(G, betweenness, "betweenness")

# Convert graph to GeoDataFrame
gdf_edges = ox.graph_to_gdfs(G, nodes=False)

# Load retail locations (Example: CSV with latitude & longitude)
gdf_retail = gpd.read_file("retail_locations.geojson")  # Replace with actual file

# Convert CRS to match street network
gdf_retail = gdf_retail.to_crs(gdf_edges.crs)

# Plot results
fig, ax = plt.subplots(figsize=(10, 10))
gdf_edges.plot(column="betweenness", cmap="plasma", ax=ax, linewidth=1, legend=True)
gdf_retail.plot(ax=ax, color="red", markersize=10, label="Retail Locations")
plt.legend()
plt.show()
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    File /opt/anaconda3/envs/envGEOG0051/lib/python3.13/site-packages/osmnx/geocoder.py:173, in _geocode_query_to_gdf(query, which_result, by_osmid)
        172 try:
    --> 173     result = _get_first_polygon(results)
        174 except TypeError as e:


    File /opt/anaconda3/envs/envGEOG0051/lib/python3.13/site-packages/osmnx/geocoder.py:239, in _get_first_polygon(results)
        238 # if we never found a polygon, raise an error
    --> 239 raise TypeError


    TypeError: 

    
    The above exception was the direct cause of the following exception:


    TypeError                                 Traceback (most recent call last)

    Cell In[41], line 3
          1 # Load street network
          2 place = "Walthamstow, London"
    ----> 3 G = ox.graph_from_place(place, network_type="walk")  # or 'drive' for cars
          5 # Convert to DiGraph for centrality measures
          6 G = ox.convert.to_digraph(G)


    File /opt/anaconda3/envs/envGEOG0051/lib/python3.13/site-packages/osmnx/graph.py:386, in graph_from_place(query, network_type, simplify, retain_all, truncate_by_edge, which_result, custom_filter)
        318 """
        319 Download and create a graph within the boundaries of some place(s).
        320 
       (...)
        383 documentation for caveats.
        384 """
        385 # extract the geometry from the GeoDataFrame to use in query
    --> 386 polygon = geocoder.geocode_to_gdf(query, which_result=which_result).union_all()
        387 msg = "Constructed place geometry polygon(s) to query Overpass"
        388 utils.log(msg, level=lg.INFO)


    File /opt/anaconda3/envs/envGEOG0051/lib/python3.13/site-packages/osmnx/geocoder.py:125, in geocode_to_gdf(query, which_result, by_osmid)
        123 # geocode each query, concat as GeoDataFrame rows, then set the CRS
        124 results = (_geocode_query_to_gdf(q, wr, by_osmid) for q, wr in zip(q_list, wr_list))
    --> 125 gdf = pd.concat(results, ignore_index=True).set_crs(settings.default_crs)
        127 msg = f"Created GeoDataFrame with {len(gdf)} rows from {len(q_list)} queries"
        128 utils.log(msg, level=lg.INFO)


    File /opt/anaconda3/envs/envGEOG0051/lib/python3.13/site-packages/pandas/core/reshape/concat.py:382, in concat(objs, axis, join, ignore_index, keys, levels, names, verify_integrity, sort, copy)
        379 elif copy and using_copy_on_write():
        380     copy = False
    --> 382 op = _Concatenator(
        383     objs,
        384     axis=axis,
        385     ignore_index=ignore_index,
        386     join=join,
        387     keys=keys,
        388     levels=levels,
        389     names=names,
        390     verify_integrity=verify_integrity,
        391     copy=copy,
        392     sort=sort,
        393 )
        395 return op.get_result()


    File /opt/anaconda3/envs/envGEOG0051/lib/python3.13/site-packages/pandas/core/reshape/concat.py:445, in _Concatenator.__init__(self, objs, axis, join, keys, levels, names, ignore_index, verify_integrity, copy, sort)
        442 self.verify_integrity = verify_integrity
        443 self.copy = copy
    --> 445 objs, keys = self._clean_keys_and_objs(objs, keys)
        447 # figure out what our result ndim is going to be
        448 ndims = self._get_ndims(objs)


    File /opt/anaconda3/envs/envGEOG0051/lib/python3.13/site-packages/pandas/core/reshape/concat.py:504, in _Concatenator._clean_keys_and_objs(self, objs, keys)
        502     objs_list = [objs[k] for k in keys]
        503 else:
    --> 504     objs_list = list(objs)
        506 if len(objs_list) == 0:
        507     raise ValueError("No objects to concatenate")


    File /opt/anaconda3/envs/envGEOG0051/lib/python3.13/site-packages/osmnx/geocoder.py:124, in <genexpr>(.0)
        121     raise ValueError(msg)
        123 # geocode each query, concat as GeoDataFrame rows, then set the CRS
    --> 124 results = (_geocode_query_to_gdf(q, wr, by_osmid) for q, wr in zip(q_list, wr_list))
        125 gdf = pd.concat(results, ignore_index=True).set_crs(settings.default_crs)
        127 msg = f"Created GeoDataFrame with {len(gdf)} rows from {len(q_list)} queries"


    File /opt/anaconda3/envs/envGEOG0051/lib/python3.13/site-packages/osmnx/geocoder.py:176, in _geocode_query_to_gdf(query, which_result, by_osmid)
        174     except TypeError as e:
        175         msg = f"Nominatim did not geocode query {query!r} to a geometry of type (Multi)Polygon."
    --> 176         raise TypeError(msg) from e
        178 elif len(results) >= which_result:
        179     # else, if we got at least which_result results, choose that one
        180     result = results[which_result - 1]


    TypeError: Nominatim did not geocode query 'Walthamstow, London' to a geometry of type (Multi)Polygon.


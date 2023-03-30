![](https://i.imgur.com/iywjz8s.png)


# Collaborative Document - Day 1

2023-03-30 Introduction to Geospatial Raster and Vector Data with Python


Welcome to The Workshop Collaborative Document.

This Document is synchronized as you type, so that everyone viewing this page sees the same text. This allows you to collaborate seamlessly on documents.

----------------------------------------------------------------------------


This is the Document for today: [link](https://tinyurl.com/2023-03-30-geospatial-python)

Collaborative Document day 1: [link](https://tinyurl.com/2023-03-30-geospatial-python)

Collaborative Document day 2: [link](https://tinyurl.com/2023-03-31-geospatial-python)


## 👮Code of Conduct

Participants are expected to follow these guidelines:
* Use welcoming and inclusive language.
* Be respectful of different viewpoints and experiences.
* Gracefully accept constructive criticism.
* Focus on what is best for the community.
* Show courtesy and respect towards other community members.
 
## 🎓 Certificate of attendance

If you attend the full workshop you can request a certificate of attendance by emailing to s.girgin@utwente.nl .

## ⚖️ License

All content is publicly available under the Creative Commons Attribution License: [creativecommons.org/licenses/by/4.0/](https://creativecommons.org/licenses/by/4.0/).

## 🙋Getting help

To ask a question, just raise your hand.

If you need help from a helper, place a pink post-it note on your laptop lid. A helper will come to assist you as soon as possible.

## 🖥 Workshop website

[link](https://www.itc.nl/research/research-facilities/labs-resources/itc-big-geodata/training/introduction-to-geospatial-raster-and-vector-data-with-python/)

## 🛠 Setup

🐍 Python & Environment: 
* Local setup: [link](https://carpentries-incubator.github.io/geospatial-python/setup.html) (recommended)
* CRIB: https://crib.utwente.nl
* UT JupyterHub: https://jupyter.utwente.nl

📚 Download files:
- [brpgewaspercelen_definitief_2020_small.gpkg](https://figshare.com/ndownloader/files/37729413)
- [brogmwvolledigeset.zip](https://figshare.com/ndownloader/files/37729416)
- [status_vaarweg.zip](https://figshare.com/ndownloader/files/37729419)

`search.json` file produced during the satellite scene search: [link](https://raw.githubusercontent.com/esciencecenter-digital-skills/2023-03-30-dc-geospatial/main/files/search.json)


## 👩‍🏫👩‍💻🎓 Instructors

Francesco Nattino, Ou Ku

## 🧑‍🙋 Helpers

Serkan Girgin


## 🗓️ Agenda

**Day 1**
 
| Time  | Topic                                         |
| ----- | --------------------------------------------- |
| 09:30 | Welcome and icebreaker                        | 
| 09:45 | Introduction to raster, vector, and CRS       |
| 10:00 | Access satellite imagery using Python         |
| 11:00 | *Coffee break*                                |
| 11:15 | Read and visualize raster data                |
| 12:30 | *Lunch break*                                 |
| 13:30 | Vector data in Python                         |
| 14:30 | *Coffee break*                                |
| 14:45 | Crop raster data with rioxarray and geopandas |
| 15:45 | *Coffee break*                                |
| 16:00 | Raster Calculations in Python                 |
| 16:45 | Wrap-up                                       |
| 17:00 | END                                           |
 
 
**Day 2**

| Time  | Topic                                   |
| ----- | --------------------------------------- |
| 09:15 | Welcome and icebreaker                  | 
| 09:30 | Calculating Zonal Statistics on Rasters |
| 10:45 | *Coffee break*                          |
| 11:00 | Parallel raster computations using Dask |
| 12:15 | Wrap-up                                 |
| 12:30 | END                                     |


## 👩‍💻👩‍💼👨‍🔬🧑‍🔬🧑‍🚀🧙‍♂️🔧 Roll Call
Name/ pronouns (optional) / job, role / social media (twitter, github, ...) / background or interests (optional) / city

** Deleted information ** 

## 🧊⛏️ Ice breaker

If you could live wherever - where would you live?
 
** Deleted information ** 

## 🔧 Exercises

* To ask for help use your pink sticker
* To run a command you can use Ctrl + Enter
* Datasets to download are indicated above. You don't need to unzip them.

#### Exercise: Downloading Landsat 8 Assets
In this exercise we put in practice all the skills we have learned in this episode to retrieve images from a different mission: [Landsat 8](https://www.usgs.gov/landsat-missions/landsat-8). In particular, we browse images from the [Harmonized Landsat Sentinel-2 (HLS) project](https://lpdaac.usgs.gov/products/hlsl30v002/), which provides images from NASA’s Landsat 8 and ESA’s Sentinel-2 that have been made consistent with each other. The HLS catalog is indexed in the NASA Common Metadata Repository (CMR) and it can be accessed from the STAC API endpoint at the following URL: https://cmr.earthdata.nasa.gov/stac/LPCLOUD.

- Using `pystac_client`, search for all assets of the Landsat 8 collection (`HLSL30.v2.0`) from February to March 2021, intersecting the point with longitude/latitute coordinates (-73.97, 40.78) deg.
- Visualize an item’s thumbnail (asset key `browse`).

```python
api_url_cmr = "https://cmr.earthdata.nasa.gov/stac/LPCLOUD"

client_cmr = client.open(api_url_cmr)

search_cmr = client_cmr.search(
    collections=["HLSL30.v2.0"],
    datetime="2021-02-01/2021-03-30",
    intersects=Point(-73.97, 40.78),
)

search_cmr.matched()

items_cmr=search_cmr.get_all_items()

items_cmr[0].assets.keys()

items_cmr[0].assets["browse"].href
```

#### Exercise: select fields within 500m from the wells

This time, we will do a selection the other way around. Can you find and plot the field polygons (stored in `fields_cropped.shp`), which intersect the 500m radius of any wells (stored in `brogmwvolledigeset.zip`)? 
- Hint1: `brogmwvolledigeset.zip` is in CRS 4326. Don't forget the CRS conversion.
- Hint2: `brogmwvolledigeset.zip` is big. To improve the performance, you may want to crop it first.


```python
import geopandas
import matplotlib.pyplot as plt
import pyproj


buffer_size = 500


fields = geopandas.read_file("fields_cropped.shp")
xmin, ymin, xmax, ymax = fields.total_bounds
xmin, ymin, xmax, ymax = xmin - buffer_size, ymin - buffer_size, xmax + buffer_size, ymax + buffer_size


transformer = pyproj.Transformer.from_crs(fields.crs, "EPSG:4326")
ymin, xmin, ymax, xmax = transformer.transform_bounds(xmin, ymin, xmax, ymax)
bounds = xmin, ymin, xmax, ymax


wells = geopandas.read_file("data/brogmwvolledigeset.zip", bbox=bounds)
wells = wells.to_crs(fields.crs)
buffer = wells.buffer(buffer_size)
fields_clipped = fields.clip(buffer)
fields_filtered = fields[fields.intersects(fields_clipped)]


fig, ax = plt.subplots(1, 1)

fields_filtered.plot(ax=ax)
fields_clipped.plot(color="green", ax=ax)
wells.plot(markersize=0.5, color="red", ax=ax)

plt.tight_layout()
plt.show()
```

## 🧠 Collaborative Notes

### Setup Check
```
import rioxarray
import geopandas
```

### Data Access
```Markdown
# Data Access
```

```python
api_url = "https://earth-search.aws.element84.com/v0"

from pystac_client import Client

client = Client.open(api_url)

for collection in client.get_collections():
  print(collection)

collection_id = "sentinel-s2-l2a-cogs"

from shapely.geometry import Point

# First parameter is longitude, second parameter is latitude
point = Point(4.89, 52.37)
point

# Establish a search
search = client.search(
        collections= [collection_id],
        intersects=point,
        max_items=10,)

# inspect search results
search


# Inspect total matched results
search.matched()

# Retrieve metadata of items matched
# If max_items is specified, only that number of items will be returned
items = search.get_all_items()

# Display number of items
len(items)

# Display information on items
for item in items:
  print(item)

# Get the first item
item = items[0]

# Inspect the item
item

[# Display date and time of the item
item.datetime](https://)

# Display geometry information of the item
item.geometry

# Display geometry using shapely
from shapely.geometry import shape

shape(item.geometry)

# Display item properties
item.properties

# Define bounding box by creating a buffer around the point
bbox = point.buffer(0.01).bounds

# Set date range
date_range = "2020-03-20/2020-03-30"

# Set cloud coverage
query_cloud_cover = "eo:cloud_cover<10"

# Query the catalog for the selected bounding box, date range, and cloud cover
search = client.search(
    collections=[collection_id],
    bbox=bbox,
    datetime=date_range,
    query=[query_cloud_cover],
)

# Display the number of matched items
search.matched()

# Get metadata of the matched items
items = search.get_all_items()

# Save results
# You can see the contents in JupyterLab JSON viewer (double click to open search.json)
# On CRIB you save as search.geojson and see it on the map by double click
items.save_object("search.json")

# Get the first item
item = items[0]

# Display assets of the item
item.assets

# Display keys of the item assets
item.assets.keys()

# Get the thumbnail asset
asset = item.assets["thumbnail"]

# Display URL address of the thumbnail
# If you click on the URL address, you will see the thumbnail
asset.href

# Import RasterIO XArray library
import rioxarray

# Get blue band URL address
b01_href = item.assets["B01"].href

# Get blue band data
# This is a lazy operation, i.e. data is not retrieved, but it will be retrieved once it is required
b01 = rioxarray.open_rasterio(b01_href)

```

### Raster Data

```python
import rioxarray

import pystac

1. # Read items (metadata) saved previously in a file
items = pystac.ItemCollection.from_file("search.json")
items

# Get URL address of the band data
b09_href = items[0].assets["B09"].href

# Display URL address of the band data
b09_href

# Open band data (lazy)
b09 = rioxarray.open_rasterio(b09_href)
b09

# Display spatial reference information
# REMARK: Information is stored as properties, not coordinates
b09.coords["spatial_ref"]

# Get CRS by using RasterIO object
b09.rio.crs

# Get no data value
# REMARK: This is a property (i.e. no paranthesis)
b09.rio.nodata

# Get bounds
# REMARK: This is a method
b09.rio.bounds()

# Get values (i.e. band data)
# REMARK: This will download data
b09.values

# Display data by using plot() convenience method
# REMARK: matplotlib is used to plot the data
b09.plot()

# Increase contrast
b09.plot(robust=True)

# Type of CRS object
type(b09.rio.crs)

# Get EPSG
b09_epsg = b09.rio.crs.to_epsg()

# Import PyPROJ library
from pyproj import CRS

# Get detailed information on CRS
crs = CRS(b09_epsg)
crs

# Reproject to WGS84
b09_repr = b09.rio.reproject("EPSG:4326")

# Plot reprojected image
b09_repr.plot()

# Calculate minimum value
# REMARK: Return an xarray object
b09.min()

# Calculate minimum value on x axis
# REMARK: Return a value for each y coordinate; hence, it is an array (xarray)
b09.min(dim="x")

# Import numpy
import numpy as np

# Show not-a-number value
np.nan

# Open band again by masking out missing values (i.e. no-data)
b09_nodata = rioxarray.open_rasterio(b09_href, masked = True)

# Confirm that no data value is changed to nan instead of 0
b09_nodata.rio.nodata

# Get mean
# REMARK: This will be different from non-masked version, because no data values will be discarded.
b09_nodata.mean()
```

```python
# Import GeoPandas package
import geopandas as gpd

# Read data (rows)
fields = gpd.read_file("data/brpgewaspercelen_definitief_2020_small.gpkg")

# Get geometry column
fields["geometry"]

# Get the first geometry
fields["geometry"].iloc[0]

# Check type of the geometry
# REMARK: It is a shapely polygon object
type(fields["geometry"].iloc[0])

# Plot dataset
fields.plot()

# Set a smaller bounding box
xmin, xmax = (115_000, 140_000)
ymin, ymax = (480_000, 510_000)
bbox = (xmin, ymin, xmax, ymax)

# Load data by using the smaller extent
fields = gpd.read_file("data/brpgewaspercelen_definitief_2020_small.gpkg", bbox = bbox)

# Plot it
fields.plot()

# Get smaller extent by using an indexer (in memory, i.e. after loading the data)
fields_cx = fields.cx[120_000:135_000, 485_000:500_000]

# Get smaller extent by using a clip method
fields_rect = fields.clip_by_rect(120_000, 485_000, 135_000, 500_000)

# Plot them to see the difference
# REMARK: cx is a "select by bounding box" operation, whereas clip_by_rect is "clip by bounding box"
# REMARK: clip_by_rect returns a GeoSeries object, i.e. only geometries
# REMARK: clip_by_rect will include the same number of features of the original, but non-intersecting will be empty
fields_cx.plot()
fields_rect.plot()

# Display them side by side
# REMARK: matplotlib is used 
from matplotlib import pyplot as plt

fig, ax = plt.subplots(2, 1)

fields_cx.plot(ax=ax[0])
ax[0].set_xlim([119500, 121000])
ax[0].set_ylim([499000, 500500])

fields_rect.plot(ax=ax[1])
ax[1].set_xlim([119500, 121000])
ax[1].set_ylim([499000, 500500])

# Check the shape of fields_cx
# REMARK: Only selected features will be listed, i.e. (4872, 6)
fields_cx.shape

# Check the shape of fields_rect
# REMARK: All features will be listed, i.e. (11839,0 )
# REMARK: Zero value indicated that it only includes geometries, but not attributes
fields_rect.shape

# Update attributes by creating a copy of the dataset
fields_temp = fields.copy()
fields_temp["geometry"] = fields_rect
fields_temp.shape

# R

```

### Vector Processing

```python
# Import GeoPandas
import geopandas as gpd

# Load wells
wells = gpd.read_file("data/brogmwvolledigeset.zip")

# Load fields
fields = gpd.read_file("data/brpgewaspercelen_definitief_2020_small.gpkg")

# Get CRS of the fields dataset
fields.crs

# Reproject wells to fields CRS
# REMARK: It produces a new object, i.e. reprojection is not in place
# REMARK: Check inplace argument if you want in place operation
wells = wells.to_crs(epsg=28992)

# Check if CRS is changed
wells.crs

# Clip wells by fields
wells_clip = wells.clip(fields)

# Plot clipped wells
wells_clip.plot()

# Create a 50 m buffer around the fields
# REMARK: buffer() method returns a geoseries, not a geodataframe
buffer = fields.buffer(50)

# To get attributes, we need to create a copy of the dataset and update geometry
fields_buffer = fields.copy()
fields_buffer["geometry"] = buffer

# Dissolve field polygons to have a single polygon
fields_buffer_dissolve = fields_buffer.dissolve()

# Find wells intersecting with the dissolved fields
wells_buffer = wells.clip(fields_buffer_dissolve)

# Plot wells
wells_buffer.plot()

# Plot wells and fields
from matplotlib import pyplot as plt

fig, ax = plt.subplots(1, 1)

wells_buffer.plot(ax=ax, color="r")
fields.plot(ax=ax)

```

## 📚 Resources

[STAC Catalog of Sentinel-2](https://radiantearth.github.io/stac-browser/#/external/earth-search.aws.element84.com/v0?.language=en)

[STAC browser one example scene](https://radiantearth.github.io/stac-browser/#/external/earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2B_52VDL_20230330_0_L2A)

[PySTAC Client Documentation](https://pystac-client.readthedocs.io/en/stable/)

[PySTAC Client Documentation `client.search` function](https://pystac-client.readthedocs.io/en/stable/api.html#pystac_client.Client.search)

[`rioxarray.open_rasterio` documentation](https://corteva.github.io/rioxarray/html/rioxarray.html#rioxarray-open-rasterio) Accodring to this doc, the coordinates refer to the center of each pixel


## Tips and Tops 

### Tips (what can be improved)

** Deleted information ** 

### Tops (what went well)

** Deleted information ** 

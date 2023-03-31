![](https://i.imgur.com/iywjz8s.png)


# Collaborative Document - Day 2

2023-03-31 Introduction to Geospatial Raster and Vector Data with Python


Welcome to The Workshop Collaborative Document.

This Document is synchronized as you type, so that everyone viewing this page sees the same text. This allows you to collaborate seamlessly on documents.

----------------------------------------------------------------------------


This is the Document for today: [link](https://tinyurl.com/2023-03-31-geospatial-python)

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

Files produced on Day 1:
* `search.json` (satellite scene metadata): [link](https://raw.githubusercontent.com/esciencecenter-digital-skills/2023-03-30-dc-geospatial/main/files/search.json)
* `fields_cropped.shp.zip` (cropped cropfields): [link](https://github.com/esciencecenter-digital-skills/2023-03-30-dc-geospatial/raw/main/files/fields_cropped.shp.zip)

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

## 🔧 Exercises


## 🧠 Collaborative Notes

Yesterday we finished with a question: how to select features from a dataset that intersect with features from another dataset? One option is to use spatial join functions.

### Vector data sjoin

```python
import geopandas as gpd

# Load wells
# REMARK: Reasons for not using Shapefiles:
# http://switchfromshapefile.org/
wells = gpd.read_file("data/brogmwvolledigeset.zip")

# Load fields
fields = gpd.read_file("fields_cropped.shp")

# Reproject wells to the CRS of the fields
wells = wells.to_crs(fields.crs)

# Get the bounds of the fields
xmin, ymin, xmax, ymax = fields.total_bounds

# Select wells by bounding box plus buffer distance (500 m)
wells_cx = wells.cx[xmin-500:xmax+500, ymin-500:ymax+500]

# Plot selected wells
wells_cx.plot()

# Create a buffer
# REMARK: Like all spatial operations, this well return geoseries, not a geodataframe
buffer = wells_cx.buffer(500)

# Create a buffer geodataframe by copying the original dataset and merging the buffer geoseries
wells_cx_buffer = wells_cx.copy()
wells_cx_buffer["geometry"] = buffer

# Plot buffered wells dataset
wells_cx_buffer.plot()

# Performs a spatial join to select fields intersecting wells
# REMARK: sjoin() is a powerful method, please check the documentation for more details
fields_wells_cx_buffer = fields.sjoin(wells_cx_buffer)

# Check the shape
fields_wells_cx_buffer.shape

# Check the original shape
fields.shape

# List indexes
fields_wells_cx_buffer.index

# Get unique indexes
index = fields_wells_cx_buffer.index.unique()

# Select fields by unique indexes
fields_selected = fields.iloc[index]

# Plot selected fields
fields_selected.plot()
```


### NDVI Calculation

```python
import pystac

# Load items saved to search.json previously
items = pystac.ItemCollection.from_file("search.json")

# Check the items are loaded
len(items)

# Get the assets of the first item
assets = items[1].assets

# List key of the assets
assets.keys()

# Display titles of the assets by iterating through a for loop
for key, asset in assets.items():
    print(key, ":", asset.title)

# Get URL addresses of red and NIR bands
red_href = assets["B04"].href
nir_href = assets["B08"].href


# Import xioarray package to load band data
import rioxarray

# Load read band masking no-data values
red = rioxarray.open_rasterio(red_href, masked=True)

# Load NIR band masking no-data values
nir = rioxarray.open_rasterio(nir_href, masked=True)

# Display information on red band
red

# Display CRS by using the underlying rasterio object
red.rio.crs

# Define AIO
bbox = (629_000, 5_804_000, 639_000, 5_814_000)

# Clip bands by using the AOI
# REMARK: Asterisk before bbox parameter unpacks it (i.e. each element of bbox becomes a parameter to the clip method)
red_clip = red.rio.clip_box(*bbox)
nir_clip = nir.rio.clip_box(*bbox)

# Load smaller version available
red_overview = rioxarray.open_rasterio(red_hred, overview_level=2)

# Display clipped red band
red_clip.plot(robust=True)

# Display clipped NIR band
nir_clip.plot(robust=True)

# Display CRS
red_clip.rio.crs, nir_clip.rio.crs

# Reproject and match (if required)
red_clip.reproject_march(nir_clip)

# Calculate NDVI
# REMARK: Raster should be on the same grid
ndvi = (nir_clip - red_clip) / (nir_clip + red_clip)

# Plot NDVI
ndvi.plot()

# Plot histogram
ndvi.plot.hist(bins=5)

# Get mask of missing values
ndvi.isnull()

# Count how many missing values exist
ndvi.isnull().sum()

# Interpolate missing values in x direction
ndvi_interpolated = ndvi.interpolate_na(dim="x")

# After interpolation, this should return 0
ndvi_interpolated.isnull().sum()

# Save NDVI to a file
# REMARK: Check the documentation for supported file formats and options (e.g. compression)
ndvi_interpolated.rio.to_raster("NDVI.tif")
```

### Parallel Raster Operations

```python
import pystac

# Read items from stored JSON file
items = pystac.ItemCollection.from_file("search.json")

# Get the last item's assets
# REMARK: Index -1 means the last item
assets = items[-1].assets

# Get the URL address of the composite visual band
visual_href = assets["visual"].href

# Import rioxarray package
import rioxarray

# Open visual band
visual = rioxarray.open_rasterio(visual_href)

# Load data
# REMARK: Enforce loading of data, so that it becomes readily available
visual = visual.load()

# Plot data
# REMARK: Plots as a histogram instead of a map, because it doesn't know what to do
visual.plot()

# Plat data as an image
visual.plot.imshow()

# Create a rolling window (i.e. filter)
visual.rolling(x=7, y=7)

# Calculate rolling median
median = visual.rolling(x=7, y=7).median()

# Plot median image
# REMARK: We need to normalize, so that it can be displayed
# REMARK: robust=True does (a kind of) normalization
(median / 255).plot.imshow()
median.plot.imshow(robust=True)

# Open data as chuncked Dask array
# REMARK: xarray supports Dask arrays out of box
visual = rioxarray.open_rasterio(visual_href, overview_level=2, chunks=(3, 500, 500))

visual = visual.persist(scheduler="threads", num_workers=4)

# Calculate median (lazy)
# REMARK: Calculation won't be done
median = visual.rolling(x=7, y=7).median()

import dask

dask.visualize(median)

median = median.persist(scheduler="threads", num_workers=4)


```

## 📚 Resources




## Post-workshop survey

** Deleted information ** 


Post workshop Survey: [link](https://www.surveymonkey.com/r/X3GX792)


# A few handy geospatial code snippets

```python

polycoords = []

for feature in fiona.open(poly_fp):
    assert feature['geometry']['type']=='Polygon', "Feature is not a polygon"
    
    for lat, lon in feature['geometry']['coordinates'][0]:
        polycoords.append((lon, lat))

poly = Polygon(polycoords)

```


### Converting a raster from one projection to EPSG:4326 (WGS)

```python
import rasterio
from rasterio.warp import calculate_default_transform, reproject, Resampling

fp_const = 'path/to/filename'
fp_const_corr = 'path/to/other/filename'
dst_crs = 'EPSG:4326'

with rasterio.open(fp_const) as src:
    transform, width, height = calculate_default_transform(src.crs, dst_crs, 
                                                           src.width, src.height,
                                                           *src.bounds)

#     print(src.indexes)
    src_data_max_band = src.read(indexes=[1])[0] # only interested in the 2-D array from band 1.
    
    dst_shape = (height, width)
    
    # initialize numpy array to contain all nulls (value -9999.0), will write over relevant ones.
    destination = np.ones(dst_shape, np.double)*-9999.0 
    
    reproject(src_data_max_band,
                destination,
                src_transform=src.transform,
                src_crs=src.crs,
                src_nodata=-9999.0,
                dst_transform=transform,
                dst_crs=dst_crs,
                resampling=Resampling.nearest
               )

        # optional write method
    with rasterio.open(fp_const_corr, 'w',
                       driver='GTiff',
                        width=width,
                        height=height,
                        count=1,
                        dtype=np.double,
                        nodata=-9999.0,
                        transform=transform,
                        crs=dst_crs) as dst:
        dst.write(destination, indexes=1)
```
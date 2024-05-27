# geomockimages

A module to programmatically create geotiff images which can be used for unit tests.

The underlying idea is that in order to write unit tests for geospatial image processsing algorithms,
it is necessary to have an actual input image file or array. Organising these test images becomes a chore over time,
they should not be stored in git as they are large binary data and when stored outside, there always
is the danger that they are not updated according to changes in the code repo.

**geomockimages** provides a solution to the problem by providing simple code that allows to create
geospatial images (so far geotiffs) in a parameterised way.

**geomockimages** is able to create both electro-optical as well as SAR images. More specificially, 4-band optical images and SAR images with one or two polarizations are supported explicitely. It is also possible to create images with more or less images, but the results will be less realistic.

Besides singe image tests, **geomockimages** can also create image pairs with small differences. This allows to do basic testing of change detection algorithms.

## Install package
```bash
pip install geomockimages
```

## Usage

In the following an example unit test for a hypothetical NDVI function.

```python
import numpy as np
import rasterio as rio
from pathlib import Path

from rasterio.transform import from_origin
from my_image_processing import ndvi
from geomockimages.imagecreator import GeoMockImage

def test_ndvi():
    """
    A unit test if an NDVI method works in general
    """
    # Create 4-band image simulating RGBN as needed for NDVI
    test_image, _ = GeoMockImage(
        300,
        150,
        4,
        "uint16",
        out_dir=Path("/tmp"),
        crs=4326,
        nodata=0,
        nodata_fill=3,
        cog=False,
    ).create(seed=42, transform=from_origin(13.428596, 52.494384, 0.000006, 0.000006))

    ndvi_image = ndvi(test_image)

    with rio.open(str(ndvi_image)) as src:
        ndvi_array = src.read()
        # NDVI only has one band of same size as input bands
        assert ndvi_array.shape == (1, 150, 300)
        # NDVI has float values between -1 and 1
        assert ndvi_array.dtype == np.dtype('float32')
        assert np.nanmin(ndvi_array) >= -1
        assert np.nanmax(ndvi_array) <= 1

```

## Documentation

**geomockimages** was written for python engineers. The best way to learn about the option it offers do:

```python
from geomockimages.imagecreator import GeoMockImage
help(GeoMockImage)
```

## More examples
...can be found within two jupyter notebooks:

[Basic 4-band optical example](https://github.com/markusuwe/geomockimages/blob/main/notebooks/example1_radiance.ipynb)

[SAR images and change detection](https://github.com/markusuwe/geomockimages/blob/main/notebooks/example2_sar_imagepair.ipynb)

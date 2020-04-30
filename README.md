# GIS-Technical-Challenge

Cervest is interested in developing a platform enabling arbitrary geospatial queries against large datasets in a highly perfomant manner, for which the [Pangeo](https://pangeo.io/index.html) tools appear well suited. One obstacle we've encountered is the time it takes to read data from disk into memory. We would like to understand what technical options exist for improving these read times.

### Technical Challenge

We have roughly 10 years' worth of relative humidity data stored in both NetCDF and [zarr](https://zarr.readthedocs.io/en/stable/tutorial.html) format, totalling around 7.2G and 8.6GB, respectively.

The data is defined on a 0.25 degree x 0.25 degree grid, within a lat-lon bounding box specified by [-27, 32, 46, 74]. The NetCDF files are named according to the timestamps of the data they contain. For instance, `R.1979010100_1979010123.nc` contains relative humidity values on 1979-01-01 from 00:00 - 23:00 hrs.

`xarray` can only read data from zarr formats which have been written using `xarray`. The zarr data is created using the default Blosc compression algorithm and default chunks chosen by `xarray`.  

The raw data is read into an `xarray.Dataset` object. The time taken to do so depends upon the file format chosen.

* NetCDF: ~ 3min 30s. Call: `xarray.open_mfdataset(nc_file_paths, combine='by_coords').load()`
* Zarr: ~ 30s: Call `xarray.open_zarr(zarr_path).load()`

This is executed on a single machine. Given modern SSD bandwidths of ~2.5GB/s, we'd hope this might take around 3-4s. 

The challenge is to bring this read time down as close to that 4s target as possible.

### Desiderata

* We are not tied to these file formats. If better performance can be achieved with Cloud-Optimised GeoTiff or TileDB formats, please use those!
* The advantage of the `xarray` module is its use of `dask` under the hood. Dask exposes a consistent API, independent of the job scheduler chosen. This means we can easily prototype the work on our on-prem workstations, but also distribute the work across a cloud-based cluster without any headaches - dask can do both. It would be ideal for any solution to maintain this transferability.

## Replication steps

### Requirements

* [pyenv](https://github.com/Cervest/ds-mlstats/wiki/Environment-Management--Using-pyenv) Python version & virtual environment manager.
* [poetry](https://python-poetry.org/) package manager. [Installation](https://github.com/python-poetry/poetry#installation)

### Installation

Create and activate a virtualenv using `pyenv`:

```
pyenv install 3.8.2
pyenv virtualenv 3.8.2 gis-venv
pyenv activate gis-venv
```

Append `poetry` executable to `$PATH`, if not already done so.

```bash
PATH=~/.poetry/bin:$PATH
```

Install project dependencies using `poetry`:

```bash
poetry install
```

Create an ipython kernel for the virtualenv (which will enable running the notebook code):

```bash
python -m ipykernel install --user --name=gis-venv
```

Download the data from S3:

```
aws s3 cp s3://cervest-google-accelerator/relative-humidity-data <some_path>
```

Within the `relative-humidity-data/` directory, you will find the data in both NetDCF and zarr formats.

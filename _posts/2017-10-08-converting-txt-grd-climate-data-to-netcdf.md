---
id: 760
title: 'Converting .TXT/.GRD Climate Data Files to netCDF Format'
date: '2017-10-08T03:49:22+00:00'
author: Anthony Louis D'Agostino
layout: post
guid: 'https://anthonylouisdagostino.com///?p=760'
permalink: /converting-txt-grd-climate-data-to-netcdf/
image: /wp-content/uploads/2017/06/screen-shot-2017-06-02-at-9-48-19-am.png
categories:
    - 'climate change'
    - computing
    - data
    - India
    - Python
    - R
---

Climate data is packaged and distributed in <del>too</del> many file formats. Under ideal circumstances, you could easily convert data from formats you’re not familiar with (and don’t have scripts to handle), to those that you do. This is why analogous tools like [Stat/Transfer](http://www.stattransfer.com/) for statistical databases often used by social scientists, are so helpful. If a stranger on the street gives you SPSS data, you can on-the-fly convert it to something which is Stata-readable. Albeit, the value of software like Stat/Transfer diminishes as more stat packages have comprehensive in-built conversion tools, like R’s [`readstata13`](https://cran.r-project.org/web/packages/readstata13/README.html){:target="_blank"} and [`read stata`](http://pandas.pydata.org/pandas-docs/version/0.20/generated/pandas.read_stata.html){:target="_blank"} in pandas. Getting similar functionality with climate data requires a bit more lift.\
<br>


This post is a tutorial and link to scripts that can convert the .TXT/.GRD file combination format used by the [India Meteorological Department [IMD](http://www.imd.gov.in/){:target="_blank"} into formats that are more usable for people working with climate data. And if you’re not trained in the sciences, but rather as an economist, figuring out how to use this data often comes with its own frustrations. I hope this helps.\
<br>

I rely on the [Climate Data Operators [CDO](https://code.mpimet.mpg.de/projects/cdo){:target="_blank"} to do the heavy lifting in my climate data workflow and work almost exclusively with the [netCDF4 file format.](https://www.unidata.ucar.edu/software/netcdf/){:target="_blank"} Since IMD provides their climate data in .TXT/.GRD format, extra work is required to turn those files into more familiar formats. Here we’ll convert it to netCDF, which after processing can then be exported to a spreadsheet.\
<br>

## Data Setup and Processing Steps
<br>

IMD provides you daily data for each weather variable (TMAX, TMIN, TAVG) in year-specific .TXT and .GRD files. The .TXT file includes the lat/lon grid boundaries, timestamp, and daily values for each pixel. The .GRD file somehow converts this long .TXT into an array stack.\
<br>

- Step 1 – We first generate year-specific .CTL files which contain the data header, since IMD provides us only a single .CTL. I found doing this in R to be relatively straightforward, since a .CTL can be read as a standard text file. Each of the generated .CTL files designate the source data and the spatial/temporal grid resolution. Here I use a modulo operator to differentiate leap years and accordingly modify the number of time-steps. If leap year date data is not included, then this wouldn’t be a concern. The following R code is an example of how those .CTLs can be auto-generated for a specified year range.\
<br>

```r

# Filename: convert_to_GrADS.R
# Author: Anthony Louis D'Agostino (ald at stanford dot edu)
# Date Created: September 16, 2015
# Last Edited: October 07, 2017
# Purpose: Modifies the NCC-provided CTL file and generates unique versions for each year of data, each type of temperature
# max, min, mean.
# Notes: While perhaps an overkill, this is generalizable to a setting where grids are file-specific.

rm(list = ls())
root.path <- "/tmp"

# -- create path for generated GrADS control .CTL files.
ctl.path <- paste(root.path, "CTL_Files", sep = "/")
if (file.exists(ctl.path) == FALSE){
  dir.create(ctl.path)
}

# -- data stored in three separate variable-specific folders
temp_types = c("MinT", "MaxT", "MeanT")
for (x in temp_types) {
  type.path <- paste(ctl.path, x, sep = "/")
  if (file.exists(type.path) == FALSE){
    dir.create(type.path)
  }

  # -- year range for which data is available
  for (i in 1951:2014){
    print(paste0("Now processing year ", i, " for variable ", x))

  # -- read in and update template .CTL provided by IMD
  ctl_file <- read.delim(paste0(root.path, "/Temp.ctl"), header = FALSE, sep = " ", stringsAsFactors = FALSE)

  # -- file reference for source GRD
  ctl_file[1,"V2"] <- paste0(root.path, "/", x, "/", x, "_", i, ".GRD")
  if (i %% 4 == 0){
    ctl_file[7,"V2"] <- 366 # account for leap years in total number of timesteps
    } else {
      ctl_file[7, "V2"] <- 365
    }

  ctl_file[7, "V5"] <- paste0("1JAN", i)

  # -- write updated .CTL to file
  write.table(ctl_file, paste(type.path, paste0(x, "_", i, ".ctl"), sep = "/"), quote = FALSE, col.names = FALSE, na = "", row.names = FALSE)

  }
}

```
<br>
- Step 2 - CDO enables easy conversion between GrADS and netCDF which we'll exploit. This could be done at the command line, for reproducibility we'll use the Python wrappers which can be installed via pip. The following script demonstrates how those binaries can be read in, converted to netCDF, and concatenated by temperature variable.\
<br>

```python

# Filename: IMD_convertRawData.py
# Author: Anthony Louis D'Agostino (ald at stanford dot edu)
# Date Created: 06/01/2017
# Last Edited: 10/07/2017
# Data: from NCC ZIP file   
# Purpose: Reads in GRaDS data files, concatenates them, and then exports netCDF versions  
# Notes: To be run after "convert_to_GrADS.R"


import os
from cdo import *
from netCDF4 import Dataset
import numpy as np
#import pandas as pd


cdo = Cdo()


root_path = "/tmp"
ctl_root = os.path.join(root_path, "CTL_Files")


def tempOutput(var, ctl, root):
	"""
	Read in binary data and output as a netCDF file.
	var: weather variable   
	ctl: root path for all .ctl's
	root: project root path
	"""

	# -- rename variables for consistency with other projects
	temp_rename = {"MaxT": "tmax", "MinT": "tmin", "MeanT": "tmean"}

	# initialize using first year's data
	t = cdo.import_binary(input = os.path.join(ctl, var, var + "_1951.ctl"))

	# -- loop through each remaining year
	for y in range(1952,2015):
		print "Now processing " + str(y)
		fn = var + "_" + str(y) + ".ctl"
		print "Processing " + str(os.path.join(ctl_root, var, fn))

		# -- concatenate into a single file
		data = cdo.import_binary(input = os.path.join(ctl_root, var, fn))
		t = cdo.cat(input = " ".join([t,data]))

	# -- save variable-specific file
	t = cdo.copy(input = t, options = "-f nc", output = os.path.join(root, temp_rename[var] + "Proc.nc"))


tempOutput("MaxT", ctl_root, root_path)
tempOutput("MinT", ctl_root, root_path)
tempOutput("MeanT", ctl_root, root_path)

```\
<br>

- Step 3 – Now the concatenated output file has been saved as a netCDF, which means you can perform all the standard CDO operators on it. You can also simply read your files in R, and run functions like spatial averaging with a minimum of code, as in [this example](https://gis.stackexchange.com/questions/213493/area-weighted-average-raster-values-within-each-spatialpolygonsdataframe-polygon){:target="_blank"}.\
 <br>

As always, happy to field any questions you might have on the code and workflow!\
<br>

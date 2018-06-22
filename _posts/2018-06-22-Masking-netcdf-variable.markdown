---
title: Masking variables in netCDF files with a landsea mask
layout: post
tags: netcdf nco
---

These notes record how to mask (set to a no-data or missing-data value) a single netCDF variable, using a landsea mask.

**Problem**: We have a netCDF file containing a variable mapped over the globe. The variable should not have any values over the sea areas, because that doesn't make sense for this particular data. (Let's say it's output from some post processed satellite data for argument). 
All the data points over seas and oceans are zero, but we can't use zero as 'NoData' becuase zero is a valid value over land as well. 

We want to use the `_FillValue` attribute of netCDF variables as a mask, so that when plotting or calculations are done using the variable, the sea values are ignored. 

We also have a binary 1/0 land-sea mask in a separate file, at the same resolution as out JULES out put data. The land values have 1, and the seas/oceans have 0.

**Input files**

`landsea.nc`

![The land sea mask with 0 for seas, and 1 for land](images/landsea_step1.png)


`data_to_be_masked.nc`

![Input data to be masked](images/landsea_beforemask.png)


**Steps**

1. First copy over the landsea mask variable from landsea.nc to the file to be masked

    ```
    ncks -A -v lsmask landsea.nc data_to_be_masked.nc
    ```

2. Then make sure that the variable in the other file (gpp_gb) has a _FillValue set correctly (using -9999 here, but any convention will do (not zero though...)

    ```
    ncatted -a _FillValue,gpp_gb,o,f,-9999 data_to_be_masked.nc
    ```

3. Then use ncap2 to set all the gpp values to _FillValue (the mask) whereever the landsea mask is not land (i.e. not == 1) 

This is written to a new file.

    ```
    ncap2 -s 'where(lsmask != 1) gpp_gb=gpp_gb@_FillValue' data_to_be_masked.nc data_after_masking_done.nc
    ```

The output variable `gpp_gb` now has the sea values set to the Nodata FillValue.

![Masked GPP](images/landsea_aftermask.png)

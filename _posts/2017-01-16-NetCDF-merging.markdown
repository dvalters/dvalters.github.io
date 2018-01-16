---
title: Merging netCDF files with NCO/CDO
layout: post
tags: netcdf
categories: data modelling
---

You can merge netcdf files (say, along a time dimension) with the`nco` utility. (nco is actually a set of linux command line utilities for performing common operations on netcdf files.)

To concatenate files, use `ncecat`

```
ncecat *nc -O merged.nc
```

This will merge all the netcdf files in a folder, creating a new _record_ dimension if one does not exist. The record dimension is often the time dimension, for example if you have a set of netCDF files, with each one representing some spatial field at a given timestep. If appropriate, you can rename this _record_ dimension to something more useful using the `ncrename` utility (another utility in the NCO package).

```
ncrename -d record,time merged.nc
```

The -d flag specifies that we are going to rename the _dimension_ in the netcdf file. (There are also flags to rename other attributes, see the manual here: https://linux.die.net/man/1/ncrename





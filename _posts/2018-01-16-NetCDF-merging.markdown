---
title: Merging netCDF files with NCO/CDO
layout: post
tags: netcdf
categories: data modelling
---

## NCO (NetCDF Operators)

You can merge netcdf files with the `nco` package. NCO is a set of linux command line utilities for performing common operations on netcdf files. This is useful for mergeing a set of files such as:

```
ModelRun_Jan.nc
ModelRun_Feb.nc
ModelRun_Mar.nc
...

```

### Concatenating files, creating a new dimension in the process

To concatenate files, use `ncecat`:

```
ncecat *nc -O merged.nc
```

This will merge all the netcdf files in a folder, creating a new _record_ dimension if one does not exist. The record dimension is often the time dimension, for example if you have a set of netCDF files, with each one representing some spatial field at a given timestep. If appropriate, you can rename this _record_ dimension to something more useful using the `ncrename` utility (another utility in the NCO package).

### Renaming dimensions

```
ncrename -d record,time merged.nc
```

The -d flag specifies that we are going to rename the _dimension_ in the netcdf file., from "record" to "time". There are also flags to rename other attributes, [see the ncrename manual page](https://linux.die.net/man/1/ncrename)


### Removing degenerate dimensions

To remove degenerate dimensions (by averaging over the dimension to be removed):

```
ncwa -a dim_name input.nc output.nc
```
ncwa ("Weighted average") will average variables over the specified dimension. If our dimension is degenerate (dim = 1), then this is effectively a way to remove that dimension without changing any of the variable data (Since it is averaging the variable over 1).

### Adding a new variable

If we need to add a new variable, this can be done with `ncap2` (`ncap` is deprecated). 

```
ncap2 -s'new_dim[$new_dim]=1234'
```
Note that this will add a single value of the time variable: 1234.

### Changing a variable to vary over a newly added dimension

If we add a new dimension, the existing variables will not automatically be functions of this new dimension. So if we were to add a time dimension, we need to recreate our variables to remap over this new dimension (assuming this is correct and appropriate for that particular variable/dimension combination.)

```
ncap2 -s 'Var_new[$dim1, $dim2, $new_dim3]=Var_old' input.nc output.nc
```

## Further nco utilities


The available utilities with nco are:

The NCO utilities are

 - ncap2 - arithmetic processor
 - ncatted - attribute editor
 - ncbo - binary operator
 - ncdiff - differencer
 - ncea - ensemble averager
 - ncecat - ensemble concatenator
 - ncflint - file interpolator
 - ncks - kitchen sink (extract, cut, paste, print data)
 - ncpdq - permute dimensions quickly
 - ncra - running averager
 - ncrcat - record concatenator
 - ncrename - renamer
 - ncwa - weighted averager


## CDO (Climate Data Operators)

This is an equally capable set of netCDF tools written by the Max Planck Institute for Meteorology. 

[CDO tools page](https://code.mpimet.mpg.de/projects/cdo/)



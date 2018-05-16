---
title: Deleting and adding attributes and variables with NCO
layout: post
tags: netcdf nco
---

Here are some more common tasks I've come across when needing to edit netCDF files. This is usually when they need to be ingested into different models or post-processing scripts that require the netCDF files to be in a certain format. 

## Deleting a global attribute

You want to delete a single global attribute from a netCDF file.

This can be done using `ncatted`, e.g.: 

```
ncatted -a global_attr_name,d,, infile.nc outfile.nc
```

This command takes for arguments separated by commas. Since we are specifying deletion, (d), only the first two arguments are needed, but the remaining commas bust be typed in.

## Convert a variable type

A variable is of incorrect type and you need to change it. You can use `ncap2` (nc arithmetic processing).

```
ncap2 -s 'time=float(time)'
```

Assumes you already have the variable defined. The `-s` option specifies that we are providing an inline script, within the quote marks.

## Add a variable mapped over a certain dimension

You want to a variable that iterates over a given dimension, such as time. The variable should increase montonically (i.e. increase by _n_ each time until the end of the dimension length is reached. I often find I need to do this after having merged netCDF files that were single time slices from a model output or satellite data or otherwise. `ncap2` is used. 

```
ncap2 -s 'time[$time]=array(54760,30,$time)' infile.nc outfile.nc
```

We are assigning the current time variable (assuming we have already added this) an array of values, specified by the `(start_point, step, dimension)`. In this case, we get an array of values starting at 54760, increasing by 30 each point, as long as the `time` dimension. The `-s` option simply means we are giving an inline script as the input to the `ncap2` program. 


## Add an attribute one at a time

You want to an _attribute_ to a _variable_. (I.e. metadata attributes for variables, such as units, etc.). We can use ncatted for this. (netCDF attribute editor).

```
ncatted -a attribute,variable,a,c,"Atrribute Value" infile.nc
```

The `-a` option specifies append mode, and so we only need to supply the input file `infile.nc`. The value of the attribute is given in the quotation marks. The nco documentation suggested also putting single quotation marks around the comma-separated arguments as well, but I found this produced unexpected results where the double quotes were escaped and inserted into the actual attribute value as well. Could possibly be a unix thing though...






## A netcdf bug to watch out for...

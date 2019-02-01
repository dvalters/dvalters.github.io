---
title: Date and time manipulations with CDO
layout: post
tags: netcdf cdo
---

Several tasks relating to manipulating netCDF files and dates/times within them are noted here.

1. Converting the units and values of the `time` variable.

# Setting the time 'axis'

I have a netcdf file that contains a time variable. The values in the variable's data array represent `time units` since `date time string`. Some model output and data formats use `months since` as the unit, which strictly speaking is non-standard and not CF compliant. (The integer values in the time variable array simply count increasing months, for example.)

Instead, I would like these to be represented as `days since`. So the integer values for time should now represent months.

I also want the month value to be reperesented at the mid-point of the month (The 15th), so I have chained in the `-settaxis` command as well.

```
cdo settunits,days -settaxis,2001-01-15,00:00,1month INFILE.nc NEW_OUTFILE.nc
```

The command `settunits` will change the `time:units` value as well, if it is in standard CF format. 

It is useful to check the output is correct using `cdo showdate` or `cdo showdatetime`

```
2001-01-15  2001-02-15  2001-03-15  2001-04-15  2001-05-15....
```




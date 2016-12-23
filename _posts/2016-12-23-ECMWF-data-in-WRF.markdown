---
layout: post
title: Initialising the WRF model with ECMWF ERA-20C data
tags: WRF era20c NWP
---

The Weather Research and Forecasting model (WRF) can be initialised with a range of input data sources for simulations. The initialisation step describes the setting of grid parameters within the model domain (pressure, surface variables, etc.) as well as defining the boundary conditions for the model. If you have followed the excellent [WRF tutorial](http://www2.mmm.ucar.edu/wrf/OnLineTutorial/Introduction/index.html) and run a few of the [case studies with real data](http://www2.mmm.ucar.edu/wrf/OnLineTutorial/CASES/index.html) the input data is provided for you and is already tested to ensure it can be pre-processed relatively painlessly by the WPS (WRF pre-processing system). Datsets from North American providers are extenisvely tested with WRF, (i.e. GFS data (global), AWIP (North American continent area))

 I've recently begun using [ECMWF](www.ecmwf.int) data to intialise WRF simulations and a few extra steps not shown in the standard tutorials were required to get the data pre-processed correctly by WPS. Certain ECWF datasets, such as reanalysis data, can be accessed and downloaded freely from their [public data portal](http://apps.ecmwf.int/datasets/). Global reanalysis data is availble for the 20th century and interim data for the most recent years. In this example I'm using the ERA-20C global reanalysis data set to set up a case study of some severe storm events over Great Britain in 2005. (The ERA-20C actually extends into the 21st century as well now). 

The data come in several different sets, as a minimum for intialising the WRF model you will need some surface variable data, and then pressure level data (pressures at different heights in the atmosphere). You could also use model level data directly, but WRF can interpolate this for you from the pressure level data. There is one other data set you need, which is a land-sea surface mask. This is called invariant data, as it does not change over time, and is found under the 'invariant' tab on the ECMWF page. Technically, WRF already has invariant data bundled with it but I found I had to download the ECMWF land-sea mask separately for WPS to work correctly. 

To summarise, you need three separate data files from the reanalysis data:

1. Surface variable data
2. Pressure level data (or model levels)
3. Land-sea mask.

## Downloading the data 
You'll be required to select which surface varibles you want to download, as well as which pressure levels, too. The land-sea mask is just a single file. Although probably not the most efficient method, I tend to just select all the surface variable for the surface data, (you don't actually need all of them to intiailise the model, but I find it easier to just download everything in case it's required later.) Once you've selected the date range, and varaibles of interest, you can proceed directly to the download by clicking the GRIB or netCDF download buttons. 

### Downloading via Python script
ECMWF have provided a very handy python API for downloading data without necessarily having to use the web interface, which can save time if you already know exactly which data fields you want. The details of the [Python API are here](https://software.ecmwf.int/wiki/display/WEBAPI/Access+ECMWF+Public+Datasets), it's well explained so I will only summarise here what you need to do:

1. Register on the ECMWF site
2. Download an access key to placed on the computer or system you are downloading to.
3. Install the ecmwfapi Python module.
3. Either write your own python script using the API documentation above, or have the ECMWF web interface generate one for you. (Click the 'view MARS request' after making your selections, and then copy the Python script that is displayed. An example python script looks like this:

{% highlight python %}
#!/usr/bin/env python
from ecmwfapi import ECMWFDataServer
server = ECMWFDataServer()
server.retrieve({
    "class": "e2",
    "dataset": "era20c",
    "date": "2009-06-01/to/2009-06-30",
    "expver": "1",
    "levtype": "sfc",
    "param": "15.128/16.128/17.128/18.128/31.128/32.128/33.128/34.128/35.128/36.128/37.128/38.128/39.128/40.128/41.128/42.128/53.162/54.162/55.162/56.162/57.162/58.162/59.128/59.162/60.162/61.162/62.162/63.162/64.162/65.162/66.128/66.162/67.128/67.162/68.162/69.162/70.162/71.162/72.162/73.162/74.162/75.162/76.162/77.162/78.128/78.162/79.128/79.162/80.162/81.162/82.162/83.162/84.162/85.162/86.162/87.162/88.162/89.162/89.228/90.162/90.228/91.162/92.162/131.228/132.228/134.128/136.128/137.128/139.128/141.128/148.128/151.128/159.128/164.128/165.128/166.128/167.128/168.128/170.128/174.128/183.128/186.128/187.128/188.128/198.128/206.128/229.128/230.128/231.128/232.128/235.128/236.128/238.128/243.128/244.128/245.128/246.228/247.228",
    "stream": "oper",
    "time": "00:00:00",
    "type": "an",
    "target": "CHANGEME",
})
{% endhighlight %}

ECMWF provide the data in two different file formats, GRIB (gridded-binary) and netCDF (.nc files). WPS comes with the ungribber tool (ungrib.exe) so I've gone for the grib data format here. (Selected by default in the Python download script).

## Important: Retrieveing the data in Gaussian gridded format
By default, the ECMWF site will download the data on a spherical harmonic grid, and the version of ungrib supplied in WPS v3.8.1 will not be able to decode this properly. (You may get the error ``Unknown ksec2(4): 50`` in the ungrib error log, if you try this). Ungrib expects a regular Gaussian grid, and I couldn't find a way to easily access this from the web portal interface. However, using the Python script above, you can easily request Gaussian gridded data by adding the parameter:

``"grid": "160",``

to the python download script, and your grib data will be supplied in Gaussian grid format. The `CHANGEME` value is the name of the downloaded file and you should probably change it to something meaningful. The python script will download the grib file to the same directory it is run in. For the example in this post, I ended up with three python scripts, one for the surface data, one for pressure levels, and one for the land-sea mask.

## Ungribbing the data
Now we need to 'ungrib' data to convert into the WPS intermediate file format, before running metgrid. This has to be done in two stages - one for the surface and pressure level data, and one for the land-sea mask. This is because the land-sea mask has a 'start date' of 1900-01-01 if you try to run ungrib with the date of your case study, ungrib will fail, complaining that the dates specified could not be found. In the namelist.wps file, I set the the `&ungrib` section to the following:

{% highlight bash %}
&ungrib
  out_format = 'WPS',
  prefix = 'FIX',
{% endhighlight %}

Link your land-sea mask to the WPS directory with the `link_grib.sh` script and run ungrib. Repeat again for the surface and pressure data but with the following section in the namelist.wps file:

{% highlight bash %}
&ungrib
  out_format = 'WPS',
  prefix = 'FILE',
{% endhighlight %}

To be honest, you can use whatever file prefixes you like, but I like to use this naming convention.

Make sure you have linked the correct Vtable before running ungrib. The ECMWF Vtable supplied with WRF v3.8.1 worked fine without any modifications in this case. The Vtable can be found in `WPS/ungrib/Variable_Tables/Vtable.ECMWF`

## Metgrid
Metgrid interpolates your ungribbed data files over the model domain. (I haven't gone through the model domain generation stage with geogrid.exe as this blog post is only about prepping the input data.) 

If you are using the sea surface temparatures field from the ECMWF data, you'll need to make a few changes to the METGRID.TBL file for it to correctly interpolate and mask the sea surface temparatures around land. IN the METGRID.TBL file (located in WPS/metgrid/), change the entry of the SST field to the following:

{% highlight bash %}
SST
  interp_option=sixteen_pt+four_pt+wt_average_4pt+wt_average_16pt+search
  missing_value=-1.E30
  masked=land
  interp_mark=LANDMASK(1)
  fill_missing=0.
  flag_in_output=FLAG_SST 
{% endhighlight %}

The changes are to make the interp_mask use the LANDMASK mask instead of LANDSEA (the default), and to change the interpolation option slightly. Without the changes, I found that for my inner domain the sea surface temperatures were incorrectly masked, and had been interpolated over land as well. Although metgrid.exe did not complain when run, the met files generated caused an error when real.exe was run - generating an error message saying:

{% highlight bash %}
-------------- FATAL CALLED ---------------
FATAL CALLED FROM FILE:  <stdin>  LINE:    2970
mismatch_landmask_ivgtyp
-------------------------------------------
{% endhighlight %}

The changes to METGRID.TBL should remedy this error message.

Before running metgrid, there is one last change to make to the namelist.wps file:

{% highlight bash %}
&metgrid
  fg_name = 'FILE',
  constants_name = 'FIX:1900-01-01_00',
{% endhighlight %}

This tells metgrid to use the invariant data we downloaded earlier as a constant field (i.e. the land-sea mask doesn't change over time, so it doesn't need to be interpolated for each period.) Use whichever name you have used for this invariant file. 

Now you can run metgrid. Hopefully it will produce all the input met files needed, and correctly interpolated. It's a good idea before running real.exe and wrf.exe to check that the fields look reasonable. In particular, check the SST field if you are using sea-surface temperatures. Check the nested domains as well - I found that my SST field had been incorrectly interpolated in the inner domain, which required the change to the METGRID.TBL file above. Ncview is a useful utility for checking the met files generated by metgrid. 

## Real and WRF
You are now set to run real.exe to generate the lateral boundary conditions and initialise the model, followed (finally) by wrf.exe to run the simulation. 


## Sources

The following discussion boards and mailing list answers were helpful in preparing this post:

http://www2.mmm.ucar.edu/wrf/users/FAQ_files/FAQ_wps_input_data.html
http://forum.wrfforum.com/viewtopic.php?f=22&t=1001
http://forum.wrfforum.com/viewtopic.php?f=6&t=4512




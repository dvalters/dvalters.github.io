---
layout: post
title: Floats and Ints in LSDRaster
modified: 2015-13-07
categories: [LSDRaster, LSDIndexRaster, data format]
tags: 
  - LSDRaster, LSDIndexRaster, data format
comments: true
---

 I have been having trouble with bil files if I import ASTER data on my windows machine. It seemed to be limited to windows but I now think the problem could be systematic.

I *think* I have fixed it but you'll need to check by rerunning some test analyses and making sure the behaviour is what you expect.

ENVI files have a 'DATATYPE' entry in the hdr file that give the number of bytes in each data point. ASTER data is 2 bytes. It turns out this is a 'short int' type. It can only go up to ~32000.

So 
1) I have changed the read raster functions to read short ints now
2) I have changed the LSDIndexRaster to go back to printing normal 4 bit ints so that we don't run out of numbers...I think this has happened to big DEMs where we need the NodeIndex. 


---
layout: page
title: Software
comments: yes
permalink: /software/
---

Some of the software projects I work on:

### Cylc
[Cylc](https://cylc.github.io/cylc/) is a Python application for managing workflows consisting of cycling tasks. Workflows in this context means any kind of ordered set of tasks that must be run when certain dependencies have been met. The software is heavily used in weather forecasting systems and climate modelling, where workflows may consist of many complex dependencies and thousands of cycling tasks. However Cylc is general purpose and can be used for a wide variety of applications, not just scientific.

### Rose
[Rose](http://www.metoffice.gov.uk/research/modelling-systems/rose) is a tool for configuring and running meteorological suites. For example numerical weather prediction models and climate models.

### LSDTopoTools
[LSDTopoTools](https://lsdtopotools.github.io/) is a set of programs for topographic analysis and modelling, as well as visualising topographic data from the analyses. The core analysis algorithms are written in C++ and the visualisation software, [LSDMappingTools](https://github.com/LSDtopotools/LSDMappingTools) in Python.

### HAIL-CAESAR
[HAIL-CAESAR](https://github.com/dvalters/HAIL-CAESAR) is a hydrological-land surface erosion model for simulating river catchment processes, such as flooding, fluvial erosion, and sedimentation. It is a spin off model from the CAESAR-Lisflood model developed by Tom Coulthard and others. It was developed during my PhD when I needed a portable model for running ensemble simulations of river catchments on cluster computers and HPC services.

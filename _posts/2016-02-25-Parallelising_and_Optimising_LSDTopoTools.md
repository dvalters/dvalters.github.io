---
layout: post
title: Parallelising and Optimising LSDTopoTools - DAV
categories: articles
tags: openmp parallelisation hpc
---

Some notes and experiences from scaling LSDTopoTools code to parallel computing facilities (HPC). I'm using the OpenMP standard, which is widely used and is applicable to all kinds of computing architectures. It will also work on your desktop pc/laptop if you have a multicore CPU.

### Identifying hotspots and 'parallelisable' functions

An example with LSDCatchment model: First, I profiled the code to find where the bottlenecks are. I used gprof for this. Before you compile the code however, remember to enable the profiling flags `-pg` in the makefile. When you run the compiled code again, you will get the gmon.out file. This contains all the profiling data. To generate some useable output from this, run:

{% highlight console %}
gprof MyLSDExecutable.out gmon.out > analysis.txt
{% endhighlight %}

The anlysis text file will give you some output like so:

{% highlight console %}
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls  ms/call  ms/call  name    
 77.71   5993.06  5993.06   234993    25.50    25.50  LSDCatchmentModel::qroute()
 17.63   7352.97  1359.91   234993     5.79     5.79  LSDCatchmentModel::depth_update()
  3.58   7629.10   276.13    46998     5.88     5.88  LSDCatchmentModel::scan_area()
  1.02   7707.40    78.29   234993     0.33     0.34  LSDCatchmentModel::catchment_water_input_and_hydrology(double)
  0.03   7709.43     2.03   234993     0.01     0.01  LSDCatchmentModel::water_flux_out(double)
  0.01   7710.29     0.86        7   122.86   191.43  LSDCatchmentModel::get_area4()
  ...
{% endhighlight %}

The analysis tells us that there is one function in this code taking up the bulk of the compute time: `qroute()`. (In this case it's a water-routing algorithm. Followed by a water depth update function and an area scanning function. This is quite a good scenario (we don't have to parallelise dozens of function to get a good return).

### Parallelisation

I used the OpenMP library to parallelise the code. OpenMP is a magical set of preprocessor directives that you add to sections of your code to run them in parallel. It does some clever stuff to your code when it compiles. For some light bedtime reading, you could consult the [568-page specification document.](http://www.openmp.org/mp-documents/openmp-4.5.pdf)

Okay, now you've read that, you  will know that you need to add the `#include <omp.h>` include file in one of your header or source files. The OpenMP libraries are fairly standard bundles with most compilers. If you have the gcc-devel package installed, you will almost certainly have the OpenMP libraries and thus the `omp.h` header file.

Secondly, you need to identify the part of the function that can be parallelised. I use the `scan_area()` function as a simple example. (`qroute()` is not so straightforward...). This function has a for loop that iterates over every element in a 2D raster matrix. It then checks to see if the water depth  is greater than zero, and sets a value in another array based on this test. This loop is easily parallelisable because the iterations do not depend on each other, and could be calculated in any order. 

{% highlight cpp %}
  #pragma omp parallel for
  for (int j=1; j <= jmax; j++)
  {
    int inc = 1;
    for (int i=1; i <= imax; i++)
    {
      // zero scan bit..
      down_scan[j][i] = 0;
      // and work out scanned area by checking the 8 surrounding cells and centre cell.
      if (water_depth[i][j] > 0
          || water_depth[i][j - 1] > 0
          || water_depth[i][j + 1] > 0
          || water_depth[i - 1][j] > 0
          || water_depth[i - 1][j - 1] > 0
          || water_depth[i - 1][j + 1] > 0
          || water_depth[i + 1][j - 1] > 0
          || water_depth[i + 1][j + 1] > 0
          || water_depth[i + 1][j] > 0
          )
      {
        down_scan[j][inc] = i;
        inc++;
      }
    }
  }

{% endhighlight %}

When the code runs with OpenMP enabled, the iterations in this loop will be distributed to different threads and cpus. (More on this later). I put in similar `#pragma omp parallel for` lines in the two other expensive functions found by the profiler.

Note that jmax and imax are variables declared out of scope of the for loop. We don't need to worry about them being modified or changing their value since we only check their value. Same for `water_depth[][]` - we are just reading the array, not modifying it. When the parallel region is reached, it is divided up between the number of threads specified before runtime (see later section). Since the value if jmax is known, each thread will perform *jmax/threads* number of iterations. Each thread will have its own private copy of all the variables declared inside the loop. 

What happens at the inner loop? Each thread must carry out the inner loop over its full number of iterations, i.e. *imax*. (There is no further 'nested' parallelism here.) The shared `down_scan[][]` array is modified, but there is no chance of accessing/overwriting the wrong part of the matrix because: 1) the j indices are unique to each outer loop thread - this is taken care of by the `#pragma omp for` statement, and 2) inc is always local to the inner loop iterations, and always starts at 1 in every thread.

Note the variable/incrementer `inc++`. Luckily, `inc` is declared as a local variable inside the outer for loop. I.e. every outer loop iteration starts with a fresh version of `int inc = 1;`. If this was not the case, and inc was a global variable, we would be in trouble - inc could be read and written to by different threads at different times, so the final value could be off. (There is a solution to this using a [reduction](https://computing.llnl.gov/tutorials/openMP/#REDUCTION), but thankfully that is not the case here - although it is a common occurrence). You will need to check which loops are suitable for parallelisation in your own code. This is the tricky bit - not all loop iterations are independent of each other and parallelise so easily. It is best to consult a brief tutorial on OpenMP for how to do this, and look at other functioning examples. So I won't go into this in any more details. Here are some useful tutorials:

[Dr Dobbs - Getting Started with OpenMP](http://www.drdobbs.com/getting-started-with-openmp/212501973)

[Lawerence Livermore National Lab - OpenMP tutorial](https://computing.llnl.gov/tutorials/openMP/)

ARCHER (the UK national supercomputing centre) also list details of upcoming training courses, and you can look through their training slides and exercises here:

[Open MP Tutorial by ARCHER - Course Materials](http://www.archer.ac.uk/training/course-material/2015/12/ShMem_OpenMP_York/index.php)

### Compiling and running

You need to add just one extra flag to your makefile: `-fopenmp` (for gcc users). That's it. You probably want to turn on an optimisation flag as well (e.g. -O2), unless you are debugging. 

Once compiled, you can run the executable on your computer/cluster. All you need to do beforehand is set the number of threads that you want the code to use. (It can't always detect this from your environment, so best to set it explicitly). In the linux terminal this is done like so (I have no idea how you do it in Windows).

{% highlight console %}
export OMP_NUM_THREADS=8
{% endhighlight %}

Where 8 should usually be the same as the number of available cores on your machine. Usually this means 1 physical processor, which has several physical cores on the chip. If you are running this in some sort of cluster-environment, OpenMP is limited to accessing a single compute node and the memory associated with that node. You may have more than one 'compute node' available, but an OpenMP application cannot distribute a program across multiple separate nodes. (Read the Dr Dobbs link above if this is unclear.) 

Then just run the executable as normal. You can check the usage of your cpus using the `top` command in linux. Then press 1 to get a breakdown of usage by CPU. You should hopefully see that all of your cpus (cores) are now in use, compared to just one that would have been in use when running the code serially.

### Results from porting to ARCHER

ARCHER is a massive HPC cluster housed in Edinburgh used for academic reserach in the UK. It's made up of a load of compute nodes (2000+ of them). Each node has 2x 12 core processors sharing 64GB of RAM. There is some clever technology called HyperThreading which you can turn on to 'double' the number of logical cores available, giving you a maximum of 48 processing units. (Confusingly, these 48 logical cores are referred to as CPUs, even though they aren't physical CPUs like we might think of normally.)

I did some simulations with LSDCatchmentModel with a variety of core/thread configurations. The test simulation was 48hrs (real time) of the Boscastle storm and flooding in 2004. In serial mode (i.e. no parallelisation) the simulation took around c. 4 hours. With the optimised, parallel code, I could get this down to about 11 minutes in the best case scenario. 


| Cores/CPUs   |Hyperthreading|  OpenMP Threads  |  Time        |
|-------------:|:------------:|:----------------:|:------------:|
| 1*           | no           |  1               | c. 4 hours   |
| 8*           | yes          |  8               | c. 50 mins   |
| 12           | no           |  12              | 28 mins      |
| 24           | yes          |  24              | 23 mins      |
| 24           | no           |  24              | 15 mins      |
| 48           | yes          |  48              | 11 mins      |

*These 2 simulations were done on a desktop PC with broadly comparable single CPU to the ARCHER nodes, so it's not quite a fair test, but close enough. The rest were done on the standard Archer compute node. 48 cores is the maximum available on a single compute node.

#### Comments on scaling

Linear speed is theoretically possible (i.e. speed up simply a multiplier of the number of processors), but unlikely to happen perfectly in practice. The scaling here is roughly sub linear. Hyper-threading (running in effect two logical cores on a physical core, 'doubling' the number of cpus available) is rarely as effective as true core increases, though the gains are noticeable here. See [Amdahl's Law](https://en.wikipedia.org/wiki/Amdahl%27s_law) if you are really curious about how far this could scale...I think (as a rough guess), it's approaching the region of diminshing returns. Unfortunately I don't have enough CPUs/cores to test it further!

### Some general thoughts about LSDTopoTools and parallelisation

> Warning: These ideas/ramblings have not been verified by an actual computer scientist...

A lot of the algorithms in LSDTT involve iteration over a loop on elements of an array or 2D matrix. On a high resolution or large DEM this is potentially hundreds of thousands, if not *millions*, of calculations in a single function call. Other algorithms, such as the segment-fitting algorithm, check millions of permuations. I am generalising here, but many of the algorithms' iterations are independent of one another. In other words, you don't always need the answer from the current iteration before you can start the next one. This makes a lot of the analyses ripe for parallelisation. The hillshade algorithm is probably a classic simple example.

{% highlight cpp %}
    for (int i = 1; i < NRows-1; ++i){
        for (int j = 1; j < NCols-1; ++j){
            float slope_rad = 0;
            float aspect_rad = 0;
            float dzdx = 0;
            float dzdy = 0;

            if (RasterData[i][j] != NoDataValue){
                dzdx = ((RasterData[i][j+1] + 2*RasterData[i+1][j] + RasterData[i+1][j+1]) -
                       (RasterData[i-1][j-1] + 2*RasterData[i-1][j] + RasterData[i-1][j+1]))
                        / (8 * DataResolution);
                dzdy = ((RasterData[i-1][j+1] + 2*RasterData[i][j+1] + RasterData[i+1][j+1]) -
                       (RasterData[i-1][j-1] + 2*RasterData[i][j-1] + RasterData[i+1][j-1]))
                       / (8 * DataResolution);

                slope_rad = atan(z_factor * sqrt((dzdx*dzdx) + (dzdy*dzdy)));

                if (dzdx != 0){
                    aspect_rad = atan2(dzdy, (dzdx*-1));
                    if (aspect_rad < 0) aspect_rad = 2*M_PI + aspect_rad;
                }
                else{
                    if (dzdy > 0) aspect_rad = M_PI/2;
                    else if (dzdy < 0) aspect_rad = 2 * M_PI - M_PI/2;
                    else aspect_rad = aspect_rad;
                }
                hillshade[i][j] = 255.0 * ((cos(zenith_rad) * cos(slope_rad)) +
                                  (sin(zenith_rad) * sin(slope_rad) *
                                  cos(azimuth_rad - aspect_rad)));

                if (hillshade[i][j] < 0) hillshade[i][j] = 0;
            }
        }
{% endhighlight %}

Basically the algorithm looks around to the cell neighbours and performs some fairly trivial calculation. But none of the calculations depend on the answers from other iterations (Again...I haven't tested this yet, see below for an actual tested and working example). You could probably parallelise this by placing a single statement before the outer `for` loop:

{% highlight cpp %}
#pragma omp parallel for
for (int i = 1; i < NRows-1; ++i){
    for (int j = 1; j < NCols-1; ++j){
    // Do stuff...
{% endhighlight %}

Finally, most of the computers we use have some multi-core capability. Even your laptop probably has 2 or 4 cores. All this computing power is sitting there waiting to be put to good use! Depending on how parallelisable the algorithm is, you would expect to get something like a 3-3.5x speed up on a 4 core machine compared to running the algorithm in serial. (You rarely get *n* times speed up (where *n* is number of cores) because of overheads, communicating between cores/memory etc.)





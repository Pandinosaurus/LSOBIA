LSOBIA
======

__Large Scale Object Based Image Analysis__ (LSOBIA) is a module for
the [Orfeo Toolbox](https://www.orfeo-toolbox.org/) (OTB). It provides
several tools for object based, large scale remote sensing image,
analysis.

The module contains 5 OTBApplications:

* __LSSegmentation__ (Large Scale Segmentation) provides several
  methods to perform segmentation of very high resolution images.
* __LSSmallRegionsMerging__ (Large Scale Image Small Regions Merging)
  provides a method to perform small regions merging of very high
  resolution images.
* __Polygonize__ provides several methods to perform polygonization of
  high resolution images.
* __LSPolygonize__ (Large Scale Polygonize) is a distributed version
  of Polygonize. _work in progress_
* __ComputeAttributes__ computes several attributes of a vector file.

# Table of contents
1. [Getting started](#getting-started)
2. [Some results](#some-results)
3. [Under the hood](#under-hood)
4. [License](#license)

<a name="getting-started"></a>
# Getting started

## Dependencies

* [OTB](https://www.orfeo-toolbox.org/)
* The C++ library for the [Message Passing
  Interface](https://www.mpi-forum.org/) (MPI) (developed and
  validated with mpich-3.2)

Since LSOBIA is a module for the OTB, one need to install the OTB in
order to build it.

LSOBIA allows the distribution of the calculation on a computation
cluster, therefor it uses MPI for the communications between the
clusters.

## Building
LSOBIA can be built like any other [otb remote
module](https://wiki.orfeo-toolbox.org/index.php/How_to_write_a_remote_module)
You can build it either from within OTB's sources or outside it.

Don't forget to activate C++14 by setting the cmake parameter
_"CMAKE\_CXX\_FLAGS"_ to _"-std=c++14"_.

## Usage

LSOBIA can run on a single processor, or be distributed over multiple processors.

### Mono-processor execution

To run an application on a silgle processor, one need to call the
application using the OTB application launcher process:

```bash
otbApplicationLauncherCommandLine ${AppName} ${AppDirectory} ${Parameters}
```

* ${AppName} is the name of the application (ie: LSSegmentation or
  Polygonize).
* ${AppDirectory} is the path to the directory containing the compiled
  applications.
* ${Parameters} are the parameters for the application.

Example:

```bash
otbApplicationLauncherCommandLine LSSegmentation /home/me/bin/lsobia/lib/otb/applications -io.im inputimage.tif -io.out.dir /home/me/out -io.temp /tmp -algorithm baatz -algorithm.baatz.maxiter 45 -processing.memory 10000 -processing.nbproc 8 -processing.nbtilesperproc 2 -processing.writeimages on -processing.writegraphs off
```

### Multi-processor execution

Simply add "*mpirun -np ${NumberProcessor}*" before the previous
command. ${NumberProcessor} is the number of processors to be used for
the computation.

Example:

```bash
mpirun -np 8 otbApplicationLauncherCommandLine LSSegmentation /home/me/bin/lsobia/lib/otb/applications -io.im inputimage.tif -io.out.dir /home/me/out -io.temp /tmp -algorithm baatz -algorithm.baatz.maxiter 45 -processing.memory 10000 -processing.nbproc 8 -processing.nbtilesperproc 2 -processing.writeimages on -processing.writegraphs off
```

<a name="some-results"></a>
# Some results

## Baatz segmentation

We applied the LSSegmentation application with the Baatz algorith on
this image, and obtained the following label image as output. The
process used a single processor for the computation.

```bash
otbApplicationLauncherCommandLine LSSegmentation /home/me/bin/lsobia/lib/otb/applications "-io.im" "${INPUT_IMAGE}" "-io.out.dir" "${OUTPUT_DIRECTORY}" -io.out.labelimage "LabelImage" "-io.temp" "${TEMP}" "-algorithm" "baatz" "-algorithm.baatz.numitfirstpartial" "5" "-algorithm.baatz.numitpartial" "5" "-algorithm.baatz.stopping" "40" "-algorithm.baatz.spectralweight" "0.5" "-algorithm.baatz.geomweight" "0.5" "-algorithm.baatz.aggregategraphs" "on" "-processing.writeimages" "on" "-processing.writegraphs" "on" "-processing.memory" "2000" "-processing.maxtilesizex" "1000" "-processing.maxtilesizey" "1000"
```

Input image:

![input file](figures/sample_image1.png)

Result:

![baatz-segmentation](figures/sample_result1.jpg)

<a name="under-hood"></a>
# Under the hood

## Base data structure

### Adjacency graph

Adcacency graph are often used when dealing with OBIA
segmentation. They are a way to represent the objects on the image,
and their proximity. Each node of the graph represents an object
(eather spectral or geometric), and each edge of the graph represent
the fact that two nodes are neighbors.

In LSOBIA, we use a list to represent a graph. The list contains the
nodes. Each node contains an other list: the edges to the adjacent
nodes. As a reminder, two nodes are spatially adjacent if they share a
common border in the image.

For instance, let's consider a 3x3pixels image, each pixel being a
node of the graph. Let's call *0* the pixel in the corner top left,
and *8* the pixel in the corner bottom right. The corresponding
agjacency graph would be as follow:

![adjacency graph](figures/adj_graph.png)

### Contour

The position of the objects in the image is important. Therefor, each
node also contains a contour representing the object encoded as a
[freeman
chain](http://www.mif.vu.lt/atpazinimas/dip/FIP/fip-Contour.html).



<a name="license"></a>
# License
This project is licensed under the Apache License 2.0. Please see the
[LICENSE file](LICENSE) for legal issues on the use of the software.

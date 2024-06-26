ROI segmentation with Cellpose
================

- [Overview](#overview)
- [Image pre-processing](#image-pre-processing)
- [Cellpose to get cell and/or nuclear
  outlines](#cellpose-to-get-cell-andor-nuclear-outlines)
  - [Locally](#locally)
    - [Installation on Windows](#installation-on-windows)
    - [Note: Mac (as of October 2022)](#note-mac-as-of-october-2022)
    - [The GUI](#the-gui)
    - [Jupyter Notebook](#jupyter-notebook)
  - [On a remote server](#on-a-remote-server)
    - [In the command line or a
      script](#in-the-command-line-or-a-script)
- [References](#references)

## Overview

This pipeline is designed to provide single-cell resolution gene
expression data to the qualitative, visual data generated by v3
HCR-FISH. This pipeline assumes basic familiarity with the command line,
[Anaconda package manager](https://anaconda.org), and Jupyter Notebook
or a text editor like Vim.

## Image pre-processing

After imaging you should have already processed the raw images with
AiryScan processing, stitching, and max intensity projection. Then you
will need to divide your processed whole-tissue, multi-channel image to
separate out tissue of interest (blastema, epithelium, and mesenchyme,
for example) as well as split the color channels. [The instructions for
doing this are here](./Tissue_Isolation.md).

## Cellpose to get cell and/or nuclear outlines

Cellpose is a machine-learning algorithm designed for cell and nucleus
segmentation in both 2D and 3D. There is [support for a GUI](#the-gui)
as well as [use in the command line](#in-the-command-line-or-a-script)
and [Jupyter Notebook](#jupyter-notebook). It also [has
documentation](https://cellpose.readthedocs.io/en/latest/) and [a
detailed GitHub page](https://github.com/MouseLand/cellpose) that makes
it easy to navigate.

### Locally

#### Installation on Windows

Follow [the instructions here](https://github.com/MouseLand/cellpose)
for how to install Cellpose on a Windows or Linux system, including if
you want GUI support or not.

#### Note: Mac (as of October 2022)

There is a dependency of Cellpose that is currently too out-of-date to
be compatible with the newest Mac software. If you follow the
instructions above for a Mac/Linux system, you will successfully install
Cellpose, but it will not be able to utilize your computer’s GPU, and
instead will resort to CPUs (which it will tell you if you try to run it
in the command line or a Jupyter Notebook). Cellpose will attempt to
run, but it could take upwards of an hour to segment one image, and eat
up enough RAM in the process that it could crash your machine. If you
have a Mac, and this dependency issue is not yet resolved, I recommend
to [instead use a computing cluster](#on-a-remote-server).

#### The GUI

If you installed Cellpose with the GUI above, you will have the ability
to open a desktop application to run Cellpose through. This is good for
when you only have a few images, but as your numbers grow, you’ll want
to consider automating the segmentation in [something like
Jupyter](#jupyter-notebook) or [in a script
file](#in-the-command-line-or-a-script). [Visit the Cellpose
documentation](https://cellpose.readthedocs.io/en/latest/gui.html) for
more information about the GUI.

The most important thing is that, when Cellpose is done segmenting and
you’re happy with the results, go to `File` in the top-left and **save
the text outlines** of the segmentation. No other output is necessary
for this pipeline.

#### Jupyter Notebook

There is [an example notebook for running Cellpose segmentation
here](https://nbviewer.org/github/MouseLand/cellpose/blob/master/notebooks/run_cellpose.ipynb).
I also have [a Notebook for running Cellpose in a loop
here](./scripts/run_cellpose.ipynb). Be sure to **always save the text
outlines**! You don’t need any other output for the purposes of this
analysis.

### On a remote server

Reach out to your organization’s computing cluster support to gain
access to your cluster. With `anaconda` loaded in your `$PATH`, follow
[the instructions on the Cellpose GitHub
page](https://github.com/MouseLand/cellpose) to install Cellpose exactly
as you do locally, but without the GUI. You may have to use the steps to
install `cudatoolkit` for Linux if you are unable to connect to GPU. The
details of installing and running Cellpose on a remote cluster can be
particular to your organization’s infrastructure, so reach out to your
cluster’s support if you need help.

#### In the command line or a script

With Cellpose installed in an Anaconda environment, it’s possible to run
Cellpose directly from the command line or a job script. There is [a
breakdown of command line commands
here](https://cellpose.readthedocs.io/en/latest/command.html). I’ve
found that Cellpose sometimes struggles with `--diameter` set to `0` for
automatic diameter estimation, so play around to find a good value. This
may take some trial and error to find a diameter that consistently
segments well. The other important part is to use the `--save_txt` flag
to save the text outlines.

You can also write these commands into a job script for batch
segmentation. The job submission guidelines will be particular to your
organization’s job manager. Create your `conda` environment as before.
Then, use `vim` to create `cellpose.sh`:

``` bash
$ vim cellpose.sh
```

You can then enter the following ([a copy is available in the scripts
directory](./scripts/cellpose.sh)):

``` bash
#!/bin/bash

conda activate cellpose

# Edit the parameters & paths below for your segmentation
# Descriptions of parameters and additional flags are available in the Cellpose documentation
python -m cellpose \
  --verbose \
  --use_gpu \
  --dir </path/to/images/> \
  --pretrained_model cyto2 \
  --chan 0 --chan2 0 \
  --diameter 0 \
  --save_txt \
  --no_npy
```

Look at [the cellpose
documentation](https://cellpose.readthedocs.io/en/latest/command.html)
for more information on how to run Cellpose in the command line. For a
grayscale cytoplasm or nuclear image with no other color, `--chan` and
`--chan2` should both be `0`.

## References

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-RN149" class="csl-entry">

Choi, Harry M. T., Maayan Schwarzkopf, Mark E. Fornace, Aneesh Acharya,
Georgios Artavanis, Johannes Stegmaier, Alexandre Cunha, and Niles A.
Pierce. 2018. “Third-Generation in Situ Hybridization Chain Reaction:
Multiplexed, Quantitative, Sensitive, Versatile, Robust.” Journal
Article. *Development* 145 (12): dev165753.
<https://doi.org/10.1242/dev.165753>.

</div>

<div id="ref-9387490" class="csl-entry">

Granger, Brian E., and Fernando Pérez. 2021. “Jupyter: Thinking and
Storytelling with Code and Data.” *Computing in Science & Engineering*
23 (2): 7–14. <https://doi.org/10.1109/MCSE.2021.3059263>.

</div>

<div id="ref-RN152" class="csl-entry">

Pachitariu, Marius, and Carsen Stringer. 2022. “Cellpose 2.0: How to
Train Your Own Model.” Journal Article. *Nature Methods* 19 (12):
1634–41. <https://doi.org/10.1038/s41592-022-01663-4>.

</div>

<div id="ref-RN151" class="csl-entry">

Stringer, Carsen, Tim Wang, Michalis Michaelos, and Marius Pachitariu.
2021. “Cellpose: A Generalist Algorithm for Cellular Segmentation.”
Journal Article. *Nature Methods* 18 (1): 100–106.
<https://doi.org/10.1038/s41592-020-01018-x>.

</div>

</div>

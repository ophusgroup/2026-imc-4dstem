---
title: Phase contrast 4D-STEM imaging of simple systems
short_title: Ptychography basics
---

:::{admonition} Learning goals
:class: tip
- Run DPC, parallax, and single-slice ptychography reconstructions on an experimental dataset.
- Calibrate rotation and defocus from the data itself, and seed the ptychography with parallax estimates.
- Judge reconstruction quality: convergence, phase sign, and artifacts.
:::

This hands-on session walks the full phase-retrieval ladder on a *simple system*: a thin, well-behaved experimental dataset (gold nanoparticles on carbon) where every method works and the differences between them are easy to see.

## The workflow

1. **Load and inspect.** Mean and max patterns, virtual BF/ADF images. With a defocused probe the virtual images look blurry; that is expected, and it is exactly the overlap that ptychography will exploit:

:::{figure} ../assets/movies/scan-defocused-probe.mp4
:alt: Animation of a defocused probe rastered over a 2D crystal, showing the smooth bright field disk in vacuum
:width: 75%
The input data: a defocused probe rastered over a 2D crystal. Without the sample, the enlarged bright field disk is smooth.
:::

:::{figure} ../assets/movies/scan-defocused-sample.mp4
:alt: Animation of the same scan with the sample, showing interference speckle inside the overlapping disks
:width: 75%
With the sample in place, the overlapping disks fill with interference speckle. That speckle encodes the relative phases of the diffracted beams, and it is exactly what direct (SSB) and iterative ptychography decode.
:::

2. **DPC.** Compute the center of mass, solve for the scan–detector rotation (clean dipoles in CoMx/CoMy), and integrate to get a first phase image. Check the sign: nanoparticles must come out as *positive* phase.
3. **Parallax.** Align the virtual images from within the BF disk; the cross-correlation shifts visualize the aberration surface, fit the defocus and astigmatism, and kernel-density upsampling recovers detail beyond the scan step.
4. **Ptychography.** Seed a single-slice reconstruction with the rotation and defocus measured by parallax, then iterate: the solver jointly refines the complex object and the probe. The reconstructed object is complex, with both amplitude and phase:

:::{figure} ../assets/figures/phase-amplitude-wheel.jpg
:alt: Color wheel showing phase as hue and amplitude as brightness
:width: 45%
Reconstructed objects are complex-valued: phase (hue) and amplitude (brightness). For thin samples nearly all of the information is in the phase.
:::

## Why this matters

Phase contrast reconstructions are far more dose-efficient than incoherent imaging for thin samples: the same information comes out at a fraction of the fluence, which is what makes atomic-resolution imaging of beam-sensitive materials possible. The iterative framework also opens the door to super-resolution beyond the aperture limit [](doi:10.1038/s41586-018-0298-5), aberration correction in software, and multislice reconstructions that recover depth information [](doi:10.1126/science.abg2533).

An overview of the open-source implementations of these methods (and how they relate) is on the [software page](./software.md); the iterative algorithms themselves are reviewed in [arXiv:2309.05250](https://arxiv.org/abs/2309.05250).

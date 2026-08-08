---
title: Phase contrast 4D-STEM imaging
short_title: Phase contrast 4D-STEM
---

:::{admonition} Learning goals
:class: tip
- Understand why phase contrast is the dose-efficient way to image weakly scattering samples.
- Follow the ladder of 4D-STEM phase-retrieval methods: center of mass / DPC, tilt-corrected bright field (parallax), direct ptychography, and iterative ptychography.
- Know what changes at high convergence angle, where the Bragg disks overlap, and why dose (not just contrast transfer) decides which method wins.
:::

The nanobeam sessions used well-separated diffraction disks and asked *where* the diffraction spots are. The afternoon asks a different question: *what phase shift did the sample imprint on the electron wave?* Thin samples are nearly transparent to fast electrons; they barely absorb, but they do shift the phase of the beam in proportion to their projected electrostatic potential. Recovering that phase gives the most dose-efficient imaging available for thin, weakly scattering, and beam-sensitive samples [](doi:10.1017/S1431927619000497).

## From nanobeam to overlapping disks

The convergence angle now plays the opposite role from the morning. At small α, the diffracted disks are separate and their *positions* carry the information. At large α, the disks overlap, and the *interference* in the overlap regions carries the information: the phase difference between the direct beam and each diffracted beam is encoded in intensity modulations that shift as the probe scans.

:::{figure} ../assets/movies/dpc-concept.mp4
:alt: Animation of a probe scanning across a one-dimensional potential, with the bright field disk responding to the local field
:width: 75%
A probe scanning across a sample potential (red) at two convergence angles: the bright field disk responds to the local electric field. This is the signal that center-of-mass imaging reads out.
:::

:::{figure} ../assets/movies/com-dpc-overlap.mp4
:alt: Animation showing the center of mass of a large overlapping bright field disk shifting as the probe crosses the potential
:width: 75%
The same measurement in the overlapping-disk regime: the center of mass (cyan) of the full diffraction pattern tracks the momentum transferred to the probe as it crosses the potential.
:::

## The method ladder

- **Center of mass / DPC.** The center of mass of each diffraction pattern measures the average in-plane momentum transferred to the beam, which is proportional to the electric field of the sample. The field is the gradient of the potential, so integrating the center-of-mass signal reconstructs the phase [](doi:10.1038/nphys2337). Fast, simple, and a good diagnostic: a correct scan–detector rotation gives clean dipole contrast along x in CoMx and along y in CoMy, so this doubles as the rotation calibration.
- **Tilt-corrected bright field (parallax).** Every pixel inside the bright-field disk is a plane-wave image of the sample from a slightly different angle. With a defocused probe, these images are shifted copies of each other; cross-correlating and aligning them recovers the phase, measures the aberrations (the shift field is the gradient of the aberration surface), and can be upsampled beyond the scan sampling.
- **Direct (single side-band) ptychography.** Integrate the interference in the disk-overlap ("double overlap") regions analytically, correcting phase and amplitude variations in a single pass.
- **Iterative ptychography.** Solve for the complex object (amplitude and phase) that explains *all* of the recorded interference at once, while simultaneously recovering the probe [](doi:10.1063/1.1823034), [](doi:10.1016/j.ultramic.2009.05.012). Iterative solvers deconvolve the probe from the object and reach resolutions beyond the aperture limit [](doi:10.1038/s41586-018-0298-5).

How well each method transfers spatial frequencies is summarized by its contrast transfer function (CTF), and the ranking depends strongly on focus [](doi:10.1093/mam/ozag005):

:::{figure} ../assets/figures/ctf-ladder.jpg
:alt: Contrast transfer functions of center of mass, parallax, direct ptychography, and iterative ptychography, in focus and at 200 angstrom defocus
:width: 100%
The method ladder in transfer-function form: center-of-mass imaging wants to be in focus, parallax requires defocus, direct ptychography recovers the parallax zero crossings, and iterated ptychography approaches unity transfer. Adapted from Varnavides et al. [](doi:10.1093/mam/ozag005).
:::

## Beyond contrast transfer: dose decides

The CTF describes the *maximum* transmittable signal and ignores the Poisson statistics of finite electron dose, so it can badly overestimate practical performance in exactly the low-dose regime where phase contrast matters most. The spectral signal-to-noise ratio (SSNR) is the finite-dose metric: center-of-mass, parallax, and direct ptychography have dose-independent SSNR shapes, while iterative ptychography behaves like direct ptychography at low dose and only unlocks its full transfer as fluence increases [](doi:10.1093/mam/ozag005):

:::{figure} ../assets/figures/ssnr-dose.jpg
:alt: Spectral signal-to-noise ratio curves for the phase retrieval methods at increasing dose
:width: 100%
Spectral SNR at finite dose: iterative ptychography (bottom row) is dose-*dependent*, converging to direct ptychography at low fluence and saturating at high fluence. Method choice is a dose question, not just a CTF question. Adapted from Varnavides et al. [](doi:10.1093/mam/ozag005).
:::

## Practical notes

- **Defocus is a design parameter.** Center-of-mass imaging and virtual imaging want a focused probe; parallax and ptychography *want* defocus, because the enlarged probe footprint creates the real-space overlap between neighboring scan positions that the reconstructions exploit.
- **The 180° rotation ambiguity is physical.** An incorrect rotation solution inverts the reconstructed phase. Check the sign: atoms are positive potentials, so the phase shift over an atomic site must be positive.
- **Phase wraps on strong scatterers.** Thin, light samples are ideal; for thicker crystals, multiple scattering demands multislice-aware reconstructions.

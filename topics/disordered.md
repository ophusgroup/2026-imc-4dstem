---
title: "Disordered materials: polymers and pair distribution functions"
short_title: Polymers & PDF
---

:::{admonition} Hands-on tutorials
:class: important
- ▶️ [Flowline mapping of a semicrystalline polymer](https://drive.google.com/file/d/17G99GpRyoZBK7xFpPUVF-2AHsrq8nsnq/view?usp=drive_link): from diffuse arcs to orientation histograms to flowline maps, working in polar coordinates.
- ▶️ [Pair distribution function of amorphous Ta](https://drive.google.com/file/d/1-fHnUWhyzVWk207TTyJSy-cSfJaQIXL8/view?usp=drive_link): compute I(k), S(k), and G(r), and validate them against the ground-truth atomic coordinates of the simulated sample.
:::

:::{admonition} Learning goals
:class: tip
- Design a 4D-STEM experiment within the dose budget of a beam-sensitive material.
- Render orientation fields as flowline maps and read connectivity, domains, and defects from them.
- Describe disordered structure with pair distribution functions, and read the signatures of nanocrystalline, medium-range-ordered, and liquid-like structure in g(r).
- Compute an electron PDF from 4D-STEM data: polar transform, background fit, structure factor, sine transform.
:::

This session covers the two main classes of disordered and partly ordered materials, and the workshop will take an audience vote on which hands-on demo to run: polymer flowline mapping or the pair distribution function of an amorphous metal. Both notebooks are linked above, so you can run the other one at home.

## Semicrystalline polymers

Semicrystalline polymers and small-molecule organic films (conjugated polymers for organic electronics, polyolefins, peptide and protein assemblies) derive their properties from nanoscale crystalline domains embedded in an amorphous matrix. Charge transport in an organic semiconductor, for example, depends on how the π-stacking direction of crystallites connects across the film. These materials are essentially impossible to characterize by conventional high-resolution imaging: they are extremely beam sensitive, with critical fluences of order 1–100 e⁻/Å², destroyed long before an atomic-resolution image can be formed [](doi:10.1021/acs.accounts.1c00073).

Nanobeam 4D-STEM sidesteps this. Diffraction concentrates the structural information from the whole illuminated volume into a few sharp features, so a useful diffraction pattern can be recorded with orders of magnitude fewer electrons than an image [](doi:10.1016/j.micron.2016.05.008), and with a fast camera, the dose is spread over the full field of view in a single low-fluence pass.

### From diffraction patterns to orientation maps

Polymer crystallites typically produce a strong, characteristic reflection (e.g., the ~3.6 Å π–π stacking peak in P3HT, or lamellar/backbone reflections at lower angles). Because of local disorder, these appear as azimuthal *arcs* rather than sharp spots:

:::{figure} ../assets/figures/polymer-diffraction.png
:alt: A nanobeam diffraction pattern from a semicrystalline polymer showing azimuthal arcs
:width: 55%
A single nanobeam pattern from a semicrystalline polymer: the characteristic reflections appear as azimuthal arcs whose angular position encodes the local crystallite orientation.
:::

The workflow:

1. **Detect the arcs.** At each probe position, locate the azimuthal position of the characteristic reflection. Polar transformation of each pattern makes the azimuthal intensity distribution easy to fit, and peak *prominence* (rather than absolute intensity) is the robust detection criterion for weak, diffuse signal.
2. **Map orientation.** The azimuthal angle of the arc gives the local crystallite orientation (modulo the symmetry of the reflection); its intensity gives the degree of crystallinity/alignment. The result is an orientation field over the scanned area.
3. **Draw flowlines.** Orientation fields are hard to read as color maps alone. **Flowline maps** [](doi:10.1038/s41563-019-0387-3), streamlines integrated through the orientation vector field drawn with density proportional to local alignment, render the connectivity of the crystalline regions directly, in images reminiscent of van Gogh's *Starry Night*:

:::{figure} ../assets/figures/flowline-maps.jpg
:alt: Orientation color map and flowline rendering of crystalline domains in a semicrystalline polymer film
:width: 100%
From orientation field to flowlines: the same region shown as an orientation color map (left) and as a flowline map (right). Connectivity, domain size, and topological defects in the orientation field become immediately readable.
:::

### Dose-limited experiment design

Everything about these experiments is a dose budget negotiation:

- **Total fluence** must stay below the damage threshold; measure the critical dose for your material first (e.g., by watching a reflection fade under repeated exposure).
- **Step size vs. probe size:** large scan steps (often ≫ probe size) spread the dose; the orientation field is smooth enough that sparse sampling still captures it.
- **Cryo helps:** cooling typically increases critical dose by a factor of a few.
- **Detector:** electron counting and high frame rates let you work at the shot-noise limit; pattern-level denoising and radial-symmetry priors can be applied in analysis.

## Amorphous materials and pair distribution functions

Glasses, amorphous thin films, liquids, and highly disordered solids have no unit cell, so the crystallographic toolkit of the morning sessions (Bragg disks, lattice vectors, orientation libraries) does not apply. But these materials are far from structureless: they have well-defined bond lengths, coordination shells, and often *medium-range order* (MRO) extending over 1–3 nm. This module covers how to quantify that structure from nanobeam diffraction [](doi:10.1146/annurev.matsci.35.082803.103337).

:::{figure} ../assets/figures/crystal-amorphous.png
:alt: A simulated sample that is crystalline on one side and amorphous on the other, with the corresponding spotted and diffuse-ring diffraction patterns
:width: 90%
Crystalline order produces sharp Bragg spots; amorphous structure produces diffuse rings. The rings still encode quantitative structural information (bond lengths and coordination shells), which the pair distribution function extracts.
:::

### Describing disorder: n-body distribution functions

Without a lattice, structure is described statistically. The workhorse is the two-body **pair distribution function** g(r): the probability, relative to a random ideal gas, of finding an atom at distance *r* from another atom. Sharp peaks at the nearest-neighbor distance and progressively broader peaks at higher shells encode the local bonding; how quickly the oscillations decay measures the extent of order:

- A **nanocrystalline** material shows sharp, persistent peaks out to large *r*.
- A material with **medium-range order** shows a sharp first shell and damped oscillations that die out over a few nanometers.
- A **liquid or fully amorphous** structure shows a sharp minimum bond distance, a strong first shell, and essentially no structure beyond the second or third shell.

For multi-component systems the bookkeeping multiplies: two elements A and B produce three partial PDFs (A–A, B–B, A–B).

### Measuring the PDF with electron diffraction (ePDF)

Diffraction measures the Fourier transform of g(r). From a 4D-STEM dataset of an amorphous sample the workflow is:

1. **Polar transform.** Convert each diffraction pattern from Cartesian (kx, ky) to polar (φ, k) coordinates. This step is exquisitely sensitive to the pattern origin: an incorrect center produces wavy azimuthal artifacts that corrupt everything downstream. Automatic origin refinement (minimizing the standard deviation of intensity along the azimuthal direction at each probe position) makes this robust at scale.
2. **Azimuthal average → I(k).** Average over φ to get the radial scattering intensity:

:::{figure} ../assets/figures/amorphous-rings.png
:alt: An energy-filtered diffraction pattern of an amorphous film showing diffuse rings
:width: 55%
The starting point: a diffuse-ring diffraction pattern from an amorphous film. Reaching high scattering vectors (several Å⁻¹) and filtering the inelastic background both pay off directly in PDF quality.
:::

3. **Background subtraction.** Fit and remove the smooth single-atom scattering background, e.g. with a model of the form

   $$B(k) = c + i_0 \exp\!\left(-\frac{k^2}{2 s_0^2}\right) + i_1 \exp\!\left(-\frac{k^4}{2 s_1^4}\right)$$

4. **Reduced structure factor.** Convert the background-corrected intensity to the reduced structure factor F(k) = k·(S(k) − 1).
5. **Sine transform → G(r).** A windowed sine transform of F(k) gives the reduced PDF G(r). Truncation of the k-range produces spurious low-r oscillations (atoms cannot be 0.5 Å apart!); damping schemes that iteratively fit S(k) and estimate the atomic number density ρ₀ suppress these artifacts [](doi:10.7566/JPSJ.91.104602).
6. **Normalize → g(r).** With the density in hand, $g(r) = 1 + \dfrac{G(r)}{4 \pi r \rho_0}$:

:::{figure} ../assets/figures/pdf-curve.png
:alt: A pair distribution function curve with a sharp first-neighbor peak and decaying oscillations
:width: 70%
The result: g(r) with its sharp first-neighbor peak and decaying coordination-shell oscillations, the quantitative fingerprint of the local atomic structure.
:::

Because each step has failure modes, validation matters: in the tutorial we use a *simulated* 4D-STEM dataset of amorphous tantalum (built on the liquid/glass models of [](doi:10.1073/pnas.1705723114)), so the measured g(r) can be compared against the ground-truth PDF computed directly from the atomic coordinates.

### Beyond the mean: mapping disorder in 4D

Unlike a selected-area or powder ePDF, a 4D-STEM measurement retains spatial resolution: each probe position carries its own diffuse-scattering signal, so you can *map* local structure:

- **Spatially resolved PDFs** distinguish amorphous phases, map crystalline/amorphous phase fractions, and track devitrification.
- **Fluctuation electron microscopy (FEM):** the *variance* of the diffracted intensity between probe positions (as a function of k and probe size) is sensitive to medium-range order that is invisible in the mean pattern [](doi:10.1088/0034-4885/68/12/R06), [](doi:10.1016/S0304-3991(02)00155-9):

:::{figure} ../assets/figures/fem-schematic.png
:alt: A probe scanned over a partly disordered sample producing diffraction patterns whose speckle varies between positions
:width: 85%
Fluctuation electron microscopy: as the probe moves across a disordered sample, the diffuse speckle changes from position to position. The variance of the intensity between positions measures medium-range order that the mean pattern averages away.
:::

- **Elliptic distortions of the amorphous halo are a signal, not just a nuisance:** under mechanical load, the halo of a metallic glass distorts measurably, so fitting ellipses to the rings maps *strain in amorphous materials*, extending the morning's strain-mapping ideas to samples with no lattice at all:

:::{figure} ../assets/figures/ellipse-strain.jpg
:alt: Schematic of an in-situ tension experiment on a metallic glass, with best-fit ellipses to the amorphous ring under load
:width: 75%
Strain mapping without a lattice: in-situ tension on a dog-bone metallic glass sample. Under load, the amorphous ring becomes measurably elliptical, and the fitted ellipse parameters give the local strain tensor.
:::

### Practical notes

- Reach high enough k (several Å⁻¹): camera length down, and mind the detector corners.
- Energy filtering removes the inelastic background and markedly improves S(k) at low k.
- Amorphous halos also calibrate elliptical distortion for all the crystalline analyses; see the [data handling module](./disk-detection-strain.md).

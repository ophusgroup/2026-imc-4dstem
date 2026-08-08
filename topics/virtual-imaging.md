---
title: Virtual dark field, digital dark field, and ML clustering
short_title: VDF, DDF & ML clustering
---

:::{admonition} Hands-on tutorials
:class: important
- ▶️ [Virtual dark field and digital dark field (BFBT ceramic)](https://drive.google.com/file/d/1SwcFZd6bZJytR38UXeSiqpVuTk6yJnbX/view?usp=sharing)
- ▶️ [Clustering-based digital dark field imaging](https://colab.research.google.com/drive/1hKaw7YiaZiXWbvSQ1oKRuM_84tz7xUma?usp=sharing)
:::

:::{admonition} Learning goals
:class: tip
- Form virtual bright field, dark field, and custom-mask images from a 4D dataset.
- Use digital dark field imaging to map grains, domains, and superlattice ordering from individual reflections.
- Apply unsupervised decomposition (PCA/NMF) and clustering (from k-means to density-based methods like DBSCAN) to 4D-STEM data.
- Interpret component patterns and cluster maps physically, and validate them against the raw data.
:::

Because a 4D-STEM dataset contains the *entire* diffraction pattern at every probe position, any STEM detector geometry can be applied *after* the experiment: integrate the intensity inside a chosen region of the diffraction pattern at each probe position, and the result is an image. This is **virtual imaging**, and it is usually the first, and often the most informative, analysis applied to any 4D-STEM dataset. Virtual bright field and dark field reconstruction in this sense was popularized by Rauch and co-workers alongside scanning precession diffraction and ACOM [](doi:10.1111/jmi.12065), [](doi:10.1016/j.matchar.2014.08.010), and generalized to arbitrarily shaped virtual apertures by Gammer et al. [](doi:10.1016/j.ultramic.2015.03.015).

## Virtual bright field and dark field

A circular mask over the central beam gives a virtual bright field (BF) image; an annulus outside it gives a virtual annular dark field (ADF). The two are complementary: electrons scattered out of the BF disk land in the DF detector, so regions that darken in BF brighten in DF. Unlike physical detectors, virtual detectors are free: you can try any inner/outer radius, any shape, any position, and iterate until the contrast isolates the feature you care about [](doi:10.1017/S1431927620024307):

:::{figure} ../assets/figures/virtual-masks.png
:alt: Virtual detector geometries applied to a diffraction pattern: a single aperture, an annular aperture, and an array of apertures over the diffracted spots
:width: 95%
Virtual detectors are defined in software after the experiment: a single aperture over one reflection, an annulus, or an aperture array matched to a whole family of diffracted spots.
:::

- **Detector design matters.** The BF detector should capture the unscattered disk (slightly expanded to tolerate residual descan); the ADF annulus geometry controls whether contrast is dominated by diffraction (low angles) or thickness/Z (high angles).
- **Selected-area diffraction in reverse:** integrating diffraction patterns over a *real-space* region of interest gives a virtual selected-area pattern from exactly that region, ideal for identifying which reflections belong to which microstructural feature.

## Digital dark field

Classical dark field TEM tilts the beam (or shifts the objective aperture) so one chosen Bragg reflection forms the image:

:::{figure} ../assets/figures/df-tem-schematic.png
:alt: Ray diagram of dark field imaging in conventional TEM using an objective aperture in the back focal plane
:width: 60%
Conventional dark field TEM: an objective aperture in the back focal plane selects a single diffracted beam to form the image: one reflection, one exposure, one tilt condition at a time.
:::

The 4D-STEM equivalent, **digital dark field (DDF)** [](doi:10.1093/mam/ozae104), places a virtual aperture over one (or several) specific Bragg reflections and maps where in real space that reflection is excited. Every reflection in the dataset is available simultaneously, from a single scan:

:::{figure} ../assets/figures/ddf-workflow.jpg
:alt: Flowchart of the digital dark field workflow, from probe template and peak finding through single apertures, aperture arrays, or points-list reduction to virtual dark field images
:width: 75%
The DDF workflow: after peak finding and g-vector identification, images can be formed from a single virtual aperture, from a mask built on a whole array of aperture positions, or by reducing the detected points list directly against the aperture array.
:::

This is enormously powerful for microstructure:

- **Grain and domain mapping:** each grain lights up only in the reflections it produces, so DDF images segment grains, twins, and ferroelastic/ferroelectric domains, even when they are invisible in BF/ADF contrast [](doi:10.1017/S1431927620024411).
- **Superlattice and ordered phases:** placing the aperture on superlattice reflections maps ordered regions and antiphase domains directly:

:::{figure} ../assets/figures/ddf-superlattice.jpg
:alt: Digital dark field maps of a perovskite film formed from different superlattice and higher-order Laue zone reflections, each highlighting a different ordered region
:width: 100%
Digital dark field in action on a perovskite thin film: virtual apertures on different superlattice and higher-order Laue zone reflections (right) map chemically and structurally distinct ordered regions through the film (left), all from one 4D-STEM scan.
:::

- **Tracking spots, not just masking them:** in real datasets the reflections move (strain, rotation, descan), so robust DDF implementations follow the peak within a window rather than using a fixed mask; this is where Bragg-vector-based approaches and fast implementations ([Kelvin_STEM](https://github.com/maclariz/Kelvin_STEM), py4DSTEM, pyxem) come in.
- From the ensemble of DDF images, phase and domain maps of the whole field of view can be assembled, the manual counterpart of the machine-learning clustering approaches below.

## Machine-learning clustering

A 4D-STEM dataset from a complex microstructure can contain thousands of distinct diffraction patterns: different phases, orientations, domains, and overlaps. Designing virtual detectors by hand (as in the previous section) works when you know what you are looking for; unsupervised machine learning lets the data tell you *what distinct patterns exist* and *where they occur*, with no prior assumptions about the structures present [](doi:10.1186/s40679-019-0063-3).

### The idea

Treat each probe position as one observation (a vector of detector-pixel intensities) and ask: what small set of characteristic patterns, mixed in varying proportions, explains the whole dataset? Two families of methods are widely used:

- **Matrix decomposition** (PCA, non-negative matrix factorization): factorize the data into a set of component patterns and their real-space loading maps. NMF's non-negativity constraint suits diffraction data, since patterns and mixing weights are both inherently non-negative. PCA is often used first for denoising and to estimate how many components the data supports (scree plot).
- **Clustering**: group similar observations, giving a hard segmentation of the field of view into phases/domains, with each cluster's mean pattern available for crystallographic interpretation. Centroid methods (k-means, Gaussian mixtures) require choosing the number of clusters up front; **density-based methods** (DBSCAN and its relatives) instead take density threshold parameters and find as many clusters as the data supports [arXiv:2606.23201](https://arxiv.org/abs/2606.23201). The observations need not be whole patterns: clustering the detected Bragg peaks themselves in combined real and diffraction space is the basis of the clustering-based digital dark field workflow in this session's tutorial.

:::{figure} ../assets/figures/clustering-methods.jpg
:alt: Comparison of many clustering algorithms applied to toy 2D datasets, showing how each algorithm partitions differently shaped clusters
:width: 100%
No single "right" clustering algorithm: the scikit-learn comparison grid shows how k-means, spectral, agglomerative, DBSCAN, Gaussian-mixture, and other methods partition the same toy datasets very differently. The same is true for diffraction data.
:::

A typical workflow: preprocess (align the zero beam, mask the central disk or take a log/power scaling so weak reflections count, optionally bin) → reduce dimensionality → decompose or cluster → inspect the component patterns *as diffraction patterns* and interpret them crystallographically → refine.

### Why this works so well for diffraction data

Unlike natural images, diffraction patterns from a given phase/orientation are highly reproducible; the "signal manifold" is low-dimensional. Clustering therefore tends to recover physically meaningful classes: distinct phases, orientation variants, ordered vs. disordered regions, and even subtle symmetry-breaking distortions that are hard to see by eye [](doi:10.1038/s43586-022-00095-w). The output is a phase/domain map plus a library of representative patterns, obtained in minutes from datasets far too large to inspect manually:

:::{figure} ../assets/figures/cluster-classes.png
:alt: A grid of mean diffraction patterns for each cluster found in a 4D-STEM dataset
:width: 100%
The payoff: each cluster's mean diffraction pattern, extracted automatically from a 4D-STEM scan. Each class corresponds to a distinct diffraction condition (a phase, orientation variant, or overlap), ready for crystallographic interpretation.
:::

### Caveats

- Components are mathematical objects, not guaranteed physics: decomposition can mix or split physical phases (e.g., NMF components need not correspond one-to-one with real structures). Always validate components against the raw patterns and against crystallographic simulation.
- Intensity scaling choices (log, power, masking the direct beam) strongly affect what the algorithms consider "similar"; dynamical intensity variations within one grain can otherwise dominate over phase differences.
- Density-based clustering does not assign every point, and that is fine. The unclustered leftovers include very small diffracting objects, false detections, and poor detections shifted well off the true disk centroid. The density parameters set the smallest object you wish to see (e.g. `min_samples = 20`), and even without analysing every disk you get far more clusters than you would ever analyse manually.

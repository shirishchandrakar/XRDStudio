# XRD Studio

**[shirishchandrakar.github.io/XRDStudio](https://shirishchandrakar.github.io/XRDStudio/)**

Pick a chemical system from the periodic table and read the calculated powder X-ray
diffraction pattern of every structure on file for it — mathematically, from real crystal
structures, not a lookup of scanned reference patterns.

The whole app is one file, **[xrd-studio.html](./xrd-studio.html)**, which is what's hosted
at the link above.

## Running it

Open the link above, or run it locally with no server at all — double-click the file, or open
it directly as `file:///…/xrd-studio.html`. To host your own copy (GitHub Pages or any static
host), just serve that one file; there's no build step, no `base` path to configure, no
dependencies to install.

> GitHub Pages serves whatever was last pushed — it doesn't pick up new fixes automatically.
> If the live link above looks out of date, push the current `xrd-studio.html` in this repo
> to it again.

## What it does

1. **Click elements on the periodic table.** The panel below fills with every structure on
   file whose elements are a subset of your selection — pick Ti and O, get every Ti-O phase
   in the library.
2. **Click an entry** to plot it. Click a few to overlay them (up to six), each colour-coded,
   each independently normalised so peak heights are comparable regardless of how strong the
   phase's scattering is.
3. **Set the radiation.** Choose a lab source (Cu/Co/Cr/Fe/Mo/Ag Kα) or a synchrotron energy
   from the dropdown, or type any wavelength or photon energy directly — they stay linked.
   The 2θ window and peak width follow automatically; there's nothing else to configure.
4. **Read the plot.** Drag across it to zoom, double-click to reset. The crosshair reports 2θ,
   d-spacing and Q at once. A d-spacing ruler runs along the top, hkl tick marks sit below the
   trace in Rietveld convention and are hoverable for indices and d.
5. **Check the tables.** "Structures shown" gives space group, lattice parameters and density
   for each active phase; "Reflections" lists every peak above 0.5% relative intensity with
   hkl, 2θ, d, Q, multiplicity and relative intensity. Both export — `.xy` for the pattern,
   CSV for the reflection list.

## The library

171 entries built in: 65 elemental reference structures (every element ASE ships a reference
state for) plus 106 curated compounds — oxides, carbides, nitrides, hydrides, intermetallics,
perovskites, spinels, common semiconductors, and a few calibration standards (Si, LaB₆, CeO₂).
83 elements are covered between them; anything greyed out on the periodic table isn't in a
structure yet.

Every entry passed a stoichiometry audit before being included — its computed composition was
checked against its declared formula, so nothing in the library was hand-typed and left
unverified.

### Adding a structure the library doesn't have

**Upload a CIF.** Search the [Crystallography Open Database](https://www.crystallography.net/cod/search.php)
by formula or elements, download a structure's CIF, and upload it with the button above the
periodic table. Parsing — cell, symmetry operations, atom sites — happens entirely in the
browser; nothing is sent anywhere. The imported structure shows up as a normal entry, tagged
**CIF**, and plots exactly like the built-in ones.

**Load a bigger library.** The "Library" control in the header accepts a JSON file in the same
schema the app uses internally — useful if you've harvested a large structure set elsewhere
(e.g. with a script hitting Materials Project, OQMD, or similar) and want to browse it here
without rebuilding the app. Malformed entries are rejected individually with a count, so a
partly-bad file still loads whatever's good.

## What's deliberately not here

**No live network fetching.** An earlier version queried public crystal-structure databases
(COD, OQMD, AFLOW, and others) directly from the browser. In practice, essentially none of
them allow that — browsers block cross-origin requests unless the server explicitly permits
it (CORS), and confirmed by testing a real deployed copy, almost every public provider
refuses. Rather than ship a feature that reliably fails and litters the browser console with
network errors, that path was removed. CIF upload is the reliable replacement: whatever you
can find and download, you can bring in.

**No AI/API structure lookup.** A "type a material name" panel calling language-model APIs
directly from the browser was tried and removed for the same reason — Anthropic's and
Materials Project's APIs both block direct cross-origin browser requests. A key doesn't fix
that; it isn't an authentication problem.

## The physics, briefly

Kinematic diffraction: structure factors from Cromer-Mann scattering parameters with
isotropic Debye-Waller, intensity as |F|²·multiplicity·Lorentz-polarisation, peaks as
Gaussian profiles at a resolution that scales sensibly with wavelength. No absorption,
texture, or size/strain broadening — this is for indexing and comparing phases, not
refining a measured pattern against a model.

## Known limitations

- Library structures are literature cells at room temperature; no solid-solution lattice
  shift is applied, so e.g. an alloy's matrix phase sits at the pure-element's angles unless
  you supply a structure with the actual composition (via CIF).
- A CIF's reduced formula is computed from the expanded, symmetry-generated cell — correct,
  but it means the displayed formula reflects what the structure factor calculation actually
  uses, not necessarily the formula string printed elsewhere in the CIF file's header.
- Cells with very large unit cells (hundreds of atoms) or huge lattice parameters are
  rejected on import rather than silently taking a long time to compute.

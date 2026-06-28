# OpenADMET-PXR-Structure-Challenge

This repository is to share my methods tested in the 2026 OpenADMET PXR structure challenge and some small python/jupyter scripting that I found useful.

I used this challenge as an opportunity to explore how diffusion based co-folding did against other more traditional methods of ligand placements.

I set myself the additional challenge of being limited to open-source packages to ensure reproducibility of results in others hands and to promote the work of open source developers.

As such - I explored multiple packages including:
-OpenFold3
-Protenix
-Boltz2
-Roshambo (ligand based shape/pharamacophore)
-xxina (indirectly - borrowed the results of "N1NC1O")

Analysis/data prep is a key component.  Packages used here include:
-RDKIT (of course!)
-biotite
-gemmi
-spyrmsd
-Pymol (Schrodinger open source version)
-scipy (clustering routines)
-Weights & Biases (free version on website for model fine tuning)
-all driven by python and its standard data tools (Pandas, numpy, matplotlib etc)


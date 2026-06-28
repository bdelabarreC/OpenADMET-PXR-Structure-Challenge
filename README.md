# OpenADMET-PXR-Structure-Challenge

<b>NB - this is a work in progress.  Happy to recieve feedback.  Details will be filled in after close of the competition July 1.</b>

This repository is to share my methods tested in the 2026 OpenADMET PXR structure challenge and some small python/jupyter scripting that I found useful.

I used this challenge as an opportunity to explore how diffusion based co-folding did against other more traditional methods of ligand placements.

I set myself the additional challenge of being limited to open-source packages to ensure reproducibility of results in others hands and to promote the work of open source developers.

I explored multiple packages including:

- OpenFold3
- Protenix
- Boltz2
- Roshambo (ligand based shape/pharmacophore)
- xxina (indirectly — borrowed the results of "N1NC1O")

Analysis/data prep is a key component. Packages used here include:

- RDKit (of course!)
- biotite
- gemmi
- spyrmsd
- PyMOL (Schrödinger open source version)
- scipy (clustering routines)
- Weights & Biases (free tier)
- all driven by Python and its standard data tools (pandas, numpy, matplotlib, etc.)

I made heavy use of both Gemini and Claude throughout to install/debug and to help create code for filters & selection methods.  Gemini declined in quality noticeably over the 2 month effort.

Gemini was relentlessly sycophantic about my ideas and enusred that each new entry I put in would place me at the top of the leaderboard.  Claude was supportive but a little more balanced in its feedback to the ideas I threw at it.  It also had a much better memory of efforts that were attempted weeks and even months ago.  I would not have been able to cover the ground I did without the use of either of them.   I did occasionally wonder if my ideas or work from others in the competition could have been shared via the use of these LLM based agents....

Additionally - I use VSCode with Github Co-pilot for finetuning python/jupyter scripts provided by the above agents.

I used the LDDT-PLI from the leaderboard (computed by the OpenADMET huggingface portal from a fixed random subset of 50% of the 184 ligands) as guidance for method development.

My starting point for understanding the PXR binding site was a collection of ~50 co-complexes pulled from the PDB.  This was reduced to ~45 'Tier 1' structures that were (somewhat) chemically diverse and showed good electron density for the ligand in the pocket.  PXR crystallizes as a dimer.  Generally the 'A' copy was taken as representative but in a few cases the 'B' copy showed either a slightly different pose or was the only one that had a molecule bound.

These structures were aligned by protein backbone and the resulting ligand coordinates were used to drive shape based matching methods.

The binding pocket of PXR is fairly constant.  It is also large and ligand binding appears to be drivent primarily by pi and hydrophobic type interactions.   I constructed a manual pharmacaphore based on the binding sites which reduces it to only 3-4 actual binding mini-pockets driving overall ligand placement.  This scored reasonably well when combined with shape based ligand matching but was still largely outperformed by diffusion/co-folding methods.

Because I was new to all of them, a variety of options were used to run each diffusion/cofolding package.  There were several efforts employed to rank the output of the diffusion/cofolding - clustering of results to find the most reproducible pose followed by top 'iptm' measure was best in my hands for creating the highest scoring entries.  Aside - I think the clustering approach may represent reality better than a single crystal structure.  The results look more like molecular dynamic frames and typically clustered around very reasonable binding options.  The clustering provides useful output in that portions of the molecule that need to be modified can be clearly identified regardless of whether the exactly correct crystallography pose is predicted.

Overall the best result came from a running Protenix in one of my initial efforts.  I wish there was a way to know I had hit my best results as this run took less than an hour to run on my Nvidia 4700 GPU....

In the case of OpenFold3, fine-tuning was attempted with the Tier 1 complexes, with 5 examples pulled randomly to act as reference points to evaluate the training.  This approach worked well, and ultimately produced the highest scoring values from OpenFold.

Each method has slightly different metrics to guide them.  Diffusion cofolding metric that provides the closest to what is being scored by openADMET is the "IPTM" or ligand-protein interface score produced by the cofolding method itself.  Like any docking score, this is an artificial metric as it does not know the true ligand placement.  Shape/PH4 fitting has an 'overlap' score that can drive selection and docking methods have standard docking scores.  I experimented a bit with combining methods (eg diffusion based co-folding followed by shape based ranking).

Ranking of Results:

<img width="719" height="560" alt="image" src="https://github.com/user-attachments/assets/56948ec1-332e-442c-a215-2de35ef59b63" />

My progress over the competition:

<img width="777" height="463" alt="image" src="https://github.com/user-attachments/assets/06d80522-f65c-4eda-865a-c8462d056ad4" />


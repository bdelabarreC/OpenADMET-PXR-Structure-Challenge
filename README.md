# OpenADMET-PXR-Structure-Challenge

<b>NB - Updated 4 Aug 2026 with more detailed method description (at bottom of page).</b>

This repository is to share my methods tested in the 2026 OpenADMET PXR structure challenge and some small python/jupyter scripting that I found useful.

I used this challenge as an opportunity to explore how diffusion based co-folding did against other more traditional methods of ligand placements.

I set myself the additional challenge of being limited to open-source packages to support extensibility & reproducibility of my results here and to promote the work of open source developers.

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

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/87486162-58e9-4dfa-8536-85f4d39e41df" />


These structures were aligned by protein backbone and the resulting ligand coordinates were used to drive shape based matching methods.

The binding pocket of PXR is fairly constant.  It is also large and ligand binding appears to be drivent primarily by pi and hydrophobic type interactions.   I constructed a manual pharmacaphore based on the binding sites which reduces it to only 3-4 actual binding mini-pockets driving overall ligand placement.  This scored reasonably well when combined with shape based ligand matching but was still largely outperformed by diffusion/co-folding methods.

Because I was new to all of them, a variety of options were used to run each diffusion/cofolding package.  There were several efforts employed to rank the output of the diffusion/cofolding - clustering of results to find the most reproducible pose followed by top 'iptm' measure was best in my hands for creating the highest scoring entries.  Aside - I think the clustering approach may represent reality better than a single crystal structure.  The results are reminiscent of molecular dynamic frames (see run8_x00011_2clusters.mp4 in files section on github) and typically clustered around very reasonable binding options.  The clustering provides useful output in that portions of the molecule that need to be modified can be clearly identified regardless of whether the exactly correct crystallography pose is predicted.

Overall the best result came from a running Protenix in one of my initial efforts with that package in mid-May.  This particular took less than an hour to run on my consumer grade Nvidia 4700 GPU with 16Gb VRAM (~$500).

In the case of OpenFold3, fine-tuning was attempted with the Tier 1 complexes, with 5 examples pulled randomly to act as reference points to evaluate the training.  This approach worked well, and ultimately produced the highest scoring values from OpenFold.

Each method has slightly different metrics to guide them.  Diffusion cofolding metric that provides the closest to what is being scored by openADMET is the "IPTM" or ligand-protein interface score produced by the cofolding method itself.  Like any docking score, this is an artificial metric as it does not know the true ligand placement.  Shape/PH4 fitting has an 'overlap' score that can drive selection and docking methods have standard docking scores.  I experimented a bit with combining methods (eg diffusion based co-folding followed by shape based ranking).

Ranking of Results:

<img width="719" height="560" alt="image" src="https://github.com/user-attachments/assets/56948ec1-332e-442c-a215-2de35ef59b63" />

My progress over the competition:

<img width="777" height="463" alt="image" src="https://github.com/user-attachments/assets/06d80522-f65c-4eda-865a-c8462d056ad4" />

I copy/pasted my entries along with occasional snapshots of the leaderboard to track my progress.  Here is where I landed amongst the other efforts (red dot):


<img width="839" height="583" alt="image" src="https://github.com/user-attachments/assets/c65f5cf0-6a6e-426c-8065-d415ea956351" />

Update: Summary of my 82 ligand score for my 20 entries.  I used the LDDT-LP score to evaluate my progress during the competition along with visual inspection of prediction quality.  Partway through I adopted PoseBuster evaulation to ensure that all entries had reasonable geometry for the small molecule.    


https://github.com/bdelabarreC/OpenADMET-PXR-Structure-Challenge/blob/main/All_TCB_entries_final.xlsx


<img width="3070" height="1052" alt="image" src="https://github.com/user-attachments/assets/3cb580d0-25eb-478f-a017-5d3c267d23c2" />

Further notes:

Except for fine-tuning efforts, all computations were run on an 18 cpu / 96 Gb system with an Nvidia 4070 GPU with 18Gb of memory, all running under Ubuntu.

Fine-tuning efforts were done on a RunPod GPU instance with an H100 (96 Gb).  Computing costs were under $100 for all efforts for this competition.

<b>Protenix</b> run in default mode (5 predictions per submission no template guidance) with their v1 model gave the highest score - for this reason I focused most of my efforts on this primary co-folding and used a variety of settings.  I tended towards larger runs - creating 20-50 predictions per run and then using clustering to pick reproducible predictions that provided the highest protein-ligand score.  I experimented with picking both the top score per cluster as well as the median score per cluster.  Top score produced better results as measured per the 82 compound LDDT-LP.  I discarded any singletons regardless of their score.  NB - the v2 weights (obtained on hugging face) did significantly poorer than the v1 weights.  The V2 weights enabled use of a more physics based ligand geometry checking system implemented within Protenix ("tfg") but this performed the poorest of all Protenix runs.  Fine-tuning of the v1 weights was considered but could not be implemented due to a lack of familiarity and/or clear directions on how to do so.

<b>OpenFold3</b> was comperable to Protenix and was the only co-folding method that was amenable to fine-tuning in my hands.  For the finetuning I curated a set of ~50 structures from the PDB PXR examples, holding out 8 structures for comparison by spyRMSD scores.  I used the weights & biases webservice to track my model training.  1000 cycles of training were used initially.  The model appeared to deteriorate after the first 100 cycles, so a second run was done with finer slicing to get to 60 cycles being optimal.  This was apparent both the distogram loss plot, the spyRMSD calculations across the holdout set, and the competition based LDDT-LP score.  For the latter, a set using an under-trained model (30 cycles) and an overtrained model (90 cycles) was submitted for scoring.  The 90 cycle model was significantly worse.  The optimally tuned weights yielded a result which was statistically insignificant from the best Protenix submission.   

<b>Distogram Loss Plot while training OpenFold3</b>
<img width="882" height="436" alt="image" src="https://github.com/user-attachments/assets/6889bc92-632a-49ed-95ef-782051de5d4a" />

<b>spyRMSD and Prediction Comparison after Optimal Tuning of OpenFold3 </b>
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/c655d541-3cdd-449f-b6d3-370169389248" />

NB - no effort was made to pay heed to data cutoffs in training of the model.  My underlying assumption was that I could reinforce correct predictions for PXR:ligand interactions.  It is therefore possible that training data was leaked into the test set.  Ideally one would have a much larger training set and model that had not seen any of the predicted results.  The competition LDDT-PL score was the closest I could get to that ideal setup.

<b>Boltz2</b> gave the lowest score of all co-folding methods tried here.  Attempts were made to combine its output with some of the shape/PH4 matching algorithms but these performed even worse.  With no ability to tune Boltz2, it was abandoned.

<b>Direct placement/shape matching</b>:  Using the curated set of structures of PXR complexes, an attempt was made to do shape and pharmacophore matching with the open-source Roshambo package.  Pharmacophore used default (RDKIT) defintions.  The blend of shape vs pharmacophore (aka 'color') score was decided empirically by selecting the most consistent set of poses.  Direct placement by this method was the least effective approach, but it could be rescued to a reasonable result that overlapped with some of the co-folding efforts by allowing for a relaxation of sidechains and optimization with a short openMM run.  Similar efforts using MOE were observed but not submitted for the competition in order to stay within the open source concept.

<b>Custom pharmacophore model</b>.  Interactions in the pocket are dominated by vdw and pi-pi type interactions.   A custom protein based pharmacophore model was created based on clustering of the PDB structures.  Three distinct binding modes using pi-pi interactions could be observed.  This model was used to filter output from the co-folding efforts but did not improve the output.

<b>SMINA based docking</b>  I did not carry this out but have borrowed a submission made by "N1NC10" who disclosed methods early in the competition.  They used a 'tuned SMINA' and deployed clustering to identify top hits.  This approach did not score well by LDDT-PL but was helpful as a reference to confirm my intuition that direct docking into a fixed structure would not yield good results.

<b>Summary</b> -

I mostly agree with the conclusions from Daniel Saltzberg OpenADMET webinar that as a group we have made a set of 'ok' predictions.  Unfortunately - 'ok' doesn't really cut it for drug discovery!  Progress has been made - but we are not there yet.

It was interesting to see the common conclusion that co-folding outperformed all other methods.  The question remains as to whether it can be truly predictive or if it is simply a 'fancy look-up method'.  The computation has become easy enough, so perhaps 'fancy look-up' isn't such a bad thing here.  The performance of the various weights (Boltz2 vs Protenix v1.0 vs Protenix v2.0 vs OpenFold3 tuned/untuned) suggests that some of this is in fact fancy-lookupism.  Nominal rescue by incorporating physics based methods underscores what a difficult problem this was.

While I have not been able to compare all 20 of the prediction sets sitting on my harddrive against the full 184 structure dataset, I also agree that the primary difficulty arose from selecting the correct prediction out of lot of other possibilities.  Multiple seeds/runs produced a range of results and associated metrics simply showed which satisfied a particular but generic protein:ligand interaction model (iPTM).  I tried making several hybrid approaches (shape, pharmacophore, clustering) but none seem to improve on simply picking the best iPTM score.   

It will be interesting to see if a co-folding model finetuned on the full 184 structural dataset would be an improvement over current prediction rates.  At a minimum we should have a better train/test set split here such that the test can more directly tell when training has been optimized.  This makes co-folding an ideal approach for a common structure such as PXR faced during drug discovery.  However - most projects I have worked on rarely get past 25 structures before we are either able to nominate a lead or kill the program, so individual datasets will remain small.  Dataset federation may work, but only for those highly competitive targets where most people are unwilling to share truly enabling data.  It remains to be seen if the predictions can be improved when smaller datasets of protein:ligand complexes are available, or for more conformationally dynamic targets. 

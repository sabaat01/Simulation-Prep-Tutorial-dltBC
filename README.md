# Simulation-Prep-Tutorial-dltBC
walkthrough of preparing a new system for simulating, using dltBC as an example
## Intro and Goals
dltBCDX is a multi-protein complex embedded in the cell wall of gram-positive bacteria and is involved in decorating the outer surface of the cell wall with D-alanine residues. These D-alanines attach to LTAs (lipotechoic acid) on the extracellular surface and provide the bacterium with a positive charge, which protects the bacterium from attack by antibiotics or an immune response.

Since D-alanine passes through the dltBCDX complex via sequential covalent interactions, it can't be modeled through a single simulation. This tutorial is based on the preparation of dltB alone and dltBC simulations.

We are going to prepare our system using various software packages, then initiate simulations on Savio (UC Berkeley's HPCC).

dltB has 415 residues (depends on species).
dltBC has 415 (B) + 79 (C) residues.
For both simulations, we will be using a POPC bilayer to simulate the bacterium cell wall.

Steps for setting up a simulation:
1. Prepare protein
	- Acquire appropriate PDB files
	- Clean and prepare the protein
2. Add environment / solvate
   	- Add waters, ions, and bilayer/water box
3. Generate starting files
	- Parameter + inpcrd files
	- Eq, Min, and Heat files
	- bash file script for running sim
4. Prepare Savio
	- folder for each replicate, duplicate input files, set up symlinks, specify run params
5. Kick off sims, monitor
   	- tips for monitoring/checking statuses
   	- how to download trajectories and visualize


# 1: Prepare protein
From https://erikh.gitlab.io/group-page/protsetup.html:
- Is the protein a monomer, dimer? or n-mer?
- Are there Cys-Cys- crosslinks?
- Are there missing parts of the structure (the first amino acids are often not resolved, check REMARK 465)
- Are there non-standard residues?
- At what pH was the structure obtained?

## Acquire original PDB
Usually, one would download a crystal structure from PDB (link to dltB: https://www.rcsb.org/structure/6BUG). In this prep, however, we used a crystal structure that contained the full complex, then isolated the proteins we wanted to simulate.
  
- Open .pdb file in PyMol
- Label by chain, color by chain
- Select only what is needed:
	- Command: select (chain B or chain C) and DltBCDX
	- Hide unselected
	- A → copy selection to object → new → rename → dltB_alone / dlt_BC

## Orient into a membrane frame
We want our system oriented in 3D space such that if the bilayer was a book, x and y are the length and width of the book cover, and z would be the depth/thickness.

Orientations of Proteins in Membrane (OPM) is a site that publishes structures of proteins from the PDB pre-oriented into a bilayer. If you downloaded a crystral structure from PDB, you might be able to download the directly corresponding OPM file and delete the bilayer markers. In this case, we needed extra steps since our PDB did not have a 1-to-1 corresponding OPM file. 

- Download dltB's OPM database file. dltC is conveniently included in this pre-oriented file. (link: https://opm.phar.umich.edu/proteins/4049)
- In PyMOL: Open → find the dlt_BC_OPM file and load into PyMOL alongside dltB_alone
- Click all → A → zoom
- Color dltB_alone and dlt_BC_OPM different colors for later comparison. Color the OPM file by segi or chain
- Optional: remove dltC (chain A) from the dlt_BC_OPM object, before aligning
- Command: <align dltB_alone, dlt_BC_OPM>
	Make sure you align your protein *to* the OPM file, not the other way around. Your protein's name should come first in the command.
- Delete dlt_BC_OPM
- File → Export Molecule → Check Retain PDB IDs → Selected: <dltB_alone>, __not__ all → Save as a .pdb file

## Access Maestro virtually
Maestro requires a license. UC Berkeley's Chem Library can provide access through the Molecular Graphics Computation Facility (MGCF). You will have to access a virtual environment to run Maestro. Alternatively, you can visit the MGCF in person in Tan Hall and use Maestro directly on an MGCF monitor.

- Assuming an account with the Chem Library MGCF is already set up, ensure the following basic items are complete:
	- MobaXTerm installed
	- starting password has been changed and new password noted down
 	- X2Go installed
  	- Settings for X2Go and MobaXterm have been adjusted according to the MGCF instructions.
- Helpful tutorials for completing the above tasks:
	- New User Instructions: https://docs.google.com/document/d/10lAWGWpGKvwPK3eGNj6TzcoMzOCtDQZC3dr--E7SeYM/edit
  	- MGCF FAQs: https://mgcf.cchem.berkeley.edu/mgcf/faqs/
	- Tips on Using Maestro via X2Go: https://docs.google.com/document/d/1kWwOh6VPb9CJRTmhdaGPFgHM5Wg4Ql7vdILsjIiNjPM/view
- Start a new X2Go session. To use X2Go with a monitor, you must specify resolution in settings as either maximum possible or according to dimensions of Display 2 and select your monitor as Display 2. Dimensions of Display 2 are 1920 x 1080.
---
- MobaXTerm can be used to transfer files between your local computer and the remote filesystem.
	- SSH protocol, Port 22, specify username (type your username here), and specify an MGCF workstation name such as nano.cchem.berkeley.edu.
   	- The remote filesystem is navigable via the left sidebar
  	- To download from remote to local: select the file you want from the remote filesystem, then click download at the top of the sidebar.
  	- To upload from local to remote: click upload at the top of the sidebar, then navigate through your local filesystem in the popup and select the file(s) you want to add.
---
- Open Maestro in X2Go and create and title a new project.
- NOTE: If Maestro is not opening, there may be an issue with the computing cluster. Maestro usually only takes 60s max to open.
	- Open terminal, type "maestro", it should note that a process has been added.
 	- Work on something else for 10 minutes.
	- If it still hasn't opened, check if you have any processes running across various workstations that may be impeding opening.
	- Helpful commands to type into X2Go terminal (use with care): ps -x, ws_ps, kill -9 PID, ws_kill, ssh other_workstation_name, pkill -U username
	- If still not working after several attempts to examine processes, do something else for a while and come back to it, or email Kathy/Singam, or visit the MGCF in person and talk to Singam directly. :(

## Preprocess
- Import structure: dltB_aligned.pdb
- Style --> Color Atoms by Chain Name or by Element
- [Optional] Style --> +Ribbons --> Color by Chain Name
---
In the dlt_BC structure, there are ligands such as PNS (phosphopantethiene group, link: https://www.rcsb.org/ligand/PNS) and LMT (detergent, link: https://www.rcsb.org/ligand/LMT). We want to remove these for now.
- On the left-hand window named "Structure Hierarchy", expand the "Ligands" category.
- Right click on the ligands you wish to remove and click "delete atoms"
---
- Select P for Protein
- Build → 3D Builder → +H
	- This will add hydrogens to every residue in the protein

## Resolve issues
- Tasks (upper right) --> Search for Protein Prep Wizard and select
- Import and Process Window --> Deselect everything, EXCEPT:
	- Add hydrogens
 	- Fill in missing side chains using Prime
 	- You can also select "Create disulfide bonds". However, there is only 1 cysteine total among dltB and dltC, so I did not bother selecting this for either the dlt_B or the dlt_BC prep.
  	- Do NOT cap termini yet! Dowser cannot parse.
  	- Click Preprocess. A window may pop up confirming that you want to fill in missing side chains using Prime. Click Continue.
- View Problems: examine issues and decide if anything needs addressing
- Protein Reports:
	- Ignore everything except Missing Atoms. Check for improper or missing side chains. If you used Prime, there should be no missing atoms.
 	- If something in reports appears for every residue in the entire system, it's probably fine. Steric clashes, improper torsions, and such are okay because the system will be minimized to an optimal arrangement later.
- Ramachandran Plot: Examine any outliers. Glycine and methionine are common outliers. Nothing to worry about.
- Review and Modify: Since we already removed the two ligands, nothing should appear here. If you forgot to delete a heteroatom, delete it now.
- Save a pdb before optimization

## Check protonation states
We want to examine the local environment of each of the protein's polar/charged residues.

From: https://computecanada.github.io/molmodsim-amber-md-lesson/11-Protonation_State/index.html
"The protonation states of titratable amino acids (Arg, Lys, Tyr, Cys, His, Glu, Asp) depend on the local micro-environment and pH. A highly polar microenvironment will stabilize the charged form, while a less polar microenvironment will favor the neutral form."

Some residues have charge states that will be influenced by whether they are in a particularly positively or negatively charged region of the protein. We cannot depend on the generic charge state of these residues, because their pKa value will be affected by the region they inhabit. If a residue's pKa shifts dramatically, it may need to occupy a different protonation state from what is expected.

Example: aspartic acid (D) changing from pKa 3.71 to pKa 7.6.  A lysine (K) and aspartic acid (D) near each other may favor a protonated lysine and deprotonated aspartic acid.
      
- How to optimize charge states:
1. No Flip residues (Asn, Gln) can be ignored. Highly unlikely that anything needs changing.
2. Ignore Ser, Thr, Tyr
3. Check Lys, Asp, and Glu in detail
	- All three will usually retain a charge (Lys protonated, Asp & Glu deprotonated)
 	- What is the usual pKa for the given side chain? What is the pKa listed in the Maestro display?
  	- Has there been a dramatic shift in pKa that may influence the charge state?
   	- Are there more favorable interactions available in a different charge state/conformation?
**   	electrostatic interactions (mauve) >> hbonds (yellow) >> aromatic hbonds (teal)
**
4. Check His in detail
	- 3 possible states: HIE (H on one of the two N's), HID (H on the opposite N), HIP (positively charged, both N's have an H)
   	- The extra (+) charge in HIP is not favorable. Only select the HIP state if it will induce the formation of a salt bridge/electrostatic interaction.
     	- If the pKa of His has been shifted by its surroundings to be >7-8, then you can protonate it. This usually occurs if there's an acidic residue (such as Asp/D or Glu/E) next to it encouraging a salt bridge.
6. For Lys, Asp, Glu, His: If there are two unique bonding patterns possible, both equally favorable, you may need to set up two different simulation sets and analyze both datasets.
7. common residues that pop up
	- Gln (Glutamine, Q, polar uncharged)
 	- Asn (Asparagine, N, polar uncharged)
  	- Tyr (Tyrosine, Y, hydrophobic)
	- Cys (Cysteine, C, special res)
	- Ser (Serine, S, polar uncharged)
	- Thr (Threonine, T, polar uncharged)
**    	- Lys (Lysine, K, positive charge)
	- Asp (Aspartic Acid, D, negative charge)
	- Glu (Glutamic Acid, E, negative charge)
	- His (Histidine, H, positive charge)**
---
- (Round One) Refine:
	- select Label pKas
	- Run Interactive Optimizer (new window opens)
   		- Select Label pKas, use PROPKA 3.0
     		- click Analyze Network
  		- click on the first species and click through possible states using the "<" ">" buttons. Repeat for every species in the list and optimize each charge state. Navigate to the next species using the up and down arrow keys.
- Label this structure with "preprocessed_noCaps" in the sidebar and export PDB from Maestro. Use MobaXTerm to download from remote filesystem to local computer, then scp using the dtn and add it to Savio filesystem.
  
- (Round Two) Add internal water molecules to stabilize the charged residues in the protein
	- Dowser is built into the Savio bash, so just call < dowser filename.pdb >
 	- Confirm that dowser completes correctly, does not crash
  	- scp dowserwat.pdb onto local filesystem, then upload to remote using MobaXTerm
  	- Import Structure into Maestro --> select dowserwat.pdb to import
  	- Shift+Select both the "preprocessed" pdb from prior and "dowserwat.pdb" --> right click --> select "Merge" from menu
  - Rename the merged structure "prepped_dowsed_noCaps"
  	  
- (Round Three) Recheck protonation
	- Protein Prep Wizard --> Import and Process tab --> check Cap termini --> Preprocess
 	- Add "capped" the name of this new structure. Make sure you have the correct structure selected. Double check on problems, missing atoms, and review & modify tab.
  	- Refine tab --> re-run the Interactive Optimizer now with capped + dowsed protein. Adust charge states as needed.
  - Label new structure with "opt"
  	  
- (Round Four) Refine --> Restrained Minimization (hydrogens only, OPLS4)
- Label new structure with "min"
- Export prepared structure as PDB file from Maestro to remote. Download PDB from remote to local, then scp to Savio

- What is Cap Termini?
The goal is to neutralize the protein backbone by adjusting the C and N-termini and spreading their charge as evenly as possible.
- N terminus (first residue): the NH2 group gets a carboxyl (acetyl, ACE) tacked on.
- C terminus (last residue): the carboxyl gets an amide (N-methyl, NME) tacked on.
- For more on termini: https://www.cup.uni-muenchen.de/ch/compchem/tink/tink2.html 

# 2: Add environment / solvate
Depending on the system you are preparing you may need to perform some or all of the following steps:
- Add internal waters
- Add ions
- Surround protein in a water box
- Surround protein in a membrane/lipid bilayer

For our system, we have already added internal waters using dowser. Now we will add ions and surround our protein in a lipid bilayer. Our environment will also include a buffer layer of waters surrounding the bilayer.
We will perform this step using packmol-memgen. Here is the introductory paper to the software, it is very short and informative: https://pubs.acs.org/doi/epdf/10.1021/acs.jcim.9b00269.
Call run packmol-memgen - - help for a detailed list of parameters
Can also look up an Amber manual and navigate to the packmol-memgen section. Amber21 manual (link: https://ambermd.org/doc12/Amber21.pdf) --> section 13.6, page 220

Here is the packmol command we will use:
< source activate AmberTools 21
packmol-memgen --pdb dlt_BC_oriented_prepped_dowsed_capped_opt_min.pdb --lipids POPC --ratio 1 --preoriented --notprotonate --nottrim --salt --salt_c Na+ --saltcon 0.15 --dist 15 --dist_wat 17.5 --ffwat tip3p --ffprot ff14SB --fflip lipid17 --nloop 50 >
- lipid bilayer: of POPC with a ratio of 1:1
- preoriented: into a bilayer, so that packmol does not try to rotate our structure for us
- notprotonate, nottrim: do not change protonation states
- salt, salt_c, saltcon: add ions. use Na+ (not K+), and Cl- is the default acceptor. add to a concentration of 150 mM (0.150 M)
- dist: minimum distance to the box border in x,y,z directions
- dist_wat: distance between edge of bilayer and edge of box --> this is the width of the water buffer included around our lipid bilayer
- ff: list force fields of the water molecules, protein, and lipids. To use combinations of lipids to more accurately resemble a Gram-positive bacterium cell well, you must have Lipid21 or higher installed.
- nloop: packing iterations per molecule, usually 20, but I set it to 50 to see if we could get any better minimizing. This is not necessary, and may make packmol-memgen intolerably slow.

Notes on packmol-memgen:
- Takes a long time to run. Make sure your connection to the HPCC does not time out, or laptop shut down.
- If you read through the run log, you will notice that it signals that minimization has not converged. This is okay. It is already extremely close.
- Might populate lipids into the core of your transmembrane protein! Visualize in VMD (after generating a trajectory) and ensure this is not the case!

Output of packmol-memgen: pdb file containing protein structure, internal waters, bilayer, ions, and water box

"A recent study has found that large membrane patches buckle using the Monte Carlo barostat, whilst smaller patches show a systematic depression of the area per lipid. The buckling behaviour is corrected using a force-switch for non-bonded interactions....The area per lipid depression is also found with Lipid21. However, POPC remains near the experimental area per lipid. The Berendsen barostat is thus preferable over the Monte Carlo barostat when simulation time allows."

The dimensions of the system are by default estimated by packmol-memgen based on the size of the protein to be packed.
PG:PE:CL (65:27:8)
packmol-memgen --pdb dltB_aligned_prepped_dowsed_minimized.pdb --lipids POPG:POPE:CL --ratio 65:27:8 --preoriented --notprotonate --nottrim --log packmol_log.txt --salt --salt_c Na+ --saltcon 0.15 --dist 12 --dist_wat 20

→ needs updating to a new version of AmberTools (needs Lipid 21 or higher to use CL)
→ https://ambermd.org/AmberTools.php 
→ re-compile 
→ to update: https://ambermd.org/GetAmber.php#amber
→ create a new miniconda in our environment called AmberTools23

→ conda create new env name … instructions listed in the Getamber.php site


--keep --parametrize (to load in the cardiolipin parameter files)
—-

system 1: POPC
system 2: POPG/E + cardiolipin bilayer

Palmitic acid | 16:0 P
Oleic acid | 18:1(9) O
Phosphatidylcholine PC
Phosphatidylethanolamine PE
Phosphatidylglycerol PG

# 3: Parametrize system
after running the packmol command, prepare to run tleao

This will generate a .pdb file and leap.in file, which can be edited by the user if required.
should we adjust coordinates? no
adjust pdb file for compatibility with leap
do we need a build.in → yes, naomi already has it (useful commands)


but first, let’s fix tleap’s issue.
before running leap and ending up with .prmtop and .inpcrd and .pdb, remove atoms (?? what is this about??? find out) something needed removing but can't remember what

check useful commands for the full command set for tleap, and that’s it until HMR + charge calcs.

build.in

addPath /global/home/groups/fc_zebramd/software/amber18/dat/leap/cmd
addPath /global/home/groups/fc_zebramd/software/amber18/dat/leap/parm
addPath /global/home/groups/fc_zebramd/software/amber18/dat/leap/lib
addPath /global/home/groups/fc_zebramd/software/amber18/dat/leap/prep

source /global/home/groups/fc_zebramd/software/amber18/dat/leap/cmd/leaprc.protein.ff14SB
source /global/home/groups/fc_zebramd/software/amber18/dat/leap/cmd/leaprc.water.tip3p
source /global/home/groups/fc_zebramd/software/amber18/dat/leap/cmd/leaprc.gaff2
source /global/home/groups/fc_zebramd/software/amber18/dat/leap/cmd/leaprc.lipid17

loadamberparams /global/home/groups/fc_zebramd/software/amber18/dat/leap/parm/frcmod.ionsjc_tip3p

(commented out) load mol2 file
p = loadPDB bilayer_dltB_aligned_prepped_dowsed_minimized.pdb

setbox p centers 0.0

check p

saveamberparm p BCDX_dltB_ff14SB_TIP3P.prmtop BCDX_dltB_ff14SB_TIP3P.inpcrd
savepdb p BCDX_dltB_ff14SB_TIP3P.pdb

quit

tleap -f build.in

732 residues had naming warnings.
Thus, there are split residues;
residue sequence numbers will not correspond to those in the pdb.

Created a new atom named: OW within residue: .R<WAT 437>
  Added missing heavy atom: .R<WAT 437>.A<O 1>
  total atoms in file: 128746
  Leap added 21 missing atoms according to residue templates:
       20 Heavy
       1 H / lone pairs
  The file contained 22 atoms not in residue templates


FATAL:  Atom .R<WAT 436>.A<OW 4> does not have a type.
FATAL:  Atom .R<WAT 437>.A<OW 4> does not have a type.

Replaced HB2 (18) in Met (2) with H2? → there is also just an H2 there at pos 15
782 HG is not understood CYX (46)

everything fixed!!!
Box dimensions:  121.510000 121.042000 102.674000


next, we need to use leap to add parameters to our system and acquire sim files
what does HMR do again?
command in tleap: charge pdb
check box size and charge warnings, adjust charges
commands in tleap
check charge on system
run parmed -p <filename> netCharge
if very close to 0, it’s fine

if we need to remove an atom from the system


double check with parmed that we got down to zero
do HMR


Leap processing
PDB File Compatibility Conversion: convert the prepared PDB file to a PDB file compatible with Leap, which is the Amber tool for simulation setup
Combine the lipid membrane PDB file + the protein/water PDB file for processing by leap
That should result in .prmtop and a .inpcrd file for simulation
Force field
amber’s ff14sb, tip3p
determine overall charge on system
no D-ala contribution
the protein overall should have its own net charge
in Maestro, make sure you cap the protein termini (option in the Protein Prep Wizard) to neutralize them
in the final step using Leap, we add ions such that we neutralize the box
usually Naomi does a quick back of the envelope calculation to determine how many Na and Cl to add
adds enough Nas and Cls to reach 150 mM of each, but the exact quantity will probably be different

Last minute checks:
- salt balanced
- checkValidity

---
# 3.2: Generate other input files
## Minimize, Heat, Equilibrate, Production
lastly, we start our simulation
Bilayer depression/buckling
Berendsen barostat recommended rather than Monte Carlo
If using Monte Carlo, a 10 Å or less LJ cutoff can induce buckling in large patches (200x200 A). While smaller patches do not undergo extreme distortion, they do deviate from experimentally derived parameters due to the effective compression.
Using a switching function for non-bonded interactions instead of a hard LJ cutoff avoids this undesirable result.

sam do thinking on backing thing sup to wasabi and do a tutorial on how partitions / external hard drives work

tleap will give a warning if net charge is not zero
change cut to 10.0
what is the ref file set to (Min_3 vs prev.rst)

## Run Bash
  
# 4: Kick off simms
for savio setups:
folder structure:
sims → dltBCDX
	→ dltB_POPC_14sb_tip3p → v1 → 
		→ input file title: BCDX_dltB_ff14SB_TIP3P.pdb/.prmtop/.inpcrd
		→ job run title: dltB_POPC_14sb_TIP3P_v1_r1
	→ dltB_PGPECL_14sb_tip3p → v1 → all prep, run files here

for run_amber.sh:
general params
job title specified, email address specified
.out and .err files specified
open-mode = append
dependency = singleton
partitions possible:
want to use GPU nodes only?
savio2_1080ti
savio3_gpu
savio2_gpu
savio2
savio4_gpu
partition, qos, and gres to use follow the table below
number of gpus or cpus to use, amount of memory, per run
time: 48 hrs
no nodes excluded atm
no CPU only params in use right now
1 node used for each run, each run is 1 task, 2 CPUs per tasks
this is 2 CPU cores used per sim run
GPU params, these are in use
1 node, 1 task, 2 CPUs per task = 2 CPU cores per sim run
gres = gpu:1, or gres=gpu:GTX2080TI:1

the remainder of the run amber files checks if Min, Heat, and Eq steps have all completed. If something is incomplete, the script prompts running of that file using the correct inpcrd/prmtop files

then it starts Prod_1. once Prod_1 is done, it calls a regex expression to figure out what the final outputted Prod step is. It continues the simulation from there while outputting .out, .nc., and .rst files for that Prod run.

to figure out the available account, QoS, and partition to use:
sacctmgr -p show associations user=$USER

# 5: Monitor sim progress and troubleshoot

# 6: Add images:
   	- gram-positive bacteria cell wall with LTAs + D-alanyl
	- PyMol OPM
   	- Maestro charge states
   	- ACE, NME groups, normal state
   	  
# 7: Process/clean out of tutorial later:
- coordinates may be slightly shifted?
- to delete extra charges and balance the system, you can just delete ions from the membrane pdb file
- membrane systems often have bad clashes of lipid chians that need resolving via the minimization steps

how to confirm sim is stable:
The M2 receptor has RMSD hovering around 2.5 A (using carbon-alpha atoms), whereas the iperoxo agonist is more stable, with RMSD 1 - 1.5 A. These results are acceptable and indicate a stable simulation.
cpptraj m2_IXO.prmtop<image.trajin
cpptraj m2_IXO.prmtop<prot_rmsd.trajin
cpptraj m2_IXO.prmtop<lig_rmsd.trajin
from justin’s tutorial

- common debug issues (bilayer populating, pdb file naming, capping before dowsing, ..)
-  description of the various ligands
- what does HMR do again?
- compare using maestro for water molecule placement to using 3D-RISM
- must remove waters placed in the membrane region?
- antechamber for checking partial charges?

- - Justin’s tutorial mentions using 3D-RISM to add waters to the system. We used Maestro’s add waters function → how to find the diff?
- antechamber → used to specially parametrize ligands, but we have no ligand (also no.lib or .frcmod files)
- we are working with Lipid_ext of AmberTools21(expanded out from Lipid21)
done
- read manual for packmol-memgen
- found ambertools2 manual
- read packmol paper

so we need to surround protein with a lipid bilayer + some water (packmol-memgen)
decide box size + bilayer size → 
box + bilayer size generated by packmol-memgen, and water layer can be added

determine whether to remove waters before making the membrane box → it seems not since naomi’s instructions mention “protein/water file”, but why/why not?
re-exported the .mae to .pdb for use with packmol
should i specify a unique box size in packmol? or is determines independently

read into gram-cell negative bacteria / staph/strep bilayer compositions
86 POPC and 42 POPG for the bacterial model, as well as 7500 water molecules → S aureus
major membrane lipid species of S. aureus: 
S. aureus
PG, CL, Lyso-PG, GPL, Lysyl-PG, Type I LTA, FA

Few architectures have been developed where the inner and outer leaflets contain the most common lipid species or analogues thereof for GN (PE, PG and CL) and GP (PG, CL, and lysyl-PG) bacteria.
These models often contain 2 or more different lipid species asymmetrically arranged in a bilayer, with the outer and inner leaflets composed primarily of LPS (restricted to the outer leaflet) and/or a mixture of PE, PG and sometimes CL.
We first monitored the interaction of NBD-labeled peptides with liposomes composed of cardiolipin and phosphatidylglycerol (CL∶PG), anionic lipids composing membranes of streptococci [39] but lacking all cell wall components including peptidoglycan and LTAs. 
 The most abundant phospholipid found in Gram-positive bacterial membranes, including staphylococcal membranes, is phosphatidylglycerol (PG). PG can be converted to cardiolipin (CL) and lysyl-phosphatidylglycerol (L-PG)
https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8921416/#CR272 
informative abstract: https://www.sciencedirect.com/science/article/pii/S0005273610004104 

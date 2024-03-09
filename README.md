# Simulation-Prep-Tutorial-dltBC
walkthrough of preparing a new system for simulating, using dltBC as an example
## Intro and Goals
dltBCDX is a multi-protein complex embedded in the cell wall of gram-positive bacteria and is involved in decorating the outer surface of the cell wall with D-alanine residues. These D-alanines attach to LTAs (lipotechoic acid) on the extracellular surface and provide the bacterium with a positive charge, which protects the bacterium from attack by antibiotics or an immune response.

Since D-alanine passes through the dltBCDX complex via sequential covalent interactions, it can't be modeled through a single simulation. This tutorial is based on the preparation of dltB alone and dltBC simulations.

We are going to prepare our system using various software packages, then initiate simulations on Savio (UC Berkeley's High Performance Computing Cluster).

dltB has 415 residues (depends on species).
dltBC has 415 (B) + 79 (C) residues.
For both simulations, we will be using a POPC bilayer to simulate the bacterium cell wall, and Amber's FF14SB and TIP3P water model.

### Steps for setting up a simulation:
1. Prepare protein
2. Add environment / solvate
3. Parametrize system
4. Kick off sims
5. Monitor, troubleshoot

# 1: Prepare protein
### Extra Notes
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
- Command: `align dltB_alone, dlt_BC_OPM`
	Make sure you align your protein *to* the OPM file, not the other way around. Your protein's name should come first in the command.
- Delete dlt_BC_OPM
- File → Export Molecule → Check Retain PDB IDs → Selected: dltB_alone, __not__ all → Save as a .pdb file

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
   
- Start a new X2Go session. To use X2Go with a monitor, you must specify the resolution in settings as either maximum possible or according to dimensions of Display 2 and select your monitor as Display 2. Dimensions of Display 2 are 1920 x 1080.
- Open Maestro in X2Go and create and title a new project.
	- NOTE: If Maestro is not opening, there may be an issue with the computing cluster. Maestro usually only takes 60s max to open.
		- Open terminal, type "maestro", it should note that a process has been added.
 		- Work on something else for 10 minutes.
		- If it still hasn't opened, check if you have any processes running across various workstations that may be impeding opening.
		- Helpful commands to type into X2Go terminal (use with care): ps -x, ws_ps, kill -9 PID, ws_kill, ssh other_workstation_name, pkill -U username
		- If still not working after several attempts to examine processes, do something else for a while and come back to it, or email Kathy/Singam, or visit the MGCF in person and talk to Singam directly. :(  

## File Transfers
- MobaXTerm can be used to transfer files between your local computer and the remote filesystem.
	- SSH protocol, Port 22, specify username (type your username here), and specify an MGCF workstation name such as nano.cchem.berkeley.edu.
   	- The remote filesystem is navigable via the left sidebar
  	- To download from remote to local: select the file you want from the remote filesystem, then click download at the top of the sidebar.
  	- To upload from local to remote: click upload at the top of the sidebar, then navigate through your local filesystem in the popup and select the file(s) you want to add.

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
### Intro
We want to examine the local environment of each of the protein's polar/charged residues.  
From: https://computecanada.github.io/molmodsim-amber-md-lesson/11-Protonation_State/index.html  
"The protonation states of titratable amino acids (Arg, Lys, Tyr, Cys, His, Glu, Asp) depend on the local micro-environment and pH. A highly polar microenvironment will stabilize the charged form, while a less polar microenvironment will favor the neutral form."  
  
Some residues have charge states that will be influenced by whether they are in a particularly positively or negatively charged region of the protein. We cannot depend on the generic charge state of these residues, because their pKa value will be affected by the region they inhabit. If a residue's pKa shifts dramatically, it may need to occupy a different protonation state from what is expected.  
  
Example: aspartic acid (D) changing from pKa 3.71 to pKa 7.6.  A lysine (K) and aspartic acid (D) near each other may favor a protonated lysine and deprotonated aspartic acid.  
  
### How to optimize charge states:
1. No Flip residues (**Asn, Gln**) can be ignored. Highly unlikely that anything needs changing.
2. Ignore Ser, Thr, Tyr
3. Check **Lys, Asp, and Glu** in detail
	- All three will usually retain a charge (Lys protonated, Asp & Glu deprotonated)
 	- What is the usual pKa for the given side chain? What is the pKa listed in the Maestro display?
  	- Has there been a dramatic shift in pKa that may influence the charge state?
   	- Are there more favorable interactions available in a different charge state/conformation?  
	__electrostatic interactions (mauve) >> hbonds (yellow) >> aromatic hbonds (teal)__
  
4. Check **His** in detail
	- 3 possible states: HIE (H on one of the two N's), HID (H on the opposite N), HIP (positively charged, both N's have an H)
   	- The extra (+) charge in HIP is not favorable. Only select the HIP state if it will induce the formation of a salt bridge/electrostatic interaction.
     	- If the pKa of His has been shifted by its surroundings to be >7-8, then you can protonate it. This usually occurs if there's an acidic residue (such as Asp/D or Glu/E) next to it encouraging a salt bridge.
  
5. For Lys, Asp, Glu, His: If there are two unique bonding patterns possible, both equally favorable, you may need to set up two different simulation sets and analyze both datasets.
  
6. Common residues that pop up
	- Ignorable: Gln (Glutamine, Q, polar uncharged), Asn (Asparagine, N, polar uncharged), Tyr (Tyrosine, Y, hydrophobic), Cys (Cysteine, C, special res), Ser (Serine, S, polar uncharged), Thr (Threonine, T, polar uncharged)
	- Examine closely: Lys (Lysine, K, positive charge), Asp (Aspartic Acid, D, negative charge), Glu (Glutamic Acid, E, negative charge), His (Histidine, H, positive charge)

### Steps
- Round One: Manual optimization
	- Protein Prep Wizard --> Refine tab
 	- select Label pKas
	- Run Interactive Optimizer (new window opens)
   		- Select Label pKas, use PROPKA 3.0
     		- click Analyze Network
  		- click on the first species and click through possible states using the "<" ">" buttons. Repeat for every species in the list and optimize each charge state. Navigate to the next species using the up and down arrow keys.
- Label this structure with "preprocessed_noCaps" in the sidebar and export PDB from Maestro. Use MobaXTerm to download from remote filesystem to local computer, then scp using the dtn and add it to Savio filesystem.
  
- Round Two: Dowse (Add internal water molecules to stabilize the charged residues in the protein)
	- Dowser is built into the Savio bash, so just call `dowser filename.pdb`
 	- Confirm that dowser completes correctly, does not crash
  	- scp dowserwat.pdb onto local filesystem, then upload to remote using MobaXTerm
  	- Import Structure into Maestro --> select dowserwat.pdb to import
  	- Shift+Select both the "preprocessed" pdb from prior and "dowserwat.pdb" --> right click --> select "Merge" from menu
  - Rename the merged structure "prepped_dowsed_noCaps"
  	  
- Round Three: Manual re-optimize
	- Protein Prep Wizard --> Import and Process tab --> check Cap termini --> Preprocess
 	- Add "capped" the name of this new structure. Make sure you have the correct structure selected. Double check on problems, missing atoms, and review & modify tab.
  	- Refine tab --> re-run the Interactive Optimizer now with capped + dowsed protein. Adust charge states as needed.
  - Label new structure with "opt"
  	  
- Round Four: Minimize
	- Refine tab --> Restrained Minimization (hydrogens only, OPLS4)
- Label new structure with "min"
- Export prepared structure as PDB file from Maestro to remote. Download PDB from remote to local, then scp to Savio
---
[What is Cap Termini?]  
The goal is to neutralize the protein backbone by adjusting the C and N-termini and spreading their charge as evenly as possible.
- N terminus (first residue): the NH2 group gets a carboxyl (acetyl, ACE) tacked on.
- C terminus (last residue): the carboxyl gets an amide (N-methyl, NME) tacked on.
- For more on termini: https://www.cup.uni-muenchen.de/ch/compchem/tink/tink2.html 
  
# 2: Add environment/solvate
## Adjust PDB for compatibility with Leap
- Use the pdbcleanup.txt file as reference  
  
Issues I've run into before:
- Incorrect spacings between columns due to having made edits
- Change NME's CA to a CH3
- Change HOH to WAT
  
If you need to check for errors, no need to re-run packmol-memgen, just send tleap the original edited pdb file, confirm the bugs were fixed, then re-start building from packmol
  
## Generate bilayer + water box
### Intro
Depending on the system you are preparing you may need to perform some or all of the following steps:
- Add internal waters
- Add ions
- Surround protein in a water box
- Surround protein in a membrane/lipid bilayer
  
For our system, we have already added internal waters using dowser. Now we will add ions and surround our protein in a lipid bilayer. Our environment will also include a buffer layer of waters surrounding the bilayer.  
We will perform this step using packmol-memgen.  

We want to set up two different systems:  
system 1: POPC  
system 2: POPG/E + cardiolipin bilayer  
  
P: Palmitic acid | 16:0  
O: Oleic acid | 18:1(9)  
PC: Phosphatidylcholine  
PE: Phosphatidylethanolamine  
PG: Phosphatidylglycerol  
CL: Cardiolipin  

### Commands
To setup POPC:    
`source activate AmberTools 21`  
  
```packmol-memgen --pdb dlt_BC_oriented_prepped_dowsed_capped_opt_min.pdb --lipids POPC --ratio 1 --preoriented --notprotonate --nottrim --salt --salt_c Na+ --saltcon 0.15 --dist 15 --dist_wat 17.5 --ffwat tip3p --ffprot ff14SB --fflip lipid17 --nloop 50```
  
To setup PG:PE:CL (65:27:8):  
```packmol-memgen --pdb dltB_aligned_prepped_dowsed_minimized.pdb --lipids POPG:POPE:CL --ratio 65:27:8 --preoriented --notprotonate --nottrim --log packmol_log.txt --salt --salt_c Na+ --saltcon 0.15 --dist 12 --dist_wat 20 --keep --parametrize```  

### Notes
- Command description:
	- lipid bilayer: of POPC with a ratio of 1:1
	- preoriented: into a bilayer, so that packmol does not try to rotate our structure for us
	- notprotonate, nottrim: do not change protonation states
	- salt, salt_c, saltcon: add ions. use Na+ (not K+), and Cl- is the default acceptor. add to a concentration of 150 mM (0.150 M)
	- dist: minimum distance to the box border in x,y,z directions
	- dist_wat: distance between edge of bilayer and edge of box --> this is the width of the water buffer included around our lipid bilayer
	- ff: list force fields of the water molecules, protein, and lipids. To use combinations of lipids to more accurately resemble a Gram-positive bacterium cell well, you must have Lipid21 or higher installed.
	- nloop: packing iterations per molecule, usually 20, but I set it to 50 to see if we could get any better minimizing. This is not necessary, and may make packmol-memgen intolerably slow.
	- keep, parametrize: to load in the cardiolipin (CL) file __didn't do this for dltB or dlt_BC though?__  
  
- Output: pdb file containing protein structure, internal waters, bilayer, ions, and water box  
	- Takes a long time to run. Make sure your connection to the HPCC does not time out, or laptop shut down.
	- If you read through the run log, you will notice that it signals that minimization has not converged. This is okay. It is already extremely close.
	- Might populate lipids into the core of your transmembrane protein! Visualize in VMD (after generating a trajectory) and ensure this is not the case!
	- The dimensions of the system are by default estimated by packmol-memgen based on the size of the protein to be packed.

- For a more detailed list of parameters, try the following:
	- Call `packmol-memgen - - help`
	- Here is the introductory paper to the software, it is very short and informative: https://pubs.acs.org/doi/epdf/10.1021/acs.jcim.9b00269.
	- Look up an Amber manual and navigate to the packmol-memgen section. Amber21 manual (link: https://ambermd.org/doc12/Amber21.pdf) --> section 13.6, page 220.  
---
### Adding a new FF
To use cardiolipin (CL) requires Lipid_ext, which is a part of the Lipid21 package, and we are using Lipid17 right now.   
To download Lipid21:
- update to a new version of AmberTools (needs Lipid 21 or higher to use CL): https://ambermd.org/AmberTools.php, then re-compile 
- to update: https://ambermd.org/GetAmber.php#amber
- create a new miniconda in our environment called AmberTools23
- conda create new env name … instructions listed in the Getamber.php site  
  
# 3: Parametrize system (prmtop, inpcrd)
- First create an input file for running tleap.
  - `vim build.in`
  - switch to Insert mode: <Ctrl+I>
  - Paste (Ctrl+V) the following:
```
addPath /global/home/groups/fc_zebramd/software/amber18/dat/leap/cmd
addPath /global/home/groups/fc_zebramd/software/amber18/dat/leap/parm
addPath /global/home/groups/fc_zebramd/software/amber18/dat/leap/lib
addPath /global/home/groups/fc_zebramd/software/amber18/dat/leap/prep

source /global/home/groups/fc_zebramd/software/amber18/dat/leap/cmd/leaprc.protein.ff14SB
source /global/home/groups/fc_zebramd/software/amber18/dat/leap/cmd/leaprc.water.tip3p
source /global/home/groups/fc_zebramd/software/amber18/dat/leap/cmd/leaprc.gaff2
source /global/home/groups/fc_zebramd/software/amber18/dat/leap/cmd/leaprc.lipid17

loadamberparams /global/home/groups/fc_zebramd/software/amber18/dat/leap/parm/frcmod.ionsjc_tip3p

#load mol2 file
p = loadPDB bilayer_dltB_aligned_prepped_dowsed_minimized.pdb

setbox p centers 0.0

check p

saveamberparm p BCDX_dltB_ff14SB_TIP3P.prmtop BCDX_dltB_ff14SB_TIP3P.inpcrd
savepdb p BCDX_dltB_ff14SB_TIP3P.pdb

quit
```

Things that may need editing:
- force fields used for protein, water, lipid
- loadPBD: name of PDB file (should be your output PDB from packmol-memgen)
- saveamberparm and savepdb: edit file names

- Then call:
```
source activate AmberTools21
tleap -f build.in
```

- Rename output files to system.*
- Check leap.log and search "box" --> the simulation box size is automatically determined by leap.
	- Box dimensions for dltB: 121.510000 121.042000 102.674000  
  	- Dimensions for dlt_BC: 120.654000 119.415000 115.541000  
- Check charges (should be zero or close to it)  
   ```
   source activate AmberTools21
   tleap
   pdb=loadpdb $smthsmth.pdb
   charge pdb
   ```
   ```
   parmed -p system.prmtop
   netCharge
   ```  
   The protein should have its own net charge. We add ions such that we neutralize the box. There is no D-ala contribution. A protein net charge of -0.0007 for example, is acceptably small.

   - To edit charge:
   	- Do back-of-envelope calculations using box size to determine the number of Na and Cl in the box.
   	- Calculate what needs adding or removing to reach 150 mM worth of molecules of each
   	- Add the following to the build.in file and re-run leap: `addIons2 Na ##` or `addIons2 Cl ##`
   	- If ions need removing...?
   	  
- parmed: `checkValidity`
- parmed: Hydrogen Mass Repartitioning
   ```
   HMassRepartition dowater
   parmout system_HMR.prmtop
   go
   ```
- Download system.prmtop and system.pdb, open in VMD
- Check for lipids within central core
   - color protein with NewCartoon and by ColorID: `protein`
   - color lipids within 15-25 of protein with VDW and by element: `not waters and not element Na and not element Cl and not resname ACE and not resname NME and not protein and within 15 of protein`
   - check for lipids populated **into** the protein  
- If lipids need removing:
   - Note down the associated residue IDs/atom numbers. For one lipid molecule, you may have multiple numbers: PA, PC, OL, etc  
   - Run the following commands (adapted/borrowed from http://archive.ambermd.org/201706/0182.html):  
     ```
     parmed -i system.prmtop
     loadRestrt system.inpcrd
     strip :64329-64343
     outparm system_new.prmtop system_new.inpcrd
     outpdb system_new.pdb
     go
     ```
- Check for lipids within residues (Y, F)
   - select Y and F of protein only: `protein and (resname TYR or resname PHE)`
   - change lipids to sticks
   - examine for any lipids jammed into the aromatic rings of Y and F

 Output of leap: .prmtop, .pdb, and .inpcrd file for simulation

# 4: Kick off sims
1. Create folders:
   - directory structure: sims → dltBCDX → dltB_POPC_14sb_tip3p → v1 → prep/runs
   - input file title: system.pdb/system_HMR.prmtop/system.inpcrd
   - job run title: dltB_POPC_14sb_TIP3P_v1_r1
   - prep
   	- files from dowsing, packmol-memgen, leap
   	- system.pdb, system.prmtop, system.inpcrd, system_HMR.prmtop
   	- sample run_amber.sh
   	- sample Eq, Min, Heat files
  - run directories: run_1 - run_5
  
2. run_amber.sh
   - Confirm that the sequence of commands in correct, and that the reference files (-ref) is set correctly to the preceding step
   - Change run time
   	- Min, Heat, Eq 1-10, Eq 11-15 = 2hrs, 2hrs, 16hrs, 12hrs
   	- First few Prod steps while confirming the sim is stable: 8 hrs
   	- After confirming sim is not crashing: 23/24 hours
   - Change as preferred:
   	- queue (savio3_gpu or savio4_gpu). partition, qos, nodes per GPU, and gres to use follow the reference table in the savio website.
   	- may need to exclude nodes. we run on GPUs. 1 node, 1 task, 2 CPUs per task = 2 CPU cores per sim run
   	- email address & notification prefs (END, FAIL, ALL)
   - Copy file to each directory
   	- Within each directory, edit the job title to r1/r2/r3..

   Explanation of code: The run_amber file checks if Min, Heat, and Eq steps have all completed. If a step is not yet executed, the script prompts the running of that file using the correct inpcrd/prmtop/ref files. Then it starts Prod_1. once Prod_1 is done, it calls a regex expression to figure out what the final outputted Prod step is. It continues the simulation from there while outputting .out, .nc., and .rst files for that Prod run.
   	  
3. Minimize, Heat, Equilibrate, Production
   Note on Bilayer depression/buckling:
     "A recent study has found that large membrane patches buckle using the Monte Carlo barostat, whilst smaller patches show a systematic depression of the area per lipid. The buckling behaviour is corrected using a force-switch for non-bonded interactions....The area per lipid depression is also found with Lipid21. However, POPC remains near the experimental area per lipid. The Berendsen barostat is thus preferable over the Monte Carlo barostat when simulation time allows."
     "If using Monte Carlo, a 10 Å or less LJ cutoff can induce buckling in large patches (200x200 A). While smaller patches do not undergo extreme distortion, they do deviate from experimentally derived parameters due to the effective compression. Using a switching function for non-bonded interactions instead of a hard LJ cutoff avoids this undesirable result."

  Params to check, adjust:
  - cut to 10.0
  - isotropic, semi-isotropic, anistropic pressure coupling
  - barostat: in bilayer simulations, use Berendsen (`barostat=1`) to prevent buckling
  - restraints (@ vs :). Changes depending on step. Common restraints are: "everything except waters and ions", "everything except lipids", "backbone atoms only"
    
  - Copy the Min, Heat, Eq, Prod files to every run sibdirectory

4. Add symlinks to the .prmtop and .inpcrd files, to each run subdirectory
   `ln -s fullpathtoprepdirectory/system_HMR.prmtop system.prmtop`
   `ln -s fullpathtoprepdirectory/system.incprd system.inpcrd`

5. Submit jobs for every directory
   From the v1 parent directory:
   ```for i in {1..5}; do cd run_$i; for j in {1..2}; do sbatch run_amber.sh; done; cd..; done;```   
   
# 5: Monitor sim progress and troubleshoot
   ```sacct --format=jobid,jobname,start,elapsed,end,node,state -S 03/07 --name=dlt_BC_POPC_14sb_TIP3P_v1_r2```  
   ```squeue -o "%.30j %.8u %.8T %.10M %.9l %.6D %R" --me --name=dltB_POPC_14sb_TIP3P_v1_r1 ``` 
   
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

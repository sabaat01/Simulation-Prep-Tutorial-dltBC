# Simulation-Prep-Tutorial-dltBC
walkthrough of preparing a new system for simulating, using dltBC as an example
## Intro and Goals
dltBCDX is a multi-protein complex embedded in the cell wall (membranes?) of (gram-positive?) bacteria and is involved in decorating the outer surface of the cell wall with D-alanine residues. These D-alanines start on the intracellular side and pass through the complex through covalent bond modifications. On the extracellular side, they attach to LTAs (lipotechoic acid) and provide the bacteria with a positive charge, which protects the bacteria from (degradation???).

In this simulation, we are only modeling dltB, which is suspected to facilitate D-ala in jumping from dltC to dltX.

We are going to prepare dltB using various software packages, then run the sims on Savio (Berkeley's HPCC).

dltB has 415-430 residues depending on species. We will be modeling the full protein within a POPC bilayer.

Generic steps for setting up a simulation:
1. Acquire appropriate PDB files (#one)
2. Clean and prepare the protein (#two)
3. Add environment / solvate (#three)
4. Generate starting files (#four)
5. Prepare Savio (#five)
6. Kick off sims, monitor (#six)
---

# 1: Prepare protein

## Acquire original PDB
- Download a crystal strxr of dltBCDX
- Open PyMol
- Remove everything not needed

- Download the new dltB structure (by Harvard group) from google drive
- Open in PyMol
- Label by chain, color by chain
- Command: select chain B and DltBCDX
- Hide unselected
- A → copy selection to object → new → rename → dltB_alone

## Orient into a membrane frame

We want everything oriented such that x and y are the 2D dimensions of the bilayer and z is the depth. Orientations of Proteins in Membrane (OPM) is a site that publishes structures of proteins from PDB oriented into a bilayer.
- Download dltB's OPM database file
	- 6BUG strxr link:
- In PyMOL: Open → find the 6BUG file and load into PyMOL alongside dltB_alone
- Click all → A → zoom
- Color dltB_alone and 6BUG different colors for later comparison. Color the OPM file by segi or chain
- Optional: remove dltC (chain A) from the 6bug_from_OPM object, prior to aligning
- Command: <align dltB_alone, 6bug_from_OPM>
- Delete 6bug_from_OPM
- File → Export Molecule → Check Retain PDB IDs → Selected: <dltB_alone>, __not__ all → Save as a .pdb file

## Access Maestro virtually
Maestro requires a license. UC Berkeley's Chem Library can provide access through the Molecular Graphics Computation Facility (MGCF). You will have to access a virtual environment to run Maestro. Alternatively, you can visit the MGCF in person in Tan Hall and use Maestro directly on an MGCF monitor.

- Assuming an account with the Chem Library MGCF is already set up, ensure the following basic items are complete:
	- MobaXTerm installed
	- starting password has been changed
 	- X2Go installed
  	- Settings for X2Go and MobaXterm have been adjusted according to the MGCF instructions
- Helpful tutorials for completing the above tasks:
	- hhfhf
	- hhhdhd
- Start a new X2Go session. To use X2Go with a monitor, you must specify resolution in settings as either maximum possible or according to dimensions of Display 2 and select the monitor as Display 2
  
- Use Cyberduck to transfer the dltB aligned pdb to MGCF remote directory/account
	- Instructions for accessing your MGCF remote directory via Cyberduck: MGCF FAQ page
   
- Open Maestro and create a new project
- Import structure: dltB_aligned.pdb

## Add hydrogens
- Select P for Protein
- Build → 3D Builder → +H
	- This will add hydrogens to every residue in the protein

## Resolve any identified issues
- Search for Protein Prep Wizard in the upper right menu and select once it appears
- In the wizard → check SS1
- Confirm there are no issues
- Missing atoms -> check for any incorrectly loaded side chains
- Examine all other Protein Reports. If something appears for every residue in the entire system it's probably fine. Steric clashes and improper torsions are okay because the system will be minimized to an optimal atomic arrangement later.
- Check Ramachandran plot
	- 3-4 outliers were visible
- Go to Import and Process window
- Check SS1 for correct parameters
- Go to Review and Modify window
- Delete heteroatoms: we had an atom that needed removing, but double-check before deleting stuff in general

## Check protonation states
We want to examine the local environment of the protein's residues. Some residues have charge states that will be influenced by whether they are in a particularly positively or negatively charged region of the protein. 
We cannot depend on the generic charge state of these residues, because their pKa value will be affected by the region they inhabit. If a residue's pKa shifts dramatically, it may need to occupy a different protonation state from expected.
Example: aspartic acid (D) changing from pKa 3.71 to pKa 7.6.  A lysine (K) and aspartic acid (D) near each other may favor a protonated lysine and deprotonated aspartic acid.
Goal: use Maestro to examine which residues are in pockets that may affect their pKa, or the pKas of nearby residues

- Interactive Optimizer - first pass
- Export PDB for Maestro and run dowser
- Recheck assignments using Interactive Optimizer again
- Perform a Restrained Minimization (hydrogens only, OPLS4)
More notes written in blue notebook!
Want to check glu, asp, lys, and his. His is its own beast
- Export prepared PDB file
	- To export from MGCF remote directory to local, open MobaXTerm. Select the file you want from the left sidebar (directory listing) and download to local computer.

## Cap termini
Goal is to neutralize the protein backbone but making an adjustment to the C and N-terminals. These are the ACE and NME caps, which add a methyl group to the end of the protein chain. You can google ACE and NME online to see the exact conformation adopted.

**do we add waters in Maestro or Savio? Or both?**
---
# 2: Add environment / solvate

## Add internal water molecules
Add internal water molecules --> [upload our PDB file to Savio where we use the dowser command to add waters]
Combine the water pdb file (dowserwat.pdb) with the exported PDB file and upload to Savio
packmol-memgen

Packmol-Memgen notes
check what lipid modeling version we are using (Lipid 21 or higher?)
instances when lipid simulations with AMBER give undesired results + best practices

what does HMR do again?
compare using maestro for water molecule placement to using 3D-RISM
must remove waters placed in the membrane region?
antechamber for checking partial charges?
sample packmol-memgen command:
packmol-memgen --pdb ../system_pdb/m2_prep.pdb --lipids POPC:CHL1 --ratio 9:1 --preoriented --salt --salt_c Na+ --saltcon 0.15 --dist 10 --dist_wat 15 --notprotonate --nottrim

run packmol-memgen - - help for parameter setting info

output of packmol-memgen
.pdb containing protein, prepared membrane, and water box
coordinates may be slightly shifted?
to delete extra charges and balance the system, you can just delete the first ion from the membrane pdb file

membrane systems often have bad clashes of lipid chians that need resolving via the minimization steps

how to confirm sim is stable:
The M2 receptor has RMSD hovering around 2.5 A (using carbon-alpha atoms), whereas the iperoxo agonist is more stable, with RMSD 1 - 1.5 A. These results are acceptable and indicate a stable simulation.
cpptraj m2_IXO.prmtop<image.trajin
cpptraj m2_IXO.prmtop<prot_rmsd.trajin
cpptraj m2_IXO.prmtop<lig_rmsd.trajin
from justin’s tutorial


A recent study has found that large membrane patches buckle using the Monte Carlo barostat, whilst smaller patches show a systematic depression of the area per lipid. The buckling behaviour is corrected using a force-switch for non-bonded interactions.

The area per lipid depression is also found with Lipid21. However, POPC remains near to the experimental area per lipid. The Berendsen barostat is thus preferable over the Monte Carlo barostat when simulation time allows.

- Justin’s tutorial mentions using 3D-RISM to add waters to the system. We used Maestro’s add waters function → how to find the diff?
- antechamber → used to specially parametrize ligands, but we have no ligand (also no.lib or .frcmod files)
- we are working with Lipid_ext of AmberTools21(expanded out from Lipid21)
done
- read manual for packmol-memgen
- found ambertools2 manual
- read packmol paper

so we need to surround protein with a lipid bilayer + some water (packmol-memgen)
decide box size + bilayer size → 
box + bilayer size generated by packmol-memgen, and water layer can be added
The dimensions of the system are by default estimated by packmol-memgen based on the size of the protein to be packed
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


PG:PE:CL (65:27:8)
packmol-memgen --pdb dltB_aligned_prepped_dowsed_minimized.pdb --lipids POPG:POPE:CL --ratio 65:27:8 --preoriented --notprotonate --nottrim --log packmol_log.txt --salt --salt_c Na+ --saltcon 0.15 --dist 12 --dist_wat 20

→ needs updating to a new version of AmberTools (needs Lipid 21 or higher to use CL)
→ https://ambermd.org/AmberTools.php 
→ re-compile 
→ to update: https://ambermd.org/GetAmber.php#amber
→ create a new miniconda in our environment called AmberTools23

→ conda create new env name … instructions listed in the Getamber.php site

—-
edit PDB
USE PACKMOL PAPER DEFAULTS FOR DIST AND DIST_WAT (15, 17.5)
—
POPC (1)
packmol-memgen --pdb dltB_aligned_prepped_dowsed_minimized.pdb --lipids POPC --ratio 1 --preoriented --notprotonate --nottrim --log packmol_log.txt --salt --salt_c Na+ --saltcon 0.15 --dist 15 --dist_wat 17.5 --ffwat tip3p --ffprot ff14SB

—-
/global/home/groups/fc_zebramd/test_install/miniconda3/envs/AmberTools21/bin/packmol-memgen --pdb ../AF2_IFNLR1_TM_5L04_forLeap.pdb --lipids POPC:CHL1 --ratio 9:1 -- preoriented --salt --salt_c Na+ --saltcon 0.15 --dist 12 --dist_wat 20 --notprotonate --nottrim --ffwat tip3p --ffprot ff14SB --fflip lipid17

 --nottrim
--log packmol_log.txt
packmol-memgen
--pdb name.pdb
--lipids POPC
--lipids POPE/POPG:CL???? --ratio 3:1 (what should the ratio be? what kind of bilayer did naomi suggest? what kind of bilayer do i think is ideal?)
--preoriented
--salt --salt_c Na+ --saltcon 0.15
--dist_wat 15: water layer thickness of 15 A on either end (what should i set it to?)
--notprotonate: don’t add H’s, we already did it
--nottrim: do not process input receptor PDB file (since we have already prepared this)
--keep --parametrize (to load in the cardiolipin parameter files)
do i need to add: tailplane, headplane, leaflet?
—-

system 1: POPC
system 2: POPG/E + cardiolipin bilayer

Palmitic acid | 16:0 P
Oleic acid | 18:1(9) O
Phosphatidylcholine PC
Phosphatidylethanolamine PE
Phosphatidylglycerol PG

actually, packmol says it didn’t find the perfect configuration, but that what was found is likely good enough. can re-run it, but not necessary.


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

# load mol2 file
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
# 4: Generate other input files
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
  
# 5: Kick off simms
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

# 6: Monitor sim progress and troubleshoot

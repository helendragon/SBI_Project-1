# Biological considerations

During the development of this project, we've had to take into account some specific details in order to elaborate functions that considered the possible biological parameters and properly build the models.

### Distinguishing the type of the chain

The inputs given to the program may be of different nature: protein or nucleic acid. These types of chains are composed by different residues and atoms. To be able to work with both of them, we had to make some adaptations. 
Those are the functions involved in this matter:  
* Labeling:
    + ```check_type()```: First of all, we made a function to determine the type of the chain, this function checks if the chain has carbons alpha, if it does, we will consider it a protein, if it doesn't, a nucleic acid. Returns a string which the label.  
* Calculating distances:  
    + ```get_seq_from_pdb()```: As the residues of a protein are in three format letter, while the nucleotides are in two, or one, we had to get in a different way the sequences from the pdb file. We used a function in the protein case (```three_to_one()```), and getting the last letter by indexing in the nucleic acid case.
    + ```get_atoms_list()```: The superimposing step recquires the use of atom lists. We decided to consider just the CA atoms in protein, while P atoms in nucleic acid.
    + ```calc_distances_residues()```: To get the interactions, we compared by distances between CA atoms. As mentioned before, nucleic acids don't have this kind of atoms. We elaborated a function that renames the C1' atoms from the nucleic acid chains to CA (```adapt_chains()```). This way, we could compute the distances the same way as we would do if we only had one type of chain.  

### Superimposing

When superimposing two atoms lists, those lists have to contain the same number of atoms. Usually, arriving to this step means that the sequences of the superimposig chains are equal. However, there are cases when the chains are similar, but not exactly the same, they have different number of atoms. We handled this creating two functions:
* Comparing the chains:
    + ```refine_for_superimpose()```: The alignment of the sequences from the two chains will reveal its differences. We obtained the sequences with a function mentioned above, then made the pairwise alignment. Based on the ouput of this tool, we obtained a pattern of 1 and 0 that revealed the positions with matching or different residues. 
* Creating the new chains:
    + ```get_chain_refined()```: Once we obtained a pattern, we knew that the number of matches would be the same for both chains. This function creates a new chain that contains only the matching residues. This allowed us to finally obtain lists with the same number of atoms.
    
### Reducing the inputs

One initial step of the program is to analyse the chain interactions given by the user, and determine if there are redundant pairs to be able to reduce the inputs and build the models more fluently. This is done in the **reduce_inputs.py** script. 
For this purpose, the program makes pairwise comparisons to detect similar sequences, and if there is similarity, those chains enter a superimposition step where the structural similarity will be tested.  
To decide if the sequence similarity between the tested sequences was high enough, we set a treshold of 0.90. This was based on the distribution of the obtained scores, which showed extreme values. Setting the treshold at this point, ensured that the actual similar sequences would obtain a greater score.  

### Dealing with chain ids  

Biopython model object doesn't allow containing chains with the same id. 
To avoid this we decided to give new ids to the chains from the input to handle them without errors during the program. We used numerical annotation to deal with big numbers of chains. This solved the "same id issue" when working with objects, but to save the model in PDB format, the chain id must be of just one character.  
When a chain is added to the current model, its id is changed again. At this point, the new id is obtained from a list of ASCII characters (*ascii_list*) located in the **utilities.py** script. This last change of id allowed us to handle the saving of the created model in PDB format, but the ASCII characters list is limited, therefore if a macrocomplex is formed by more than 83 chains, we have to create a new model to continue adding chains without trouble.
To sum up, if the macrocomplex has less than 83 chains, it will be created as one model and saved in one single file, but if it doesn't, the protein will be created in more than one model and saved splitted in different files. Besides avoiding biopython errors during the program, having two or more files for one big structure avoids issues in Chimera when labeling chains.


# Tutorial

## Prerequisites

* <b>Python</b> 3.0 Release

https://www.python.org/download/releases/3.0/

* <b>Modeller</b> 9.19 Release

https://anaconda.org/salilab/modeller

It is also recommended to have a program for interactive visualization and analysis of molecular structures and related data, such as <b>Chimera</b> or <b>VMD</b>.

https://www.cgl.ucsf.edu/chimera/

http://www.ks.uiuc.edu/Research/vmd/

## Input files

The input files must be <b>pairs of interacting chains</b> (.pdb), which has to be located into a directory decided by the user. This directory can also be in <i>tar.gz</i> format. 

In case you are interested in <b>rebuilding a macrocomplex</b>, you can introduce its <b>pdb file</b> (.pdb) from PDB."

## Python modules

A module is a file containing Python definitions and statements. It is important to consider that definitions from a module can be imported into other modules or into the <i>main</i> module.

* <b>sbi_project.py</b>: <i>main</i> module or program created to reconstruct a macrocomplex given a set of interacting pairs (prot-prot, prot-RNA). It also considers the possibility to rebuild a macrocomplex if the input is a macrocomplex (.pdb). 

  Aditionally, this module contains the ArgumentParser object, created using argparse module, which is used for command-line     options, arguments and sub-commands. The ArgumentParser object will hold all the information necessary to parse the command   line into Python data types.

* <b>get_interactions.py</b>: this module is <b>only</b> used by the <i>main</i> module if the given input is a macrocomplex (.pdb). It gets all possible interactions given a this structure and computes the distance between chains parining that ones which accomplish the following conditions: less than 8A between their carbons alpha (CA) and implication of at least 8 CA in this interaction. It creates the pair files (.pdb) which later, in the <i>main</i> program will be filtered by the imported module named <b>reduce_inputs_func.py</b> to get only the non-redundant interactions to build the model.

* <b>reduce_inputs_func.py</b>: this module is imported into the <i>main</i> module to speeds up the further process to create the model, as it is getting only the non-redundant interactions from the whole set of input pairs. From a set of input pairs (.pdb) compares each chain pair with the rest. This comparision has two steps: sequence and structural. First of all, a pairwise alignment is performed to determine similar sequences (cut-off = 0.9). The score of the alignment is normalized by the length of the longest sequence. If the normalized score is higher than the stablished cut-off, the analysis proceeds to the second step. In this step, a superimposition is performed between the similar chains. These similar chains are part of two different interaction pairs, which will be refered as fixed and moving chains. We apply the rotran matrix to the couple of the moving chain, which will be refered as alternative/new chain. Finally, the distances between the CA of the comparing chain (couple of the fixed chain) and the new chain are computed. If the distance is lower than 9A, the interactions will be considered the same (redundant). It saves the non-redundant chain pairs (.pdb) obtained from the reducing inputs process, which will be considered as the required input pairs to build the model.

* <b>functions.py</b>: this module is composed by a set of different <b>functions</b> to solve biological and technical problems during the analysis. Thus, it is imported into the other modules in order to use the defined functions.

* <b>utilities.py</b>: this module is composed by a set of different <b>variables</b> to solve biological and technical problems during the analysis. Thus, it is imported into the other modules in order to use the defined functions.

* <b>classes.py</b>: this module is composed by the definition of a <b>class</b> to check if the input variable is a directory or a compressed directory (.tar.gz).

* <b>DOPE_profile.py</b>: this module is <b>only</b> used if the argument '-e', '--energy_plot' is set. It creates a DOPE profile plot (.jpg) from a macrocomplex (.pdb), which has no acid nucleic chains using Modeller.

* <b>DOPE_comparison.py</b>: this module is <b>only</b> used if the argument '-ref', '--refine' is set. It refines a model previously generated with the <i>main</i> program according to the optimization parameters defined by Modeller. It also generates a comparison energy plot between the model generated by the program and the refined one.

## Output files

It is necessary to consider that the output files will depend on the command-line options and arguments the user determines while performing the analysis. All of them will be stored in the output directory selected by the user (-o or --output).

The arguments considered to fill the ArgumentParser object can be achieved by:

```
$ python3 sbi_project.py -h (or --help)
```

The following <b>arguments</b> are stablished:

* <b> -i INDIR, --input INDIR </b>
  
  It must be a directory provided by the user which contains the inputs pairs (compressed format is also
  available <i>.tar.gz</i>). In case you want to rebuild a macrocomplex from PDB, you can introduce its pdb file (.pdb).

   <i>(default: None)</i>
 

* <b> -o OUTDIR, --output OUTDIR</b>
 
  This is a directory which will be created during the program, structured in other subdirectories. 
  
  <i>(default:None)</i>
  
  
* <b> -v, --verbose</b>

  This option will allow the user to completely follow the progam. 

  <i>(default: False)</i>
  
  
* <b> -cc {0.25,0.5,1,1.2,1.5}, --clash_distance_cutoff {0.25,0.5,1,1.2,1.5} </b>
  Choose a cut-off distance to detect clashes between chains. 
 
  <i>(default: 1.2)</i>
  
  
* <b>-e, --energy_plot </b>     
  DOPE profile of the macrocomplex (not including nucleic acid chains) generated by Modeller.
 
   <i>(default: False)</i>
 
 
* <b> -ref, --refine </b>        
  Refines a model previously generated with the program according to the optimization parameters defined by Modeller.
 
  <i>(default: False)</i>
 
 
* <b> -ver {short,intensive}, --version {short,intensive}</b>
  Short version only considers the first model the program can build, while Intensive version considers all the possible
  models the program can build depending on the input pair it starts the construction of the model.
 
  <i> (default: short) </i>
 
 
 
If the <b>default options</b> are set, these are the following outputs:

<b>files</b>: 

 - alignments_results.txt: file genereated from the pairwise comparisons between all the inputs to create a dictionary of equivalences.
 
 - unique_chains_fasta.mfa: multifasta file containing the unique chains to allow the user to introduce the stoichiometry of the macrocomplex.
  
<b>files, directories and subdirectories(->)</b>:

 - <i>models (directory)</i> -> 1 (directory) -> .pdb: the generated model.
 
 - <i>reduced_inputs (directory) </i>: non-redundant interactions pdb files obtained with the reduction of inputs process.
 
(*) If <b>-ver, --version</b> intensive, inside the models directory all the possible models are created inside each subdirectory named with the number of the model.



Additionally to these previously mentioned outputs, if <b>-e, --energy_plot </b> is set, the following outputs are generated:

<b> directories and files</b>:
 - <i>models (directory) </i> -> 1 (directory) -> dope_profile.jpg (image of the plot) and .pdb.profile (DOPE profile file).
 - <i>temp (directory)</i>: temporary directory containing the pdb files without the acid nucleic chains, in order to that use them in the refining process, once the model has been built (to avoid Modeller errors).
 
 
 Additionally to these previously mentioned outputs, if <b>-ref, --refine</b> is set, the following outputs are generated:

<b> directories and files</b>:
 - <i>models (directory) </i> -> 1 (directory) -> .pdb = generated model and .pdb.B = refined.
 - <i>optimization_results (directory) </i> -> dope_profile, refined_models, stats and log_files (subdirectories). Inside dope_profiles, an image named '.pdb.dope_profile.jpg' comparing both models is created.
 - <i>temp (directory)</i>: temporary directory containing the pdb files (.pdb = generated model, .pdb.B = refined) without the acid nucleic chains, in order to that use them in the refining process, once the model has been built (to avoid Modeller errors).
 
## Authors

* **Marta Badia** - [Github](https://github.com/martabadiagraset)
* **Marta Ferri** - [Github](https://github.com/martaferri)
* **Aida Ripoll** - [Github](https://github.com/aidarripoll)


See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## Acknowledgments

* Hat tip to anyone who's code was used
* Inspiration
* etc

# busco2phylo-nf
Nextflow pipeline for turning a set of BUSCO results into a species tree


##  Dependencies

- conda
- nextflow

Different steps of the pipeline use `mafft`, `iqtree` and `trimal`. 

In the example config file the environment name is `orthology_env`. You can create a working a environment with `conda create -n orthology_env -y -c bioconda mafft iqtree trimal`.

## Executing the pipeline on the Sanger farm

Clone this repository:
```
git clone https://github.com/lstevens17/busco2phylo-nf.git
```

Submit an interactive job. Choose a queue appropriate to the time of execution of your pipeline. Take into account that the default config submits jobs to the 'normal' queue of 12 hours execution and if the job fails it will be resubmitted into the 'long' queue.
```
mbMem=5000; bsub -n 1 -q long -R"span[hosts=1] select[mem>${mbMem}] rusage[mem=${mbMem}]" -M${mbMem} -Is bash
```

Launch the pipeline: 
```
nextflow -C busco2phylo-nf/busco2phylo.config run busco2phylo-nf/busco2phylo.nf --busco_dir_path [absolute_path_to_BUSCO_directory] -profile farm
```
Note: the `busco_dir_path` should contain a set of BUSCO results with sensible names. The directory name will be used to create the sequence headers in the FASTAs and therefore become the tip labels in the resulting trees. 

## Output

The pipeline will produce a tree in two distinct ways: 1) supermatrix tree, where the species tree is inferred from a concatenated alignment (ie a "supermatrix") of all ortholosgs using IQ-TREE and 2) a summary tree, whereby a tree is inferred for each orthogroup and these trees are fed to ASTRAL which estimates a species tree with branch lengths in coalescent units (branch lengths are then inferred in substitutions per site using the supermatrix and IQ-TREE, but fixing the topology on the ASTRAL one) based on the support values in your tree. So `busco2phylo/astral/supermatrix.fa.iqtree` will have internal support value ranging from 0-1 and are therefore posterior probabilities rather than ranging from 0-100 which would be bootstraps.

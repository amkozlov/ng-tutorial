**NOTE:** Optional tasks are marked with an asterisk (*). 

# Lab #1: RAxML-NG Basics

## Exercise 0 : Getting ready

**1.** Check input data:
```
cd /home/phylogenomics/workshop_materials/; git clone https://github.com/amkozlov/ng-tutorial
cd ng-tutorial
ls
```
you should see roughly the following:
```
5.phy   dna.map    multi199.part  multi5.fa   prim2.part  prim.part  prim.phy.raxml.log  prot140.phy  prot21.fa.ckp  prot21.fa.out   prot21.part  README.md   terrace.part
bad.fa  fusob.phy  multi199.phy   multi5.map  prim.cons   prim.phy   prot140.part        prot21.fa    prot21.fa.log  prot21.fa.tree  rbcl.phy     terrace.fa
```

**2.** Run RAxML-NG binary without parameters
```
raxml-ng
```
this will show RAxML-NG version and short usage help:
```
RAxML-NG v. 1.2.1 released on 22.12.2023 by The Exelixis Lab.

[...]
System: Intel(R) Core(TM) i7-8550U CPU @ 1.80GHz, 4 cores, 23 GB RAM

Usage: raxml-ng [OPTIONS]

Commands (mutually exclusive):
  --help                                     display help information
[...]
```

**3.** It is always a good idea to run alignment sanity check before starting the actual analysis:

```
raxml-ng --check --msa prim.phy --model GTR+G
```
This alignment is clean, so you should see:
```
Alignment can be successfully read by RAxML-NG.
```
**4.\*** Run `--check` command again with `bad.fa` and examine error and warning messages.

## Exercise 1 : Tree search

**1.** Infer ML tree with default parameters for `prim.phy`.<br>
NOTE: use `--prefix S1`, we will use the results of this run in the following exercises.

**2.** Check log-likelihoods for all 20 resulting trees (HINT: use `grep`).

**3.** Check topological distances between all 20 trees (so-called Robinson-Foulds or RF distance):

**4.\*** Repeat steps 1, 2 and 3 for `fusob.phy`, using 5 random + 5 parsimony starting trees.

## Exercise 2 : Bootstrapping / "all-in-one"

**1.** Run "all-in-one" analysis (ML tree search + bootstrapping + branch support mapping) for `prim.phy`. 

NOTE: This is convenient, but not always possible/efficient for large datasets! We will explore an alternative approach in Lab #2.

**2.** Open `<PREFIX>.raxml.support` in the tree viewer of your choice.

**3.\*** Repeat step 1. using a fixed number of bootstrap replicates (100). Did this change the support values?

## Exercise 3: Tree likelihood evaluation
The `--evaluate` command computes the likelihood of a fixed tree topology after re-optimizing all model parameters and branch lengths. We will use it to compare different evolutionary models:
```
raxml-ng --evaluate --msa prim.phy --tree S1.raxml.bestTree --model GTR+G --prefix E_GTRG
```
**1.** Repeat the above command for `GTR+R4`, `GTR`, `JC` and `JC+G` models. Don't forget to change the `--prefix`!

**2.** Check the log-likelihood scores. Which model yields the highest log-likelihood? Does it mean that this model should be preferred?

**3.** Check AIC/BIC scores for our runs (lower values are better). Which model should be preferred according to information theoretical criteria ?

## Exercise 4: Partitioned models

**1.** Re-run tree inference for `prim.phy` using partitioned model in `prim2.part`. 

**2.** Compare the results (log-likelihood and tree topology) to the Exercise 1. 

## Exercise 5: Constrained tree search

**1.** Establish a baseline: run regular (unconstrained) tree search on `prim.phy` with 100 random + 100 parsimony starting trees.<br>
NOTE: use fixed random seed (e.g.,`--seed 1`) to ensure deterministic and comparable results!

**2.** Re-run the same search with a topological constraint defined in `prim1.cons`:

**3.**  Re-run the same search again now with a diffent constraint, `prim2.cons`:

**4.** Compare runtimes and likelihoods among all three runs. How can we interpret the results?

## Exercise 6: Protein data and ModelTest-NG

**1.** Check online help 
```
modeltest-ng --help
```
Important options are:
- `-i ALIGNMENT`
- `-d DATATYPE`, where `DATATYPE` is `nt` (default) or `aa`

**2.** Run model selection for `prot21.fa` (protein alignment).

**3.** Run tree inference with the best-scoring model determined by ModelTest-NG.

**4.\*** Re-run ModelTest-NG including the `LG4M` and `LG4X` models which are not tested by default. 

# Lab #2: Parallelization / Analyzing Large Datasets

## Exercise 7: Alignment pre-processing

Before analyzing a large dataset, it is highly recommended to convert alignment into RAxML binary format (RBA) using the `--parse` command. Doing this pre-processing step has two advantages:
- binary alignments are faster to load than PHYLIP/FASTA
- you will get estimated memory requirements and the recommended number of threads for
this dataset

**1.** Parse alignment file `fusob.phy`

**2.\*** Explore how resource estimates change depending on the selected `--model` (e.g., `GTR`, `GTR+R8`)

## Exercise 8: Automatic and manual parallelization

**1.** Run a tree search on the binary alignment from Exercise 7 with automatic parallelization to establish a baseline (NOTE: use `-seed 1`!).<br>
If tree search runs too fast (<10 seconds), increase the number of starting trees or use `--all` mode.

**2.** Re-run the same tree search with different number of `--threads` and `--workers`. Can you find a configuration which beats the default heuristic in terms of runtime?<br>
HINT: don't forget to change the `--prefix` for every run!<br>
HINT: you can use `grep "Elapsed time:" T*.raxml.log` to quickly compare runtimes.

**3.\*** Try to oversubscribe CPU cores by using 9 or more threads. What do you observe? 

For more details on RAxML-NG parallelization, please read: https://github.com/amkozlov/raxml-ng/wiki/Parallelization

## Exercise 9: Bootstrapping revisited

**1.** For `prim.phy`, infer 100 bootstrap replicate trees in two batches of 50.

**2.** Combine bootstrap trees froma all runs into a single file (HINT: use `cat`):

**3.** Compute and map bootstrap support values to the best ML tree (`S1.raxml.bestTree`).

**4.** Open `<PREFIX>.raxml.support` in the tree viewer of your choice.

**5.\*** Recompute support values using the Transfer Bootstrap Expectation (TBE) algorithm. Did this change the support values? <br>
HINT: you do NOT need to re-infer the bootstrap trees!

## Exercise 10: Adaptive search

**1.** For this exercise, we will be using an experimental version of RAxML-NG. Please make sure that you specify the correct binary, `raxml-ng-adaptive`:
```
raxml-ng-adaptive
```
```
RAxML-NG v. 1.2.1-adaptive released on 12.01.2024 by The Exelixis Lab.
```
**2.** Run the adaptive tree search for the dataset from the Exercise 5 (`prim.phy` with partitioned model). Compare runtimes, likelihoods, and best-found ML tree topologies. Can you spot the differences in the log files?

```
raxml-ng-adaptive --adaptive --msa prim.phy --model prim2.part --prefix P1A
```
**3.\*** Run the adaptive tree search for `fusob.phy`. Compare the results with Excercise 1.4.

## Exercise 11: ParGenes

**0.** Check ParGenes options 
```
python ~/software/.source/ParGenes/pargenes/pargenes.py  --help
```
**1.** Analyze all alignments in the  `~/software/.source/ParGenes/examples/data/small/fasta_files/` folder using default ParGenes settings

```
python ~/software/.source/ParGenes/pargenes/pargenes.py  -a ~/software/.source/ParGenes/examples/data/small/fasta_files/  -o parout -c 4 -m --scheduler fork
```

**2.\***. Run model testing and tree inference from 1 parsimony + 5 random starting trees for the `prot21.fa` alignment.

```
echo "prot21.fa" > msa_filter.txt

python ~/software/.source/ParGenes/pargenes/pargenes.py  -a ~/workshop_materials/ng-tutorial/ --msa-filter msa_filter.txt -o parout2 -c 4 -m --scheduler fork -d aa -s 5 -p 1
```

**3.** Examine the results from both runs

# Further reading

RAxML-NG documentation: https://github.com/amkozlov/raxml-ng/wiki

Detailed RAxML-NG tutorial: https://github.com/amkozlov/raxml-ng/wiki/Tutorial

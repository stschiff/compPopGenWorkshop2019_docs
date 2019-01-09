# F Statistics

## Data
For this lecture, we will be using data from the (fully public) version of
 Lazaridis et al 2014, that can be downloaded from
https://reich.hms.harvard.edu/datasets. This data has an unique ascertainment
scheme, so throughout we need to keep ascertainment bias in mind. 

In order to save time, I subset the data and converted it to various formats,
namely the `plink` and `eigenstrat` formats that we will be using throughout.

### Overview
We'll be using three subsets: The `world`-dataset contains a small set of samples
from all continents. The `europe`-dataset contains data from all European
samples only, The `ancient`-dataset contains two high-coverage archaic hominins, as well as some primate outgroups.

Let's haven an overview over the data:
Both datasets are in multiple formats. In particular, the Eigenstrat format
consists of three files, ending with `.geno`, `.ind`, and `.snp`, corresponding
to the genetic data, annotations for individuals and annotations for SNP,
respectivelu. Similarly, the `plink`-format data has files ending with `.bed`,
`.bim` and `.fam`, with the same roles, respectively. The `geno` and `bed`-files
are in binary, whereas the other files can be read by any text editor and `R`.

To see which samples we have available, let's load the `world.ind` into `R`
(using the `read.table` function). 

Q1: which populations are present in this file?
Q2: which (and how many) populations are present in the `europe`-data set?


## F2, Fst and allelic covariance
Next, we will use R to explore these data sets and to examine differences
between F2, FST and covariance. For this purpose, we load the script
`analysis.R`

```R
rm(list=ls())
source("scripts/analysis.R")
ls()
```
As you can see, we loaded 4 functions into `R`'s memory, which are called
`fst_mat`, `fst_pair`, `read_data` and `rm_missing`.

`read_data` will load data into memory, `fst_pair` will calculate F2, FST and a
covariance between two populations, and fst_mat will do so for all possible
pairs. `rm_missing` is a helper function to deal with missing data

Q3: have a look at the fst_pair function in `R` (by typing its name in the
console) can you figure out how f2 is calculated? Does this match with the
formula from the lecture?

next, pick one of the data sets and load it into memory.
```R
data <- read_data("data/world.1.frq.strat")
data
f_stats <- fst_pair(data, "Sardinian", "French")
f_stats
```
The `.frq.strat`-file has all allele counts in each population (it can be
generated using `plink`). 

Q4: Which is larger, FST or F2? Is that what you expect?

Next, let's calculate the statistics for all possible pairs
```
m <- fst_mat(data)
fst <- m[[1]]
f2 <- m[[2]]
cov_mat <- m[[3]]

library(gplots)
heatmap.2(fst, symm=T, trace="n")
dev.new()
heatmap.2(f2, symm=T, trace="n")
dev.new()
heatmap.2(cov_mat, symm=T, trace="n")

```

Q5: How do the matrices differ? Is this what you would expect?
Q6: Also calculate the fst-matrix for the european data (loading the
`data/europe.1.frq.strat` file. How does this matrix differ from the worldwide
one? (Hint: If it takes a long time to compute, you can speed it up by only
looking at a subset of snp, i.e. `m <- fst_mat(data[1:5e5,])`

## Admixture F3 statistics

We will use the `world`-dataset that contains a small set of samples from all
continents. We'll use the program `qp3pop` from `admixtools` to calculate
F3-stats. 

Now, we'll hypothesize that Uygurs are an admixed group resulting from the mixture of two populations. We'll try to find the pair of populations in our panel that can best stand in for those two populations. For this, we'll resort to using Admixture F3 statistics.

To run `qp3pop`, we need two input files, one with the populations we are
interested in, and one that tells `qp3pop` which files to use.

for the first file, we can use  a script `make_f3list.R` that takes two
arguments: The target population and the output filename. Hence type (e.g.)

```
Rscript make_f3list.R Uygur uygurs.poplist
```

This file is used in the other input file:

```
genotypename:   data/world/world.geno
snpname:	data/world/world.snp
indivname:	data/world/world.ind
popfilename:    uygurs.poplist
```

Finally, run qp3Pop:

```
qp3Pop -p AdmixtureF3.par > AdmixtureF3.out
```


Q7: Are any F3 statistics negative?  Are they significant? (|Z| > 3?)? Which pairs of populations do these correspond to? What could this mean?


## Outgroup F3 statistics

Next, we can compute outgroup-F3 stats in a very similar fashion. To do so,
we'll use the European data. If you looked carefully, you'll find a population
labeled as `unknown`. Which population could they be most closely related to?

We'll begin by computing outgroup F3 statistics to determine which 
are the populations that share the most drift with them. We'll use Mbuti Pygmies as the outgroup. A requirement when computing F3 statistics in AdmixTools is to have a list of population triplets. 

Run the script `Rscript scripts/make_f3outgroup_list.R MbutiPygmy Unknown OutgroupF3.txt` 
Note that the first column corresponds to our population of interest, the second column iterates over a series of populations we'll compare them to, and the third column is a fixed outgroup, in this case Mbuti.

```
Unknown  Papuan  MbutiPygmy
Unknown  Sardinian MbutiPygmy
Unknown  Yoruba  MbutiPygmy
....
```

We also need to create a parameter file (OutgroupF3.par) to specify the location of our input files:

```
genotypename:  data/europe/europe.geno 
snpname:	data/europe/europe.snp
indivname:	data/europe/europe.ind
popfilename:    OutgroupF3.txt
```

Now run the program qp3Pop, which comes in the AdmixTools package:

```
qp3Pop -p OutgroupF3.par > OutgroupF3.out
```

Q8: Look at the output log file. Which population shares the largest amount of drift with the Unknown population? Which shares the least amount of drift? How do you interpret these results?
Q9: Create a visualization of this data using R


## D-statistics

next, we can test for admixture in the past history of humans by lookin for an excess of ABBA or an excess of BABA sites in a 4-population tree. 

For this purpose, we use the `ancient` dataset, which contains 
a subset of the Lazaridis-data with Chimpanzee, Oranguan,
Neandertal, Denisovan and a small number of modern Human population samples.

To calculate D-stats is very similar to running `qp3Pop`: we need an input with 
the data files, and a second input with the specific comparisons we'd like.

For the second file, write a script generating all possible combinations of the
following:
1. one non-human primate
2. one archaic hominin
3/4. all possible pairs of modern humans

```
Primate_Chimp Ancient_Neandertal MbutiPygmy French
...
```

The parameter file (Dstats.par) should include the following information:

```
genotypename:   data/ancient/ancient.geno
snpname:        data/ancient/ancient.snp
indivname:      data/ancient/ancient.ind
popfilename:    Dstats.pops
```

Now, run the qpDstat program in AdmixTools:

```
qpDstat -p Dstats.par > Dstats.out
```

Look at the output log file. The first numerical column corresponds to the value of the D-statistic (defined here as (BABA-ABBA)/(BABA+ABBA)). The second numerical column is the Z-score corresponding to this D-statistic. The 3rd and 4th numerical columns are the BABA and ABBA counts, respectively, and the last column is the total number of SNPs available for analysis.

Q10: Are any of the statistics significant? In what direction? (i.e. is there an excess of ABBA or BABA patterns, relative to what you would expect under a 4-population tree?). Is this consistent with what you might expect?
Q11: Create a visualization of this data using R

## TreeMix
After exploring the hypothesis-driven F-statistics, let's see how 
this compares with TreeMix, an (almost-)unsupervised admixture graph program. 
We'll use the same data sets we used for the f-statistic analyses. For
simplicity, I already converted them to the custom input format of treemix,
they can be found in the subfolder data/treemix/

```
mkdir treemix
```

Now, let's sequentially run treemix with 0 - 4 migration events for the `world` and `ancient` data
sets (you can also try the europe data set, but it might take a while), to test its limits.

```
for rep in {0,1,2} do
for mig in {0,1,2,3,4}; do
treemix -i data/treemix/world.in.gz -o treemix/world_m${mig}_${rep} -m $mig -root Khomani -k 1000
done; done
```

We can visualize the results using R scripts that can be downloaded along with the treemix program. For example, for a tree with no migration events, we can plot the corresponding graph as follows.

```
R
source("scripts/treemix_plotting.R")
plot_tree("treemix/world_m0_0")
```

Plot the other graphs and study their respective topologies. Which admixture events do you observe? Do these make sense based on your knowledge of human history? Are there differences between the replicates/ Note that some migration events may be added because of poor representation of certain populations that may have been important in human history. y

Take a look at the length of the branches in the tree. Why are some branches much longer than others? What does the length here represent?

You can also plot the residual fits from each graph. For example, for the graph containing no migrations:

```
R
source("scripts/treemix_plotting.R")
plot_resid("treemix/world_m0_0", "data/world/world.poplist")
```

Q12: Take a look at these residuals. Which pairs of populations are worst-fitted under each graph? How could that be improved?
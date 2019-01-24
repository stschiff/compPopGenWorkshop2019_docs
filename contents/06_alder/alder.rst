Estimating Admixture times with *Alder*
=======================================

Overview
--------

Admixture between two distinct gene pools generate an linkage disequilibrium (LD) that extends even megabases across genomes. This "admixture LD" extends far longer than the typical LD in non-recently-admixed human populations, which often decays within few tends of kilobases. Once formed, admixture LD decays, like the typical LD, over time due to recombination. Therefore, a relationship between the strength of LD the distance between markers provides a powerful window to look into the temporal aspect of admixture. Such a feature is by definition not visible by F-statistics, which does not consider LD.

The demographic admixture process is far more complicated than any simple model can fully capture, but even a surprisingly simple model can provides a useful insight into the characteristics of the admixture process. Here we explore the ``alder`` program that is based on the simplest possible model: a single pulse-like admixture.


Data
----

In this session, we will use a small ``EIGENSTRAT`` format genotype data, including the following populations::

  /data/alder/popgen_alder_test_190120.geno
  /data/alder/popgen_alder_test_190120.ind
  /data/alder/popgen_alder_test_190120.snp

.. admonition:: Exercise

    How many individuals are in the data set? How many groups? Could you count the number of individuals in each population?


Preparing the parameter file
----------------------------

We will use ``alder`` v1.03 in this session. Parameter file for alder takes a format similar to that for the programs in the ADMIXTOOLS package.::

    genotypename: <your_genotype_data>.geno
    snpname: <your_genotype_data>.snp
    indivname: <your_genotype_data>.ind
    admixpop: TARGET
    refpops: REF1;REF2
    raw_outname: <output>.txt

The "admixpop" argument is for the name of your target population. The name should match with the group ID in the 3rd column of ``.ind`` file.

The "refpops" argument is for a list of reference populations for testing admixture in the target. The standard setting is to provide two populatuions (semicolon separated). You can provide more than two (also semicolon separated). In that case, all pairs of references will be run one by one. You can also provide only a single reference population: alder will run only the so-called "1-reference" model. This will still provide you with an estimate of admixture date in case that you do not have a good proxy for the contributing ancestry. However, "1-ref" mode is prone to false positives and does not provide a proper test of admixture signal.

Two important optional parameters are ``mindis: NUMBER`` and ``jackknife: YES``. The "mindis" option sets up the minimum distance (in Morgan) between two SNP bins that you want to include in exponential curve fitting. If not set up, ``alder`` calculate a correlation of LD decay between the target and each reference population and take the minimum distance that shows no correlation in both references. If it continues beyond 2 cM (due to either a strong bottleneck or admixture in a reference), the program aborts. ``jackknife: YES`` is to output 22 additional weighted LD files, each representing a leave-one-chromosome-out jackknifing results. It is useful if you want to do exponential fitting on your side.

Let's write two parameter files, one without mindis argument, and the other including ``mindis: 0.005``. For both tests, we use ``Uygur`` as the target and ``Daur`` and ``Georgian`` as references.::

    pt1=($(pwd)"/")
    fn1="/data/alder/popgen_alder_test_190120"
    target="Uygur"
    refs="Daur;Georgian"

    for i in $(seq 1 2); do
        echo 'genotypename: '${fn1}'.geno' > test${i}.par
        echo 'snpname: '${fn1}'.snp' >> test${i}.par
        echo 'indivname: '${fn1}'.ind' >> test${i}.par
        echo 'admixpop: '${target} >> test${i}.par
        echo 'refpops: '${refs} >> test${i}.par
        echo 'raw_outname: '${pt1}'alder.test'${i}'.raw.txt' >> test${i}.par
        if [ "$i" -eq 2 ]; then echo 'mindis: 0.005' >> test${i}.par; fi
    done



Running the programs
--------------------

Running command line is similar to the other programs (e.g. ``qpDstat`` or ``smartpca``). Let's run the following two tests::

    alder -p test1.par > test1.log
    alder -p test2.par > test2.log

If you want to see the log file being written in real time, you can try pipe + tee command instead of re-direction::

    alder -p test1.par | tee test1.log
    alder -p test2.par | tee test2.log


.. note:: Running ``alder`` with this small dataset takes a few minutes, using a single core. Multithreading is  also available.


Reading the main output file (weighted LD table)
------------------------------------------------

The main output file, the name of which is designated by the "raw_outname:" argument, looks like below::

    #   d (cM)      weighted LD         bin count
    #    0.050       0.00153525           7478597
    #    0.100       0.00073147           6079776
    #    0.150       0.00049288           5940521
    #    0.200       0.00038920           5914194
    ...
    #    0.600       0.00023054           5700318
    #    0.650       0.00022210           5702649
         0.700       0.00020821           5687007
         0.750       0.00020796           5670508
    ...
        49.900      -0.00000575           3938767
        49.950      -0.00000674           3910622
           inf       0.00000416       -1089862760
    # last row: affine term computed from pairs of SNPs on different chroms


.. admonition:: Exercise

    What do you think would "#" at the beginning of some rows mean?



Reading the log file
--------------------

The first block of the log file repeats the parameter settings and input files. There are a few default settings that are good to know::

    Data filtering:
                mincount: 4

    Curve fitting:
                 binsize: 0.000500
                  mindis: auto
                  maxdis: 0.500000

    Computational options:
             num_threads: 1

Alder calculates LD between a pair of SNPs across individuals. The "mincount:" argument dictates the minimum number of individuals that you have genotype data for both SNPs. For computational efficiency, alder first split genome into small non-overlapping bins and use the distance between two bins for all pairs of SNPs between the two. The "binsize" argument dictates the size of these bins. "First-bin-then-measure-distance" approach of alder produces bigger variance of actual distance between pairs of SNPs in the same bin than "first-measure-distance-then-bin" approach of rolloff, but it is computationally much faster. You can also multithreads alder by setting the "num_threads:" argument.


If you did not set up the "mindis:" argument, the next block describes the minimum distance between bins that alder found::

    Checking LD correlation of test pop Uygur with ref pop Georgian
      binsize: 0.1 cM
      (distances are rounded down to bins; bin starting at 0 is skipped)

    d (cM)    LD corr (scaled)   bin count
     0.100     0.119 +/- 0.007      186942
     0.200     0.034 +/- 0.010      364453
     0.300     0.012 +/- 0.004     5760262
     0.400     0.018 +/- 0.006     2861516
     0.500     0.005 +/- 0.004    11396910   losing significance (1)
     0.600     0.006 +/- 0.005    11395248   losing significance (2)
    lost significance; computing bias-corrected LD corr polyache
     0.600     0.216 +/- 0.186   <-- approx bias-corrected LD corr

    Decay curves will be fit starting at the folliowing min distances (cM):
      (to override, specify the 'mindis' parameter)

                    Daur   0.700
                Georgian   0.600

The program will use 0.7 cM for the minimum distance between bins (inclusive) to fit exponential decay.


The next block describes the main results of alder: exponential fit of weighted LD decay using two references::

    ---- fit on data from 0.50 to 50.00 cM (using inter-chrom affine term) ----
    d>0.50       decay:       19.94 +/- 1.96         z = 10.16 *
    d>0.50     amp_tot:  0.00021347 +/- 0.00001428
    d>0.50     amp_exp:  0.00021139 +/- 0.00001462   z = 14.46 *
    d>0.50     amp_aff:  0.00000416 +/- 0.00000174
    
    ---- fit on data from 0.60 to 50.00 cM (using inter-chrom affine term) ----
    d>0.60       decay:       19.32 +/- 1.98         z = 9.77 *
    d>0.60     amp_tot:  0.00020732 +/- 0.00001400
    d>0.60     amp_exp:  0.00020524 +/- 0.00001434   z = 14.31 *
    d>0.60     amp_aff:  0.00000416 +/- 0.00000174
    
    ---- fit on data from 0.70 to 50.00 cM (using inter-chrom affine term) ----
    d>0.70       decay:       18.82 +/- 1.97         z = 9.58 *
    d>0.70     amp_tot:  0.00020218 +/- 0.00001398
    d>0.70     amp_exp:  0.00020010 +/- 0.00001431   z = 13.98 *
    d>0.70     amp_aff:  0.00000416 +/- 0.00000174
    
    ---- fit on data from 0.80 to 50.00 cM (using inter-chrom affine term) ----
    d>0.80       decay:       18.45 +/- 1.98         z = 9.30 *
    d>0.80     amp_tot:  0.00019840 +/- 0.00001406
    d>0.80     amp_exp:  0.00019632 +/- 0.00001438   z = 13.66 *
    d>0.80     amp_aff:  0.00000416 +/- 0.00000174

    ---- fit on data from 0.90 to 50.00 cM (using inter-chrom affine term) ----
    d>0.90       decay:       17.97 +/- 2.11         z = 8.52 *
    d>0.90     amp_tot:  0.00019337 +/- 0.00001429
    d>0.90     amp_exp:  0.00019129 +/- 0.00001465   z = 13.06 *
    d>0.90     amp_aff:  0.00000416 +/- 0.00000174

This block shows estimates using the above-found minimum distance (0.7 cM), as well as values around it (0.2 cM range). The most important number is "decay:", which shows the estimate for admixture date and its block jackknifing standard error. It corresponds to the "n" parameter in the following exponential curve formula::

.. math::

    E[Y] = M \times e^-nd + K

.. admonition:: Exercise

    In which unit is the decay parameter estimate? Let's convert it to the unit of years.

Another important estimate is "amp_exp:", which corresponds to the "M" parameter in the above formula. Naively speaking, it becomes bigger when your choice of references matches the true sources better.

.. admonition:: Exercise

    What will the "d" parameter represent? In which unit will it be then?


.. admonition:: Exercise

    Check the log file of "test2" run, for which we provided "mindis: 0.005" argument. Do you see the difference in the results on this block? Combining the results from the two runs, do you see any trend in the decay parameter estimates over the minimum distance values?

  
Then, alder repeats fitting with the "1-ref" models. In these runs, alder uses one of the two references and the target itself to set up weight.

The last block summarizes test results. The last lines will tell repeat the most important estimates and tell you whether alder considers the test of admixture as either successful or failed.::

    Test SUCCEEDS (z=9.58, p=1e-21) for Uygur with {Daur, Georgian} weights

    DATA:   success (warning: decay rates inconsistent)     1e-21   Uygur   Daur    Georgian        9.58    6.34    8.57    28%     18.82 +/- 1.97  0.00020010 +/- 0.00001431    16.22 +/- 2.56  0.00004300 +/- 0.00000456       21.55 +/- 2.52  0.00005970 +/- 0.00000650

    DATA:   test status     p-value test pop        ref A   ref B   2-ref z-score   1-ref z-score A 1-ref z-score B max decay diff %        2-ref decay     2-ref amp_exp        1-ref decay A   1-ref amp_exp A 1-ref decay B   1-ref amp_exp B



.. admonition:: Exercise

    Let's set up new parameter files that use Daur and French as two references (instead of Georgian). Then run alder and compare the results with the current runs.



Plotting weighted LD decay curve in R
--------------------------------------

For an easy plotting in R, let's re-format the main output file (the LD table) using the follwoing bash commands::

    for i in $(seq 1 2); do
        echo -e 'Dist\tweightedLD\tnpairs\tuse' > alder.test${i}.txt
        head -n -2 alder.test${i}.raw.txt | tail -n +2 | awk '{OFS="\t"} {if ($1 == "#") print $2,$3,$4,"N"; else print $1,$2,$3,"Y"}' >> alder.test${i}.txt
    done


This is to make it into a proper table format and remove irregular lines (the last two).

In R, you can import this file, create a scatter plot, and add the fitted curve based on the estimates in the log file. For example, let's use "test1" results::

    fn1 = "alder.test1.txt"

    n = 18.82       ## decay parameter
    M = 0.00020524  ## amp_exp
    K = 0.00000416  ## amp_aff

    d1 = read.table(fn1, header=T)
    xv = as.vector(d1$Dist)        ## distance between SNP bins (cM)
    yv = as.vector(d1$weightedLD)  ## Weighted LD
    fv = as.vector(d1$use) == "Y"  ## a boolean vector marking if each bin is used in fitting
    prdv = M * exp(-1 * n * xv / 100) + K

    plot(xv[fv], yv[fv], xlab="Genetic distance (cM)", ylab = "weighted LD", pch=4, col="blue")
    points(xv[fv], prdv[fv], type="l", lwd=1.5, col="red")


.. admonition:: Exercise

    How does the exponential decay curve look? Does it seem to fit data well?






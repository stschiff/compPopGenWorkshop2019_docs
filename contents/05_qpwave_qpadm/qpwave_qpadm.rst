Admixture modelling with *qpWave* and *qpAdm*
=============================================

In the previous session, we learned how to calculate D/F\ :sub:`4` statistics and how to interpret output. F\ :sub:`4` statistics are extremely useful to test simple phylogenetic hypotheses, such as whether the relationship of four test populations fits into a tree. However, an individual f\ :sub:`4`-statistic cannot provide specifics of this "non-treeness": such as i) what is the magnitude and direction of gene flow, ii) how many gene flows are required to model the relationship between the populations used, and iii) whether the populations used are sufficient to model their relationships (or an unsampled lineage is required).

QpWave and its derivative qpAdm are tools for summarizing information from multiple F-statistics to make such sophisticated inferences possible to a substantial degree. In this section, we focus on the following specific questions:
    1) Detecting the minimum number of independent gene pools to explain a set of target populations (*qpWave*)
    2) Testing sufficienty of an admixture model within the resolution of data (*qpAdm*)
    3) Estimating admixture proportions (*qpAdm*)



Overview: summarizing a matrix of F4 statistics
-----------------------------------------------

In the *qpWave/qpAdm* scheme, you choose m "right" populations (or outgroups) and n "left" populations (targets for *qpWave*, target and references for *qpAdm*). Taking the first population on each side as the point of comparison, you build a :math:`(m-1)\times(n-1)` matrix of :math:`f_4` statistics in the following way:

.. math::

    F_{ij} = F_4 (L_1, L_j; R_1, R_j)
  
Given that you provide a sufficient number of distinct outgroup (m > n), this matrix has maximum n-1 independent columns (= rank) and minimum zero nontrivial column (i.e. zero matrix). For example, two unadmixed Native American left populations are a sister group to each other against all non-Native American right populations. Then, any :math:`f_4(\text{American} 1, \text{American} 2; \text{non-American} 1, \text{non-American} 2)` will be zero.

Assuming that this matrix has rank r, i.e. there are r independent columns and the rest is a linear combination of the r columns, we can model F matrix into product of two matrices, A and B.

.. math::
        
        F = A \cdot B

Where A is an :math:`(m-1) \times r` matrix, and B is an :math:`r \times (n-1)` matrix. You can think that columns of A matrix represents r independent :math:`F_4` statistic columns. In turn, j-th column of B matrix represents the weight values for combining A columns to reproduce the j-th column of the original F matrix.

The observed :math:`f_4` statistic is an estimate of true parameter, :math:`F_4` , with a noise/error. Therefore, the actual model fitting becomes to estimate A and B matrices in the following formula:

.. math::
            
            F = A \cdot B + E
            
where E = error matrix:

.. math::
            E = \text{F-observed} - A \cdot B

Assuming that :math:`(m-1) \times (n-1)` entries of the E matrix follows a multivariate normal distribution with mean zero (because it is the error matrix), you can write down log-likelihood and compare different models based on log-likelihood.

``qpAdm`` is a special case of ``qpWave``, in which you assume that the first left population ("target)" is a mixture of the remaining left populations ("references"), and therefore not independent. It becomes a *qpWave* case with rank (n-2) with additional constraint for scaling to make sum of admixture coefficient to 1.



Data
----

In this session, we will use a small ``EIGENSTRAT`` format genotype data, including the following populations::

  /data/qpAdm/popgen_qpAdm_test_190120.geno
  /data/qpAdm/popgen_qpAdm_test_190120.ind
  /data/qpAdm/popgen_qpAdm_test_190120.snp


Preparing "left" and "right" populations
----------------------------------------

Both "left" and "right" population lists are a simple text file, including one population name per line. In the EIGENSTRAT format input genotype data, population name matches entries in the third column of the ``.ind`` file. "Right" population list includes outgroup populations that are distantly but (potentially) differentially related to "left" populations. "Left" population list includes populations of your interest. For ``qpWave``, the order of left populations does not have specific meaning. However, for ``qpAdm``, the first left population (one at the top of the file) is the target of admixture modeling, and the remaining ones serve as reference populations for the target.

For this session, let's generate two "left" and one "right" population list files using the following code bits::

    target="Corded_Ware_Germany"
    refs="Yamnaya_Samara LBK_EN"
    og1s="Mbuti Natufian Onge Iran_N Villabruna Mixe Ami Nganasan Itelmen"

    echo ${refs} | sed s/" "/"\n"/g > left1.pops
    echo ${target} ${refs} | sed s/" "/"\n"/g > left2.pops
    echo ${og1s} | sed s/" "/"\n"/g > right.pops


.. admonition:: Exercise

    If you run ``qpAdm`` using left2.pops file, which population is your target for admixture modeling? What admixture modeling are you proposing by using this list? How would you write the left population list file if you want to test three-way admixture model, including "WHG" (Mesolithic Western European Hunter-gatherers) as the third reference?



Preparing the parameter file
----------------------------

We will use ``qpWave`` and ``qpAdm`` programs in the ``ADMIXTOOLS`` package. Parameter files for both programs have the same structure, similar to parameter files for other programs included in the package.::

    genotypename: <qpWave_qpAdm_test>.geno
    snpname: <qpWave_qpAdm_test>.snp
    indivname: <qpWave_qpAdm_test>.ind
    popleft: left1.pops
    popright: right.pops
    details: YES
    maxrank: 7

You can write two parameter files, using ``left1.pops`` and ``left2.pops`` as the left population file, respectively, using a simple loop in bash::

    pt1=($(pwd)"/")
    fn1="/home/choongwon/qpAdm/popgen_qpAdm_test_190120"

    for i in $(seq 1 2); do
        echo 'genotypename: '${fn1}'.geno' > test${i}.par
        echo 'snpname: '${fn1}'.snp' >> test${i}.par
        echo 'indivname: '${fn1}'.ind' >> test${i}.par
        echo 'popleft: '${pt1}'left'${i}'.pops' >> test${i}.par
        echo 'popright: '${pt1}'right.pops' >> test${i}.par
        echo -e 'details: YES\nmaxrank: 7' >> test${i}.par
    done


An important optional parameter is ``useallsnps: YES``. In default setting, both programs use only the SNPs that are not missing in any of the left or right populations. Therefore, all f\ :sub:`4` statistics used in the program are calculated across the exactly same set of SNPs. When ``useallsnps: YES`` is set, each f\ :sub:`4` statistic is calculated using SNPs that are present in the four populations included in the test.


Running the programs
--------------------

Running command lines are similar to the other programs (e.g. ``qpDstat`` or ``smartpca``). Let's run the following three tests::

    qpWave -p test1.par > qpWave.test1.log
    qpWave -p test2.par > qpWave.test2.log
    qpAdm -p test2.par > qpAdm.test2.log

.. admonition:: Exercise

    Run ``qpWave`` and ``qpAdm`` with the prepared parameter file.
  
.. note:: Running ``qpWave`` or ``qpAdm`` with this dataset takes 1-2 minutes, using a single core (no multithreading is available).



Reading the *qpWave* log file
-----------------------------

Both ``qpWave`` and ``qpAdm`` log files begin by iterating the parameter file, showing the current version of the program, listing left and right populations in the given order::

    ### THE INPUT PARAMETERS
    ##PARAMETER NAME: VALUE
    genotypename: /home/choongwon/qpAdm/popgen_qpAdm_test_190120.geno
    snpname: /home/choongwon/qpAdm/popgen_qpAdm_test_190120.snp
    indivname: /home/choongwon/qpAdm/popgen_qpAdm_test_190120.ind
    popleft: /home/choongwon/qpAdm/left1.pops
    popright: /home/choongwon/qpAdm/right.pops
    details: YES
    maxrank: 7
    ## qpWave version: 410

    left pops:
    Yamnaya_Samara
    LBK_EN

    right pops:
    Mbuti
    Natufian
    ...


The next block of the log file shows quantity of data used in the analysis::

      0       Yamnaya_Samara    9
      1               LBK_EN    6
      2                Mbuti   10
      3             Natufian    6
      4                 Onge   11
      5               Iran_N    5
      6           Villabruna    1
      7                 Mixe   10
      8                  Ami   10
      9             Nganasan   33
     10              Itelmen    6
    jackknife block size:     0.050
    snps: 593124  indivs: 107
    number of blocks for block jackknife: 711
    dof (jackknife):   612.975
    ...


.. admonition:: Exercise

    How many SNPs are used in the analysis according to the test1 log file?


The next block contains the actual test results for ``qpWave``. Because we have two left populations in test1 setting (Yamnaya_Samara and LBK_EN) and 9 right populations, ``qpWave`` operates on 8-by-1 f\ :sub:`4` matrix. Therefore, the maximum rank of the matrix is 1 (i.e. two left populations are differentially related to the outgroups). Rank 0 here means that the f\ :sub:`4` matrix is indistinguishable from a zero vector (i.e. two left populations are symetrically related to all outgroups listed).::

    f4rank: 0 dof:      8 chisq:   609.975 tail:      1.67697608e-126 dofdiff:      0 chisqdiff:     0.000 taildiff:                    1
    f4rank: 1 dof:      0 chisq:     0.000 tail:                    1 dofdiff:      8 chisqdiff:   609.975 taildiff:      1.67697608e-126

This block contains multiple sub-blocks, showing results from rank zero to min(maximum rank, maxrank value in the parfile). You can see the structure better with test2 log file, where you have three left populations and thus maximum rank value two::

    f4rank: 0 dof:     16 chisq:   690.774 tail:      1.18836874e-136 dofdiff:      0 chisqdiff:     0.000 taildiff:                    1
    f4rank: 1 dof:      7 chisq:    31.660 tail:       4.69503474e-05 dofdiff:      9 chisqdiff:   659.114 taildiff:      4.23672911e-136
    f4rank: 2 dof:      0 chisq:     0.000 tail:                    1 dofdiff:      7 chisqdiff:    31.660 taildiff:       4.69503474e-05

Each line contains results for a certain rank value ("f4rank: r-val"). For each rank, it returns results for two different tests.
  1) A comparison of the rank "r-val" model and the "full" model (i.e. rank = maxrank): dof/chisq/tail
  2) A comparison of the rank "r-val" model and the rank "r-val - 1" model: dofdiff/chisqdiff/taildiff

The "full" model provides a perfect fit to data because one free parameter is assigned to each entry of the f\ :sub:`4` matrix. Small *p*-value (<< 0.05) for the "tail:" cell means that the reduced rank model under consideration fits data substantially worse than the full model. That means, you need higher rank to properly explain data. Again, that means that you need more streams of independent ancestry to explain the left populations!

To obtain the minumum number of distinct ancestries required, it is often convenient to go over the "taildiff: " cell from higher to lower ranks. If the presented *p*-value is significant (i.e. < 0.05), it means that the current rank fits data significantly better than the simpler model with rank-1.

.. admonition:: Exercise

    How many streams of distinct ancestries are required to explain left1 and left2 population sets? What would that mean for the left2 populations, given that our goal is to check if Corded Ware individuals from Germany can be modeled as a mixture of Yamnaya and LBK?

.. admonition:: Exercise

    Let's make another list of left populations, this time including WHG as an additional population. Then, let's run qpWave on this list of four left populations. What is the rank of the f\ :sub:`4` matrix?


Last, each non-zero rank model is followed by the estimates of A and B matrices. In case of left2 populations and rank 2,::

    B:
              scale     1.000     1.000
           Natufian    -1.270     0.487
               Onge     0.547    -0.560
             Iran_N     0.343    -0.517
         Villabruna     0.652     2.647
               Mixe     1.570    -0.158
                Ami     0.613     0.046
           Nganasan     1.032     0.077
            Itelmen     1.279    -0.379
    A:
              scale   906.839  3447.047
     Yamnaya_Samara     0.296    -1.383
             LBK_EN    -1.383    -0.296


The product of A and B matrices is the expected f\ :sub:`4` matrix. Based on this number, you can have a good idea on which right populations are useful to distinguish between left populations.



Reading the *qpAdm* log file
----------------------------

``qpWave`` and ``qpAdm`` are essentially the same program. You can see this in the first part of the ``qpAdm`` log file, which simply repeats the corresponding ``qpWave`` results. The first block of the log file shows *qpWave* test results for the full rank and (full rank - 1), including degree of freedom, chi-square value, *p*-value, and A/B matrices.

The next block shows the main results of ``qpAdm``: admixture coefficient estimates and standard errors::

    best coefficients:     0.765     0.235
    Jackknife mean:      0.764524556     0.235475444
          std. errors:     0.034     0.034

    error covariance (* 1000000)
          1133      -1133
         -1133       1133

In the case of two-way admixture model, admixture coefficients are perfectly correlated. That is, if the coefficient for the reference 1 is alpha, that for the reference two is (1 - alpha).

The next block presents model fit measures (i.e. *p*-values) for the proposed admixture model and all "submodels" of it::

        fixed pat  wt  dof     chisq       tail prob
               00  0     7    31.536     4.95055e-05     0.765     0.235
               01  1     8    75.490     3.93452e-13     1.000    -0.000
               10  1     8   389.320               0     0.000     1.000
    best pat:           00      4.95055e-05              -  -
    best pat:           01      3.93452e-13  chi(nested):    43.954 p-value for nested model:     3.36113e-11

"tail prob" cell shows the comparison of the proposed admixture model (i.e. maxrank - 1) and the full model (i.e. maxrank). If the value is small, it suggests that the target population significantly deviate from the proposed admixture model (= a linear combination of reference populations).

"fixed pat" cell shows which (sub)model is being tested. "0" means that the corresponding reference population is included in the model, while "1" means that the reference is fixed to have zero admixture coefficient.

.. admonition:: Exercise

    Judging from the tail prob of two submodels, "01" and "10", which reference seems genetically closer to Corded Ware? Does it match with the admixture coefficients of the two-way admixture model?

"p-value for the nested model" is a useful metric for exploring whether there is a small set of references that can adequately explain the target population. Here it is provided only for the "best" submodel, but you can calculate it easily using dof and chisq values in the table.

.. admonition:: Exercise

    In R shell, let's run the following command:

        1 - pchisq(75.490-31.536, df=8-7)

    What number do you get? Can you calculate nested p-value for the worse submodel "10"?


Although the above two blocks provide all of the essential results for ``qpAdm``, the following two blocks actually harbor quite useful information. The next "dscore" block shows which outgroup is useful to distinguish between reference populations and which outgroup makes the model deviate from data when the model does not fit well. For each outgroup except for the "base" outgroup (i.e. one at the top of the right population list), you get the following sub-block::

    ## dscore:: f_4(Base, Fit, Rbase, right2)
    details:  Yamnaya_Samara        Natufian    -0.000610   -3.188475
    details:          LBK_EN        Natufian     0.001895    8.003780
    dscore:        Natufian f4:    -0.000021 Z:    -0.119830

The first two lines ("details") show the actual entries of f\ :sub:`4` matrix and Z-score, calculated by dividing the f\ :sub:`4` by its block jackknifing standard error value. That is, the above two lines mean::

    f4(Corded_Ware_Germany, Yamnaya_Samara; Mbuti, Natufian)
    f4(Corded_Ware_Germany, LBK_EN        ; Mbuti, Natufian)

The last "dscore" line makes a linear combination of the above lines, using the admixture coefficient estimates::

    f4(CW, Model; Mbu, Nat) = 0.765 * f4(CW, Yam; Mbu, Nat) + 0.235 * f4(CW, LBK; Mbu, Nat)
    -0.0000213 = -0.000610 * 0.765 + 0.001895 * 0.235

If you are testing a three-way model, you get::

    ## dscore:: f_4(Base, Fit, Rbase, right2)
    details:  Yamnaya_Samara        Natufian    -0.000630   -3.182212
    details:          LBK_EN        Natufian     0.001961    8.018318
    details:             WHG        Natufian     0.000319    1.297454
    dscore:        Natufian f4:    -0.000045 Z:    -0.255258

.. admonition:: Exercise

    Let's recover the "dscore" value by taking a linear combination of the above three numbers.


The last block is a linear combination of "dscore" rows to obtain the contrast between two non-base outgroups (in z-score scale). For example, if you want to obtain f4(Corded Ware, Model; Natufian, Onge)::

    f4(CW, Model; Nat, Ong) = f4(CW, Model; Mbu, Ong) - f4(CW, Model; Mbu, Nat)
    


**You now know how to run *qpWave*/*qpAdm* and how to interpret results!**

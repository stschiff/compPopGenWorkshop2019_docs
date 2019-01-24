Slides for this lesson are available :download:`here <./qpGraph_Posth.pdf>`.

Estimating Admixture Graphs with ``qpGraph``
============================================

Based on previous analyses we can try to reconstruct a model that includes the described observations in an admixture graph, which combines in an unique model the relationships between multiple groups that can be single individuals or larger populations. For that we are going to use ``qpGraph`` (also known as "ADMIXTUREGRAPH") part of the package ADMIXTOOLS (`Patterson et al. 2012`_. _qpGraph_ relies on an user-defined topology of the admixture graph and calculates the best-fitting admixture proportions and branch lengths based on the observed f-statistics. 

.. _Patterson et al. 2012: http://www.genetics.org/content/192/3/1065.short

Overview method
---------------

The general approach of an admixture graph is to reconstruct the genetic relationships between different groups through a phylogenetic tree allowing for the addition of admixture events. The method operates on a defined graph’s topology and estimates *f2*, *f3*, and *f4*-statistic values for all pairs, triples, and quadruples of groups, compared to the expected allele frequency correlation of the tested groups. For a given topology, ``qpGraph`` provides branch lengths (in units of genetic drift) and mixture proportions. Groups that share a more recent common ancestor will covary more than others in their allele frequencies due to common genetic drift. We should keep in mind that the model is based on an unrooted tree and that while we show graphs with a selected outgroup as the root, the results should not depend on the root position.

.. note:: The reconstructed graph is never meant to reflect a comprehensive population history of the region under study but the best fitted model to the limit of the available groups and method’s resolution.

As reported in the Method section of `Lipson and Reich 2017`_:

    “Our usual strategy for building a model in ADMIXTUREGRAPH is to start with a small, well-understood subgraph and then add populations (either unadmixed or admixed) one at a time in their best-fitting positions. This involves trying different branch points for the new population and comparing the results. If a population is unadmixed, then if it is placed in the wrong position, the fit of the model will be poorer, and the inferred split point will move as far as it can in the correct direction, constrained only by the specified topology. Thus, searching over possible branching orders allows us to find a (locally) optimal topology. If no placement provides a good fit (in the sense that the residual errors are large), then we infer the presence of an admixture event, in which case we test for the best-fitting split points of the two ancestry components. After a new population is added, the topology relating the existing populations can change, so we examine the full model fit and any inferred zero-length internal branches for possible local optimizations.”

.. _Lipson and Reich 2017: https://academic.oup.com/mbe/article/34/4/889/2838774

Data
----

For this session, we are going to use modern and ancient DNA data analysed in the recently published study about the settlement of the Americas `Posth et al. 2018`_.

.. _Posth et al. 2018: https://www.sciencedirect.com/science/article/pii/S0092867418313801

The genotype data is in "EIGENSTRAT" format and exclude from the "1240K panel" a subset of SNPs that are transitions in CpG sites. Those Cytosines are prone to be methylated and if deamination (the typical chemical modification of ancient DNA) occurs at the same position Cytosine is directly converted into Thymine without becoming Uracil. Thus, the resulting CtoT modification cannot be removed with an enzymatic reaction like performing uracil-DNA glycosylase (UDG) treatment. This results in additional noise in ancient DNA data that can be reduced by excluding those SNPs. This is especially relevant when analysing ancient samples that have been processed using different laboratory protocols.

The path to the data we are going to work on is the following::

    /data/qpGraph/Posth2018_Americas.packedancestrymapgeno
    /data/qpGraph/Posth2018_Americas.ind
    /data/qpGraph/Posth2018_Americas.snp

.. admonition:: Exercise
   
   Check how many SNPs are in this dataset.

Preparing the parameter file
----------------------------

In order to run ``qpGraph`` we need to prepare a parameter file that we can call "parQpgraph" and where we change "filename" with the names above::

    DIR: /data/qpGraph
    genotypename: DIR/<filename>.packedancestrymapgeno
    snpname: DIR/<filename>.snp
    indivname: DIR/<filename>.ind
    outpop:  NULL
    useallsnps: YES
    blgsize: 0.05
    forcezmode: YES
    lsqmode: YES
    diag:  .0001
    bigiter: 6
    hires: YES
    lambdascale: 1


Let’s go through the most relevant parameter options. ``outpop: NULL`` does not use an outgroup population to normalize f-stats by heterozygosity e.g. selecting a group in the graph in which SNPs must be polymorphic. ``useallsnps: YES`` each comparison uses all SNPs overlapping in that specific test, otherwise the program looks only at the overlapping SNPs between all groups. ``blgsize: 0.05`` is the block size in Morgans for Jackknife. ``diag:  .0001`` uses the entire matrix form of the objective function to avoid the basis dependence of the least-squares version of the computation. ``lambdascale: 1`` in order to preserve the standard scaling of the f-statistics without an extra denominator. ``lsqmode: YES`` otherwise unstable for large graphs. ``hires: YES`` controls output when more decimals are desired.

Preparing the graph topology
----------------------------

We start by constructing a scaffold graph based on previously published studies (`Lipson and Reich 2017`_, Moreno-Mayar et al. 2018 and Scheib et al. 2018). The following can be saved in a file called "Figure3a"::

    root    R
    label    Mbuti.DG    Mbuti.DG
    label    Han.DG    Han.DG
    label    Onge.DG    Onge.DG
    label    MA1    MA1
    label    USR1   USR1
    label    USA_SanNicolas_4900BP    USA_SanNicolas_4900BP
    label    Canada_Lucier_4800BP_500BP    Canada_Lucier_4800BP_500BP

    edge    a R Mbuti.DG
    edge    b R nonAfrica
    edge    c1 nonAfrica EastEurasia
    edge    c2 EastEurasia EastAsia
    edge    c3 EastEurasia Onge.DG
    edge    c4 EastAsia EastAsia2
    edge    c5 EastAsia EastAsia3
    edge    c6 EastAsia2 Han.DG
    edge    c7 EastAsia2 EastAsia4
    edge    c8 nonAfrica WestEurasia
    edge    c9 WestEurasia E_HG2 
    edge    c10 WestEurasia E_HG3
    admix   ANE E_HG2 EastAsia3
    edge    c11 ANE MA1
    admix   FA EastAsia4 E_HG3
    edge    c12 FA Beringia
    edge    c13 Beringia USR1
    edge    c14 Beringia NA
    edge    c15 NA Canada_Lucier_4800BP_500BP
    edge    c16 NA NA2
    edge    c17 NA2 USA_SanNicolas_4900BP

Running the program
-------------------

To run ``qpGraph`` we need to specify the parameter file "parQpgraph" and three output files that are ``.ggg`` , ``.dot`` and ``.out``::

    qpGraph -p parQpgraph -g Figure3a -o Figure3a.ggg -d Figure3a.dot > Figure3a.out

.. note:: Running ``qpGraph`` with this dataset takes 1-2 minutes.

Reading the output files
------------------------

First let’s inspect the ``.out`` file. If we open that on the terminal with ``less Figure3a.out`` you can directly go to the bottom of the file with ``shift-g`` sequence. Among other values we should see reported the total number of individuals used for all groups ``indivs`` and the total number of ``snps`` available from at least one individual. 

.. note:: as mentioned above, not all those SNPs are used for each f-statistic

::

    outpop: NULL
    population:   0             Mbuti.DG    5
    population:   1               Han.DG    4
    population:   2              Onge.DG    2
    population:   3                  MA1    1
    population:   4                 USR1    1
    population:   5 USA_SanNicolas_4900BP   17
    population:   6 Canada_Lucier_4800BP_500BP    6
    before setwt numsnps: 882908  outpop: NULL
    setwt numsnps: 681581
    number of blocks for moving block jackknife: 713
    snps: 681581  indivs: 36
    lambdascale:     1.000

At the very bottom of the ``.out`` file are reported the outlier f4-statistics, which show the lowest or highest Z-scores. Those are calculated based on the difference between the fitted and the observed f4 values. The only worst f4-statistic identified in the model we just run is some un-modelled affinity between "USA_SanNicolas_4900BP" and "USR1" that is anyway below 3 standard deviations::

    outliers:
                                                          Fit          Obs         Diff   Std. error         Z 
    
    
    worst f-stat:       Mbu        USR        USA        Can       0.000000    -0.001849    -0.001849     0.000681    -2.717



We can now visualize the graph in the ``.dot`` file using the program *graphviz* with the following command and then look at the resulting ``.png`` from the jupyter file browser. Dashed arrows represent admixture edges while solid arrows drift edges reported in units of :math:`\text{FST}\times 1,000`. On the very top, the worst f4-statistic is again reported::


    dot -Tpng Figure3a.dot -o Figure3a.png


Finally we can have a look at the ``.ggg`` file, which provides detailed proportions for admixture edges and drift lengths for each branch.

.. note:: generally it is important to not have zero-length edges because it might signify that the modelled edge does not exist. Also terminal edges for ancient groups, especially if composed by a single individual, are artificially long and should not be considered.

Adding new groups to the scaffold graph
---------------------------------------

Once confirmed that the scaffold graph has a good fit, we carry on adding the new Central and South American groups released in Posth et. al. 2018.
To the file ``Figure3a`` add the following *labels* ``Belize_MayahakCabPek_9300BP, PERu_Cuncaicha_9000BP, Peru_Lauricocha_8600BP`` as well as the following additional *edges*, save it with a new name like ``Figure3b`` and run ``qpGraph`` as shown before::

    edge    c18 NA2 CA
    edge    c19 CA Belize_MayahakCabPek_9300BP
    edge    c20 CA SA
    edge    c21 SA SA2
    edge    c22 SA2 PERu_Cuncaicha_9000BP
    edge    c23 SA2 Peru_Lauricocha_8600BP

The worst f4-statistic is -2.809 and despite in the ``.dot`` file once converted into ``.png`` there are some zero-length branches a more careful examination of the ``.ggg`` file indicates that those edges are in fact different from zero.

Continuing to fit new groups
----------------------------

Now we can create a file called ``Figure3c`` where we add the last three group *labels* ``Brazil_LapaDoSanto_9600BP, Argentina_ArroyoSeco2_7700BP, Chile_LosRieles_5100BP``. From node "SA" add an *edge* to form a new node called "SA3" that splits into ``Brazil_LapaDoSanto_9600BP`` and a new node ``SA4``. Finally "SA4" splits itself into the two Southern Cone populations that are ``Argentina_ArroyoSeco2_7700BP`` and ``Chile_LosRieles_5100BP``. After running it we can visualise the resulting ``.dot`` file as a ``.png``. That is the final graph reported in Figure 3 of `Posth et al. 2018`_!

Test the robustness of the graph topology
-----------------------------------------

Starting from the final graph ``Figure3c`` we can try, for example, to invert ``Belize_MayahakCabPek_9300BP`` with ``USA_SanNicolas_4900BP`` in a file called ``Figure3c.v2`` to test for branching patterns between North and Central American groups. For more advanced modelling we can instead invert the entire "SA3" node  with ``Belize_MayahakCabPek_9300BP`` and call the file ``Figure3c.v3`` to test for Central-South America branching patterns.

.. admonition:: Exercise:: 

     What do you observe when inspecting the respective ``.out`` files? Which of the models fit and which not? How do you interpret that?

Adding admixture edges
----------------------

We finally want to add in our working graph the oldest genome published so far from South America called ``CHIle_LosRieles_10900BP``. We initially try to position it as departing from each node without invoking admixture. One example is the following ``FigureS5a`` file that we can copy and run as seen before::

    root    R
    label    Mbuti.DG    Mbuti.DG
    label    Han.DG    Han.DG
    label    Onge.DG    Onge.DG
    label    MA1    MA1
    label    USR1    USR1
    label    USA_SanNicolas_4900BP    USA_SanNicolas_4900BP
    label    Canada_Lucier_4800BP_500BP    Canada_Lucier_4800BP_500BP
    label    Belize_MayahakCabPek_9300BP    Belize_MayahakCabPek_9300BP
    label    PERu_Cuncaicha_9000BP    PERu_Cuncaicha_9000BP
    label    Peru_Lauricocha_8600BP    Peru_Lauricocha_8600BP
    label    Brazil_LapaDoSanto_9600BP    Brazil_LapaDoSanto_9600BP
    label    Argentina_ArroyoSeco2_7700BP    Argentina_ArroyoSeco2_7700BP
    label    Chile_LosRieles_5100BP    Chile_LosRieles_5100BP
    label    CHIle_LosRieles_10900BP    CHIle_LosRieles_10900BP

    edge    a R Mbuti.DG
    edge    b R nonAfrica
    edge    c1 nonAfrica EastEurasia
    edge    c2 EastEurasia EastAsia
    edge    c3 EastEurasia Onge.DG
    edge    c4 EastAsia EastAsia2
    edge    c5 EastAsia EastAsia3
    edge    c6 EastAsia2 Han.DG
    edge    c7 EastAsia2 EastAsia4
    edge    c8 nonAfrica WestEurasia
    edge    c9 WestEurasia E_HG2 
    edge    c10 WestEurasia E_HG3
    admix   ANE E_HG2 EastAsia3
    edge    c11 ANE MA1
    admix   FA EastAsia4 E_HG3
    edge    c12 FA Beringia
    edge    c13 Beringia USR1
    edge    c14 Beringia NA
    edge    c15 NA Canada_Lucier_4800BP_500BP
    edge    c16 NA NA2
    edge    c17 NA2 USA_SanNicolas_4900BP
    edge    c18 NA2 CA
    edge    c19 CA Belize_MayahakCabPek_9300BP
    edge    c20 CA SA
    edge    c21 SA SA2
    edge    c22 SA SA3
    edge    c23 SA2 SA6
    edge    c24 SA2 CHIle_LosRieles_10900BP
    edge    c25 SA6 PERu_Cuncaicha_9000BP
    edge    c26 SA6 Peru_Lauricocha_8600BP
    edge    c27 SA3 Brazil_LapaDoSanto_9600BP
    edge    c28 SA3 SA4
    edge    c29 SA4 Argentina_ArroyoSeco2_7700BP
    edge    c30 SA4 Chile_LosRieles_5100BP

At the bottom of the newly produced ``.out`` file there are several f4-statistics that have Z-scores below -3 or above 3. The worst one is the following statistics that indicates some un-modelled affinity between the two Chilean samples ``CHIle_LosRieles_10900BP`` and ``Chile_LosRieles_5100BP``::

    worst f-stat:       Han        Chi        Bra        CHI       0.000000     0.003372     0.003372     0.000862     3.912 

We can then model a contribution from the oldest Chilean individual into the younger one. Change the last part of ``FigureS5a`` with the following **edges** and one **admixture event**, save it as ``FigureS5a.v2`` and run it again::


    edge    c24 SA2 SA7
    edge    c25 SA7 CHIle_LosRieles_10900BP
    edge    c26 SA6 PERu_Cuncaicha_9000BP
    edge    c27 SA6 Peru_Lauricocha_8600BP
    edge    c28 SA3 Brazil_LapaDoSanto_9600BP
    edge    c29 SA3 SA4
    edge    c30 SA4 Argentina_ArroyoSeco2_7700BP
    edge    c31 SA4 SA5
    admix   SA8 SA7 SA5
    edge    c32 SA8 Chile_LosRieles_5100BP

In the ``.out`` file we see that most of the outlier f4-statistics are gone while the worst statistic is still present but reduced (Zscore=3.2). This suggests the presence of un-modelled affinity between the two oldest South American groups ``Brazil_LapaDoSanto_9600BP`` and ``CHIle_LosRieles_10900BP``, that might represent a shared ``Anzick-1``-related ancestry that we investigate in detail in Figure 4 and Figure 5 of `Posth et al. 2018`_.
The resulting admixture graph suggests that a component from the oldest Chilean individual contributed at least marginally to the younger individual despite being more than 5,000 years apart!

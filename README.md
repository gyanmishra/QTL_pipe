QTL_pipe
========

The pipeline do the job of mapping QTL in RILs.

1. Preparation
1.1 trait

create trait dir

mkdir ../input/trait

put trait file trait dir

cp /rhome/cjinfeng/HEG4_cjinfeng/RILs/QTL/input/trait/May28_2013.RIL.trait.table ./


parse trait file, generate trait matrix for parents and RILs, draw distribution fig for each trait.

perl /rhome/cjinfeng/HEG4_cjinfeng/RILs/QTL_pipe/bin/scritps/trait/RIL_trait.pl --trait ../input/trait/May28_2013.RIL.trait.table


2. Mapping Reads (Maq, replace with bwa?)

3. Recombination Map (MPR package, Xie et al, 2010 PNAS)

4. QTL using R (Rqtl, Broman et al, 2003 Bioinfromatics)

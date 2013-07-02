QTL_pipe
========

The pipeline do the job of mapping QTL in RILs.

1. Preparation

1.1 trait

create "trait dir" and put trait file in "trait dir"

     mkdir ../input/trait
     cd ../input/trait
     cp /rhome/cjinfeng/HEG4_cjinfeng/RILs/QTL/input/trait/May28_2013.RIL.trait.table ./

parse trait file, generate trait matrix for parents and RILs, draw distribution fig for each trait.

     perl /rhome/cjinfeng/HEG4_cjinfeng/RILs/QTL_pipe/bin/scritps/trait/RIL_trait.pl --trait ../input/trait/May28_2013.RIL.trait.table

The results files will be in "trait dir":

May28_2013.RIL.trait.table.QTL.parents.txt

May28_2013.RIL.trait.table.QTL.trait.txt

DrawQTLtrait.R

DrawQTLtrait.pdf

1.2 reference

Create "reference dir" and put reference sequence, dbSNPs in "reference dir"

     mkdir ../input/reference
     cd ../input/reference
     ln -s /rhome/cjinfeng/HEG4_cjinfeng/RILs/Depth_Evaluation/input/HEG4_dbSNP.vcf ./
     ln -s /rhome/cjinfeng/HEG4_cjinfeng/seqlib/MSU_r7.fa ./

Format dbSNPs and reference sequence

     grep -v "#" HEG4_dbSNP.vcf | awk '{print $1"\t"$2"\t"$5"\t"$4}' > HEG4_dbSNP.table
     perl /rhome/cjinfeng/HEG4_cjinfeng/RILs/QTL_pipe/bin/scritps/reference/formatfa.pl --fa MSU_r7.fa --project Nipponbare
     rm MSU_r7.fa
     ln -s MSU_r7.reform.fa MSU_r7.fa

Generate parent2 reference sequence (HEG4) 

     perl /rhome/cjinfeng/HEG4_cjinfeng/RILs/QTL_pipe/bin/scritps/reference/PseudoMaker_cjinfeng.pl HEG4_dbSNP.table MSU_r7.fa HEG4


1.3 fastq

create "fastq dir" and put fastq of RILs in  "fastq dir"

     mkdir ../input/fastq/
     cd ../input/fastq/
     ln -s /rhome/cjinfeng/Rice/RIL/Illumina/ ./

Get sample of fastq of RILs (0.1 X)

     perl /rhome/cjinfeng/HEG4_cjinfeng/RILs/QTL_pipe/bin/scritps/fastq/prefastq.pl --RIL ./Illumina > prefastq.list

Or run using qsub

     qsub runprefastq.sh

Put all sample into a project dir

     mkdir RIL_1X
     mv GN* RIL_1X

2. Mapping Reads (Maq, replace with bwa?)

Run shell "step00.mapping.sh" using shell

	bash step00.mapping.sh

which use "RIL_MAQ_qsub.pl" to map reads to reference and do pileup.

	perl $scripts/mapping/RIL_MAQ_qsub.pl --ref /rhome/cjinfeng/HEG4_cjinfeng/RILs/QTL_pipe/input/reference/MSU_r7.fa --fastq /rhome/cjinfeng/HEG4_cjinfeng/RILs/QTL_pipe/input/fastq/RILs_0.2X

The output files:

GN*.Maq.p1.map.pileup, will be present in where read is and used by genotyping script "step01.genotype.sh".


3. Recombination Map (MPR package, Xie et al, 2010 PNAS)

3.1 Genotyping RILs

Run shell "step01.genotype.sh" using qsub

	qsub step01.genotype.sh

which include these two steps:

Convert SNPs into parents files, which will be used to genotype the RILs.

	perl $scritps/genotype/RIL_VCF2Parents.pl --vcf ../input/reference/HEG4_dbSNP.vcf

Genotype Maq results of RILs using parents file 

	perl $scritps/genotype/RIL_SNP_MAQ.pl --ref ../input/reference/MSU_r7.fa --fastq ../input/fastq/012 --parents NB.RILs.dbSNP.SNPs.parents

The output files:

NB.RILs.dbSNP.SNPs.parents, parents genotypes obtained from SNPs data.
NB.RILs.dbSNP.SNPs.Markers, parent1 genotypes that used as reference genome.
NB.RILs.dbSNP.SNPs.RILs, genotypes of RILs on each markers.
NB.RILs.dbSNP.SNPs.sub.parents, parents genotypes only for these markers have polymorephism in RILs.
NB.RILs.dbSNP.SNPs.sub.Marker, parent1 genotypes only for these markers have polymorephism in RILs.
NB.RILs.pdf, test runs of Rqtl for QTL mapping of first trait

NB.RILs.dbSNP.SNPs.RILs,NB.RILs.dbSNP.SNPs.Markers,NB.RILs.dbSNP.SNPs.sub.parents will be used in "MPR_hmmrun.R" of step02.recombination_bin.sh 

3.2 Construct recombination map

Run shell "step02.recombination_bin.sh" using qsub

	qsub step02.recombination_bin.sh

which include these two steps:

Construct recombination bin using MPR package

	cat $scripts/recombination_map/MPR_hmmrun.R | /rhome/cjinfeng/software/tools/R-2.15.3/bin/R --slave

Draw bin map for each RILs and for each chromosome

	perl $scripts/recombination_map/RIL_drawbin.pl --MPR ./ --chrlen ../input/reference/MSU7.chr.inf

The output files:

MPR.allele.MPR
MPR.base.data
MPR.cross.csv
MPR.cross.fill.cro, cross file of qtlcart format
MPR.cross.fill.map, map file of qtlcart format
MPR.geno.bin, raw recombination bin of RILs
MPR.geno.bin.fill, filled bin of RILs
MPR.geno.bin.uniq, unique bin of RILs
MPR.geno.data
MPR.geno.data.HMMcr
MPR_bin/, directory of pdf contains recombination bin for each RIL and each chromosome

4. QTL using R (Rqtl, Broman et al, 2003 Bioinfromatics)

Run shell "step03.QTL.sh" using qsub

	qsub step03.QTL.sh

It will use the script "./scripts/QTL/RIL_QTL.pl" to do QTL mapping using Rqtl. The input files is MPR.cross.fill.cro and MPR.cross.fill.map, which given by "--qtlcart MPR.cross.fill"

	perl $scripts/QTL/RIL_QTL.pl --qtlcart MPR.cross.fill

The output files:

MPR.cross.fill.QTL.pdf, which contains the genetic map and QTL signal for each trait using MR and EM method.
MPR.cross.fill.QTL.mr.table, which contains the LOD score of each markers for each trait.
MPR.cross.fill.QTL.mr.table.LOD_threshold, which contains the LOD threshold for each trait.
MPR.cross.fill.QTL.mr.table.test, which contains the significant markers. 






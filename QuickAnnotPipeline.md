# A quick tool to annotate new genome assembly based on closed related species' annotation
Nan Hu / Mar. 2023

Based on liftoff software in Linux

---
For a lot of non-model systems, it is hard and complicated to get a reliable annotation even after a draft assembly is done. However, a rough annotation is to be considered super useful in multiple bioinformatic analyses including sequence-level synteny, functional gene-based evolution, chromosomal duplication and rearrangement, inter-species phylogenetics, etc. Luckily, in some taxa, closed-related species shared similar gene structures and homologous regions, which set the base for homologous annotation based on a well-annotated closed-related species. 

In general, currently there are several different software and pipeline to fulfil the homologous annotation. Here, we selected a pipeline that can quickly annotate a closed-related genome in a relatively good quality.

## 1. Software Environment Preparation
Software requirement:
1. [liftoff](https://github.com/agshumate/Liftoff)
2. [minimap2](https://academic.oup.com/bioinformatics/article/34/18/3094/4994778)

We recommend to use conda to install these packages in a separate environment
```bash
conda create --name annot1
conda activate annot1
conda install -c bioconda liftoff
conda install -c bioconda minimap2
```
In some specific systems, you may need to install python3 separately.

## 2. Input File Preparation
Liftoff takes 3 input files. They are: **Targeted unannotated FASTA file**, **reference genome FASTA file** (from closed-related species), and **reference annotation GFF3 file**. The reference annotation can be in different form include but not limited to gene annotation, cds annotation, exons annotation, etc.

## 3. Sample Run
Here is a example for using the liftoff to run a homologous annotation pipeline. We planned to annotate our genome assembly of *Salix exigua* male assembly. Instead of de novo annotation of our genome assembly, *S. purpurea* was selected for homologous annotation due to its well established transcriptome data.

Reference genome and annotation could be found on Phytozome (JGI) under "*Salix purpurea* v5.1". 

The prepared files are listed below:
1. Targeted unannotated FASTA file: SE20220629_main_sorted.fasta
2. Reference genome FASTA file: Spurpurea_519_v5.0.fa
3. Reference annotation GFF3 file: Spurpurea_519_v5.1.gene_exons.gff3
> We choose the GFF3 file for only the annotated exons but there are several other choices.

Under the working directory, create a new director for the results.
```bash
mkdir sexigua_exons_annot
```

Then, submit a job under Slurm job submission environment.
```bash
#!/bin/bash
#SBATCH -J liftoff
#SBATCH -o %x.o%j
#SBATCH -e %x.e%j
#SBATCH -p nocona
#SBATCH -N 1
#SBATCH -n 64

liftoff -g ./Spurpurea_519_v5.1.gene_exons.gff3 \
        -dir ./sexigua_exons_annot/ \
        -o ./sexigua_exons_annot/sexigua.liftoff.gff3 \
        ./SE20220629_main_sorted.fasta \
        ./Spurpurea_519_v5.0.fa
```
As shown above, `-g` refers to the GFF3 file, `-dir` refers to the working directory, `-o` refers to the output GFF3 file, which is the homologous annotation for the *S. exigua*. This step for a genome assemble around 350M takes about 40min-1h long running on a node requesting 64 CPU cores.

Here is a sample view of the results:
| seqid 	| source  	| type            	| start 	| end   	| score 	| strand 	| phase 	| attributes                                                                                                                                                                                     	|
|-------	|---------	|-----------------	|-------	|-------	|-------	|--------	|-------	|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| Chr01 	| Liftoff 	| gene            	| 7123  	| 7446  	| .     	| -      	| .     	| ID=Sapur.001G017100.v5.1;Name=Sapur.001G017100;coverage=0.990;sequence_ID=0.902;valid_ORFs=0;extra_copy_number=0;copy_num_ID=Sapur.001G017100.v5.1_0                                           	|
| Chr01 	| Liftoff 	| mRNA            	| 7123  	| 7446  	| .     	| -      	| .     	| ID=Sapur.001G017100.1.v5.1;Name=Sapur.001G017100.1;pacid=41822182;longest=1;Parent=Sapur.001G017100.v5.1;matches_ref_protein=False;valid_ORF=False;missing_stop_codon=True;extra_copy_number=0 	|
| Chr01 	| Liftoff 	| exon            	| 7123  	| 7446  	| .     	| -      	| .     	| ID=Sapur.001G017100.1.v5.1.exon.1;Parent=Sapur.001G017100.1.v5.1;pacid=41822182;extra_copy_number=0                                                                                            	|
| Chr01 	| Liftoff 	| CDS             	| 7123  	| 7446  	| .     	| -      	| .     	| ID=Sapur.001G017100.1.v5.1.CDS.1;Parent=Sapur.001G017100.1.v5.1;pacid=41822182;extra_copy_number=0                                                                                             	|
| Chr01 	| Liftoff 	| gene            	| 84543 	| 85098 	| .     	| +      	| .     	| ID=Sapur.001G016800.v5.1;Name=Sapur.001G016800;coverage=1.0;sequence_ID=0.922;valid_ORFs=0;extra_copy_number=0;copy_num_ID=Sapur.001G016800.v5.1_0                                             	|
| Chr01 	| Liftoff 	| mRNA            	| 84543 	| 85098 	| .     	| +      	| .     	| ID=Sapur.001G016800.1.v5.1;Name=Sapur.001G016800.1;pacid=41824577;longest=1;Parent=Sapur.001G016800.v5.1;matches_ref_protein=False;valid_ORF=False;missing_stop_codon=True;extra_copy_number=0 	|
| Chr01 	| Liftoff 	| exon            	| 84543 	| 85098 	| .     	| +      	| .     	| ID=Sapur.001G016800.1.v5.1.exon.1;Parent=Sapur.001G016800.1.v5.1;pacid=41824577;extra_copy_number=0                                                                                            	|
| Chr01 	| Liftoff 	| CDS             	| 84549 	| 84888 	| .     	| +      	| .     	| ID=Sapur.001G016800.1.v5.1.CDS.1;Parent=Sapur.001G016800.1.v5.1;pacid=41824577;extra_copy_number=0                                                                                             	|
| Chr01 	| Liftoff 	| five_prime_UTR  	| 84543 	| 84548 	| .     	| +      	| .     	| ID=Sapur.001G016800.1.v5.1.five_prime_UTR.1;Parent=Sapur.001G016800.1.v5.1;pacid=41824577;extra_copy_number=0                                                                                  	|
| Chr01 	| Liftoff 	| three_prime_UTR 	| 84889 	| 85098 	| .     	| +      	| .     	| ID=Sapur.001G016800.1.v5.1.three_prime_UTR.1;Parent=Sapur.001G016800.1.v5.1;pacid=41824577;extra_copy_number=0                                                                                 	|
| Chr01 	| Liftoff 	| gene            	| 89564 	| 91827 	| .     	| -      	| .     	| ID=Sapur.001G017200.v5.1;Name=Sapur.001G017200;coverage=0.999;sequence_ID=0.758;valid_ORFs=0;extra_copy_number=0;copy_num_ID=Sapur.001G017200.v5.1_0                                           	|
| Chr01 	| Liftoff 	| mRNA            	| 89564 	| 91827 	| .     	| -      	| .     	| ID=Sapur.001G017200.1.v5.1;Name=Sapur.001G017200.1;pacid=41821970;longest=1;Parent=Sapur.001G017200.v5.1;matches_ref_protein=False;valid_ORF=False;missing_stop_codon=True;extra_copy_number=0 	|
| Chr01 	| Liftoff 	| exon            	| 89564 	| 91827 	| .     	| -      	| .     	| ID=Sapur.001G017200.1.v5.1.exon.1;Parent=Sapur.001G017200.1.v5.1;pacid=41821970;extra_copy_number=0                                                                                            	|
| Chr01 	| Liftoff 	| CDS             	| 89564 	| 91827 	| .     	| -      	| .     	| ID=Sapur.001G017200.1.v5.1.CDS.1;Parent=Sapur.001G017200.1.v5.1;pacid=41821970;extra_copy_number=0                                                                                             	|
| Chr01 	| Liftoff 	| gene            	| 90784 	| 98993 	| .     	| +      	| .     	| ID=Sapur.001G016900.v5.1;Name=Sapur.001G016900;coverage=0.998;sequence_ID=0.693;valid_ORFs=0;extra_copy_number=0;copy_num_ID=Sapur.001G016900.v5.1_0                                           	|
| Chr01 	| Liftoff 	| mRNA            	| 90784 	| 98993 	| .     	| +      	| .     	| ID=Sapur.001G016900.1.v5.1;Name=Sapur.001G016900.1;pacid=41822040;longest=1;Parent=Sapur.001G016900.v5.1;matches_ref_protein=False;valid_ORF=False;missing_stop_codon=True;extra_copy_number=0 	|

This dataset would be useful for downstream analysis. In the rest part of this repository, we will introduce some applications using the GFF3 files for genomic evolution and population genetics studies.

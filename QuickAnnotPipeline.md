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


## 3. Sample Run



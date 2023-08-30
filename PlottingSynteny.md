# Plotting Synteny Groups Using GFF3 Homologous Annotation
Nan Hu / Aug. 2023

---
The first application of using the homologous annotation is to easily plot a synteny map between sequences annotated with the same reference (or synteny with the reference genome).

Continuing with the dataset we generated from last step, a GFF3 file. 
> Edited. 08/30/2023. These steps are included in the R scripts.
<del>Firstly, we will filter the data down to only the gene region. If your interest is the exon-per-exon synteny or other types, it is up to your own setting ups.
```bash
# We filter both the target and the reference annotations.
grep "gene" sexigua.liftoff.gff3 > sexigua.gene.gff3
grep "gene" Spurpurea_519_v5.1.gene_exons.gff3 > spurpurea.gene.gff3
```</del>
Here, we have all the files that ready for the synteny plot. Before we plug in the data into the R code, I would like to discuss the rationale of using only the GFF3 file (without sequence files and alignment) for the synteny map. We annotated the genome using liftoff, where it called minimap2 to align our target genome to the reference for each annotation unit. Thus, each annotation items that identified in the final GFF3 file has passed the critria of the aligner. In other word, each annotation gene names shared between new GFF3 and the reference GFF3 refers to a qualified alignment between target and the reference.

```R
# Load required libraries
library(ggplot2)
library(readr)
library(dplyr)
library(tidyr)

# Read in the data
file1 <- file.choose()
data1 <- read_tsv(file1, col_names = FALSE)

# Remove columns 2, 3, 6-8
data1 <- data1[data1$X3 == "gene",]
data_clean <- data1 %>%
  separate(col = 9, into = c("ID", "Name", "coverage", "sequence_ID", "valid_ORFs", "extra_copy_number", "copy_num_ID"), sep = ";") %>%
  select(-c(2, 3, 6:8)) %>%
  select(c(1,2,3,Name))


# View cleaned data
head(data_clean)

species1 <- data_clean 

file1 <- file.choose()
data <- read_tsv(file1, col_names = FALSE)

# Remove columns 2, 3, 6-8
# Remove columns 2, 3, 6-8
data <- data[data$X3 == "gene",]
data_clean <- data %>%
  separate(col = 9, into = c("ID", "Name", "coverage", "sequence_ID", "valid_ORFs", "extra_copy_number", "copy_num_ID"), sep = ";") %>%
  select(-c(2, 3, 6:8)) %>%
  select(c(1,2,3,Name))

# View cleaned data
head(data_clean)

species2 <- data_clean 


target_contig = "X_contigs"
# link genes
species1_sel <- species1 %>% 
  mutate(pos = (X4 + X5) / 2) %>%
  select(-c(X4, X5)) %>%
  filter(X1 == target_contig)

species2_sel <- species2 %>% 
  mutate(pos = (X4 + X5) / 2) %>%
  select(-c(X4, X5)) %>%
  filter(X1 == "Chr15")


# Match datasets based on the Name column.
matched_data <- inner_join(species1_sel, species2_sel, by = "Name") %>% 
  select(pos.x, pos.y)


# Set constants
chr1 <- max(species1_sel$pos) + 20000
chr2 <- max(species2_sel$pos) + 20000

# Read data from input file
data <- matched_data

# Set up the plot
ggplot(data, aes(x = pos.x, y = pos.y)) +
  xlim(-10000, 10000) +
  ylim(0, 1000) +
  geom_segment(aes(x = (-8000 + pos.x / chr2 * 16000), xend = (-8000 + pos.y / chr2 * 16000),
                   y = 650, yend = 200), color = grey(0.8), linewidth = 0.7) +
  geom_segment(aes(x = -8000, y = 650, xend = (-8000 + chr1 / chr2 *16000), yend = 650), 
               linetype = "solid", color = "black", linewidth = 2) +
  geom_segment(aes(x = -8000, y = 200, xend = 8000, yend = 200), 
               linetype = "solid", color = "black", linewidth = 2) +
  labs(title = target_contig)+
  theme_bw() +
  theme(panel.grid = element_blank(),
        axis.line = element_blank(),
        axis.text = element_blank(),
        axis.title = element_blank(),
        panel.border = element_blank(),
        plot.background = element_blank(),
        panel.background = element_blank(),
        axis.ticks = element_blank())
ggsave(paste0(target_contig, ".png"), plot=last_plot())
```
# Gene Prevalence Estimation Tool for Enterobacteriaceae

## Overview
This tool is designed for estimating the prevalence of a specific gene in Enterobacteriaceae taxa, integrating NCBI genome retrieval, BLAST database construction, and automated query analysis.

## Features
**Genomic Data Acquisition**
  - A pre-built BLAST database has been constructed for complete genomes. To ensure reproducibility, a metadata file listing all assembly accessions of genomes used in the BLAST database is also provided. However, since some complete genomes are relatively small and may not be representative of the full genetic diversity of a taxon, users may choose to enable **heavy mode** to include draft genomes in their analysis.
  - For draft genomes, users must provide their target taxa and have the option to define the sample size (default: 100) for each iteration, allowing for a controlled and flexible selection process.
  - Draft genome accessions are randomly chosen and retrieved using ncbi-genome-download, then downloaded using the datasets tool. The draft genomes are iteratively sampled due to their large number, ensuring representative sampling across taxa.
  - The number of iterations is determined by the total draft genomes (contigs) available in GenBank divided by the sample size, with a cap of 50 iterations to ensure computational feasibility.
**Genome files Organization**
- Creates structured directories per genus, species, and Salmonella enterica subspecies enterica serotype.
- Maintains an aggregated directory for each genus and *Salmonella enterica*.
- Draft genomes are downloaded randomly in iterations and stored separately within the draft genome directory.
- The script to download complete genomes is download_complete_genomes.sh and the script to build the BLAST database is makeblastdb.sh.
```
complete_genomes
│   ├── Escherichia/
│   │   ├── aggregated/
│   │   ├── Escherichia_coli/
│   │   ├── Escherichia_fergusonii/
│   │   ├── Escherichia_albertii/
│   │   ├── ...
│   ├── Salmonella/
│   │   ├── aggregated/
│   │   ├── Salmonella_enterica/
│   │   │   ├── aggregated/
│   │   │   ├── Typhimurium/
│   │   │   ├── Infantis/
│   │   │   ├── Newport/
│   │   │   ├── Heidelberg/
│   │   │   ├── ...
│   │   ├── Salmonella_bongori/
│   ├── Shigella/
│   │   ├── aggregated/
│   │   ├── Shigella_flexneri/
│   │   ├── Shigella_sonnei/
│   │   ├── Shigella_boydii/
│   │   ├── ...
│   ├── Klebsiella/
│   │   ├── aggregated/
│   │   ├── Klebsiella_pneumoniae/
│   │   ├── Klebsiella_oxytoca/
│   │   ├── ...
│   ├── Enterobacter/
│   │   ├── aggregated/
│   │   ├── Enterobacter_cloacae/
│   │   ├── Enterobacter_hormaechei/
│   │   ├── ...
│   ├── Citrobacter/
│   │   ├── aggregated/
│   │   ├── Citrobacter_freundii/
│   │   ├── Citrobacter_koseri/
│   │   ├── ...
│   ├── Cronobacter/
│   │   ├── aggregated/
│   │   ├── Cronobacter_sakazakii/
│   │   ├── Cronobacter_malonaticus/
│   │   ├── ...
```
**BLAST Query & Analysis**
- BLAST analysis is conducted in two stages: First, against pre-built complete genome databases to leverage high-quality genome assemblies. Then, against draft genome databases, which are iteratively constructed during runtime to ensure representative sampling for each genus, species, and *Salmonella enterica* serotypes group.
- Query gene file is provided by users (supports batch processing of multiple genes).
- Filters results by user-defined minimum identity & coverage thresholds.
**Gene prevalence calculation**
- The prevalence of the gene is calculated based on hits in complete genomes only.
- Draft genomes may have variable quality, by incorporating multiple iterations, and the final prevalence estimate is averaged over all iterations, ensuring robustness against sampling bias.
------
## Installation
To run this pipeline, set up a Conda environment with the required dependencies.
1. Clone the Repository
```sh
git clone https://github.com/Weifanwu66/SGP.git
cd SGP
```
2. Create and Activate the Conda Environment
The pipeline requires a Conda environment with all necessary dependencies. To create and activate it, run:
```sh
conda env create -f environment.yml
conda activate SGP
```
To verify the installation, check if all tools are installed:
```sh
conda list | grep -E "sra-tools|seqkit|trimmomatic|skesa|blast|ncbi-datasets-cli|entrez-direct"
```
If any package is missing, please install it manually:
```sh
conda install -c bioconda <package_name>
```
-----
## Dependencies
1. NCBI Datasets: https://github.com/ncbi/datasets
2. ncbi-genome-download: Blin, K. (2023). ncbi-genome-download (0.3.3). Zenodo. https://doi.org/10.5281/zenodo.8192486
3. NCBI BLAST: https://blast.ncbi.nlm.nih.gov/doc/blast-help/downloadblastdata.html
-----
## Flags and Options
The pipeline provides several flgas to customize execution:
```sh
Usage: SGP.sh -g GENE_FILE -s SEROTYPE_FILE [--mode light|heavy] [--sra on|off] [--sra_number N] [-c COVERAGE] [-i IDENTITY] [-o OUTPUT_FILE]
-g GENE_FILE : FASTA file with target gene sequence (required).
-s SEROTYPE_FILE : File containing Salmonella serotypes (required).
--mode MODE : Choose between heavyweight mode and lightweight mode (default: light).
--sra on|off : Enable or disable SRA assembly (default: off).
--sra_num N : Specify the number of SRA files to retrieve (max: 100, default 50).
-c COVERAGE : The minimum genome coverage (default: 80%).
-i IDENTITY : The minimum percentage of identity (default: 90%).
-o OUTPUT_FILE : Output result file (default: gene_summary.tsv).
```
## Example usage:
```sh
bash SGP.sh -g target_genes.fasta -t serotype_list.txt --mode heavy -c 95 -i 95 -o output.csv
```

## Output
Examples of the final output file:

Light mode
| Serotype            | Gene_ID                     | Min_coverage | Min_percentage_of_identity | Total_draft_genomes |  Total_complete_genomes | Complete_genomes_with_target_gene | Percentage_with_target_gene_in_complete_genomes |
|---------------------|-----------------------------|--------------|----------------------------|---------------------|-------------------------|-----------------------------------|-------------------------------------------------|
| Enteritidis         | NC_003197.2:1707344-1707789 | 95           | 95                         | 41408               | 165                     | 164                               | 99.00%                                          |
| Typhimurium         | NC_003197.2:1707344-1707789 | 95           | 95                         | 37382               | 226                     | 226                               | 100.00%                                         |
| Typhimurium var. 5- | NC_003197.2:1707344-1707789 | 95           | 95                         | 825                 | 11                      | 11                                | 100.00%                                         |
| Heidelberg          | NC_003197.2:1707344-1707789 | 95           | 95                         | 4905                | 51                      | 51                                | 100.00%                                         |
| Infantis            | NC_003197.2:1707344-1707789 | 95           | 95                         | 16915               | 80                      | 80                                | 100.00%                                         |
| Newport             | NC_003197.2:1707344-1707789 | 95           | 95                         | 8116                | 92                      | 92                                | 100.00%                                         |
| Uganda              | NC_003197.2:1707344-1707789 | 95           | 95                         | 1254                | 12                      | 12                                | 100.00%                                         |
| Braenderup          | NC_003197.2:1707344-1707789 | 95           | 95                         | 2565                | 5                       | 5                                 | 100.00%                                         |
| Muenchen            | NC_003197.2:1707344-1707789 | 95           | 95                         | 2384                | 22                      | 22                                | 100.00%                                         |
| Montevideo          | NC_003197.2:1707344-1707789 | 95           | 95                         | 4831                | 32                      | 31                                | 96.00%                                          |
| Javiana             | NC_003197.2:1707344-1707789 | 95           | 95                         | 1748                | 8                       | 8                                 | 100.00%                                         |
| Reading             | NC_003197.2:1707344-1707789 | 95           | 95                         | 1973                | 12                      | 12                                | 100.00%                                         |
| Dublin              | NC_003197.2:1707344-1707789 | 95           | 95                         | 3831                | 28                      | 28                                | 100.00%                                         |
| Oranienburg         | NC_003197.2:1707344-1707789 | 95           | 95                         | 1543                | 5                       | 5                                 | 100.00%                                         |
| Potsdam             | NC_003197.2:1707344-1707789 | 95           | 95                         | 128                 | 1                       | 1                                 | 100.00%                                         |
| Thompson            | NC_003197.2:1707344-1707789 | 95           | 95                         | 1727                | 17                      | 17                                | 100.00%                                         |
| Saintpaul           | NC_003197.2:1707344-1707789 | 95           | 95                         | 3873                | 24                      | 24                                | 100.00%                                         |
| Hadar               | NC_003197.2:1707344-1707789 | 95           | 95                         | 1927                | 15                      | 15                                | 100.00%                                         |
| Schwarzengrund      | NC_003197.2:1707344-1707789 | 95           | 95                         | 3038                | 27                      | 27                                | 100.00%                                         |
| Anatum              | NC_003197.2:1707344-1707789 | 95           | 95                         | 4843                | 40                      | 40                                | 100.00%                                         |
| Berta               | NC_003197.2:1707344-1707789 | 95           | 95                         | 604                 | 3                       | 3                                 | 100.00%                                        

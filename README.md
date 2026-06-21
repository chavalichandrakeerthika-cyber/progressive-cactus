# PROGRESSIVE CACTUS
Using Progressive Cactus for Multiple Sequence Alignment and Ancestral Genome Reconstruction  
<br>

## Downloading Genome Assemblies From NCBI  
Set up the ncbi_datasets conda environment  
[Link to installation documentation](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/command-line-tools/download-and-install/)
```bash
conda create -n ncbi_datasets -c conda-forge ncbi-datasets-cli
conda activate ncbi_datasets
```
\
Download a genome assembly using accession number  
[Link to genome download documentation](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/how-tos/genomes/download-genome/)
```bash
datasets download genome accession "GCF_009873245.2" --filename "path/Balaenoptera_musculus_dataset.zip"
```
\
Download multiple genomes in a loop

- Create a list containing genome assembly accesion numbers
```bash
genome_list=("GCF_028564815.1" "GCA_004363455.1" "GCA_029224305.1")
```

- Create a loop that downloads every genome from list into its own directory
```bash
for n in ${genome_list[@]}; do 
  mkdir -p path/genomes/${n}; 
  datasets download genome accession ${n} --filename "path/genomes/${n}/${n}_dataset.zip"; 
  echo "${n} done"; 
done
```
<br>

## Soft-masking Genomes with RepeatMasker
RepeatMasker masks genomes by comparing it to a known library of repeats  
(For de novo repeats, use RepeatModeler to build a custom repeat library and then give that to RepeatMasker)
\

Install RepeatMasker as a conda environment (installs search engine RMBlast and TRF as well)
```bash
conda create -n RepeatMasker -c conda-forge -c bioconda repeatmasker -y
conda activate RepeatMasker
```
\
Check already downloaded FamDB partitions 
```bash
cd $CONDA_PREIX/share/RepeatMasker
python famdb.py info
```
\
Download required partition (eg: Partition 7 (Mammalia) for masking Cetaceans)




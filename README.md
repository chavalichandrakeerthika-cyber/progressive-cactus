# PROGRESSIVE CACTUS
Using Progressive Cactus for Multiple Sequence Alignment and Ancestral Genome Reconstruction

## DOWNLOADING GENOME ASSEMBLIES FROM NCBI  
Setting up the ncbi_datasets conda environment  
[Link to installation documentation](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/command-line-tools/download-and-install/)
```bash
conda create -n ncbi_datasets -c conda-forge ncbi-datasets-cli
conda activate ncbi_datasets
```  
  
Downloading a genome assembly using accession number  
[Link to genome download documentation](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/how-tos/genomes/download-genome/)
```bash
datasets download genome accession "GCF_009873245.2" --filename "path/Balaenoptera_musculus_dataset.zip"
```

Downloading multiple genomes in a loop

1. Create a list containing genome assembly accesion numbers
```bash
genome_list=("GCF_028564815.1" "GCA_004363455.1" "GCA_029224305.1")
```

2. Create a loop that downloads every genome from list into its own directory
```bash
for n in ${genome_list[@]}; do 
  mkdir -p path/genomes/${n}; 
  datasets download genome accession ${n} --filename "path/genomes/${n}/${n}_dataset.zip"; 
  echo "${n} done"; 
done
```

## SOFT-MASKING GENOMES USING REPEATMASKER


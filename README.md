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
```bash
datasets download genome accession "GCF_009873245.2" --filename "path/Balaenoptera_musculus_dataset.zip"

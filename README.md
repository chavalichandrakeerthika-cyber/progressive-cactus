# PROGRESSIVE CACTUS  
Using Progressive Cactus for Multiple Sequence Alignment and Ancestral Genome Reconstruction  
<br>

## Downloading Genome Assemblies From NCBI  

Set up the ncbi_datasets conda environment:  
[Link to installation documentation](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/command-line-tools/download-and-install/)
```bash
conda create -n ncbi_datasets -c conda-forge ncbi-datasets-cli
conda activate ncbi_datasets
```  
\
Download a genome assembly using accession number:  
[Link to genome download documentation](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/how-tos/genomes/download-genome/)
```bash
datasets download genome accession "GCF_009873245.2" --filename "path/Balaenoptera_musculus_dataset.zip"
```
\
Download multiple genomes in a loop:  
- Create a list containing genome assembly accesion numbers
- Create a loop that downloads every genome from list into its own directory
```bash
genome_list=("GCF_028564815.1" "GCA_004363455.1" "GCA_029224305.1")

for n in ${genome_list[@]}; do 
  mkdir -p path/genomes/${n}; 
  datasets download genome accession ${n} --filename "path/genomes/${n}/${n}_dataset.zip"; 
  echo "${n} done"; 
done
```
<br>

## Soft-masking Genomes with RepeatMasker
RepeatMasker masks genomes by comparing sequences to a known library of repeats from FamDB  
To find de novo repeats, use RepeatModeler to build a custom repeat library and then give that to RepeatMasker  

The following commands are for:  
_RepeatMakser Version: 4.2.2_ which comes with _FamDB Format Version: 2.0.0_ which supports the _Dfam 3.9_ repeat library  

---
Install RepeatMasker as a conda environment (installs search engine RMBlast and TRF as well):
```bash
conda create -n RepeatMasker -c conda-forge -c bioconda repeatmasker=4.2.2 -y
conda activate RepeatMasker
```
\
Check already downloaded FamDB partitions: 
```bash
cd $CONDA_PREFIX/share/RepeatMasker
python famdb.py info
```
\
Download required partition into the famdb directory (eg: Partition 7 [Mammalia] for masking Cetaceans)  
Get Partition Download Links [Here](https://www.dfam.org/releases/Dfam_3.9/families/FamDB/)  
```bash
cd $CONDA_PREFIX/share/RepeatMasker/Libraries/famdb
aria2c -x 16 -s 16 -c --dir=$CONDA_PREFIX/RepeatMasker/Libraries/famdb https://www.dfam.org/releases/Dfam_3.9/families/FamDB/dfam39_full.7.h5.gz

# Do "python famdb.py info" again and confirm that it downloaded
```
\
Check number of repeats available for your taxon of interest
Repeat
```bash
python famdb.py lineage -ad cetacea
python famdb.py -ad -f totals cetacea




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
```
\
Check number of repeats available for the ancestors and descendants of your taxon of interest
```bash
python famdb.py lineage -ad cetacea
python famdb.py -ad -f totals cetacea
```
\
Run RepeatMasker on genome assemblies  
[Link to RepeatMasker Documentation](https://github.com/Dfam-consortium/RepeatMasker/blob/master/repeatmasker.help)  
```bash
RepeatMasker -species cetacea -pa 6 -q -xsmall -dir /path/to/output genomic.fna
```
<details>
<summary>Output Files Produced</summary>
<ul><li>.masked → Masked fasta file</li>
<li>.tbl → Summary statistics of repeat content</li>
<li>.out → Annotation table of all detected repeats</li>
<li>.cat -> Raw match data (can be used to generate the other output files without rerunning the whole program)</li></ul>
</details>  

\
If the .cat file exists and you only want to generate the .masked .tbl .out files again:  
```bash
ProcessRepeats -species cetacea -xsmall -maskSource genome.fna genomic.fna.cat.gz
```
\
NOTE:  When using the -species <> flag, RepeatMasker creates a cache of species/taxon-specific libraries extracted from FamDB, making it faster next time the same -species <> argument is used. This cache may require a bit of space (eg: ~50GB for cetacea), so consider masking all sequences using the same -species <> argument at once and deleting the cache before using a different -species <> argument.  
\
Masking multiple genomes in a loop:
- Create a list of genome assemblies (remove .fna from end)
- Create a loop to mask every genome in list with output files for each in their own directories
```bash
genome_list=("GCF_028023285.1_mBalRic1.hap2_genomic" "GCF_028564815.1_mEubGla1.1.hap2._XY_genomic" "GCA_041834305.1_ASM4183430v1_genomic" "GCF_949987535.1_mBalAcu1.1_genomic" "GCF_965194805.1_mBalBor1.hap1.1_genomic" "GCF_965194825.1_mBalPhy2.hap1.1_genomic")

for n in ${genome_list[@]}; do 
	mkdir -p "/path/to/RepMask/${n}"; 
	RepeatMasker -species cetacea -pa 6 -q -xsmall -dir "/path/to/RM/${n}" "${/}.fna"; 
	echo "${n} done"; 
done
```
\




# PROGRESSIVE CACTUS  
Using Progressive Cactus for Multiple Sequence Alignment and Ancestral Genome Reconstruction  
<br>

## Downloading Genome Assemblies From NCBI  

1\. Set up the ncbi_datasets conda environment:  
[Link to Installation Documentation](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/command-line-tools/download-and-install/)
```bash
conda create -n ncbi_datasets -c conda-forge ncbi-datasets-cli
```
```bash
conda activate ncbi_datasets
```  
\
2. Download a genome assembly using accession number:  
[Link to Genome Download Documentation](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/how-tos/genomes/download-genome/)
```bash
datasets download genome accession "GCF_009873245.2" --filename "path/Balaenoptera_musculus_dataset.zip"
```
\
3. Download multiple genomes in a loop:  
- Create a list containing genome assembly accesion numbers
- Create a loop that downloads every genome from list into its own directory
```bash
genome_list=("GCF_028564815.1" "GCA_004363455.1" "GCA_029224305.1")
```
```bash
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
_RepeatMakser Version: 4.2.2_ and _FamDB Format Version: 2.0.0_ which supports the _Dfam 3.9_ repeat library  

---
1\. Install RepeatMasker as a conda environment (installs search engine RMBlast and TRF as well):
```bash
conda create -n RepeatMasker -c conda-forge -c bioconda repeatmasker=4.2.2 -y
```
```bash
conda activate RepeatMasker
```
\
2. Check already downloaded FamDB partitions: 
```bash
cd $CONDA_PREFIX/share/RepeatMasker
```
```bash
python famdb.py info
```
\
3. Download required partition into the famdb directory (eg: Partition 7 [Mammalia] for masking Cetaceans)  
[Link to Download Partitions](https://www.dfam.org/releases/Dfam_3.9/families/FamDB/)  
```bash
cd $CONDA_PREFIX/share/RepeatMasker/Libraries/famdb
```
```bash
aria2c -x 16 -s 16 -c --dir=$CONDA_PREFIX/RepeatMasker/Libraries/famdb https://www.dfam.org/releases/Dfam_3.9/families/FamDB/dfam39_full.7.h5.gz
```
\
4. Check number of repeats available for the ancestors and descendants of your taxon of interest:  
```bash
python famdb.py lineage -ad cetacea
```
```bash
python famdb.py -ad -f totals cetacea
```
\
5. Run RepeatMasker on genome assemblies:  
[Link to RepeatMasker Documentation](https://github.com/Dfam-consortium/RepeatMasker/blob/master/repeatmasker.help)  
<details>
<summary>Output Files Produced</summary>
<ul><li>.masked → Masked fasta file</li>
<li>.tbl → Summary statistics of repeat content</li>
<li>.out → Annotation table of all detected repeats</li>
<li>.cat -> Raw match data (can be used to generate the other output files without rerunning the whole program)</li></ul>
</details>  

```bash
RepeatMasker -species cetacea -pa 6 -q -xsmall -dir /path/to/output genomic.fna
```
\
6. If the cat file exists and you only want to generate the masked, tbl, out files again:  
```bash
ProcessRepeats -species cetacea -xsmall -maskSource genome.fna genomic.fna.cat.gz
```
\
7. Masking multiple genomes in a loop:  
- Create a list of genome assemblies (remove .fna from end)
- Create a loop to mask every genome in list with output files for each in their own directories

```bash
genome_list=("GCF_949987535.1_mBalAcu1.1_genomic" "GCF_965194805.1_mBalBor1.hap1.1_genomic" "GCF_965194825.1_mBalPhy2.hap1.1_genomic")
```
```bash
for n in ${genome_list[@]}; do 
	mkdir -p "/path/to/RepMask/${n}"; 
	RepeatMasker -species cetacea -pa 6 -q -xsmall -dir "/path/to/RM/${n}" "${/}.fna"; 
	echo "${n} done"; 
done
```
\
NOTE:  When using the -species <> flag, RepeatMasker creates a cache of species/taxon-specific libraries extracted from FamDB, making it faster next time the same -species <> argument is used. This cache may require a bit of space (eg: ~50GB for cetacea), so consider masking all sequences requiring the same -species <> argument before deleting the cache and using a different -species <> argument.  
<br>

## Downloading and Installing Cactus  
1\. Set up cactus conda environment:  
```bash
conda create -n cactus -c bioconda -c conda-forge cactus=3.1.4
```
```bash
conda activate cactus
```
2\. Test Progressive Cactus works their given example data:   
```bash
wget https://raw.githubusercontent.com/ComparativeGenomicsToolkit/cactus/master/examples/evolverMammals.txt
```
```bash
cactus /path/to/temporary/jobStore /path/to/input_file/evolverMammals.txt /path/to/output_file/evolverMammals.hal
```
NOTE: Temporary jobStore files can take up to hundreds of GB depending on the number/size of genomes being aligned. Also make store jobStore directory doesn't already exist! 
<br>

## Running Progressive Cactus  
1\. Creating the Cactus Input Seq File
- Construct a phylogenetic tree in newick format
- Every leaf in the tree must have a sequence provided
- Branch lengths represent substitutions per site (default value: 1)
- Nodes can be named or are automatically named Anc0, Anc1, etc
```bash
cat > cetaceans.txt
((Bmus:0.00266658863064363509,Bede:0.00301108031167500301)AncBlue:0.00085135825141383284,Erob:0.00386804054184409878)AncGray;
Bmus /path/to/soft/masked/genome/GCF_009873245.2_mBalMus1.pri.v3_genomic.fna.masked
Bede /path/to/soft/masked/genome/GCA_052818205.1_BalEdn.hic.v1_genomic.fna.masked
Erob /path/to/soft/masked/genome/GCF_028021215.1_mEscRob2.pri_genomic.fna.masked
```
Ctrl + D  
\
2. Run progressive cactus  
[Link to Cactus Documentation](https://github.com/ComparativeGenomicsToolkit/cactus/blob/master/doc/progressive.md)  
```bash
cactus /path/to/intermediate/jobStore /path/to/input_file/cetaceans.txt /path/to/output_file/cetaceans.hal --maxCores 24 --maxMemory 80G
```
\
## HAL to MAF Conversion  
1\. Convert binary HAL file to human-readable MAF file:  
[Link to Cactus-hal2maf Documentation](https://github.com/ComparativeGenomicsToolkit/cactus/blob/master/doc/progressive.md#maf-export)  
```bash
podman run --privileged --rm -v /path/to/cactus/directory:path/to/cactus/directory quay.io/comparative-genomics-toolkit/cactus:v3.2.1;
cactus-hal2maf  /path/to/jobStore /path/to/input_hal/cetaceans.hal /path/to/output_maf/cetaceans.maf.gz --refGenome Bmus --chunkSize 500000 --batchCount 1 --maxCores 20 --batchParallelHal2maf 10 --batchParallelTaf 4 --batchMemory 70G --maxMemory 80G --workDir /path/to/tmp
```
NOTE: Cactus conda package doesn't have the cactus-hal2maf tool, which is why we are using the cactus docker image.    
Also, make sure the --worDir tmp directory exists in a place with sufficient space.  
\
2. Extract alignment blocks corresponding to only a particular gene from HAL file  
- Get the gene's coordinates with respect to the --refGenome <> from NCBI [Here](https://www.ncbi.nlm.nih.gov/datasets/gene/)
```bash
podman run --privileged --rm -v /path/to/cactus/directory:/path/to/cactus/directory quay.io/comparative-genomics-toolkit/cactus:v3.2.1;
cactus-hal2maf /path/to/jobStore /path/to/input_hal/cetaceans.hal /path/to/output_maf/cetaceans_gene1.maf.gz --refGenome Bmus --refSequence NC_090843.1 --start 58000972 --length 13531 --maxCores 20 --batchCores 20 --batchParallelHal2maf 10 --batchParallelTaf 4 --batchMemory 70G --maxMemory 80G --workDir /path/to/tmp
```

## Downstream Analysis
1\. Extract

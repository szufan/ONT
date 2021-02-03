# ARTIC SARS-CoV-2 ONT Pipeline
## Background
An introduction to amplicon sequencing bioinformatics can be found [here](https://artic.network/quick-guide-to-tiling-amplicon-sequencing-bioinformatics.html). Information regarding the core pipeline can be found [here](https://artic.readthedocs.io/en/latest/minion/).

Installation and procedure have been adapted from [ARTIC Network](https://artic.network/ncov-2019/ncov2019-it-setup.html).

## Starting from basecalled, demuxed reads
This protocol is for use with **basecalled** AND **demultiplexed** reads. For further information regarding basecalling and demultiplexing of amplicon reads please refer to the ARTIC Network's bioinformatics [SOP](https://artic.network/ncov-2019/ncov2019-bioinformatics-sop.html). 

### Input requirements
This protocol requries the file inputs:
* A comma separated ```sample_sheet.txt``` in the format ```ID,Species,Barcode,RunID```
> Note: barcodes must be in the format ```barcode##```.

> Suggestions: Species is not needed for the pipeline, can be omitted. An ```amplicon_version``` column would be helpful.
* [Concatenated FASTQ file](#Batch-concatenating-FASTQ-files) of each barcoded sample named ```${RunID}_${Barcode}.fastq```

### Activate the ARTIC Environment
Activate Conda:
```
conda activate
```
Activate the ```artic-ncov2019-medaka``` environment:
```
conda activate artic-ncov2019-medaka
```
### Setup the project directory
Create the following project directory structure and navigate to the project folder.
```
mkdir ${RunID} ${RunID}/input
cd ${RunID}
```

### Batch execute all barcoded samples
```
sed 1d sample_sheet.txt | while IFS=, read ID Species Barcode RunID; \
	do mkdir ${ID}; \
	artic minion --medaka --normalise 200 --threads 72 \
	--scheme-directory ~/artic-ncov2019/primer_schemes \
	--read-file input/${RunID}_${Barcode}.fastq \
	nCoV-2019/V1200 \
	${ID}/${ID}; \
done
```

#### Key output files
* ```${ID}.rg.primertrimmed.bam``` - BAM file for visualisation after primer-binding site trimming
* ```${ID}.trimmed.bam``` - BAM file with the primers left on (used in variant calling)
* ```${ID}.merged.vcf``` - all detected variants in VCF format
* ```${ID}.pass.vcf``` - detected variants in VCF format passing quality filter
* ```${ID}.fail.vcf``` - detected variants in VCF format failing quality filter
* ```${ID}.primers.vcf``` - detected variants falling in primer-binding regions
* ```${ID}.consensus.fasta``` - consensus sequence


#
## Additional instructions
### Batch concatenating FASTQ files
During an ONT sequencing run, reads are written to file in batches (default = 4000), leading to multiple FASTQ files for each sample at the end of the run. The input requirement for ```artic minion``` is one FASTQ file per barcode named ```${RunID}_${Barcode}.fastq```. 

Follow these steps to concatenate all samples from the run at once:
```
# Convert sample sheet to csv (if necessary)

in2csv sample_sheet.xlxs > sample_sheet.txt

# Set the PATH to the FASTQ directory

FASTQ_PATH="/PATH_TO/fastq_pass"

# Concatenate FASTQ files, copy to local input/

sed 1d sample_sheet.txt | while IFS=, read ID Species Barcode RunID; \
	do cat ${FASTQ_PATH}/${Barcode}/*.fastq > input/${RunID}_${Barcode}.fastq; \
done
```

### Adding custom primer schemes

Files associated with Nikki Freed's V1200 amplicon scheme can be downloaded [here](https://drive.google.com/file/d/1Y85hs7-HWSsiZxU6Gh2qcij2xkW7HbZ-/view).

The following files must be in the new primer scheme folder, e.g. ```~/artic-ncov2019/primer_schemes/nCoV-2019/V1200```:
* ```nCoV-2019.bed```
* ```nCoV-2019.reference.fasta```
* ```nCoV-2019.reference.fasta.fai```
* ```nCoV-2019.scheme.bed```
* ```nCoV-2019.tsv```

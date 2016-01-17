# Octopus README file


###1. Requirements


Octopus requires Python 2.7 or greater. It has not been tested with the Python 3.X series.

It also requires the following Python packages: pysam v0.8.4, itertools, argparse, os and sys. 



###2. Running Octopus


Octopus is a tool to pre-process RNA-seq reads prior to variant calling. Currently, variant callers do not generally behave well with reads encompassing intronic regions. Octopus splits the reads to get rid of the intronic parts. At the same time, it performs several quality control measures such as discarding secondary alignments, poorly mapped reads and read pairs where the two reads are pointing outwards or in the same direction, or that have been mapped in different chromosomes. Octopus also discards duplicate reads based on the start and end positions of the read pair. Furthermore, it merges overlapping paired-end reads.

Octopus has been designed to work specifically with Platypus, but it can be used equally well with other variant callers such as GATK.


Octopus can be run as follows:

    python Octopus.py --BamFile=input.bam --OutFile=output.bam

Octopus has been tested to work with BAM files that have been aligned with TopHat and Star. Whether or not the aligner uses soft clips should be specified with the parameter *SoftClipsExist* (set *False* for TopHat, *True* for Star).

The BAM file given as input to Octopus should be sorted according to read position. The reads should include the MD tag in the optional field. If MD tags have not been provided, they can be generated with samtools calmd. Octopus does not currently handle reads with hard clips and these will be discarded.

Octopus expects that base qualities have been expressed in the standard Illumina encoding starting at 33. If another encoding has been used (such as the encoding used in earlier Illumina platforms, which starts at 64), the base qualities should be first converted to the standard encoding with e.g. GATK.

Octopus sorts the output file and creates a corresponding index file .bai.


You can see a list of all the possible input options by running the following command:

    python Octopus.py --help

You should adjust the input parameters according to which mapper has been used for aligning the reads in the input BAM file.



###3. Main command-line arguments to Octopus


*--BamFile* : (Required) BAM file name, or entire path if the file is not located in the same directory as where the code is run.

*--MinFlank* : Ignore base-changes closer than MinFlank bases to the end of the read. The corresponding base qualities will be set to 0. Default = 0

*--MinFlankStart* : Ignore base-changes closer than MinFlankStart bases to the start of the read. The corresponding base qualities will be set to 0. Default = 0

*--SoftClipsExist* : If set to False (default), no soft clips are expected in the reads, and MinFlank and MinFlankStart are applied to all reads. If set to True, then MinFlank and MinFlankStart are only applied to reads with soft clips. The value typically depends on the aligner used, e.g. for TopHat, it should be False, and for Star, it should be True.

*--MapCutoff* : Minimum mapping quality of a read pair. If either read in the pair has a lower mapping quality, both will be discarded. Default = 40

*--ProperlyPaired* : If set to True (default), only properly paired reads are considered.

*--KeepMismatches* : If set to False (default), overlapping paired-end reads having at least one base mismatch within the overlap region are discarded.

*--OutFile* : (Required) Output BAM file name, or entire path if the file will not be created in the same directory as where the code is run.



##4. Suggested parameters when calling variants with Platypus


If, after running Octopus, variants are called using Platypus, we suggest keeping the Platypus settings at a minimum. This is because the reads have already been formatted in a way to take into account the various options included in Platypus.

    python Platypus.py --callVariants --bamFiles input.bam --refFile ref.fasta --filterDuplicates 0 --minMapQual 0 --minFlank 0 --maxReadLength 500 --minGoodQualBases 10 --minBaseQual 20 -o variants.vcf

It should be noted that *minGoodQualBases* also defines the minimum acceptable read length to be considered for variant calling. Using a value below the suggested 10 here may cause segmentation fault.

By default, Platypus flags variants that do not fulfill all of its filtering criteria (please see Platypus publication). These criteria have been designed to make the most out of DNA data. The same criteria can well be used with RNA-seq data if the user wants to maximize precision at the cost of sensitivity. However, if the user seeks a greater balance between precision and sensitivity, it would be advisable to include variants flagged as 'badReads', 'SC', and 'Q20' among the final variants.


###5. Release History


####0.1.


Released in January 2016. First stable release of Octopus.



###Contact

Laura Oikkonen: firstname.surname (AT) well.ox.ac.uk


###License

Octopus is available under the GPL v3 license.
## Mapping Exercise

MIP 280A4
---

## In this exercise, we will learn how to create an index from a reference sequence, then map reads to that reference sequence

### Downloading the boa constrictor (mitochondrial) genome.

The dataset you downloaded last week was created by sequencing a library made from boa constrictor liver RNA.  We will map the reads in this dataset to the boa constrictor genome sequence to demonstrate read mapping.

First, we need to *find* the boa constrictor genome.  As usual, there are few ways we could go about this:

1. navigate through the NCBI [Taxonomy database](https://www.ncbi.nlm.nih.gov/taxonomy/)
2. navigate through the NCBI [Genome database](https://www.ncbi.nlm.nih.gov/genome/)
3. navigate through another genome database, like [Ensembl](http://www.ensembl.org/index.html) or [UCSC](https://genome.ucsc.edu/)
4. google 'boa constrictor genome sequence'  (not a terrible way to do it)

- Let's choose option 1, and go through the NCBI Taxonomy database.  Navigate to https://www.ncbi.nlm.nih.gov/taxonomy/
   - Search for `boa constrictor`.
   - Click on Boa constrictor link, then click the Boa constrictor link again
   - You should see a table in the upper right corner showing linked records in various NCBI (Entrez) databases.
   - Click on the `Nucleotide` link in that table to go to boa constrictor sequences in the NCBI Nucleotide database

   - There are ~600 boa constrictor nucleotide sequences in this database.  We want the mitochondrial genome, which happens to be the only nucleotide sequence in the NCBI RefSeq database.
   - Click on "RefSeq" filter on the left hand side of the page.
   - You should see a link to the boa constrictor mitochondrial genome sequence.
   - Click on this 'NC_007398.1' RefSeq link

Now we need to download the sequence.  We'll do this through the browser.  In the upper right hand corner of the page, note the 'Send' drop down

- Click Send->Complete Record->File->Format->FASTA->Create File

You should have downloaded a fasta file of ~19 kb, named sequence.fasta, or something like that.

Now download the sequence in GenBank format too.  Note that this file is larger (~42 kb), because it contains annotation as well as the actual sequence.

Note that the downloaded files have unhelpful names: `sequence.fasta` and `sequence.gb` or similar.  Rename these files and transfer them to the thoth01 server, where you have the fastq-format sequences that you downloaded and trimmed last time.  Rename them to:

- boa_mtDNA.fasta
- boa_mtDNA.gb


### Once you have these files on the thoth01 server

**Change to the directory where your trimmed fastq files are and make sure these new boa_mtDNA files are in the same directory.**

Let's inpsect the contents of these files.  You can use the cat (or less) commands to output the contents of these files:
```
# cat outputs the contents of a file all at once
cat boa_mtDNA.fasta
cat boa_mtDNA.gb

# less allows you to page through files
less boa_mtDNA.fasta
less boa_mtDNA.gb
```

Hint: press `space` to advance a page in less and press `q` to exit


We want these files in Geneious too.  Drag them into Geneious:
 - Open Geneious
 - Create a new folder in Geneious
 - Drag and drop these files into Geneious




### Create a bowtie index from the boa constrictor mitochondrial genome sequence

We will map reads in the SRA dataset that we downloaded and trimmed the other day to the boa constrictor mitochondrial genome.

Read mapping tools map reads very quickly using pre-built indexes of the reference sequence(s) to which reads will be mapped.  We'll use the [Bowtie2](http://www.nature.com/nmeth/journal/v9/n4/full/nmeth.1923.html) mapper. Bowtie2 has a nice [manual](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml) that explains how to use this software. 

Note that there are a variety of other good read mapping tools, such as [STAR](https://github.com/alexdobin/STAR) and [BWA](https://github.com/lh3/bwa).

The first step will be to create an index of our reference sequence (the boa constrictor mitochondrial genome).

Make sure you are in the right directory (our working directory):
```
pwd
```

Now confirm that the boa constrictor mtDNA sequence file is there and in FASTA format: 
```
ls -lh    # should see: boa_mtDNA.fasta and 2 trimmed fastq (SRR1984309_1_trimmed.fastq SRR1984309_2_trimmed.fastq)
```

Now, we'll use the bowtie2-build indexing program to create the index.  This command takes 2 arguments: 
(1) the name of the fasta file containing the sequence(s) you will index
(2) the name of the index (can be whatever you want)

```
bowtie2-build boa_mtDNA.fasta boa_mtDNA_bt_index 
```

Confirm that you built the index.  You should see six files with names ending in bt2, like boa_mtDNA_bt_index.3.bt2
```
ls -lh
```

Note that this index building went very fast for a small genome like the boa mtDNA, but can take much longer (hours) for Gb-sized genomes.


### Mapping reads in the SRA dataset to the boa constrictor mitochondrial genome 

Now that we've created the index, we can map reads to the boa mtDNA.  We'll map our Trimmomatic-trimmed paired reads to this sequence, as follows:

```
bowtie2 -x boa_mtDNA_bt_index \
   -q -1 SRR1984309_1_trimmed.fastq  -2 SRR1984309_2_trimmed.fastq \
   --no-unal --threads 4 -S SRR1984309_mapped_to_boa_mtDNA.sam
```

Let's deconstruct this command line (note: the comments will screw up this command: don't copy and paste from this box): 
```
 bowtie2
   -x boa_mtDNA_bt_index         # -x: name of index you created with bowtie2-build
   -q                # -q: the reads are in FASTQ format
   -1 SRR1984309_1_trimmed.fastq      # name of the paired-read FASTQ file 1
   -2 SRR1984309_2_trimmed.fastq      # name of the paired-read FASTQ file 2
   --no-unal            # don't output unmapped reads to the SAM output file (will make it _much_ smaller
   --threads 4            # since our computers have multiple processers, run on 4 processors to go faster
   -S SRR1984309_mapped_to_boa_mtDNA.sam   # name of output file in SAM format
```

- Some questions to consider:
  - What percentage of reads mapped to the boa mitochondrial genome?
  - Does this make biological sense?


The output file SRR1984309_mapped_to_boa_mtDNA.sam is in [SAM format](https://en.wikipedia.org/wiki/SAM_(file_format)).  This is a plain text format, so you can look at the first 20 lines by running this command:


```
head -20 SRR1984309_mapped_to_boa_mtDNA.sam      
```

You can see that there are several header lines beginning with `@`, and then one line for each mapped read.  See [here](http://genome.sph.umich.edu/wiki/SAM) or [here](https://samtools.github.io/hts-specs/SAMv1.pdf) for more information about interpreting SAM files.



### Visualizing aligned (mapped) reads in Geneious

Geneious provides a nice graphical interface for visualizing the aligned reads described in your SAM file.   Other tools for visualizing this kind of data include [IGV](http://software.broadinstitute.org/software/igv/) and [Tablet](https://ics.hutton.ac.uk/tablet/)

First, you need to transfer the SRR1984309_mapped_to_boa_mtDNA.sam file from thoth01 to your computer.  Use sftp or cyberduck to do this.

Second, you need to have your reference sequence in Geneious, preferably with annotations.  You can do this 2 ways:

1. Drag and drop the boa_mtDNA.gb file into a folder in Geneious
2. Download the file directly into Genious, using the NCBI->Nucleotide interface (search for NC_007398.1).  Once downloaded, drag from the NCBI download folder into another folder in Geneious.  

Once you have the boa constrictor mitochondrial genome in a folder in Geneious, you can drag and drop the SAM file that bowtie2 output into the same folder.  Geneious will tell you that it 'can't find the sequence it needs in the selected file'.  It is telling you it is trying to find the reference sequence to which you aligned reads.  Answer: 'Find a sequence with the same name in this Geneious folder' or 'Use one of the selected sequences' (after selecting the boa mtDNA sequence).

- A few Geneious tips:
  - Enlarge the Geneious window so that it fills the screen
  - Click View->Expand Document View to enlarge the alignment
  - Try playing with the visualization settings in the panels on the right of the alignment

- Some questions to consider when viewing the alignment:
  - Is the coverage even across the mitochondrial genome?  
  - What is the average coverage across the mitochondrial genome?
  - This is essentially RNA-Seq data.  Are the mitochondrial genes expressed evenly?  
  - How does this relate to coverage?
  - Are there any variants between this snake's mitochondrial genome sequence and the boa constrictor reference sequence?  
  - Is it expected that there are variants?
  - Can you distinguish true variants from sequencing errors?
  - How can you distinguish true variants from sequencing errors?
  - Is it possible that reads that derive from the boa constrictor nuclear genome are mapping to this sequence?  
  - How would you prevent nuclear reads from mapping to the mitochondrial genome?
  - Can you identify mapped read pairs?  



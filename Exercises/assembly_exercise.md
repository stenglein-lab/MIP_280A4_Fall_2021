## Mapping and Assembly Exercise

MIP 280A4
---

## We will continue the exercise we started in previous classes using reads from a shotgun sequencing library from total RNA from the liver of a boa constrictor with inclusion body disease.  

### De-novo assembly of non-mapping reads

Last time, you practiced mapping to a reference sequence by mapping reads to the boa constrictor mitochondrial genome.  Imagine instead that we *don't* have a reference sequence.  In that case, we'd need to perform de novo assembly.  

There are a variety of de novo assemblers with different strengths and weaknesses.  We're going to use the [SPAdes assembler](http://cab.spbu.ru/software/spades/) to assemble the reads in our dataset that **don't** map to the boa constrictor genome. First, let's map the reads in our dataset to the _entire_ boa constrictor genome, not just the mitochondrial genome.

The instructors have already downloaded an assembly of the boa constrictor genome from [here](http://gigadb.org/dataset/100060) and made a bowtie2 index, which can be found on thoth01 in /home/data_for_classes/2021_MIP_280A4.  We could have you make an index yourself, but that would take a long time for a Gb genome like the boa constrictor's.  The boa constrictor genome index is named boa_constrictor_bt_index.

First, change to the directory in which you have your fastq files containing trimmed sequences.  Confirm you are in the right directory by running `ls`.  You should see the trimmed fastq files.

Now, we'll run bowtie2 to map reads to the _entire_ boa constrictor genome.  This time we'll run bowtie2 a little differently:
1. We'll run bowtie2 in [local mode](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml#end-to-end-alignment-versus-local-alignment), which is a more permissive mapping mode that doesn't require the ends of the reads to map
2. We'll keep track of which reads _didn't_ map to the genome using the --un-conc option

```
bowtie2 -x /home/data_for_classes/2021_MIP_280A4/boa_constrictor_bt_index \
   --local \
   -1 SRR1984309_1_trimmed.fastq \
   -2 SRR1984309_2_trimmed.fastq \
   --no-unal \
   --threads 24 \
   -S SRR1984309_mapped_to_boa_genome.sam \
   --un-conc SRR1984309_not_boa_mapped.fastq
```

**Command line options explained:**
- -x: the bowtie2 index
- --local: run bowtie2 in local mode: don't require the ends of reads to map
- -1: the first file containing paired reads
- -2: the second file containing paired reads 
- --no-unal: don't report unmapped reads in the sam file
- --threads 24: use 24 threads (24 CPUs) to make bowtie2 run faster
- -S: name of the sam format output file that will be generated
- --un-conc: put read pairs that don't map into new fastq files with this name

#### Questions about this mapping

- What percentage of reads mapped to the nuclear boa constrictor genome (nuclear + mitochondrial)?   [This is the "overall alignment rate"]
- How does this compare to the percentage of reads that mapped to just the mitochondrial genome?
- Does it make sense that this percentage of reads mapped to the entire genome?  
- What might be the source or sources of non-mapping reads?
- The non-mapping reads should be in new files named `SRR1984309_not_boa_mapped.1.fastq` and `SRR1984309_not_boa_mapped.2.fastq`.  How many reads are in each of these files?

Bowtie2 outputs information about whether reads mapped concordantly or not and whether they mapped uniquely or not.  It's honestly confusing to wade through these different categories, so let's just map the reads to the genome without accounting for whether they are paired or not.  Now, run bowtie2 like this: 

```
bowtie2 -x /home/data_for_classes/2021_MIP_280A4/boa_constrictor_bt_index \
   --local \
   -U SRR1984309_1_trimmed.fastq \
   -U SRR1984309_2_trimmed.fastq \
   --no-unal \
   --threads 24 \
   -S SRR1984309_mapped_to_boa_genome.unpaired.sam 
```

The big difference with running bowtie2 this time is that you are telling it that all the reads are unpaired (using the -U option) instead of specifying that your reads are paired (using the -1 and -2 options).  Running bowtie2 this way makes it easier to understand what fraction of reads mapped uniquely.

#### Questions about this mapping

- What percentage of reads mapped _uniquely_ to the boa constrictor genome?
- What percentage of reads mapped _non-uniquely_ (>1 time) to the boa constrictor genome?
- What can you say about the regions of the boa constrictor genome to which these reads mapped non-uniquely? 

### Assembly

Now, we are going to assembly the boa constrictor **non-mapping** reads to try to understand what might be making this snake sick.  

We will use the non-mapping reads as input to our de novo SPAdes assembly.  Run SPAdes as follows:

```
spades.py   -o SRR1984309_spades_assembly \
   --pe1-1 SRR1984309_not_boa_mapped.1.fastq \
   --pe1-2 SRR1984309_not_boa_mapped.2.fastq \
   -m 24 -t 18
```

**Command line options explained:**
- -o:  name of new directory where SPAdes output will go
- --pe1-1:  name of first paired read input file
- --pe1-2:  name of second paired read input file
- -m 24 -t 18: use 24 Gb of RAM and 18 threads (CPUs)

SPAdes will output a bunch of status messages to the screen as it runs the assembly.  Can you tell what the different assembly steps are?

After SPAdes finishes, there will be output files in the `SRR1984309_spades_assembly` folder.  The key ones are:

- contigs.fasta:   the assembled contigs in FASTA format
- scaffolds.fasta: scaffolds in FASTA format
- assembly_graph.fastg:   de bruijn graphs used to create contigs.  Can be visualized using a tool like [Bandage](https://rrwick.github.io/Bandage/)

#### Questions about the assembly

- Spades produced an output file named scaffolds.fasta.  How is this file different from contigs.fasta? 
- What is needed to go beyond contigs to produce a scaffolded assembly?  
- What kmer sizes did Spades use during this assembly?  (Hint: run `grep started spades.log` in the spades output directory to search for the text "started" in the spades log file)

Let's look at the contigs in contigs.fasta.  Transfer this file to your computer and open it using a text editor like BBEdit or Notepad++.

The contigs are sorted in order of length and have names like NODE_1..., NODE_2..., etc..  Recall that these are contigs made from the reads that _didn't_ map to the boa constrictor genome. Let's try to figure out what the longest contigs are.  Use what you've learned during the earlier parts of this class to identify what the 5 longest contigs are (NODE_1 ... NODE_5)


#### Questions about the contigs
- What are the 5 longest contigs?  Are you confident in your conclusions?  Does this make sense?




#### Additional, time-permitting exercises 

**1. Assembly validation:**

To quote [Miller et al](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2874646/), these contigs are only "putative reconstructions" of the sequences from which the reads derived.  How could we validate these sequences as being accurate?

One way would be to use another sequencing technology, like PCR and Sanger sequencing.

Another way to validate an assembly is to re-map reads back to it using a mapping tool like bowtie2.  This might reveal errors in the assembly, or mis-assemblies.  

If time permits, use what you've learned and re-map reads back to these contigs.  To do this, you'll have to create a new bowtie index (e.g. of the first 5 contigs) using bowtie2-build, then use bowtie2 to map reads.  Then you can visualize the aligned reads in Geneious.  Can you find any problems with the assemblies?

Hint: to get the first 5 contigs in a new fasta file, run:
```
seqtk seq -A contigs.fasta | head -10 > first_5_contigs.fasta
```

**2. Sequence annotation**
Another thing you could do is annotate the virus contigs.  Geneious is a great tool for doing things like finding ORFs in sequences and adding annotations, that can then be exported in GenBank format.

**3. Assemble the entire datasets**

You could also try assembling all of the reads in the datasets, not just the ones that didn't map to the boa constrictor genome.  This will take longer, but should be doable in a minute or so on thoth01.  What are the top contigs now?  What happened to the virus contigs?

## Mapping and Assembly Exercise

GDW 2023
---

### In this exercise, we will learn how to create an index from a reference sequence, then map reads to that reference sequence, and will perform a de novo assembly.  We'll:


### Create a bowtie index from the boa constrictor mitochondrial genome sequence

We will map reads in the SRA dataset that we downloaded yesterday to the boa constrictor mtDNA sequence that we also downloaded yesterday.  

Read mapping tools map reads very quickly because they use pre-built indexes of the reference sequence(s). We'll use the [Bowtie2](http://www.nature.com/nmeth/journal/v9/n4/full/nmeth.1923.html) mapper. Bowtie2 has a nice [manual](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml) that explains how to use this software. 

There are a variety of other good read mapping tools, such as [BWA](https://github.com/lh3/bwa).  [minimap2](https://github.com/lh3/minimap2) is a mapper which works well for long read data or long sequences.

The first step will be to create an index of our reference sequence (the boa constrictor mitochondrial genome).

In your terminal window:

Make sure you are in the right directory (our working directory):
```
pwd
cd ~/gdw_working   # if necessary
```

Now confirm that the boa constrictor mtDNA sequence file is there and in FASTA format: 
```
ls -lh    # should see: boa_mtDNA.fasta
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

Now that we've created the index, we can map reads to the boa mtDNA.  We'll map our cutadapt-trimmed paired reads to this sequence, as follows:

```
bowtie2 -x boa_mtDNA_bt_index \
   -1 SRR1984309_1_trimmed.fastq \
   -2 SRR1984309_2_trimmed.fastq \
   --no-unal \
   --threads 8 \
   -S SRR1984309_mapped_to_boa_mtDNA.sam
```

Let's deconstruct this command line:

| Part | Purpose |
| ---- | ------- |
| bowtie2 | name of command |
| -x boa_mtDNA_bt_index | -x: name of index you created with bowtie2-build |
| -1 SRR1984309_1_trimmed.fastq | name of the paired-read FASTQ file 1 |
| -2 SRR1984309_2_trimmed.fastq | name of the paired-read FASTQ file 2 |
| --no-unal | don't output unmapped reads to the SAM output file (will make it _much_ smaller |
| --threads 8 | since the server has multiple processers, run on 8 processors to go faster |
| -S SRR1984309_mapped_to_boa_mtDNA.sam   | name of output file in [SAM format](https://en.wikipedia.org/wiki/SAM_(file_format)) |


---
:question: **Questions:**
- What percentage of reads mapped to the boa mitochondrial genome?
- Does this make sense biologically?  Remember that this is total RNA from snake liver tissue.  Explain.
---


The output file SRR1984309_mapped_to_boa_mtDNA.sam is in [SAM format](https://en.wikipedia.org/wiki/SAM_(file_format)).  This is a plain text format, so you can look at the first 20 lines by running this command:


```
head -20 SRR1984309_mapped_to_boa_mtDNA.sam      
```

You can see that there are several header lines beginning with `@`, and then one line for each mapped read.  See [here](http://genome.sph.umich.edu/wiki/SAM) or [here](https://samtools.github.io/hts-specs/SAMv1.pdf) for more information about interpreting SAM files.

---
:question: **Answer the following questions about the first mapped read:**
- What position in the mtDNA sequence did the first mapped read map to?
- What is the mapping quality for this read's mapping?
- What does this mapping quality score indicate?
---


### Visualizing aligned (mapped) reads in Geneious

Geneious provides a nice graphical interface for visualizing the aligned reads described in your SAM file.   Other tools for visualizing this kind of data include [IGV](https://software.broadinstitute.org/software/igv/) and [Tablet](https://ics.hutton.ac.uk/tablet/)
 

First, you need to have your reference sequence in Geneious, preferably with annotations.  You can do this 2 ways:

1. Drag and drop the boa_mtDNA.gb file into a folder in Geneious
2. Download the file directly into Genious, using the NCBI->Nucleotide interface (search for NC_007398.1).  Once downloaded, drag from the NCBI download folder into another folder in Geneious.  

Once you have the boa constrictor mitochondrial genome in a folder in Geneious, you can drag and drop the SAM file that bowtie2 output into the same folder.  Geneious will tell you that it 'can't find the sequence it needs in the selected file'.  It is telling you it is trying to find the reference sequence to which you aligned reads.  Answer: 'Find a sequence with the same name in this Geneious folder' or 'Use one of the selected sequences' (after selecting the boa mtDNA sequence).

- A few Geneious tips:
  - Enlarge the Geneious window so that it fills the screen
  - Click View->Expand Document View to enlarge the alignment
  - Try playing with the visualization settings in the panels on the right of the alignment

---
:question: **Questions to consider when viewing the alignment:**
- What is the average coverage depth across the mitochondrial genome?
- Is the coverage *even* across the mitochondrial genome?
- Would you expect coverage to be even across the genome?  (Recall that this data is derived from total RNA from liver tissue).
- Are the mitochondrial genes expressed evenly?
- Are there any variants between this snake's mitochondrial genome sequence and the boa constrictor reference sequence?
- Is it expected that there are variants between these reads and this reference sequence?  Explain your answer.
- Can you distinguish true variants from sequencing errors?
- In general, how can you distinguish true variants from sequencing errors?
- Is it possible that reads that derive from the boa constrictor nuclear genome are mapping to this sequence?
- How would you prevent nuclear reads from mapping to the mitochondrial genome?
- Can you identify mapped read pairs?
---


### **Stop here** - we will proceed to assembly after lunch.

---

### De-novo assembly of non-mapping reads

OK, now we've practiced mapping to a reference sequence.  Imagine instead that we don't have a reference sequence.  In that case, we'd need to perform de novo assembly.  

There are a variety of de novo assemblers with different strengths and weaknesses.  We're going to use the [SPAdes assembler](http://cab.spbu.ru/software/spades/) to assemble the reads in our dataset that don't map to the boa constrictor genome. First, let's map the reads in our dataset to the _entire_ boa constrictor genome, not just the mitochondrial genome.

The instructors have already downloaded an assembly of the boa constrictor genome from [here](http://gigadb.org/dataset/100060) and made a bowtie2 index, which can be found on your HDDs.  We could have you make an index yourself, but that would take a long time for a Gb genome like the boa constrictor's.  The boa constrictor genome index is named boa_constrictor_bt_index.

First, let's transfer the bowtie index from the HDD to your working folder:
```
# make sure in the right working directory
cd ~/gdw_working

# copy the boa constrictor full genome bowtie index to the pwd.  Note the . at the end of the command line
cp /Users/gdw/Desktop/GDW_Data/Mark/boa_constrictor_bt_index* .
```

Now, we'll run bowtie2 to map reads to the _entire_ boa constrictor genome.  This time we'll run bowtie2 a little differently:
1. We'll run bowtie2 in [local mode](http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml#end-to-end-alignment-versus-local-alignment), which is a more permissive mapping mode that doesn't require the ends of the reads to map
2. We'll keep track of which reads _didn't_ map to the genome using the --un-conc option

```
bowtie2 -x boa_constrictor_bt_index \
   --local \
   -1 SRR1984309_1_trimmed.fastq \
   -2 SRR1984309_2_trimmed.fastq \
   --no-unal \
   --threads 4 \
   -S SRR1984309_mapped_to_boa_genome.sam \
   --un-conc SRR1984309_not_boa_mapped.fastq
```

**Command line options explained:**
- -x: the name of the bowtie2 index
- --local: run bowtie2 in local mode: don't require the ends of reads to map
- -1: the first file containing paired reads
- -2: the second file containing paired reads
- --no-unal: don't report unmapped reads in the sam file
- --threads 4: use 4 threads (4 CPUs) to make bowtie2 run faster
- -S: name of the sam format output file that will be generated
- --un-conc: put read pairs that *don't* map into new fastq files with this name

---
:question: **Questions about this mapping**
- What percentage of reads mapped to the nuclear boa constrictor genome (nuclear + mitochondrial)?   [This is the "overall alignment rate"]
- How does this compare to the percentage of reads that mapped to just the mitochondrial genome?
- Does it make sense biologically that this percentage of reads mapped to the entire genome?
- What might be the source or sources of non-mapping reads?
- The non-mapping reads should be in new files named `SRR1984309_not_boa_mapped.1.fastq` and `SRR1984309_not_boa_mapped.2.fastq`.  How many reads are in each of these files?
---

Bowtie2 outputs information about whether reads mapped concordantly or not and whether they mapped uniquely or not.  It's honestly confusing to wade through these different categories, so let's just map the reads to the genome without accounting for whether they are paired or not.  Now, run bowtie2 like this:

```
bowtie2 -x boa_constrictor_bt_index \
   --local \
   -U SRR1984309_1_trimmed.fastq \
   -U SRR1984309_2_trimmed.fastq \
   --no-unal \
   --threads 4 \
   -S SRR1984309_mapped_to_boa_genome.unpaired.sam
```

The big difference with running bowtie2 this time is that you are telling it that all the reads are unpaired (using the -U option) instead of specifying that your reads are paired (using the -1 and -2 options).  Running bowtie2 this way makes it easier to understand what fraction of reads mapped uniquely.

---
:question: **Questions about this mapping**
- What percentage of reads mapped _uniquely_ to the boa constrictor genome?
- What percentage of reads mapped _non-uniquely_ (>1 time) to the boa constrictor genome?
- What can you say about the regions of the boa constrictor genome to which these reads mapped non-uniquely?
---

## Assembly

Now, we are going to assembly the boa constrictor **non-mapping** reads to try to understand what might be making this snake sick.
There are a variety of de novo assemblers with different strengths and weaknesses.  We're going to use the [SPAdes assembler](http://cab.spbu.ru/software/spades/), which is a great general purpose assembler.

We will use these non-mapping reads as input to our de novo SPAdes assembly.  Run SPAdes as follows:

```
spades.py   -o SRR1984309_spades_assembly \
   --pe1-1 SRR1984309_not_boa_mapped.1.fastq \
   --pe1-2 SRR1984309_not_boa_mapped.2.fastq \
   -m 12 -t 4
```

**Command line options explained:**
- -o:  name of new directory where SPAdes output will go
- --pe1-1:  name of first paired read input file
- --pe1-2:  name of second paired read input file
- -m 12 -t 4: use 12 Gb of RAM and 4 threads (CPUs)

SPAdes will output a bunch of status messages to the screen as it runs the assembly.  Can you tell what the different assembly steps are?

After SPAdes finishes, there will be output files in the `SRR1984309_spades_assembly` folder.  The key ones are:

- contigs.fasta:   the assembled contigs in FASTA format
- scaffolds.fasta: scaffolds in FASTA format
- assembly_graph.fastg:   de bruijn graphs used to create contigs.  Can be visualized using a tool like [Bandage](https://rrwick.github.io/Bandage/)


---
:question: **Questions about the assembly**
- Spades produced an output file named scaffolds.fasta.  How is this file different from contigs.fasta?
- What is needed to go beyond contigs to produce a scaffolded assembly?
- What kmer sizes did Spades use during this assembly?  (Hint: run `grep started spades.log` in the spades output directory to search for the text "started" in the spades log file)
---

Let's look at the contigs in contigs.fasta.  Navigate to that file in the Finder and open it using a text editor like BBEdit.

The contigs are sorted in order of length.  Recall that these are contigs made from the reads that _didn't_ map to the boa constrictor genome. Let's try to figure out what some of the contigs are.

Copy the first 4 contigs (the 4 longest contigs) and open a browser, navigate to the [NCBI blastn page](https://blast.ncbi.nlm.nih.gov/Blast.cgi?PROGRAM=blastn&PAGE_TYPE=BlastSearch&LINK_LOC=blasthome), and paste the sequences of the first 4 contigs into the search field.  Make sure that the megablast option is selected, and run the BLAST.  

---
:question: **Questions about contigs**
- What are the first 4 (the 4 longest) contigs?  Are you confident in your conclusions?  Do they make sense biologically?
---


#### Additional, time-permitting exercises 

**1. Assembly validation:**

To quote [Miller et al](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2874646/), these contigs are only "putative reconstructions" of the sequences from which the reads derived.  How could we validate these sequences as being accurate?

One way would be to use another sequencing technology, like PCR and Sanger sequencing.

Another way to validate an assembly is to re-map reads back to it using a mapping tool like bowtie.  This might reveal errors in the assembly, or mis-assemblies.  

If time permits, use what you've learned and re-map reads back to these contigs.  To do this, you'll have to create a new bowtie index (e.g. of the 1st 2 or 3 contigs) using bowtie2-build, then use bowtie2 to map reads.  Then you can visualize the aligned reads in Geneious.  Can you find any problems with the assemblies?

**2. Sequence annotation**
Another thing you could do is annotate the virus contigs.  Geneious is a great tool for doing things like finding ORFs in sequences and adding annotations, that can then be exported in GenBank format.

**3. Assemble the entire datasets**

You could also try assembling all of the reads in the datasets, not just the ones that didn't map to the boa constrictor genome.  This will take longer, but should be doable in a minute or two on your laptops.  What are the top contigs now?  What happened to the virus contigs?

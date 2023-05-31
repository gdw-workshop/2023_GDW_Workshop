## Genome assembly exercise, continued: Hybrid assembly

In this exercise, we will perform an assembly using just short reads (Illumina MiSeq 2x250) or a hybrid assembly using the same short reads plus long reads from a MinION run. 

These reads derive from genomic DNA from a veterinary clinical _Pseudomonas aeruginosa_ isolate.  The same gDNA was sequenced on an Illumina MiSeq to generate 433191 2x250 bp read pairs and on an Oxford Nanopore MinION to generate 82702 long reads with an average length of 2150 bp.  

- How long would you expect a _Pseudomonas aeruginosa_ genome to be?  How did you figure that out?
- Given the answer above, What is the expected coverage levels of MiSeq and MinION data?

### Illumina-only assembly

First, we will do an Illumina-only assembly.  

The first thing we need to do is copy and unpack the necessary data.  

```
# create a new directory
cd
mkdir hybrid_assembly
cd hybrid_assembly

# copy and unpack the data file
cp ~/Desktop/GDW_Data/Mark/hybrid_assembly_exercise_files.tgz .
tar xvf hybrid_assembly_exercise_files.tgz
```


Now that we have the data, let's assemble the short read data alone. Run the SPAdes assembler as follows:

```
spades.py -o illumina_only_assembly -1 illumina_R1_corrected.fastq -2 illumina_R2_corrected.fastq -t 8 -m 12 --only-assembler  -k 127
```

This should take ~3 minutes to run on your laptops.

A couple notes about this:

- SPAdes usually does multiple assemblies with different k-mer sizes and uses the best one.  Here, we are telling SPAdes to only do a k=127 assembly in order to save time
- SPAdes also normally includes an error correction step, which we are skipping by specifying the --only-assembler option.  The reads you are using have already been corrected by running SPAdes with the --only-error-correction option.  


When the assembly finishes, let's grab the contigs and use [QUAST](http://quast.sourceforge.net/quast) to get some basic statistics about the assembly.  The assembly will be in the illumina_only_assembly directory.  

What files can you see in that directory?  

```
ls illumina_only_assembly
```

The contigs are in the contigs.fasta file.  We'll copy this file to our present working directory and rename it.

```
cp illumina_only_assembly/contigs.fasta ./illumina_only_contigs.fa
```

Run QUAST to generate some statistics on the assembly:

```
quast.py illumina_only_contigs.fa -o illumina_only_quast_report
```

You will notice in the illumina_only_quast_report directory a file named report.html.  Open that in a browser by running:

```
open illumina_only_quast_report/report.html
```

View the report in your browser and consider these questions:

- what is the total length of the assembly?  How does this compare to the expected genome size from above?
- How many contigs were generated in total?  How many of these are >50,000 bp in length?
- What is the assembly N50?  How does this compare to the expected total genome size?
- Does this assembly include any gaps (these would show up as Ns)?  In other words, is this a scaffolded assembly?

Another way to assess an assembly would be to map the contigs to a suitably close reference genome.  We happen to have such a reference genome to use in this case, which is convenient and not always the case. 

Short read mappers like bowtie are not necessarily the best tools for mapping long reads or long sequences like contigs.  We'll use the [minimap2](https://github.com/lh3/minimap2) tool to map the contigs to this pre-existing refence genome.

```
minimap2 -a Pa_ref.fa illumina_only_contigs.fa --secondary=no > illumina_contigs_mapped_to_ref_genome.sam
```

Now, drag and drop the file Pa_ref.gb into Geneious.  Then, drag and drop the illumina_contigs_mapped_to_ref_genome.sam file into the same folder in Geneious.  Answer the following questions:

- Is the reference genome entirely covered by contigs? 
- Do you notice any characteristics of sequence features that exist at the ends of contigs? 
- Did all the contigs map to the reference genome?  How many didn't?  Why didn't these contigs map to the reference genome?


### Hybrid assembly

OK, let's see if we can get a better assembly by including long read data.  SPAdes is also capable of hybrid assembly.  You may be wondering how SPAdes uses these long reads.  On the SPAdes manual [webpage](http://cab.spbu.ru/files/release3.12.0/manual.html) you will find: 

```
SPAdes can take as an input an unlimited number of PacBio and Oxford Nanopore libraries.

PacBio CLR and Oxford Nanopore reads are used for hybrid assemblies (e.g. with Illumina or IonTorrent). 
There is no need to pre-correct this kind of data. SPAdes will use PacBio CLR and Oxford Nanopore reads 
for gap closure and repeat resolution.

...Oxford Nanopore reads are provided with --nanopore option.
```

So, we'd just basically need to rerun the assembly and add the --nanopore option to the command line and SPAdes will do a hybrid assembly.  

We are not going to actually do this, because it takes ~25 minutes on your laptops, so the instructors already did it for you, by running this command:

```
# This takes ~25 minutes to run
### spades.py -o hybrid_assembly -1 illumina_R1_corrected.fastq -2 illumina_R2_corrected.fastq -t 8 -m 12 --only-assembler  -k 127 --nanopore nanopore_reads.fastq
```

The assembly is in the hybrid_assembly directory.  Check it out:
```
ls hybrid_assembly
```

The contigs are in the contigs.fasta file.  Copy and rename this:
```
cp hybrid_assembly/contigs.fasta ./hybrid_contigs.fa
```

Let's run QUAST to get some stats on the assembly:

```
quast.py hybrid_contigs.fa -o hybrid_quast_report
open hybrid_quast_report/report.html
```
Answer the questions as before:

- What is the total length of the assembly?  How does this compare to the illumina-only assembly? 
- How many contigs were generated in total?  How many of these are >50,000 bp in length?
- What is the assembly N50?  

Now let's again map this hybrid assembly to the reference genome.

```
minimap2 -a Pa_ref.fa hybrid_contigs.fa --secondary=no > hybrid_contigs_mapped_to_ref_genome.sam
````

Drag this sam file into Geneious and check it out.  

- Assuming the reference genome represents the actual complete genome, is this a complete assembly?
- What is the story with the very small contigs? (NODE_3_...)  
- What about unmapped contigs?

You can see that the fact that this is a circular genome raises some issues, as depicted in [this figure](https://media.springernature.com/full/springer-static/image/art%3A10.1186%2Fs13059-015-0849-0/MediaObjects/13059_2015_849_Fig1_HTML.gif).  Using a tool like [circulator](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-015-0849-0) could perhaps help resolve the end sequences to finalize the assembly.


### Additional, time permitting exercises

One thing that we didn't do here is map the raw data to the assembly.  You could, for instance, map the illumina reads to the reference genome (or the hybrid assembly contigs) using bowtie2.  You'll have to create an index first, as we did earlier.

You could also use minimap2 to map the nanopore reads to the reference genome.  Note that minimap2 doesn't make you pre-make an index. You just specify the reference's fasta file and it implicitly does it for you.  The syntax for that command would be:

```
minimap2 -a -x map-ont Pa_ref.fa nanopore_reads.fastq > nanopore_reads_mapped_to_ref.sam
```

Then you could drag the .sam files into geneious to inspect mapped reads.

You could also use fastqc to check out the quality and length of these nanopore reads.



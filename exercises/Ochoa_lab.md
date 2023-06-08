# Population Genomics Lab

### Young scientists!  The order of the universe has been altered by a mysterious virus attacking populations of Wookies, an intergalactic endangered species.

![](https://ik.imagekit.io/robynlovescake/wp-content/uploads/2019/09/278AD2CF-1344-449C-BE81-5D0BA90628B7-915x1024.jpeg)

### Your mission: Obtain different population genomics metrics from the three surviving populations of Wookies (and a population of a closely related species) to determine the demographic/evolutionary causes enabling the proliferation of this mysterious virus.
### Throughout these exercises, we will be using the following data as input:
- all.vcf - a VCF file containing ~53,000 SNPs across the three populations of Wookies (defined as P1, P2, P3) and one population from a closely related species (defined as OUT).

- all.unlinked.vcf - a VCF file containing ~7,000 unlinked SNPs across the three populations of Wookies (defined as P1, P2, P3) and one population from a closely related species (defined as OUT).

- pops.unlinked.vcf - a VCF file containing ~1,100 SNPs across the three populations of Wookies (defined as P1, P2, P3).

- pops.unlinked.beagle.gz - a compressed file containing genotype likelihoods from ~1,100 unlinked SNPs across the three populations of Wookies (very convenient for low coverage data).

- all.K1_5.replicates.log - a file containing log likelihood data from *K* = 1-5 replicates.

### Also, we will be using different programs for the exercises:
- VCFtools: [https://vcftools.github.io/man_latest.html](https://vcftools.github.io/man_latest.html)

- PLINK: [https://www.cog-genomics.org/plink/2.0/](https://www.cog-genomics.org/plink/2.0/)

- NGSadmix: [http://www.popgen.dk/software/index.php/NgsAdmix](http://www.popgen.dk/software/index.php/NgsAdmix)

# Part 1


## 01. Observed Heterozygosity

#### Move to the working directory:
```bash
cd ~/Desktop/GDW_Data/Alex/popgen/01.HE
```

#### Use VCFtools and the *all.vcf* file to count the number of HOMOZYGOUS genotypes per sample: 

```bash
vcftools --vcf all.vcf --het --out all
```

#### This process created a tab-delimited file called *all.het*, which can be visualized by typing:
```bash
cat all.het
```


#### The *all.het* file contains five columns defined by a header (the first line of the file).  For simplicity, we will focus on three columns:
- Column 1 - sample labels
- Column 2 - counts of observed HOMOZYGOUS genotypes per sample
- Column 4 - total counts of genotypes (homozygous and heterozygous) per sample

#### Next, we will add an "observed heterozygosity" column to the header by creating a new file called *header.het*: 
```bash
grep "INDV" all.het | sed 's/F/F\tO(HET)/g' > header.het
```

#### We will then create a file called *obsHet.txt*, which will contain individual "observed heterozygosity" metrics on the last column (Column 6):
```bash
grep -v "INDV" all.het | awk -v OFS='\t' '{ print $1, $2, $3, $4, $5, 1-($2/$4) }' | cat header.het - > obsHet.txt
```
#### You can visualize the *obsHet.txt* file by typing:
```bash
cat obsHet.txt
```
#### How is the *obsHet.txt* file different from the *all.het* file?
#### Finally, we will obtain "mean observed heterozygosity" metrics by averaging individual "observed heterozygosity" metrics from each population: 
```bash
grep "OUT" obsHet.txt | awk '{ total += $6 } END { print total/NR }' > OUT.obsHet.mean.txt
grep "P1" obsHet.txt | awk '{ total += $6 } END { print total/NR }' > P1.obsHet.mean.txt
grep "P2" obsHet.txt | awk '{ total += $6 } END { print total/NR }' > P2.obsHet.mean.txt
grep "P3" obsHet.txt | awk '{ total += $6 } END { print total/NR }' > P3.obsHet.mean.txt
```
#### This process created four files (one from each population) with *.obsHet.mean.txt extensions. Type the following command line to visualize "mean observed heterozygosity" values from each population:
```bash
head *.obsHet.mean.txt
```
#### Which populations would you say have high/low genetic diversities?  On what basis?

## 02. Tajima's *D*

#### Move to the working directory:

```bash
cd ~/Desktop/GDW_Data/Alex/popgen/02.TajimaD
```
#### Next, we will create four population vcf files by typing:
```bash
cut -f 1-9,10-14 all.vcf > OUT.vcf
cut -f 1-9,15-19 all.vcf > P1.vcf
cut -f 1-9,20-24 all.vcf > P2.vcf
cut -f 1-9,25-29 all.vcf > P3.vcf
```
#### Using VCFtools, we will now obtain *D* statistics across 100-Kb windows for each population:
```bash
vcftools --vcf OUT.vcf --TajimaD 100000 --out OUT
vcftools --vcf P1.vcf --TajimaD 100000 --out P1
vcftools --vcf P2.vcf --TajimaD 100000 --out P2
vcftools --vcf P3.vcf --TajimaD 100000 --out P3
```
#### This process created four files with extensions **.Tajima.D*, which contain Tajima's *D* metrics across 100-Kb windows per population.  These files, however, need to be sorted after creating a new header and removing the missing data ("nan"): 
```bash
echo -e "CHROM\tBIN_START\tN_SNPS\tTajimaD" > header.Tajima.D
grep -v "CHROM\|nan" OUT.Tajima.D | sort -g -k 4,4 | cat header.Tajima.D - > OUT.Tajima.D.sorted.txt
grep -v "CHROM\|nan" P1.Tajima.D | sort -g -k 4,4 | cat header.Tajima.D - > P1.Tajima.D.sorted.txt
grep -v "CHROM\|nan" P2.Tajima.D | sort -g -k 4,4 | cat header.Tajima.D - > P2.Tajima.D.sorted.txt
grep -v "CHROM\|nan" P3.Tajima.D | sort -g -k 4,4 | cat header.Tajima.D - > P3.Tajima.D.sorted.txt 
```

#### Next, we will obtain "mean *D* statistics" across 100-Kb windows per population:
```bash
grep -v "CHROM" OUT.Tajima.D.sorted.txt | awk '{ total += $4 } END { print total/NR }' > OUT.Tajima.D.mean.txt
grep -v "CHROM" P1.Tajima.D.sorted.txt | awk '{ total += $4 } END { print total/NR }' > P1.Tajima.D.mean.txt
grep -v "CHROM" P2.Tajima.D.sorted.txt | awk '{ total += $4 } END { print total/NR }' > P2.Tajima.D.mean.txt
grep -v "CHROM" P3.Tajima.D.sorted.txt | awk '{ total += $4 } END { print total/NR }' > P3.Tajima.D.mean.txt
```
#### We can now visualize "mean *D* statistics" per population by typing:
```bash
head *.Tajima.D.mean.txt
```
#### Which populations would you argue are expanding/contracting?  Which populations have remained constant in size?

#### For simplicity, we will only charaterize the "top" and "bottom" three *D*-statistic outliers per population by typing:
```bash
head -4 OUT.Tajima.D.sorted.txt > OUT.Tajima.D.outliers.txt
tail -3 OUT.Tajima.D.sorted.txt >> OUT.Tajima.D.outliers.txt
head -4 P1.Tajima.D.sorted.txt > P1.Tajima.D.outliers.txt
tail -3 P1.Tajima.D.sorted.txt >> P1.Tajima.D.outliers.txt
head -4 P2.Tajima.D.sorted.txt > P2.Tajima.D.outliers.txt
tail -3 P2.Tajima.D.sorted.txt >> P2.Tajima.D.outliers.txt
head -4 P3.Tajima.D.sorted.txt > P3.Tajima.D.outliers.txt
tail -3 P3.Tajima.D.sorted.txt >> P3.Tajima.D.outliers.txt
```
#### Results from the previous step can be visualized by typing:
```bash
head *.Tajima.D.outliers.txt
```
#### What do these results mean?  What similarities/dissimilarities across populations can you find?


## 03. Runs of Homozygosity (ROHs) 

#### Move to the working directory:

```bash
cd ~/Desktop/GDW_Data/Alex/popgen/03.ROHs
```
#### Run the following command line in *PLINK* to define ROHs from each sample from the *all.vcf* file:
```bash
plink2 --vcf all.vcf --make-bed --out out_name --no-sex --no-parents --no-fid --no-pheno --allow-extra-chr --double-id
plink2 -bfile out_name --homozyg --homozyg-window-snp 20 --homozyg-snp 20 --homozyg-window-missing 1 --homozyg-window-threshold 0.01 --allow-extra-chr
```
#### Based on the parameters used, we are defining ROHs as genomic tracts with a minimum length of 20 consecutive homozygous SNPs.  Fore more information on these parameters you can go to [https://www.cog-genomics.org/plink/1.9/ibd]()
#### You can give the resulting file a quick scan by typing:
```bash
head plink.hom
tail plink.hom
```
#### What does this file mean?

#### We will now convert the *plink.hom* file to a tab-delimited one (*plink.hom.tab*): 
```bash
sed -e 's/\s\{1,\}/\t/g' plink.hom > plink.hom.tab
```
#### From this newly created file (*plink.hom.tab*), we will now capture ROH tracts from each sample by typing:
```bash
grep "OUT_1" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > OUT_1.FROH.txt
grep "OUT_2" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > OUT_2.FROH.txt
grep "OUT_3" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > OUT_3.FROH.txt
grep "OUT_4" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > OUT_4.FROH.txt
grep "OUT_5" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > OUT_5.FROH.txt
grep "P1_1" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P1_1.FROH.txt
grep "P1_2" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P1_2.FROH.txt
grep "P1_3" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P1_3.FROH.txt
grep "P1_4" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P1_4.FROH.txt
grep "P1_5" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P1_5.FROH.txt
grep "P2_1" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P2_1.FROH.txt
grep "P2_2" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P2_2.FROH.txt
grep "P2_3" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P2_3.FROH.txt
grep "P2_4" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P2_4.FROH.txt
grep "P2_5" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P2_5.FROH.txt
grep "P3_1" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P3_1.FROH.txt
grep "P3_2" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P3_2.FROH.txt
grep "P3_3" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P3_3.FROH.txt
grep "P3_4" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P3_4.FROH.txt
grep "P3_5" plink.hom.tab | awk '{ total += $10 } END { print total/169609.841 }' > P3_5.FROH.txt
```
#### Phew!!!  Command line got a bit bulky there, but we created 20 different files (all with extensions **.FROH.txt*).  Give these files a quick inspection by typing:
```bash 
head *.FROH.txt
```
#### What information is contained in these files?

#### We will now obtainin "mean FROH metrics" per population by typing:
```bash
cat OUT_*.FROH.txt | awk '{ total += $1 } END { print total/NR }' > OUT.FROH.mean.txt
cat P1_*.FROH.txt | awk '{ total += $1 } END { print total/NR }' > P1.FROH.mean.txt
cat P2_*.FROH.txt | awk '{ total += $1 } END { print total/NR }' > P2.FROH.mean.txt
cat P3_*.FROH.txt | awk '{ total += $1 } END { print total/NR }' > P3.FROH.mean.txt
```
#### Final results can be visualized by typing:
```bash
head *.FROH.mean.txt
```
#### Which populations would you argue are highly inbred?
#### How are these results (i.e., observed heterozygosity, Tajima's *D*, FROH) relevant to WILDLIFE DISEASE?


# Part 2
## 04. Principal Component Analysis (PCA)
#### Move to the working directory:

```bash
cd ~/Desktop/GDW_Data/Alex/popgen/04.PCA
```

#### Use *PLINK* and the *all.unlinked.vcf* file to produce the data (eigenvectors and eigenvalues) to build a PCA plot:
```bash
plink2 --vcf all.unlinked.vcf --allow-extra-chr 0 --double-id --pca var-wts --out all.unlinked
```
#### Give the resulting files a quick scan by typing:
```bash
cat all.unlinked.eigenval
cat all.unlinked.eigenvec
```
#### What do these files mean?
#### Next, we will obtain the eigenvalue data from the two main components by typing:
```bash
awk 'NR > max { max=NR } { tot+=$1; v[NR]=$1; d[NR]=$2 } END { for (i=1; i<=max; i++) { print v[i]*100/tot } }' all.unlinked.eigenval | head -2 > all.unlinked.eigenvalue.prcomp
cat all.unlinked.eigenvalue.prcomp
```
#### We will now retrieve the eigenvector data from the two main components by typing:
```bash
sed 's/ /\t/g' all.unlinked.eigenvec | cut -f 2-4 > all.unlinked.eigenvector.prcomp
cat all.unlinked.eigenvector.prcomp
```
#### Within the "04.PCA" directory, open the Excel file called *PCA.xlsx* and plot results using the "all.unlinked.eigenvector.prcomp" sheet as a template.  Accomodate the eigenvector data from the *all.unlinked.eigenvector.prcomp* file accordingly.  Also, don't forget to add the eigenvalue data from the *all.unlinked.eigenvalue.prcomp* file to each axis.  Side note: the eigenvalue data needs to be converted into a percentage by multiplying each entry by 100.

#### Is there any genomic structure across samples?  And if so, at what scale (e.g., species, population)?


#### In order to gain more resolution into the distribution of genetic variation between populations from the species of interest, we will repeat the previous procedure with the *pops.unlinked.vcf* file:
```bash
plink2 --vcf pops.unlinked.vcf --allow-extra-chr 0 --double-id --pca var-wts --out pops.unlinked
awk 'NR > max { max=NR } { tot+=$1; v[NR]=$1; d[NR]=$2 } END { for (i=1; i<=max; i++) { print v[i]*100/tot } }' pops.unlinked.eigenval | head -2 > pops.unlinked.eigenvalue.prcomp
cat pops.unlinked.eigenvalue.prcomp
sed 's/ /\t/g' pops.unlinked.eigenvec | cut -f 2-4 > pops.unlinked.eigenvector.prcomp
cat pops.unlinked.eigenvector.prcomp
```
#### As done previously to the *PCA.xlsx* file, plot results using the "pops.unlinked.eigenvector.prcomp" sheet as a template. Accomodate the eigenvector data from the *pops.unlinked.eigenvector.prcomp* file accordingly. Also, don't forget to add the eigenvalue data from the *pops.unlinked.eigenvalue.prcomp* file to each axis. Side note: the eigenvalue data needs to be converted into a percentage by multiplying each entry by 100.

#### What type of genomic structure resolution are we now obtaining from this data?


## 05. Ancestry and Admixture (NGSadmix)
#### Move to the working directory:

```bash
cd ~/Desktop/GDW_Data/Alex/popgen/05.NGSadmix
```
#### Run NGSadmix assuming two, three, and four genomic groups (*K* = 2, 3, 4) and visualize results by applying the following command line:
```bash
~/Desktop/GDWApps/NgsAdmix/NGSadmix -likes pops.unlinked.beagle.gz -seed 12345 -K 2 -P 4 -o K2
paste labels.txt K2.qopt | sed 's/ /\t/g' > K2.qopt.txt
cat K2.qopt.txt
```
```bash
~/Desktop/GDWApps/NgsAdmix/NGSadmix -likes pops.unlinked.beagle.gz -seed 12345 -K 3 -P 4 -o K3
paste labels.txt K3.qopt | sed 's/ /\t/g' > K3.qopt.txt
cat K3.qopt.txt
```
```bash
~/Desktop/GDWApps/NgsAdmix/NGSadmix -likes pops.unlinked.beagle.gz -seed 12345 -K 4 -P 4 -o K4
paste labels.txt K4.qopt | sed 's/ /\t/g' > K4.qopt.txt
cat K4.qopt.txt
```
#### Open the Excel file called *NGSadmix.xlsx* and plot results in their corresponding Excel sheet (i.e., K2.qopt.txt, K3.qopt.txt, K4.qopt.txt).
#### How many genomic groups (*K*) are best explained by the data?  Two, three, four?
#### All right, all right ... the last question was a tricky one.  In reality, we need to create multiple replicates from each assumed *K* in order to find the optimal number of genomic groups (*K*) defined by the data.
#### In this regard, the *all.K1**_**5.replicates.log* file contains the log likelihood values from five replicates assuming *K* = 1-5.
#### Open the *all.K1**_**5.replicates.log* file and paste the data on the Excel sheet called "all.K1*_*5.replicates.log".
#### According to the Evanno et al. method ([https://onlinelibrary.wiley.com/doi/full/10.1111/j.1365-294X.2005.02553.x](https://onlinelibrary.wiley.com/doi/full/10.1111/j.1365-294X.2005.02553.x)), the *K*-value (*x*-axis) corresponding to the greatest Delta *K* value (*y*-axis) best explains the number of genomic clusters from the data.
#### What's the optimal *K* according to this method?  Are populations genetically isolated?



## 06. FST-outlier Test
#### Move to the working directory:
```bash
cd ~/Desktop/GDW_Data/Alex/popgen/06.FST_outlier
```
#### Using VCFtools and the *pops.unlinked.vcf* file, obtain FST statistics across loci and populations:
```bash
vcftools --vcf pops.unlinked.vcf --weir-fst-pop P1.txt --weir-fst-pop P2.txt --weir-fst-pop P3.txt
```
#### We will now create a new header, sort the resulting file (out.weir.fst), and output into a file called *pops.FST.sorted* by typing:
```bash
echo -e "CHROM\tPOS\tWEIR_AND_COCKERHAM_FST" > header.FST
grep -v "CHROM\|nan" out.weir.fst | sort -r -g -k 3,3 | cat header.FST - > pops.FST.sorted
```
#### From the *pops.FST.sorted* file, now apply the following command line to output FST values belonging to the top 5th percentile:
```bash
head -54 pops.FST.sorted > pops.FST.sorted.outliers
cat pops.FST.sorted.outliers
```
#### What does this data mean and how would you improve the analysis to detect genes under selection?
#### How are patterns of genomic structure and selection related to WILDLIFE DISEASE?

# Part 3
## 07. Mutation Load
#### Go to the SIFT site ([https://sift.bii.a-star.edu.sg/]()) and click on the "SIFT sequence" link.
#### Next, copy/paste the following AMINO ACID sequence in the "Paste in your protein query sequence" entry:
```
>unknown
MPLGEYTLVRVVGKGSYGEVSLVRHRHDGKQYVIKRLNLKHASSRERKAAEQEAQLLSQLKHPNIVTYRESWEGDDGLLYIVMGFCEGGDLYHKLKEQKGQLLPESQVVEWLVQIAMALQYLHEKHILHRDLKTQNVFLTRSNIIKVGDLGIARVLENQHDMASTLIGTPYYMSPELFSNKPYSYKSDVWALGCCAYEMTTLKHAFNAKDMNSLVYRIIEGKLPPMPKDYSIQLKELIRIMLSKKPEERPSVRSILRQPYIKQQISLFLEATKAKTSKNHKKISEPNPRGSSEKAELKKENTVPQKLPDELSKNSHLNEDKCLINNKTFEVSSSKKLAPEPEKNSKNGLNSLDVSLATMSKVDILIVPLEKRNSATKQSAEHKESRNVHIPNEAEPERAKNSSELKEESLLKMTTPCSKTGSVEAKETDNGSVQEPSLSQQRRHKKESEILNKIAAFGHRQLPAYPDVNVKTKEVNTDHQRAANVLKKSKSFEVEHSKDRPLSARERRRLKQSQEDMLSSDPSVRRMSYNASTETKLPREDRCIQIAQFTSEPTTDKKKSLTGCCTEDDLSSSTSSTEKSDGDYKERKSNEMNDLLQLMTQTLKMDNIENYSQFAMSTPDSQFRLNRKYRDTLILHGKTAEEPDEFQFQEFPSDDKSGPQNIRRLVEILRADVVQGIGVKLLEKVYDAMEEEDEQKRELCLRKLMGEKYTSYSMKARHLKFLEDNVKL
```
#### Once finished, copy the following information on the "Enter the substitutions of interest" entry:
```
V9M
T140M
```
#### Click on the "Submit" button and enjoy your coffee!
#### Ok, the SIFT online platform has now had a chance to produce results.  Click on the "Your results will be at this link" link.  Then click on the "Predictions of substitutions entered" link to visualize your results. 
#### What can we say about these AMINO ACID mutations?
#### It just so happens that the V9M amino acid mutation corresponds to a G -> A DNA mutation at locus "jcf7180000039029*_*1840688," whereas the T140M amino acid mutation corresponds to a C -> T DNA mutation at locus "jcf7180000039029*_*1838320" from our original *all.vcf* file.
#### To this extent, apply the following command line to capture the loci of interest:
```bash
grep "CHROM\|jcf7180000039029_1838320\|jcf7180000039029_1840688" all.vcf > loci.txt
```
#### Open the *loci.txt* file in BBEdit.  What can we say about these loci?  With this one example in hand, what do you think the genome-wide mutation load pattern may look like across populations? 
#### Finally, perform a BLASTp search ([https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE=Proteins](https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE=Proteins)) of the protein in question to find its description.
#### Look closely at the description of the protein and search for a gene code (usually represented by a few characters).
#### Finally, go to the GeneCards site ([https://www.genecards.org/]()), introduce this gene code in the "Explore a gene" search bar, and click on the "search" button.
#### What can we say about this gene and how is it related to WILDLIFE DISEASE?
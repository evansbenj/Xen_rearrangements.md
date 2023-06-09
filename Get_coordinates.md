# Get coordinates from XB genome that match XL CDS that are greater than 200:

Working in this directory on graham:
```
/home/ben/projects/rrg-ben/for_martin
```

First extract exons from trop and laevis longest gff file:
```
grep 'CDS  ' XENTR_10.0_Xenbase_longest.gff3 > XENTR_10.0_Xenbase_longest_CDSonly.gff3
grep 'CDS  ' XENLA_10.1_Xenbase_longest.gff3 > XENLA_10.1_Xenbase_longest_CDSonly.gff
```
Now make a new bed file that also has the name of each exon in it:
```
cut -f1,4,5,9 XENTR_10.0_Xenbase_longest_CDSonly.gff3 > XENTR_10.0_Xenbase_longest_CDSonly_names.bed
cut -f1,4,5,9 XENLA_10.1_Xenbase_longest_CDSonly.gff > XENLA_10.1_Xenbase_longest_CDSonly_names.bed
```

Remove CDS that are less than 200 bp for XL
```
awk '{ $5 = $3 - $2 } 1' < XENLA_10.1_Xenbase_longest_CDSonly_names.bed > XENLA_10.1_Xenbase_longest_CDSonly_names_diff.bed
awk '$5 >= 200' XENLA_10.1_Xenbase_longest_CDSonly_names_diff.bed > XENLA_10.1_Xenbase_longest_CDSonly_names_diff_gt200.bed
```
and XT:
```
awk '{ $5 = $3 - $2 } 1' < XENTR_10.0_Xenbase_longest_CDSonly_names.bed > XENTR_10.0_Xenbase_longest_CDSonly_names_diff.bed
awk '$5 >= 200' XENTR_10.0_Xenbase_longest_CDSonly_names_diff.bed > XENTR_10.0_Xenbase_longest_CDSonly_names_diff_gt200.bed
```
Now replace the spaces that awk added with tabs so that bedtools can read it for XL:
```
awk -v OFS="\t" '{$1=$1; print}' XENLA_10.1_Xenbase_longest_CDSonly_names_diff_gt200.bed > XENLA_10.1_Xenbase_longest_CDSonly_names_diff_gt200tab.bed
```
and XT:
```
awk -v OFS="\t" '{$1=$1; print}' XENTR_10.0_Xenbase_longest_CDSonly_names_diff_gt200.bed > XENTR_10.0_Xenbase_longest_CDSonly_names_diff_gt200tab.bed
```
Now cut the first four columns for XL
```
cut -f1,2,3,4 XENLA_10.1_Xenbase_longest_CDSonly_names_diff_gt200tab.bed > XENLA_10.1_Xenbase_longest_CDSonly_names_diff_gt200tab_final.bed
```
and XT:
```
cut -f1,2,3,4 XENTR_10.0_Xenbase_longest_CDSonly_names_diff_gt200tab.bed > XENTR_10.0_Xenbase_longest_CDSonly_names_diff_gt200tab_final.bed

```
Now use the XL bed to extract fasta seqs for each exon from the XL genome:
```
module load bedtools
bedtools getfasta -name -fi ../2021_XL_v10_refgenome/XENLA_10.1_genome.fa -bed XENLA_10.1_Xenbase_longest_CDSonly_names_diff_gt200tab_final.bed -fo XENLA_10.1_Xenbase_longest_CDSonly_names_gt200.fasta
```
And now use the XT bed to extract fasta seqs for each exon from the XT genome:
```
module load bedtools
bedtools getfasta -name -fi ../2020_XT_v10_refgenome/XENTR_10.0_genome.fasta -bed XENTR_10.0_Xenbase_longest_CDSonly_names_diff_gt200tab_final.bed -fo XENTR_10.0_Xenbase_longest_CDSonly_names_gt200.fasta
```

Get best alignment between XL CDS and XB genome using blast (based on bit score)
```
module load nixpkgs/16.09 gcc/7.3.0 'blast+/2.10.1' 
blastn -query XENLA_10.1_Xenbase_longest_CDSonly_names_gt200.fasta -db ../XB_genome_concat_scafs/Xbo.v1_chrs_and_concatscafs_blastable -outfmt 6 | sort -k1,1 -k12,12nr -k11,11n | sort -u -k1,1 --merge > XLlongCDS_to_XBgenome_bestbitscore.blastn
```
Use sed to replace double colon with a tab so that the XL coordinates are in a separate column (Importnat: you need to insert a tab using "Ctrl-V tab" in the command below before the '/g' part. It will not work if you just copy and paste this command):
```
sed -i 's/\:\:/    /g' XLlongCDS_to_XBgenome_bestbitscore.blastn
```
Now get this column plus the borealis coordinates, plus the direction info
```
cut -f1,2,3,10,11 XLlongCDS_to_XBgenome_bestbitscore.blastn > XLlongCDS_to_XBgenome.txt
```
*** the XLlongCDS_to_XBgenome.txt file has the coordinates for each XL CDS gt 200 bp and the XB genome and also the XL annotation information

# Get coordinates from XL L and S subgenoms that match XT CDS gt 200 bp

First generate a XL genome with only the L (or S) subgenome
```
module load bedtools
bedtools getfasta -fi XENLA_10.1_genome.fa -bed XL_Lsubgenome.bed -fo XENLA_10.1_genome_Lsubgenomeonly.fa
bedtools getfasta -fi XENLA_10.1_genome.fa -bed XL_Ssubgenome.bed -fo XENLA_10.1_genome_Ssubgenomeonly.fa
```
using these bed files:
```
Chr1L	1	233740091
Chr2L	1	191000147
Chr3L	1	161426102
Chr4L	1	155250555
Chr5L	1	171415385
Chr6L	1	164223596
Chr7L	1	139837619
Chr8L	1	135449134
Chr9_10L	1	137811820
```
and
```
Chr1S	1	202412971
Chr2S	1	169306101
Chr3S	1	131962817
Chr4S	1	132731175
Chr5S	1	143394104
Chr6S	1	137316287
Chr7S	1	113060390
Chr8S	1	103977863
Chr9_10S	1	117266292
```
Change names in subgenome files:
```
sed -i 's/Chr1L\:1\-233740091/Chr1L/' XENLA_10.1_genome_Lsubgenomeonly.fa
sed -i 's/Chr2L\:1\-191000147/Chr2L/' XENLA_10.1_genome_Lsubgenomeonly.fa
sed -i 's/Chr3L\:1\-161426102/Chr3L/' XENLA_10.1_genome_Lsubgenomeonly.fa
sed -i 's/Chr4L\:1\-155250555/Chr4L/' XENLA_10.1_genome_Lsubgenomeonly.fa
sed -i 's/Chr5L\:1\-171415385/Chr5L/' XENLA_10.1_genome_Lsubgenomeonly.fa
sed -i 's/Chr6L\:1\-164223596/Chr6L/' XENLA_10.1_genome_Lsubgenomeonly.fa
sed -i 's/Chr7L\:1\-139837619/Chr7L/' XENLA_10.1_genome_Lsubgenomeonly.fa
sed -i 's/Chr8L\:1\-135449134/Chr8L/' XENLA_10.1_genome_Lsubgenomeonly.fa
sed -i 's/Chr9_10L\:1\-137811820/Chr9_10L/' XENLA_10.1_genome_Lsubgenomeonly.fa
sed -i 's/Chr1S\:1\-202412971/Chr1S/' XENLA_10.1_genome_Ssubgenomeonly.fa
sed -i 's/Chr2S\:1\-169306101/Chr2S/' XENLA_10.1_genome_Ssubgenomeonly.fa
sed -i 's/Chr3S\:1\-131962817/Chr3S/' XENLA_10.1_genome_Ssubgenomeonly.fa
sed -i 's/Chr4S\:1\-132731175/Chr4S/' XENLA_10.1_genome_Ssubgenomeonly.fa
sed -i 's/Chr5S\:1\-143394104/Chr5S/' XENLA_10.1_genome_Ssubgenomeonly.fa
sed -i 's/Chr6S\:1\-137316287/Chr6S/' XENLA_10.1_genome_Ssubgenomeonly.fa
sed -i 's/Chr7S\:1\-113060390/Chr7S/' XENLA_10.1_genome_Ssubgenomeonly.fa
sed -i 's/Chr8S\:1\-103977863/Chr8S/' XENLA_10.1_genome_Ssubgenomeonly.fa
sed -i 's/Chr9_10S\:1\-117266292/Chr9_10S/' XENLA_10.1_genome_Ssubgenomeonly.fa
```
Check to make sure they are ok:
```
grep '>' XENLA_10.1_genome_Lsubgenomeonly.fa
grep '>' XENLA_10.1_genome_Ssubgenomeonly.fa
```


Make a blast db for each subgenome.  This required a sbatch script:
```
#!/bin/sh 
#SBATCH --job-name=blast
#SBATCH --nodes=1 
#SBATCH --cpus-per-task=1
#SBATCH --time=8:00:00 
#SBATCH --mem=64gb
#SBATCH --output=blast.%J.out
#SBATCH --error=blast.%J.err
#SBATCH --account=def-ben

module load nixpkgs/16.09 gcc/7.3.0 blast+/2.10.1
makeblastdb -in XENLA_10.1_genome_Lsubgenomeonly.fa -dbtype nucl -out XENLA_10.1_genome_Lsubgen
omeonly_blastable

makeblastdb -in XENLA_10.1_genome_Ssubgenomeonly.fa -dbtype nucl -out XENLA_10.1_genome_Ssubgen
omeonly_blastable
```
If the script is called "2023_makeblastdb.sh" then it can be executed like this: "sbatch 2023_makeblastdb.sh"



Get best alignment between XT CDS and XL L-subgenome using blast (based on bit score)
```
module load nixpkgs/16.09 gcc/7.3.0 'blast+/2.10.1' 
blastn -query XENTR_10.0_Xenbase_longest_CDSonly_names_gt200.fasta -db ../2021_XL_v10_refgenome/XENLA_10.1_genome_Lsubgenomeonly_blastable -outfmt 6 | sort -k1,1 -k12,12nr -k11,11n | sort -u -k1,1 --merge > XTlongCDS_to_XL_Lsubgenome_bestbitscore.blastn
```
Get best alignment between XT CDS and XL S-subgenome using blast (based on bit score)
```
module load nixpkgs/16.09 gcc/7.3.0 'blast+/2.10.1' 
blastn -query XENTR_10.0_Xenbase_longest_CDSonly_names_gt200.fasta -db ../2021_XL_v10_refgenome/XENLA_10.1_genome_Ssubgenomeonly_blastable -outfmt 6 | sort -k1,1 -k12,12nr -k11,11n | sort -u -k1,1 --merge > XTlongCDS_to_XL_Ssubgenome_bestbitscore.blastn
```

Use sed to replace double colon with a tab so that the XT coordinates are in a separate column (Importnat: you need to insert a tab using "Ctrl-V tab" in the command below before the '/g' part. It will not work if you just copy and paste this command):
```
sed -i 's/\:\:/    /g' XTlongCDS_to_XL_Lsubgenome_bestbitscore.blastn
sed -i 's/\:\:/    /g' XTlongCDS_to_XL_Ssubgenome_bestbitscore.blastn
```
Now get this column plus the XL coordinates
```
cut -f1,2,3,10,11 XTlongCDS_to_XL_Lsubgenome_bestbitscore.blastn > XTlongCDS_to_XL_Lgenome.txt
cut -f1,2,3,10,11 XTlongCDS_to_XL_Ssubgenome_bestbitscore.blastn > XTlongCDS_to_XL_Sgenome.txt
```
*** the XTlongCDS_to_XL_Lgenome.txt and the XTlongCDS_to_XL_Sgenome.txt files have the coordinates for each XT CDS gt 200 bp and each XL subgenome and also the XT annotation information



# Extracting XB L and S subgenomes
bed file for L subgenome:
```
Chr1L   1       232529967
Chr2L   1       184566229
Chr3L   1       145564449
Chr4L   1       156120765
Chr5L   1       174499024
Chr6L   1       157843502
Chr7L   1       136892544
Chr8L   1       123836259
Chr9_10L        1       135078614
```
bed file for S subgenome:
```
Chr1S   1	196169796
Chr2S   1	167897111
Chr3S   1	127416162
Chr4S   1	131359388
Chr5S   1	139053354
Chr6S   1	137668413
Chr7S   1	105895006
Chr8S   1	105436522
Chr9_10S        1	110702964
```


### below not used



* for XL
```
grep 'gprin3\|ugt8\|pitx2\|metap1\|ccrn4l\|spry1\|smad1\|ednra\|hand2\|kit\|fgfr3\|wdr1\|ncbp1\|midn\|fkbp8\|tpm4\|cer1\|rps6\|bcl2l2\|dmrt1\|gna14\|ntrk2\|zmat5\|med13l\|mlec\|gltp\|fzd10\|prrc1\|noc4l\|riok2\|papd4\|nipbl\|smad7\|rax\|ctse\|rcc1\|phactr4\|aipl1\|myo1c\|ift20\|traf4\|chmp2b\|ets2\|gabpa\|gap43\|igsf11\|myog\|gjb3\|ulk2\|tbx2\|lims1\|efnb2\|cblb\|nup88\|tmem194a\|cacnb3\|col2a1\|dctn2\|map3k12\|pgr\|krt18\|pds5b\|arrb1\|ppfibp1\|clint1\|ccdc69\|sap30l\|hnrnph1\|larp1\|hmp19\|tspan17\|prickle1\|usp44\|lgr5\|guca1a\|scamp5\|celf2\|flnc\|rab27a\|rasgrf1\|nodal6\|pin1\|znf703\|psmb6\|cdca5\|foxa4\|igf2\|syt12\|pax6\|depdc7\|accs\|myod1\|hsbp1\|coq9\|fa2h\|psma4\|lhx9\|npl\|gtf2b\|mcm5\|h1f0\|mgc75753\|gmppb\|atf4\|chchd4\|t\|fbxo5\|lgalsl\|rnf8\|mix1\|bmp2\|epcam\|rtn4\|aim1\|bach2\|tdrp\|ect2\|ssr3\|slc25a36\|epha4\|sox11\|mycn\|laptm4a\|pccb\|ubxn2a\|stk17a\|ctnnb1\|hoxa4\|meox2\|bmi1\|fzd8\|klf6\|znf622\|prpf4b\|rbm24\|sox4\|tshz1\|sox17a\|gata6\|oxr1\|mmp16\|matn2\|med30\|ptp4a3\|ndrg1\|cuedc2\|slc2a9\|zranb1\|cdk1\|bicc1\|pcdh15\|vax1\|btg4\|sdhd\|atad3a\|xilr2\|spib\|cacng6\|glul\|atp6ap1.2\|apln\|rlim\|vasp\|fam199x\|bag6\|irf2bpl\|flot1\|yif1b\|meis3\|gsc\|gtf2a1\|ttc7b\|bmp4\|adssl1\|foxa1\|slc39a9\|vangl2\|znf652\|krt\|sdc4\|chmp6\|psme3\|wdr16\|grb2\|rpl13a\|dapl1\|arpc1b\|gmppa\|cxcr4\|ssb\|ag1\|ndufa10\|ikzf2\|ccnyl1\|nop58\|nde1\|nubp1\|ern2' XENLA_10.1_GCF.gff3 | grep '   gene    ' > XL_v10_session_genes.txt
```

* for trop:
```
grep 'gprin3\|ugt8\|pitx2\|metap1\|ccrn4l\|spry1\|smad1\|ednra\|hand2\|kit\|fgfr3\|wdr1\|ncbp1\|midn\|fkbp8\|tpm4\|cer1\|rps6\|bcl2l2\|dmrt1\|gna14\|ntrk2\|zmat5\|med13l\|mlec\|gltp\|fzd10\|prrc1\|noc4l\|riok2\|papd4\|nipbl\|smad7\|rax\|ctse\|rcc1\|phactr4\|aipl1\|myo1c\|ift20\|traf4\|chmp2b\|ets2\|gabpa\|gap43\|igsf11\|myog\|gjb3\|ulk2\|tbx2\|lims1\|efnb2\|cblb\|nup88\|tmem194a\|cacnb3\|col2a1\|dctn2\|map3k12\|pgr\|krt18\|pds5b\|arrb1\|ppfibp1\|clint1\|ccdc69\|sap30l\|hnrnph1\|larp1\|hmp19\|tspan17\|prickle1\|usp44\|lgr5\|guca1a\|scamp5\|celf2\|flnc\|rab27a\|rasgrf1\|nodal6\|pin1\|znf703\|psmb6\|cdca5\|foxa4\|igf2\|syt12\|pax6\|depdc7\|accs\|myod1\|hsbp1\|coq9\|fa2h\|psma4\|lhx9\|npl\|gtf2b\|mcm5\|h1f0\|mgc75753\|gmppb\|atf4\|chchd4\|t\|fbxo5\|lgalsl\|rnf8\|mix1\|bmp2\|epcam\|rtn4\|aim1\|bach2\|tdrp\|ect2\|ssr3\|slc25a36\|epha4\|sox11\|mycn\|laptm4a\|pccb\|ubxn2a\|stk17a\|ctnnb1\|hoxa4\|meox2\|bmi1\|fzd8\|klf6\|znf622\|prpf4b\|rbm24\|sox4\|tshz1\|sox17a\|gata6\|oxr1\|mmp16\|matn2\|med30\|ptp4a3\|ndrg1\|cuedc2\|slc2a9\|zranb1\|cdk1\|bicc1\|pcdh15\|vax1\|btg4\|sdhd\|atad3a\|xilr2\|spib\|cacng6\|glul\|atp6ap1.2\|apln\|rlim\|vasp\|fam199x\|bag6\|irf2bpl\|flot1\|yif1b\|meis3\|gsc\|gtf2a1\|ttc7b\|bmp4\|adssl1\|foxa1\|slc39a9\|vangl2\|znf652\|krt\|sdc4\|chmp6\|psme3\|wdr16\|grb2\|rpl13a\|dapl1\|arpc1b\|gmppa\|cxcr4\|ssb\|ag1\|ndufa10\|ikzf2\|ccnyl1\|nop58\|nde1\|nubp1\|ern2' XENTR_10.0_Xenbase.gff3 | grep '       gene    ' > XT_v10_session_genes.txt
```

* for XB
Make multifasta file with X. trop seqs for all genes

Then blast this against the Xborealis genome:
```
blastn -query test.fa -db Xbo.v1_chrs_and_concatscafs_blastable -outfmt 6 -out test.out
```





# AC2021_NANOPORE_PCR10 mix1-3_81Q_prolif_c8_c30_c47

## Basecalling

```bash
cd /lustrehome/gianluca/strique/jobs/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

./submit_chronos_gianluca guppy-gianluca-job-basecalling.json
```

<!-- #### alignment reads "both_ends" with PCR fragment
```
cd /lustrehome/gianluca/strique/jobs/AC2021_NANOPORE_PCR10/21Q_45Q_C6_12_51_PLURIP_D0_30_60_90_120/
```
Barcode 13-21 81Q
```
for n in $(seq 13 21); do cp guppy-gianluca-job-align13.json guppy-gianluca-job-align${n}.json ; done

for n in $(seq 13 21); do sed -i -e "s/barcode13/barcode${n}/g" guppy-gianluca-job-align${n}.json ; done

for n in $(seq 13 21); do ./submit_chronos_gianluca guppy-gianluca-job-align${n}.json ; done
``` -->

## WHOLE GENOME alignment

> Barcode 13-24 81Q

```bash
for n in $(seq 13 24); do cp guppy-gianluca-job-WG-align13.json guppy-gianluca-job-WG-align${n}.json ; done

for n in $(seq 13 24); do sed -i -e "s/barcode13/barcode${n}/g" guppy-gianluca-job-WG-align${n}.json ; done

for n in $(seq 13 24); do ./submit_chronos_gianluca guppy-gianluca-job-WG-align${n}.json ; done
```

## Merge multiple BAM into one

Script location:

```bash
/lustrehome/gianluca/strique/jobs/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/bash-merge_multiBAM.sh
```

```bash
for n in $(seq 13 24) ; do bash bash-merge_multiBAM.sh /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/20220124_1530_MN29119_FAR76426_8a6ff7d4/analysis/alignment/barcode${n}_WG ; done
```

## FASTQ2FASTA

```bash
for n in $(seq 13 24); do zcat /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/20220124_1530_MN29119_FAR76426_8a6ff7d4/analysis/basecalling/pass/barcode${n}/*.fastq.gz | awk '{if(NR%4==1) {printf(">%s\n",substr($0,2));} else if(NR%4==2) print;}' > /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/20220124_1530_MN29119_FAR76426_8a6ff7d4/analysis/fastq2fasta/barcode${n}_allreads.fasta ; done
```

## BLASTN 81Qbarcode

```bash
source ~/.bashrc

mkdir /lustrehome/gianluca/straglr/blastn/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

for n in $(seq 13 24); do blastn -task blastn-short -query /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/20220124_1530_MN29119_FAR76426_8a6ff7d4/analysis/fastq2fasta/barcode${n}_allreads.fasta -subject /lustrehome/gianluca/straglr/barcode_fasta/barcode81Q.fa -outfmt 6 > /lustrehome/gianluca/straglr/blastn/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/barcode${n}_blastn_81Q.txt ; done
```

## BLASTN all CAG sizes (21Q, 45Q, 107Q) for barcode13-15

```bash
source ~/.bashrc

mkdir /lustrehome/gianluca/straglr/blastn/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

for c in 21Q 45Q 107Q ; do for n in $(seq 13 15); do blastn -task blastn-short -query /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/20220124_1530_MN29119_FAR76426_8a6ff7d4/analysis/fastq2fasta/barcode${n}_allreads.fasta -subject /lustrehome/gianluca/straglr/barcode_fasta/barcode${c}.fa -outfmt 6 > /lustrehome/gianluca/straglr/blastn/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/barcode${n}_blastn_${c}.txt ; done ; done 
```

## *straglr*, for CAG count

CL Merge 100 bam:

```bash
for n in $(seq 13 24) ; do bash bash-merge_100_BAM.sh /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/20220124_1530_MN29119_FAR76426_8a6ff7d4/analysis/alignment/barcode${n}_WG ; done
```

CL Job submission:

```bash
mkdir /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

cd /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/20220124_1530_MN29119_FAR76426_8a6ff7d4/analysis/alignment/

for n in $(seq 13 24) ; do ls barcode${n}_WG/*.multi.bam | while IFS="$(printf '/')" read -r f1 f2 ; do echo condor_submit -name ettore \
-a "out_dir=/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/" \
-a "bam=/lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/20220124_1530_MN29119_FAR76426_8a6ff7d4/analysis/alignment/barcode${n}_WG/${f2}" \
-a "ref=/lustre/home/enza/cattaneo/data/Methylation-RUES2-FLG/RUES2-20CAG/reference/GCA_000001405.15_GRCh38_no_alt_analysis_set.fna" \
-a "out_prefix=barcode${n}_${f2}" \
-a "bed=/lustrehome/gianluca/test_straglr/CAG.bed" \
-a "file=${f2}" \
-a "barcode=barcode${n}" \
/lustrehome/gianluca/straglr/jobs/condor-straglr_exp.job ; done ; done | less -S
```

## Merge multiple *straglr* output

```bash
cd /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

for n in $(seq 13 24); do ls barcode${n}_split*.tsv | grep aa | while read file ; do cat ${file} > barcode${n}.tsv; done ; done

for n in $(seq 13 24); do ls barcode${n}_split*.tsv | grep -v aa | while read file ; do cat ${file} | grep -v "#" >> barcode${n}.tsv; done ; done

#cancel all .ok files created for job-control
rm *.ok
```

## Create folder to save on local computer with blastn+straglr+alignment_summary

```bash
cd /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

for n in $(seq 13 24) ; do mkdir barcode${n} ; done

ALIGNMENT=/lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/20220124_1530_MN29119_FAR76426_8a6ff7d4/analysis/alignment

BLASTN=/lustrehome/gianluca/straglr/blastn/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

STRAGLR=/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

for n in $(seq 13 24) ; do cp ${ALIGNMENT}/barcode${n}_WG/alignment_summary.txt ${STRAGLR}/barcode${n}/ &&  cp ${BLASTN}/barcode${n}* ${STRAGLR}/barcode${n}/ &&  cp ${STRAGLR}/barcode${n}.tsv ${STRAGLR}/barcode${n}/ ; done 
```

## Rsync to local computer

```bash
mkdir /Users/gianlucadamaggio/projects/cattaneo/straglr/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/

for n in $(seq 13 24) ; do rsync -avh --progress gianluca@ui02.recas.ba.infn.it:/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/barcode${n} /Users/gianlucadamaggio/projects/cattaneo/straglr/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/ ; done
```

## Re-do Straglr - version CAG_v4.bed

CL Job submission:

```bash
mkdir /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

cd /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/20220124_1530_MN29119_FAR76426_8a6ff7d4/analysis/alignment/

for n in $(seq 13 24) ; do ls barcode${n}_WG/*.multi.bam | while IFS="$(printf '/')" read -r f1 f2 ; do echo condor_submit -name ettore \
-a "out_dir=/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/" \
-a "bam=/lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/20220124_1530_MN29119_FAR76426_8a6ff7d4/analysis/alignment/barcode${n}_WG/${f2}" \
-a "ref=/lustre/home/enza/cattaneo/data/Methylation-RUES2-FLG/RUES2-20CAG/reference/GCA_000001405.15_GRCh38_no_alt_analysis_set.fna" \
-a "out_prefix=barcode${n}_v4_${f2}" \
-a "bed=/lustrehome/gianluca/test_straglr/CAG_v4.bed" \
-a "file=${f2}" \
-a "barcode=barcode${n}" \
/lustrehome/gianluca/straglr/jobs/condor-straglr_exp.job ; done ; done | less -S
```

### Merge multiple *straglr* output - version CAG_v4.bed

```bash
cd /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

for n in $(seq 13 24); do ls barcode${n}_v4_split*.tsv | grep aa | while read file ; do cat ${file} > barcode${n}_v4.tsv; done ; done

for n in $(seq 13 24); do ls barcode${n}_v4_split*.tsv | grep -v aa | while read file ; do cat ${file} | grep -v "#" >> barcode${n}_v4.tsv; done ; done

#cancel all .ok files created for job-control
rm *.ok
```

### Create folder to save on local computer with blastn+straglr+alignment_summary - version CAG_v4.bed

```bash
cd /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

for n in $(seq 13 24); do mkdir barcode${n}_v4 ; done

STRAGLR=/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47

for n in $(seq 13 24); do cp ${STRAGLR}/barcode${n}_v4.tsv ${STRAGLR}/barcode${n}_v4/ ; done 
```

### Rsync to local computer - version CAG_v4.bed

```bash
mkdir /Users/gianlucadamaggio/projects/cattaneo/straglr/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/

for n in $(seq 13 24); do rsync -avh --progress gianluca@ui02.recas.ba.infn.it:/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/barcode${n}_v4 /Users/gianlucadamaggio/projects/cattaneo/straglr/AC2021_NANOPORE_PCR10/mix1-3_81Q_prolif_c8_c30_c47/ ; done
```
-->
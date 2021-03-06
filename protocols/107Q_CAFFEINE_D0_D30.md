# AC2021_NANOPORE_PCR10 107Q_CAFFEINE_D0_D30

## Basecalling

```bash
cd /lustrehome/gianluca/strique/jobs/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30

./submit_chronos_gianluca guppy-gianluca-job-basecalling.json
```

## STRAGLR

### WHOLE GENOME alignment

Add to json file for alignment vs the whole-genome.

```json
"constraints": [["hostname", "LIKE", "hpc-gpu-1-[2,4]-[1-2].recas.ba.infn.it"]],
```

> Barcode 05 08 09 10 17 18 19 20 21 22 23 24

```bash
for n in 05 08 09 10 17 18 19 20 21 22 23 24; do cp guppy-gianluca-job-WG-align05.json guppy-gianluca-job-WG-align${n}.json ; done

for n in 05 08 09 10 17 18 19 20 21 22 23 24; do sed -i -e "s/barcode05/barcode${n}/g" guppy-gianluca-job-WG-align${n}.json ; done

for n in 05 08 09 10 17 18 19 20 21 22 23 24; do ./submit_chronos_gianluca guppy-gianluca-job-WG-align${n}.json ; done
```

### FASTQ2FASTA

```bash
mkdir /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/20220307_1326_MN37986_FAR78200_cb0c17e2/analysis/fastq2fasta

for n in 05 08 09 10 17 18 19 20 21 22 23 24; do zcat /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/20220307_1326_MN37986_FAR78200_cb0c17e2/analysis/basecalling/pass/barcode${n}/*.fastq.gz | awk '{if(NR%4==1) {printf(">%s\n",substr($0,2));} else if(NR%4==2) print;}' > /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/20220307_1326_MN37986_FAR78200_cb0c17e2/analysis/fastq2fasta/barcode${n}_allreads.fasta ; done
```

### BLASTN 107Qbarcode

```bash
source ~/.bashrc

mkdir /lustrehome/gianluca/straglr/blastn/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30

for n in 05 08 09 10 17 18 19 20 21 22 23 24; do blastn -task blastn-short -query /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/20220307_1326_MN37986_FAR78200_cb0c17e2/analysis/fastq2fasta/barcode${n}_allreads.fasta -subject /lustrehome/gianluca/straglr/barcode_fasta/barcode107Q.fa -outfmt 6 > /lustrehome/gianluca/straglr/blastn/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/barcode${n}_blastn_107Q.txt ; done
```

### *straglr*, for CAG count

CL Merge 100 bam:

```bash
for n in 05 08 09 10 17 18 19 20 21 22 23 24 ; do bash ~/straglr/jobs/bash-merge_100_BAM.sh /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/20220307_1326_MN37986_FAR78200_cb0c17e2/analysis/alignment/barcode${n}_WG ; done
```

CL Job submission:

```bash
mkdir /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30

cd /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/20220307_1326_MN37986_FAR78200_cb0c17e2/analysis/alignment/

for n in 05 08 09 10 17 18 19 20 21 22 23 24 ; do ls barcode${n}_WG/*.multi.bam | while IFS="$(printf '/')" read -r f1 f2 ; do echo condor_submit -name ettore \
-a "out_dir=/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/" \
-a "bam=/lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/20220307_1326_MN37986_FAR78200_cb0c17e2/analysis/alignment/barcode${n}_WG/${f2}" \
-a "ref=/lustre/home/enza/cattaneo/data/Methylation-RUES2-FLG/RUES2-20CAG/reference/GCA_000001405.15_GRCh38_no_alt_analysis_set.fna" \
-a "out_prefix=barcode${n}_v4_${f2}" \
-a "bed=/lustrehome/gianluca/test_straglr/CAG_v4.bed" \
-a "file=${f2}" \
-a "barcode=barcode${n}" \
/lustrehome/gianluca/straglr/jobs/condor-straglr_exp.job ; done ; done | less -S
```

### Merge multiple *straglr* output - version CAG_v4.bed

```bash
cd /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30

for n in 05 08 09 10 17 18 19 20 21 22 23 24; do ls barcode${n}_v4_split*.tsv | grep aa | while read file ; do cat ${file} > barcode${n}_v4.tsv; done ; done

for n in 05 08 09 10 17 18 19 20 21 22 23 24; do ls barcode${n}_v4_split*.tsv | grep -v aa | while read file ; do cat ${file} | grep -v "#" >> barcode${n}_v4.tsv; done ; done

#cancel all .ok files created for job-control
rm *.ok
```

### Create folder to save on local computer with blastn+straglr+alignment_summary - version CAG_v4.bed

```bash
cd /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30

for n in 05 08 09 10 17 18 19 20 21 22 23 24; do mkdir barcode${n}_v4 ; done

ALIGNMENT=/lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/20220307_1326_MN37986_FAR78200_cb0c17e2/analysis/alignment

BLASTN=/lustrehome/gianluca/straglr/blastn/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30

STRAGLR=/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30

for n in 05 08 09 10 17 18 19 20 21 22 23 24; do cp ${ALIGNMENT}/barcode${n}_WG/alignment_summary.txt ${STRAGLR}/barcode${n}_v4/ &&  cp ${BLASTN}/barcode${n}* ${STRAGLR}/barcode${n}_v4/ &&  cp ${STRAGLR}/barcode${n}_v4.tsv ${STRAGLR}/barcode${n}_v4/ ; done 
```

### Rsync to local computer - version CAG_v4.bed

```bash
mkdir /Users/gianlucadamaggio/projects/cattaneo/straglr/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/

for n in 05 08 09 10 17 18 19 20 21 22 23 24; do rsync -avh --progress gianluca@ui02.recas.ba.infn.it:/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/barcode${n}_v4 /Users/gianlucadamaggio/projects/cattaneo/straglr/AC2021_NANOPORE_PCR10/107Q_CAFFEINE_D0_D30/ ; done
```

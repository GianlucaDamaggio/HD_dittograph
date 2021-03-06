# AC2021_NANOPORE_PCR10 107Q_prolif_rep2_div30_div60_div90 + 107Q_prolif_rep2_c25_div58

## Basecalling

```bash
cd /lustrehome/gianluca/strique/jobs/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/

./submit_chronos_gianluca guppy-gianluca-job-prolif-basecalling.json

./submit_chronos_gianluca guppy-gianluca-job-prolif-c25-div58-basecalling.json
```

## STRIQUE

### STRIQUE INDEX

```bash
cd /lustrehome/gianluca/strique/jobs/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/

condor_submit -name ettore condor-striqueIndex.job

condor_submit -name ettore condor-striqueIndex_c25_div58.job
```

### alignment reads "both_ends" with PCR fragment

```bash
cd /lustrehome/gianluca/strique/jobs/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/

for n in $(seq 1 12) ; do ./submit_chronos_gianluca guppy-gianluca-job-prolif-align${n}.json ; done

./submit_chronos_gianluca guppy-gianluca-job-prolif-align_c25_div58.json
```

### numero di reads allineate con fasta 107Q flippato (senza NEO)

```bash
for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do cat barcode${n}/alignment_summary.txt | grep 107Q | wc -l ; done
```

|barcode |reads number|
|------|--------------|
| barcode01 |  211622 |
| barcode02 |  231241 |
| barcode03 |  188621 |
| barcode04 |  176890 |
| barcode05 |  198674 |
| barcode06 |  207540 |
| barcode07 |  139790 |
| barcode08 | (60881) | c25_div58_FLONG
| barcode09 |  150535 |
| barcode10 |  122977 |
| barcode11 |  176319 |
| barcode12 |  100792 |

### 3.2 - STRique count Analysis

#### 3.2.1 concat all SAM file after alignment

```bash
for n in 01 02 03 04 05 06 07 09 10 11 12 ; do bash /lustre/home/enza/cattaneo/src/concatSAM_split4STRique_PCR10.sh /lustre/home/enza/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/alignment/barcode${n} 107Q ; done

bash /lustre/home/enza/cattaneo/src/concatSAM_split4STRique_PCR10.sh /lustre/home/enza/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_c25_div58/20211124_1325_MN29119_AHI905_191b6139/analysis/alignment 107Q ; done
```

```bash
for c in 01 02 03 04 05 06 07 09 10 11 12 ; do ls barcode${c}/split_fq_* | while IFS="$(printf '/')" read -r f1 f2 ; do for n in 10 ; do echo condor_submit -name ettore -a "sam=/lustre/home/enza/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/alignment/barcode${c}/${f2}" -a "fofn=/lustre/home/enza/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/fast5/reads.fofn" -a "config=/lustre/home/enza/cattaneo/config_file/AC2021_NANOPORE_PCR10/HTT_config_AC2021_NANOPORE_PCR10_150_770bp_CAG.tsv" -a "output=/lustrehome/gianluca/strique/striqueOutput/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/barcode${c}/AC_barcode${c}_prolif.${f2}.qs10.150_770bp_CAG.tsv" -a "file=${f2}" -a "barcode=${c}" /lustrehome/gianluca/strique/jobs/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/condor-striqueCount_exp.job ; done ; done; done | less -S

for c in 08 ; do ls alignment/split_fq_* | while IFS="$(printf '/')" read -r f1 f2 ; do for n in 10 ; do echo condor_submit -name ettore -a "sam=/lustre/home/enza/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_c25_div58/20211124_1325_MN29119_AHI905_191b6139/analysis/alignment/${f2}" -a "fofn=/lustre/home/enza/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_c25_div58/20211124_1325_MN29119_AHI905_191b6139/fast5/reads.fofn" -a "config=/lustre/home/enza/cattaneo/config_file/AC2021_NANOPORE_PCR10/HTT_config_AC2021_NANOPORE_PCR10_150_770bp_CAG.tsv" -a "output=/lustrehome/gianluca/strique/striqueOutput/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/barcode${c}/AC_barcode${c}_prolif.${f2}.qs10.150_770bp_CAG.tsv" -a "file=${f2}" -a "barcode=${c}" /lustrehome/gianluca/strique/jobs/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/condor-striqueCount_exp.job ; done ; done ; done  | less -S
```

#### 4.2 Merge splitted STRique output

```bash
for n in 01 02 03 04 05 06 07 08 09 10 11 12 ;  do cp barcode${n}/AC_barcode${n}_prolif.split_fq_aa.qs10.150_770bp_CAG.tsv barcode${n}/AC_barcode${n}.qs10.150_770bp_CAG.tsv ; done

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do ls barcode${n}/AC_barcode${n}_prolif.split_fq_* | grep -v aa | while read line ; do cat $line | tail -n+2 >> barcode${n}/AC_barcode${n}.qs10.150_770bp_CAG.tsv ; done ; done
```

## STRAGLR

### WHOLE GENOME alignment

Add to json file for alignment vs the whole-genome.

```json
"constraints": [["hostname", "LIKE", "hpc-gpu-1-[2,4]-[1-2].recas.ba.infn.it"]],
```

rep2 d30-60-90

```bash
for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do cp guppy-gianluca-job-WG-align01.json guppy-gianluca-job-WG-align${n}.json ; done

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do sed -i -e "s/barcode01/barcode${n}/g" guppy-gianluca-job-WG-align${n}.json ; done

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do ./submit_chronos_gianluca guppy-gianluca-job-WG-align${n}.json ; done
```

### FASTQ2FASTA

```bash
mkdir /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/fastq2fasta

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do zcat /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/basecalling/pass/barcode${n}/*.fastq.gz | awk '{if(NR%4==1) {printf(">%s\n",substr($0,2));} else if(NR%4==2) print;}' > /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/fastq2fasta/barcode${n}_allreads.fasta ; done
```

### BLASTN 107Qbarcode

```bash
source ~/.bashrc

mkdir /lustrehome/gianluca/straglr/blastn/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do blastn -task blastn-short -query /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/fastq2fasta/barcode${n}_allreads.fasta -subject /lustrehome/gianluca/straglr/barcode_fasta/barcode107Q.fa -outfmt 6 > /lustrehome/gianluca/straglr/blastn/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/barcode${n}_blastn_107Q.txt ; done
```

### *straglr*, for CAG count

CL Merge 100 bam:

```bash
for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do bash bash-merge_100_BAM.sh /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/alignment/barcode${n}_WG ; done
```

CL Job submission:

```bash
mkdir /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90

cd /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/alignment/

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do ls barcode${n}_WG/*.multi.bam | while IFS="$(printf '/')" read -r f1 f2 ; do echo condor_submit -name ettore \
-a "out_dir=/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/" \
-a "bam=/lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/alignment/barcode${n}_WG/${f2}" \
-a "ref=/lustre/home/enza/cattaneo/data/Methylation-RUES2-FLG/RUES2-20CAG/reference/GCA_000001405.15_GRCh38_no_alt_analysis_set.fna" \
-a "out_prefix=barcode${n}_${f2}" \
-a "bed=/lustrehome/gianluca/test_straglr/CAG.bed" \
-a "file=${f2}" \
-a "barcode=barcode${n}" \
/lustrehome/gianluca/straglr/jobs/condor-straglr_exp.job ; done ; done | less -S
```

## Merge multiple *straglr* output

```bash
cd /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do ls barcode${n}_split*.tsv | grep aa | while read file ; do cat ${file} > barcode${n}.tsv; done ; done

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do ls barcode${n}_split*.tsv | grep -v aa | while read file ; do cat ${file} | grep -v "#" >> barcode${n}.tsv; done ; done

#cancel all .ok files created for job-control
rm *.ok
```

## Create folder to save on local computer with blastn+straglr+alignment_summary

```bash
cd /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do mkdir barcode${n} ; done

ALIGNMENT=/lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/alignment

BLASTN=/lustrehome/gianluca/straglr/blastn/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90

STRAGLR=/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do cp ${ALIGNMENT}/barcode${n}_WG/alignment_summary.txt ${STRAGLR}/barcode${n}/ &&  cp ${BLASTN}/barcode${n}* ${STRAGLR}/barcode${n}/ &&  cp ${STRAGLR}/barcode${n}.tsv ${STRAGLR}/barcode${n}/ ; done 
```

## Rsync to local computer

```bash
mkdir /Users/gianlucadamaggio/projects/cattaneo/straglr/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do rsync -avh --progress gianluca@ui02.recas.ba.infn.it:/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/barcode${n} /Users/gianlucadamaggio/projects/cattaneo/straglr/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/ ; done
```

## Re-do Straglr - version CAG_v4.bed

```bash
mkdir /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90

cd /lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/alignment/

for n in 01 02 03 04 05 06 07 08 09 10 11 12 ; do ls barcode${n}_WG/*.multi.bam | while IFS="$(printf '/')" read -r f1 f2 ; do echo condor_submit -name ettore \
-a "out_dir=/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/" \
-a "bam=/lustrehome/gianluca/cattaneo/data/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/20211122_1902_MN37986_FAQ69057_21d78a93/analysis/alignment/barcode${n}_WG/${f2}" \
-a "ref=/lustre/home/enza/cattaneo/data/Methylation-RUES2-FLG/RUES2-20CAG/reference/GCA_000001405.15_GRCh38_no_alt_analysis_set.fna" \
-a "out_prefix=barcode${n}_v4_${f2}" \
-a "bed=/lustrehome/gianluca/test_straglr/CAG_v4.bed" \
-a "file=${f2}" \
-a "barcode=barcode${n}" \
/lustrehome/gianluca/straglr/jobs/condor-straglr_exp.job ; done ; done | less -S
```

### Merge multiple *straglr* output - version CAG_v4.bed

```bash
cd /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90

for n in 01 02 03 04 05 06 07 08 09 10 11 12; do ls barcode${n}_v4_split*.tsv | grep aa | while read file ; do cat ${file} > barcode${n}_v4.tsv; done ; done

for n in 01 02 03 04 05 06 07 08 09 10 11 12; do ls barcode${n}_v4_split*.tsv | grep -v aa | while read file ; do cat ${file} | grep -v "#" >> barcode${n}_v4.tsv; done ; done

#cancel all .ok files created for job-control
rm *.ok
```

### Create folder to save on local computer with blastn+straglr+alignment_summary - version CAG_v4.bed

```bash
cd /lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90

for n in 01 02 03 04 05 06 07 08 09 10 11 12; do mkdir barcode${n}_v4 ; done

STRAGLR=/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90

for n in 01 02 03 04 05 06 07 08 09 10 11 12; do cp ${STRAGLR}/barcode${n}_v4.tsv ${STRAGLR}/barcode${n}_v4/ ; done 
```

### Rsync to local computer - version CAG_v4.bed

```bash
mkdir /Users/gianlucadamaggio/projects/cattaneo/straglr/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/

for n in 01 02 03 04 05 06 07 08 09 10 11 12; do rsync -avh --progress gianluca@ui02.recas.ba.infn.it:/lustrehome/gianluca/straglr/output/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/barcode${n}_v4 /Users/gianlucadamaggio/projects/cattaneo/straglr/AC2021_NANOPORE_PCR10/107Q_prolif_rep2_div30_div60_div90/ ; done
```
-->
Bulk RNA-seq analysis tissue data

Data: /media/lopez_cascales/DATA2/BulkRNAseq_tissuedataTsaiYing Raw data: X204SC26092341-Z01-F001/01.RawData Reference: GRCh38 primary assembly + Ensembl 109 GTF Aligner: STAR 2.7.10a
Contents

    Tool installation
    Samples and MD5 check
    Reference genome
    STAR index
    Mapping
        5.1 Check index and sample layout
        5.2 First sample — failed (open-file limit)
        5.3 Check file-descriptor limits
        5.4 First sample — rerun with raised limit
        5.5 Mapping QC and strandedness
        5.6 Mapping remaining samples (loop)
    Pending issues

1. Tool installation

Script: RNASEQBULK_00_install_tools.sh
Tool 	Status 	Path / version
STAR 	OK 	/usr/bin/STAR (2.7.10a)
samtools 	OK 	/usr/bin/samtools (1.13)
FastQC 	OK 	/usr/bin/fastqc
MultiQC 	OK 	1.35
deepTools (bamCoverage) 	OK 	3.5.6
R 	OK 	4.3.3
DESeq2 	OK 	1.42.1
ashr 	OK 	2.2.63
limma 	OK 	3.58.1
pheatmap 	OK 	1.0.13
ggplot2 	OK 	4.0.3
ggrepel 	OK 	0.9.8

    Required: STAR, samtools, Rscript (+ DESeq2, ashr)
    Optional: fastqc, multiqc, bamCoverage (steps skipped if missing)

Load tools:

source /home/lopez_cascales/rnaseq_tools/env.sh

2. Samples and MD5 check

Script: RNASEQBULK_01_preprocess_RNAseq.sh Parameters: threads = 20 | trim = 0 | sparse = 1

18 samples (3 patients × 3 conditions × 2 replicates):
Patient 	Samples
T31 	Acti_CSM_1A, Acti_CSM_1B, ActiCtrl_A, ActiCtrl_B, Acti_FSM_1A, Acti_FSM_1B
T34 	Acti_CSM_1A, Acti_CSM_1B, ActiCtrl_A, ActiCtrl_B, Acti_FSM_1A, Acti_FSM_1B
T96 	Acti_CSM_1A, Acti_CSM_1B, ActiCtrl_A, ActiCtrl_B, Acti_FSM_1A, Acti_FSM_1B

MD5 output:

[MD5] T31_Acti_CSM_1A
[MD5] T31_Acti_CSM_1B
T31_Acti_CSM_1B_MKRO260003428-1A_23NVTWLT4_L8_2.fq.gz: FAILED
md5sum: WARNING: 1 computed checksum did NOT match
ERROR: MD5 mismatch in T31_Acti_CSM_1B — re-download before continuing

    Status: pipeline stopped — T31_Acti_CSM_1B read 2 FASTQ (..._23NVTWLT4_L8_2.fq.gz) must be re-downloaded.

3. Reference genome

BASE=/media/lopez_cascales/DATA2/BulkRNAseq_tissuedataTsaiYing/X204SC26092341-Z01-F001/01.RawData
ls -lh $BASE/Homo_sapiens.GRCh38.dna.primary_assembly.fa $BASE/Homo_sapiens.GRCh38.109.gtf

File 	Size 	Date
Homo_sapiens.GRCh38.109.gtf 	1.4 G 	15 Dec 2022
Homo_sapiens.GRCh38.dna.primary_assembly.fa 	3.0 G 	13 Dec 2022
4. STAR index

BASE=/media/lopez_cascales/DATA2/BulkRNAseq_tissuedataTsaiYing/X204SC26092341-Z01-F001/01.RawData
FA=$BASE/Homo_sapiens.GRCh38.dna.primary_assembly.fa
GTF=$BASE/Homo_sapiens.GRCh38.109.gtf
IDX=$BASE/STAR_index_GRCh38_109
OVERHANG=149

mkdir -p "$IDX"
[[ -f $FA.fai ]] || samtools faidx "$FA"

STAR --runMode genomeGenerate --runThreadN 64 --genomeDir "$IDX" \
     --genomeFastaFiles "$FA" --sjdbGTFfile "$GTF" --sjdbOverhang "$OVERHANG" \
     --genomeSAindexNbases 14 --outFileNamePrefix "$IDX/"

Log:

STAR version: 2.7.10a
Nov 02 11:36:15 ..... started STAR run
Nov 02 11:36:15 ... starting to generate Genome files
Nov 02 11:36:52 ..... processing annotations GTF
Nov 02 11:37:09 ... starting to sort Suffix Array. This may take a long time...
Nov 02 11:37:21 ... sorting Suffix Array chunks and saving them to disk...
Nov 02 14:22:07 ... loading chunks from disk, packing SA...
Nov 02 14:24:43 ... finished generating suffix array
Nov 02 14:24:43 ... generating Suffix Array index
Nov 02 14:27:35 ... completed Suffix Array index
Nov 02 14:27:35 ..... inserting junctions into the genome indices
Nov 02 14:37:32 ... writing Genome to disk ...
Nov 02 14:37:34 ... writing Suffix Array to disk ...
Nov 02 14:39:08 ... writing SAindex to disk
Nov 02 14:39:17 ..... finished successfully

Runtime: ~3 h.

    Note — --sjdbOverhang 149: recommended value is read length − 1. Reads are 150 bp, so 149. The default (100) would work about as well, but 149 is optimal for this data.

5. Mapping
5.1 Check index and sample layout

BASE=/media/lopez_cascales/DATA2/BulkRNAseq_tissuedataTsaiYing/X204SC26092341-Z01-F001/01.RawData
IDX=$BASE/STAR_index_GRCh38_109
OUT=$BASE/analysis

ls -lh $IDX/{Genome,SA,SAindex,sjdbList.out.tab,exonInfo.tab}
rm -rf $IDX/_STARtmp 2>/dev/null
ls -d $BASE/T[0-9]*/ | head -20
ls $BASE/$(ls $BASE | grep '^T[0-9]' | head -1)/

Index files:
File 	Size
exonInfo.tab 	24 M
Genome 	3.1 G
SA 	24 G
SAindex 	1.5 G
sjdbList.out.tab 	8.8 M

Sample directories: $BASE/T31_*, $BASE/T34_*, $BASE/T96_* (18 total).

Each sample has two lanes (L8, L3) per read:

MD5.txt
T31_Acti_CSM_1A_MKRO260003427-1A_23NVTWLT4_L8_1.fq.gz
T31_Acti_CSM_1A_MKRO260003427-1A_23NVTWLT4_L8_2.fq.gz
T31_Acti_CSM_1A_MKRO260003427-1A_23NWJ3LT4_L3_1.fq.gz
T31_Acti_CSM_1A_MKRO260003427-1A_23NWJ3LT4_L3_2.fq.gz

Lanes are passed to STAR as comma-separated lists (R1 lanes, then R2 lanes).
5.2 First sample: failed (open-file limit)

STAR --runThreadN 64 --genomeDir $IDX \
     --readFilesIn "$R1" "$R2" --readFilesCommand zcat \
     --quantMode GeneCounts \
     --outSAMtype BAM SortedByCoordinate --outBAMsortingThreadN 16 --limitBAMsortRAM 30000000000 \
     --outSAMattributes NH HI AS nM \
     --outFileNamePrefix $OUT/02_star/$s/

Error:

Nov 02 15:29:45 ..... started STAR run
Nov 02 15:29:45 ..... loading genome

BAMoutput.cpp:27:BAMoutput: exiting because of *OUTPUT FILE* error: could not create output file
.../analysis/02_star/T31_Acti_CSM_1A/_STARtmp//BAMsort/20/16
SOLUTION: check that the path exists and you have write permission for this file.
Also check ulimit -n and increase it to allow more open files.

Nov 02 15:30:35 ...... FATAL ERROR, exiting

Cause: BAM sorting with 64 threads opens more temporary files than the default soft limit allows.
5.3 Check file-descriptor limits

ulimit -n     # soft limit
ulimit -Hn    # hard limit (max raisable without root)

Limit 	Value
Soft (ulimit -n) 	1024
Hard (ulimit -Hn) 	1048576
5.4 First sample: rerun with raised limit

ulimit -n $(ulimit -Hn)       # raise soft limit to the hard maximum
ulimit -n
rm -rf $OUT/02_star/$s/_STARtmp

STAR --runThreadN 64 --genomeDir $IDX \
     --readFilesIn "$R1" "$R2" --readFilesCommand zcat \
     --quantMode GeneCounts \
     --outSAMtype BAM SortedByCoordinate --outBAMsortingThreadN 16 --limitBAMsortRAM 30000000000 \
     --outSAMattributes NH HI AS nM \
     --outFileNamePrefix $OUT/02_star/$s/

Log:

Nov 02 15:33:34 ..... started STAR run
Nov 02 15:33:34 ..... loading genome
Nov 02 15:34:17 ..... started mapping
Nov 02 15:38:32 ..... finished mapping
Nov 02 15:38:34 ..... started sorting BAM
Nov 02 15:43:22 ..... finished successfully

Runtime: ~10 min per sample.

    ulimit -n only applies to the current shell. It must be set again in every new terminal / script before running STAR.

5.5 Mapping QC and strandedness

grep -E "Number of input reads|Uniquely mapped reads %|% of reads mapped to multiple|% of reads unmapped" \
     $OUT/02_star/$s/Log.final.out
head -5 $OUT/02_star/$s/ReadsPerGene.out.tab
awk 'NR>4{a+=$2;b+=$3;c+=$4} END{print "unstranded="a, "forward="b, "reverse="c}' \
     $OUT/02_star/$s/ReadsPerGene.out.tab

Mapping quality — T31_Acti_CSM_1A
Metric 	Value
Input read pairs 	36,687,894
Uniquely mapped 	87.00%
Multi-mapped 	10.03%
Unmapped: too many mismatches 	0.00%
Unmapped: too short 	2.61%
Unmapped: other 	0.30%

36.7 M input read pairs; 87.0% unique, 10.0% multi-mapped, 2.9% unmapped. Good result for human mRNA.
ReadsPerGene.out.tab (first rows)
	Col 2 (unstranded) 	Col 3 (forward) 	Col 4 (reverse)
N_unmapped 	1,089,766 	1,089,766 	1,089,766
N_multimapping 	3,678,856 	3,678,856 	3,678,856
N_noFeature 	1,750,271 	16,572,440 	15,888,588
N_ambiguous 	4,639,023 	1,791,927 	1,952,565
ENSG00000160072 	238 	115 	123
Strandedness
Column 	Assigned reads
Unstranded (col 2) 	25,529,978
Forward (col 3) 	13,554,905
Reverse (col 4) 	14,078,119

    Forward and reverse are nearly identical, and their sum ≈ unstranded.
    A dUTP (reverse-stranded) library would show ~10× more reads in reverse than forward — not the case here.
    Library is unstranded → use column 2 of ReadsPerGene.out.tab.

    Action: check which column 02_deseq2_analysis.R reads. If it uses column 4, every gene loses ~half its counts. Adjust after alignment is complete.

5.6 Mapping remaining samples (loop)

Skips samples that already have a Log.final.out; stops on the first failure.

ulimit -n $(ulimit -Hn)

for d in $BASE/T[0-9]*/; do
  s=$(basename "$d")
  if [[ -s $OUT/02_star/$s/Log.final.out ]]; then echo ">>> $s already done, skipping"; continue; fi
  echo ">>> $(date +%H:%M) $s"
  R1=$(ls $BASE/$s/*_1.fq.gz | paste -sd,)
  R2=$(ls $BASE/$s/*_2.fq.gz | paste -sd,)
  mkdir -p $OUT/02_star/$s; rm -rf $OUT/02_star/$s/_STARtmp
  STAR --runThreadN 64 --genomeDir $IDX \
       --readFilesIn "$R1" "$R2" --readFilesCommand zcat \
       --quantMode GeneCounts \
       --outSAMtype BAM SortedByCoordinate --outBAMsortingThreadN 16 --limitBAMsortRAM 30000000000 \
       --outSAMattributes NH HI AS nM \
       --outFileNamePrefix $OUT/02_star/$s/ \
    || { echo "!!! FAILED on $s"; break; }
done
echo "=== finished $(date +%H:%M) ==="

Expected runtime: ~17 samples × ~10 min ≈ 3 h.
6. Pending issues

    Re-download T31_Acti_CSM_1B_MKRO260003428-1A_23NVTWLT4_L8_2.fq.gz (MD5 mismatch) and re-check MD5
    Re-map T31_Acti_CSM_1B once the file is replaced (delete $OUT/02_star/T31_Acti_CSM_1B/ first so the loop does not skip it)
    Check which ReadsPerGene.out.tab column 02_deseq2_analysis.R reads → must be column 2 (unstranded)
    Review mapping stats for all 18 samples (MultiQC on 02_star/) #####################################################################################################################

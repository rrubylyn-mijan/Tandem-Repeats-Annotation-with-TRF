# Tandem-Repeats-Annotation-with-TRF
This workflow identifies and quantifies tandem repeats in wheat genome assemblies using Tandem Repeats Finder (TRF). It includes installation, HPC job scripts, repeat length and percentage calculations, and genome-wide density analysis for visualization.

## 1. Installation
```bash
cd /directory/this/saved/tandem-repeat-finder
git clone https://github.com/Benson-Genomics-Lab/TRF.git
cd TRF
./configure
make
cd src
```

## 2. Run TRF on Assemblies
```bash
./trf /directory/this/saved/wheat.fasta 2 7 7 80 10 50 500 -f -d
```

## 3. Example SLURM Job Script (atlas)
```bash
#!/bin/bash
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -p atlas
#SBATCH --mem=100GB
#SBATCH -J tandem_repeats_wheat
#SBATCH -A genolabswheatphg

cd /directory/this/saved/trf/src

./trf /directory/this/saved/wheat.fasta 2 7 7 80 10 50 500 -f -d
```

## 4. Calculate Total Length of Tandem Repeats
```bash
awk '$1 ~ /^[0-9]+$/ && $2 ~ /^[0-9]+$/ {total += ($2 - $1 + 1)} END {print total}' wheat.fasta.2.7.7.80.10.50.500.dat
```

## 5. Genome Size Calculation
```bash
grep -v "^>" wheat.fasta  | tr -d '\n' | wc -c
```

## 6. Tandem Repeat Percentage
```bash
genome_size=14408607908   # Example: wheat
total_length=877621698

echo "scale=2; ($total_length / $genome_size) * 100" | bc
```

## 7. Distribution by Chromosome Group (A, B, D)
```bash
awk '/^Sequence:/ {chr=$2}
     /^[0-9]/ {
       if (chr ~ /A$/) A += ($2-$1+1);
       else if (chr ~ /B$/) B += ($2-$1+1);
       else if (chr ~ /D$/) D += ($2-$1+1);
     }
     END {print "A:", A, "B:", B, "D:", D}' wheat.fasta.2.7.7.80.10.50.500.dat
```

## 8. Prepare BED Files for Genome-wide Density (Circos)
```bash
awk '/Sequence:/ {chr=$2} /^[0-9]/ {print chr"\t"($1-1)"\t"$2}' wheat.fasta.2.7.7.80.10.50.500.dat > wheat_tandem_repeats.bed

# Make windows:
ml bedtools2/2.31.1
bedtools makewindows -g wheat.fasta.fai -w 1000000 > wheat_genome_1Mb_windows.bed

# Coverage:
bedtools coverage -a wheat_genome_1Mb_windows.bed -b wheat_tandem_repeats.bed > wheat_tandem_repeat_density.txt

# Format for Circos:
awk '{gsub(/^chr/, "ta", $1); print $1, $2, $3, $7}' wheat_tandem_repeat_density.txt > x5_tandem_repeat_density_wheat
```

Maintainer:

Ruby Mijan

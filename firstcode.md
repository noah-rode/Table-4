## trim adapters and quality check reads
```
for i in *.lite.1_1.fastq
do
OUT=#{i%.lite.1_1.fastq}
fastp -i $OUT.lite.1_1.fastq -I $OUT.lite.1_2.fastq -o $OUT.lite.trim.1_1.fastq -O $OUT.lite.1_2.fastq
done
```

## BWA MAPPING
```
bwa index bbc.fasta

for i in *lite.trim.1_1.fastq
do
OUT=#{i%lite.trim.1_1.fastq}
bwa mem -t 10 bbc.fasta $OUT.lite.trim.1_1.fastq $OUT.lite.trim.1_2.fastq > $OUT.sam
done
```

## Samtools View
```

for i in *.sam
do
OUT=${i%.sam}
samtools view -b $OUT.sam -o $OUT.bam
done
```

## Samtools Sort
```

for i in *.bam
do
OUT=${i%.bam}
samtools sort $OUT.bam -o $OUT.sorted.bam
done
```

## bcftools mpileup and  call
```
bcftools mpileup -Ou -f bbc.fasta SRR10729165.sorted.bam SRR1072916
6.sorted.bam SRR10729566.sorted.bam SRR10733526.sorted.bam SRR31835375.sorted.bam SRR31835473.sorted.bam SRR31835482.sorted.bam SRR31835573.sorted.bam | bcftools call -mv -Ov -o body_size.vc
```

## VCF to Allele Frequency
```

conda activate bio_env
bcftools query -l body_size.vcf
bgzip body_size.vcf
bcftools index body_size.vcf.gz

nano group1.txt
SRR10729165.sorted.bam
SRR10729166.sorted.bam
SRR10729566.sorted.bam
SRR10733526.sorted.bam
nano group2.txt
SRR31835375.sorted.bam
SRR31835473.sorted.bam
SRR31835482.sorted.bam
SRR31835573.sorted.bam

bcftools view -S group1.txt body_size.vcf.gz -Oz -o group1.vcf.gz
bcftools view -S group2.txt body_size.vcf.gz -Oz -o group2.vcf.gz
bcftools view -m2 -M2 -v snps group1.vcf.gz -Oz -o group1_biallelic.vcf.gz
bcftools view -m2 -M2 -v snps group2.vcf.gz -Oz -o group2_biallelic.vcf.gz
bcftools +fill-tags group1_biallelic.vcf.gz -Oz -o group1_af.vcf.gz -- -t AF
bcftools +fill-tags group2_biallelic.vcf.gz -Oz -o group2_af.vcf.gz -- -t AF
bcftools query -f '%CHROM\t%POS\t%INFO/AF\n' group1_af.vcf.gz > group1_af.tsv
bcftools query -f '%CHROM\t%POS\t%INFO/AF\n' group2_af.vcf.gz > group2_af.tsv

R
g1 <- read.table("group1_af.tsv", col.names = c("CHROM", "POS", "AF1"))
g2 <- read.table("group2_af.tsv", col.names = c("CHROM", "POS", "AF2"))
g1
g2
merged <- merge(g1, g2, by = c("CHROM", "POS"))
merged
merged$CHROM
merged
merged$AF_diff <- merged$AF1 - merged$AF2
merged

pdf('merged.pdf')

plot(merged$POS, merged$AF_diff,
     pch = 19, col = "steelblue",
     xlab = "Position in gene", ylab = "Allele frequency difference (Group1 - Group2)",
     main = "Allele frequency difference along bbc")
abline(h = 0, lty = 2, col = "grey40")

dev.off()

exit
scp -r visitor@134.129.113.23:/storehouse/visitor/table_4/pigmentation/merged.pdf .
password - temp
## able to view graph now
```







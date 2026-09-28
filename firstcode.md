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

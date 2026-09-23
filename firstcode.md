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

## trim adapters and quality check reads
```
for i in *.lite.1_1.fastq
do
OUT=#{i%.lite.1_1.fastq}
fastp -i $OUT.lite.1_1.fastq -I $OUT.lite.1_2.fastq -o $OUT.lite.trim.1_1.fastq -O $OUT.lite.1_2.fastq
done


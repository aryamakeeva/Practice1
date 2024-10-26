# Practice
SNP calling 
## Date: 25.10.2024
1. Создание виртуальной среды для работы
   `conda create --name project1env python=3.13`
2. Установка пакетов
  `conda install -c bioconda seqkit`
  `sudo apt install trimmomatic`

## Date: 26.10.2024
1. Скачивание данных из NCBI

`wget GCF_000005845.2_ASM584v2_genomic.fna.gz`: seqence 

`wget GCF_000005845.2_ASM584v2_genomic.gff.gz`: annotation

2. Скачать fasta- файлы не получилось, поэтому я просто перенесла их в папку вручную

3. Изучение структуры fasta-файлов

`zcat amp_res_1.fastq.gz | head -20`

zcat чтобы не разархивировать файлы, head -20 показывает первые 20 строк файла
![image](https://github.com/user-attachments/assets/f7a07ea6-97fc-4092-b323-74669c7f2268)

` zcat amp_res_1.fastq.gz | wc -l` - смотрим количество строк

`1823504` - аналогичное было и в reverse ридах

Чтобы посчитать количество ридов в файле нужно или количество строк разделить на 4, или ввести команду: `zcat amp_res_1.fastq.gz | wc -l | awk '{print $1/4}'. Количество ридов 455876.

Теперь смотрим вывод команды `zcat amp_res_1.fastq.gz | seqkit stats`
![image](https://github.com/user-attachments/assets/c53f17a1-07a1-4935-85ce-20d7a92414fb)

4. Фильтрация ридов

`mamba install -c bioconda fastqc` установила fastqc

`fastqc -h` изучила как пользоваться командой

Запустила программу для анализа качества ридов, флаг `-t` указывает количество потоков, `-o` местоположение куда сохранить получившиеся файлы
`fastqc -t 1 -o ./QC_Trimming /amp_res_1.fastq.gz /amp_res_2.fastq.gz` 

5. Изучение FastQC Report









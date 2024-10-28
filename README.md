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

2. Скачать fasta - файлы не получилось, поэтому я просто перенесла их в папку вручную

3. Изучение структуры fasta-файлов

`zcat amp_res_1.fastq.gz | head -20`

zcat чтобы не разархивировать файлы, head -20 показывает первые 20 строк файла
![image](https://github.com/user-attachments/assets/f7a07ea6-97fc-4092-b323-74669c7f2268)

` zcat amp_res_1.fastq.gz | wc -l` - смотрим количество строк

`1823504` - аналогичное было и в reverse ридах

Чтобы посчитать количество ридов в файле нужно или количество строк разделить на 4, или ввести команду: `zcat amp_res_1.fastq.gz | wc -l | awk '{print $1/4}`. Количество ридов 455876.

Теперь смотрим вывод команды `zcat amp_res_1.fastq.gz | seqkit stats`
![image](https://github.com/user-attachments/assets/c53f17a1-07a1-4935-85ce-20d7a92414fb)

4. Фильтрация ридов

`mamba install -c bioconda fastqc` установила fastqc

`fastqc -h` изучила как пользоваться командой

Запустила программу для анализа качества ридов, флаг `-t` указывает количество потоков, `-o` местоположение куда сохранить получившиеся файлы
`fastqc -t 1 -o ./QC_Trimming /amp_res_1.fastq.gz /amp_res_2.fastq.gz` 

5. Изучение FastQC Report

_amp_res_1_fastqc_
![image](https://github.com/user-attachments/assets/61466195-e42b-4160-a29f-741d608bb91d)

Число ридов совпадает с ожидаемым
- Per base sequence quality: red, качество ридов падает к концу из-за истощения реактивов
  ![image](https://github.com/user-attachments/assets/2857ac45-476e-4711-8b3d-46cdf0eb664a)
- Per tile sequence quality: red, значение качества по регионам проточной ячейки тоже понижено, это может быть связано с появлением пузырьков воздуха (каких-то небольших дефектов ячейки)
![image](https://github.com/user-attachments/assets/dc0ac75f-5d86-44fe-9eb4-e5594b0c3602)
- Per base sequence content: yellow
- Per sequence GC content: yellow
  
_amp_res_2_fastqc_

- Per base sequence quality: red
- Per tile sequence quality: yellow
- Per base sequence content: yellow
- Per sequence GC content: yellow









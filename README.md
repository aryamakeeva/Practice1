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

`zcat` чтобы не разархивировать файлы, `head -20` показывает первые 20 строк файла
![image](https://github.com/user-attachments/assets/f7a07ea6-97fc-4092-b323-74669c7f2268)

` zcat amp_res_1.fastq.gz | wc -l` - смотрим количество строк

`1823504` - аналогичное было и в reverse ридах

Чтобы посчитать количество ридов в файле нужно или количество строк разделить на 4, или ввести команду: 

`zcat amp_res_1.fastq.gz | wc -l | awk '{print $1/4}`. 

Количество ридов `455876`.

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
- Per base sequence content: yellow, возможно из-за наличия адаптеров мы видим "шум" в начале последовательности
- Per sequence GC content: yellow
  
_amp_res_2_fastqc_

- Per base sequence quality: red
- Per tile sequence quality: yellow
- Per base sequence content: yellow
- Per sequence GC content: yellow

6. Тримминг последовательности

`mamba install -c bioconda trimmomatic` 

`trimmomatic PE -phred33 amp_res_1.fastq.gz amp_res_2.fastq.gz \
               amp_res_1_paired.fastq.gz amp_res_1_unpaired.fastq.gz \
               amp_res_2_paired.fastq.gz amp_res_2_unpaired.fastq.gz \
               LEADING:20 TRAILING:20 SLIDINGWINDOW:10:20`
               
В этой команде `PE` для парных ридов, `-phred33` используемый PhredScore, затем идут `input` файлы и 4 `output` т.к. нам нужно складывать в разные папки, если в парном чтении один из ридов прошел по качеству, а другой - нет. `LEADING` и `TRAILING` это обрезать с начала и с конца по определенному качеству, а `SLIDINGWINDOW` тримминг скользящим окном 

Посчитали количество прочтений: стало `452621`. В результате выполнения команды было отбраковано `3255` ридов

Затем сделали ещё раз проверку качества:

`fastqc -t 1 -o ./QC_Trimming /amp_res_1_paired.fastq.gz /amp_res_2_paired.fastq.gz`

После тримминга повысилось качество ридов (Per base sequence quality), поскольку мы обрезали с конца риды, но появилась желтая галочка в Sequence Length Distribution. 
![image](https://github.com/user-attachments/assets/3a81d010-d9ae-4f09-b4f5-d2f3fc27fe74)

## Date: 28.10.2024

7. Mapping Calling

- установка `bwa`

`conda install bwa` не получилось

Ошибка: `PackagesNotFoundError: The following packages are not available from current channels: bwa`

`conda install bioconda::bwa` 

Получилось :)

Индексация:

`bwa index ./Map_Call/ref_indexes /home/arina/Practice.IB/Project1/GCF_000005845.2_ASM584v2_genomic.fna.gz`

Картирование:

`bwa mem GCF_000005845.2_ASM584v2_genomic.fna.gz /Users/alinanazarova/all_important/BI/bioinf_prak/Practice1/amp_res_1_paired.fastq.gz /Users/alinanazarova/all_important/BI/bioinf_prak/Practice1/amp_res_2_paired.fastq.gz > alignment.sam`

Посмотрим статистику картирования с использованием флагов из SamTools

`samtools view -@ 1 -c alignment.sam` 

общее число выравниваний: 905500

`samtools view -@ 1 -f 4 -c alignment.sam`

число некартированных ридов: 7659

`samtools view -@ 1 -F 4 -c alignment.sam`

число картированных ридов: 897841

`samtools view -@ 1 -F 4 -c alignment.sam`

число правильно картированных ридов: 889224











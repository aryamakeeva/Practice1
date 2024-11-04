# Practice
SNP calling 
## Date: 25.10.2024
1. Создание виртуальной среды для работы
   `conda create --name project1env python=3.13`
2. Установка пакетов
  `conda install -c bioconda seqkit`
  `sudo apt install trimmomatic`

## Date: 26.10.2024
### 1. Скачивание данных из NCBI

`wget GCF_000005845.2_ASM584v2_genomic.fna.gz`: seqence 

`wget GCF_000005845.2_ASM584v2_genomic.gff.gz`: annotation

### 2. Скачать fasta - файлы не получилось, поэтому я просто перенесла их в папку вручную

### 3. Изучение структуры fasta-файлов

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

### 4. Фильтрация ридов

`mamba install -c bioconda fastqc` установила fastqc

`fastqc -h` изучила как пользоваться командой

Запустила программу для анализа качества ридов, флаг `-t` указывает количество потоков, `-o` местоположение куда сохранить получившиеся файлы
`fastqc -t 1 -o ./QC_Trimming /amp_res_1.fastq.gz /amp_res_2.fastq.gz` 

### 5. Изучение FastQC Report

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

### 6. Тримминг последовательности

При обнаружении недостаточного качества в FastQC отчёте необходимо привести риды в порядок (подрезать с концов, если качество падает или, например, убрать адапторные последовательности, если таковые обнаруживаются). 

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

### 7. Mapping Calling

- Установка `bwa`

`conda install bwa` не получилось

Ошибка: `PackagesNotFoundError: The following packages are not available from current channels: bwa`

`conda install bioconda::bwa` 

Получилось :)

- Индексация:

`bwa index ./Map_Call/ref_indexes /home/arina/Practice.IB/Project1/GCF_000005845.2_ASM584v2_genomic.fna.gz`

- Картирование:

`bwa mem GCF_000005845.2_ASM584v2_genomic.fna.gz /Users/alinanazarova/all_important/BI/bioinf_prak/Practice1/amp_res_1_paired.fastq.gz /Users/alinanazarova/all_important/BI/bioinf_prak/Practice1/amp_res_2_paired.fastq.gz > alignment.sam`

Конвертация файла в bam-формат: 

`samtools view -S -b alignment.sam > alignment.bam`

- Посмотрим статистику картирования с использованием флагов из SamTools:

`samtools view -@ 1 -c alignment.sam` 

общее число выравниваний: 905500

`samtools view -@ 1 -f 4 -c alignment.sam`

число некартированных ридов: 7659

`samtools view -@ 1 -F 4 -c alignment.sam`

число картированных ридов: 897841

`samtools view -@ 1 -F 4 -c alignment.sam`

число правильно картированных ридов: 889224

- Сортировка и индексация .bam: команды из практикума сработали без дополнительных изменения

`samtools sort alignment.bam alignment_sorted`

`samtools index alignment_sorted.bam`

После сортировки и индексации оценим покрытие, которое у нас получилось:

` samtools coverage alignment_sorted.bam`

![image](https://github.com/user-attachments/assets/6b2c34c3-746e-40bf-896e-597981089912)

rname - хромосома референса, star/endpos - стартовая/конечная позиция, numreads - число ридов, выровненных по региону (после фильтрации), coverage - % покрытых нуклеотидов, depth - глубина покрытия


- Визуализация в IGV-браузере:

Чтобы посмотреть результаты картирования и коллинга в IGV сначала загрузили индексированный референсный геном на который мы
картировали свои данные (представленные в браузере нам не подходили). После загрузки референса загрузить сортированные по координатам и индексированные bam файлы (.bam и bam.bai)

![image](https://github.com/user-attachments/assets/745f9875-058c-441d-a5fb-d1cc8f409bd7)

## Date: 29.10.2024

### Variant calling

- Далее провели поиск мутаций с помощью VarScan(varianr scaning), но для запуска его работы необходимо подготовить промежуточный файл **mpileup** который считает количество несовпадений с рефересной молекулой.

разархивировали файл `genomic.fna.gz` с помощью команды `gunzip`

Провели расчет покрытия прочтениями каждой позиции рефернса. Флаг `-f` fasta с последовательностью референсного генома:

`samtools mpileup -f GCF_000005845.2_ASM584v2_genomic.fna alignment_sorted.bam >  my.mpileup`

- Затем потербовалась установка **VarScan**, для этого нужно скачать файл **VarScan.v2.3.9.jar** с официального сайта или гитхаба разработчика и положить его в директорию где будет проходить обработка данных. Для поиска истинных мутаций запустили фильтр прочтений с следующими настройками:

`java -jar VarScan.v2.3.9.jar  mpileup2snp my.mpileup --min-var-freq 0.20 --variants --output-vcf 1 > VarScan_results.vcf`
  
где уровень **--min-var-freq** был выбран 0.2, это означает, что положения в которых отличия от референса ниже 20% отфильтровываются.

#### Результаты визуализации вариантов в IGV

<img width="1416" alt="Снимок экрана 2024-10-30 в 22 27 30" src="https://github.com/user-attachments/assets/a017d0c1-44ab-41b3-82ef-bdcc75803069"> 

- Таким образом получили 6 мутаций в бактериальном геноме _E.coli k_12_ для того чтобы понять влияние этих мутаций провели автоматическую SNP аннотацию с помощью **SnpEff**

- Установка SnpEff:

`conda install bioconda::snpeff`

- Запуск аннотации

1. Добавить в начало snpEff.config файла, который лежит в директории вместе с файлом snpEff.jar, строчку **k12.genome : ecoli_K12**
2. Вернуться в основную рабочую директорию, создать папку data, а в ней подпапку k12
3. Добавить в k12 файл с аннотацией с расширением dbff.gz разархивировать файл и переименовать в genes.gbk
4. Создать базу данных командой `snpEff build -dataDir /Users/alinanazarova/all_important/BI/bioinf_prak/Practice1/data/ -noCheckCds -genbank -v k12`
5. Провести аннотацию `snpEff ann -dataDir /Users/alinanazarova/all_important/BI/bioinf_prak/Practice1/data k12 VarScan_results.vcf > VarScan_results_annotated.vcf`

Аннотированные данные можно просмотреть в IGV подгрузив в качестве файла получившийся **VarScan_results_annotated.vcf**

## Date 03.11,2024

Альтернативный способ коллинга с помощью BCF-tools:

Установка: 

`conda install bioconda::bcftools`

`bcftools call -mv --ploidy 1 -Ob -o ./Map_Call/calls.bcf my.mpileup`

Флаг `--poidy 1` указывает, что наш организм гаплоидный, `-mv` использовать улучшенный алгоритм поиска вариантов, дать на выходе программы только сайты вариантов, `Ob` выдать результаты в бинарном формате BCF

Изучим вывод структуру файла:

`bcftools view -h calls.bcf`

`bcftools view -H calls.bcf | head -n 5`

![image](https://github.com/user-attachments/assets/b2690c84-c6e4-4097-a190-42d8867b44cd)

Теперь отфильтруем полиморфизмы, потому что некоторые мутации, которые мы видим потенциально могут быть ложно-положительными. Например, SNP на концах чтений или возле инделей потенциально - ошибки секвенирования/картирования. Или низкая глубина покрытия (мы считаем, что глубина покрытия >20 нас устраивает. Или отсутствие информации о мутации в базах данных - тоже ошибка (если речь идёт о модельных организмах).

`bcftools filter -e 'DP < 20 || QUAL < 20 || MQ < 20' -Ob calls.bcf > filtered_calls.bcf`

Альтернативный способ коллинга с помощью FreeBayers:

`conda install freebayes`

`freebayes -f GCF_000005845.2_ASM584v2_genomic.fna -p 2 -m 20 -q 20 ./Map_Call/alignment_sorted.bam >  ./Map_Call/free_calls.vcf`

`p` - плоидность, `-m` - min mapping quality Q, `-q` min base quality Q

дополнительно отфильтруем:

`bcftools filter -e 'INFO/DP > 20 || QUAL > 20' -Ob free_calls.vcf > filtered_free_calls.bcf`

Теперь проиндексируем 

`bcftools index filtered_free_calls.bcf & bcftools index filtered_calls.bcf`

Посчитаем статистику по всем полиморфизмам:

`bcftools stats filtered_free_calls.bcf > filtered_free_stats.txt`

`bcftools stats filtered_calls.bcf > filtered_calls_stats.txt`

Теперь оставим только SNP

`bcftools view -v snps -Ob filtered_calls.bcf > filtered_calls_snp.bcf`

`bcftools view -v snps -Ob filtered_free_calls.bcf > filtered_free_calls_snp.bcf`






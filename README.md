# FLAX
Анализ вариаций копийности льна по данным высокопроизводительного секвенирования

Ход работы:
1. Первым шагом необходимо конвертировать все файлы из формата vcf в bed, с заданными столбцами.
Для этого загружаем папку со всеми файлами в нужную директорию, например, /home/YOUR_NAME/bcftools-1.9/
Запускаем команду

```
for t in /home/YOUR_NAME/bcftools-1.9/NAME_FOLDER/*
do 
         bcftools query -f'%CHROM\t%POS0\t%END\t%ID\n'  $t > $t.bed
done
```
Эта команда преобразует ваш файл и выдает выходные столбцы: CHROM, POS 0, END, ID.

2.	Плотность распределения длин вариантов (делеций и дупликаций)
Полученные файлы собираем в один и фильруем по нужным параметрам, в нашем случае по делеции/дупликации, также можно отфильровать по отдельной хромосоме
Файл с кодом - mergefile1
Пример выходных данных:
```
             chr     start       end   diff
19    CP027619.1   5344000   5350000   6000
22    CP027619.1   5548000   5568000  20000
...          ...       ...       ...    ...
2132  CP027633.1  17538000  17551000  13000
2133  CP027633.1  17587000  17601000  14000
```
Используя этот файл строим графики распределения плотности по хромомсомам в Rsudio:

```
ggplot(file, aes(x = diff, group = chr)) + geom_line(stat = "density", aes(color = chromosome)) + theme_light() + labs(title ="Deletion",x="length", y="Density")+ coord_cartesian(xlim = c(0, 100000)) + scale_y_continuous(labels = scales::comma) + scale_x_continuous(limits = c(0, NA)) 
```
3. Построение графиков boxplot 
Аналогично с шагом 2, только в шаге 1 не учтен порядковый номер образца.
Для получения файла с пронумерованными образцами используем следующий код: boxplot1
На выходе получаем: 
```
    chr     start       end    ALT     id
0     CP027619.1    211000    216000  <DEL>        1
1     CP027619.1    324000    337000  <DEL>        1
2     CP027619.1    632000    637000  <DEL>        1
...          ...       ...       ...    ...      ...
2166  CP027633.1  21723000  21727000  <DEL>       99
2167  CP027633.1  21745000  21752000  <DEL>       99
```
4. Составление хромосомных карт. Поиск вариантов, затронутых делециями и дупликациями общих для всех образцов
4.1. Для составления хромососмных карт была использована библиотека RIdeogram (https://cran.r-project.org/web/packages/RIdeogram/RIdeogram.pdf)
Предварительно необходимо подготовить файлы, а именно указать количество совпадений, учитывая совпадение по началу и концу дупликации/делеции
Файл с кодом - Overlap1
4.2. Работа с библиотекой RIdeogram:

```
library("RIdeogram")
ideogram(karyotype = karyotype, overlaid = overlaid)
```
Необходимо правильно назвать все столбцы для успешного запуска кода
для karyotype:
```
karyotype<-overlapfile(Chr = c(overlapfile$V1), Start = c(overlapfile$V2),End = c(overlapfile$V3))
```
ДЛЯ overlaid:
```
 overlaid<-overlapfile(Chr = c(overlapfile$V1), Start = c(overlapfile$V2),End = c(overlapfile$V3),Value = c(overlapfile$V4))
```
5. GO enrichment 
Подготовка файдов: Используем файл полученный на этапе  3, дополнительно используем bedtools
```
bedtools intersect-f 0.5
```

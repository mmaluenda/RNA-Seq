# Guía para alineamiento 03, 04, 05, 06, 07

Luego trimgalore donde se eliminaron adaptadores y/o secuencias de baja calidad y solo quedan los reads "limpios" es el momento iniciar el protocolo de alineamiento.

Antes iniciar el protocolo debemos tener los inputs para correr ```STAR``` estos archivos se obtienen desde el directorio de trim ```../outputs/02_trimgalore/```, estos archivos por lo general vienen en pares (R1 y R2) con un formato ```fastq``` (```*_R1_val_1.fq.gz ``` y ```_R2_val_2.fq.gz```).

Una vez tenemos nuestros archivos de entrada podemos correr el script ```/03_star.sh``` ubicado simepre en el directorio ```../codes``` y debemos asignar un directorio para los archivos de salida, en este caso es ```../outputs/03_star``` como se muestra en este ejemplo:
```bash
$ ./03_star.sh /home/proyectos/Protocolos/RNA-seq/outputs/02_trimgalore/ /home/proyectos/Protocolos/RNA-seq/outputs/03_star/
```



Luego se carga el genoma de referencia y se toma la muestra con sus pares (```*R1*fq.gz``` y ```*R2*fq.gz```) si es que los tiene, y que fueron generados en el ``` trimgalore ```, el alineamiento se ejecuta usando el comando directamente en la terminal:
```bash
$ bowtie2 -q --phred33 -p 35 -x /home/resources/genomes/Genomes/t2t/t2t -1 H3K27ac_WT2_R1_val_1.fq.gz -2 H3K27ac_WT2_R2_val_2.fq.gz -S output.sam
```

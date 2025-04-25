# Guía para alineamiento 03, 04, 05, 06, 07

+ Alineamiento 03 STAR  

Luego trimgalore donde se eliminaron adaptadores y/o secuencias de baja calidad y solo quedan los reads "limpios" es el momento iniciar el protocolo de alineamiento.

STAR es un alineador ultrarrápido diseñado específicamente para RNA-seq, que puede alinear reads a un genoma considerando los sitios de corte y empalme (splice junctions), lo cual es clave en eucariotas.

Antes iniciar el protocolo debemos tener los inputs para correr ```STAR``` estos archivos se obtienen desde el directorio de trim ```../outputs/02_trimgalore/```, estos archivos por lo general vienen en pares (R1 y R2) con un formato ```fastq``` (```*_R1_val_1.fq.gz ``` y ```_R2_val_2.fq.gz```) y el genoma de referencia que se especifica dentro del codigo que se va a correr ```/03_star.sh```.

Una vez tenemos nuestros archivos de entrada podemos correr el script ```/03_star.sh``` ubicado simepre en el directorio ```../codes``` y debemos asignar un directorio para los archivos de salida, en este caso es ```../outputs/03_star``` como se muestra en este ejemplo:
```bash
$ ./03_star.sh /home/proyectos/Protocolos/RNA-seq/outputs/02_trimgalore/ /home/proyectos/Protocolos/RNA-seq/outputs/03_star/
```
Los archivo resultantes son: ```*.sortedByCoord.out.bam``` reads alineados (ordenados por coordenada), ```*Log.out ``` logs del proceso, ```*ReadsPerGene.out.tab ```, ```*Log.final.out ``` estadísticas del alineamiento (muy útil), ```*Log.progress.out ``` logs del proceso, ```*SJ.out.tab``` splice junctions detectadas y dos directorios con información sobre el genoma.

Cosas clave de STAR: Alinea cruzando exones (intron-aware), usa las anotaciones para mejorar la detección de splicing y produce ```*.bam``` ya listo para cuantificación o análisis downstream (HTSeq, featureCounts, Salmon, etc.)

+ 04 featureCounts

El cuarto paso es la clave de RNA-Seq, la cuantificación. Aquí tomas los reads alineados (el ```*.bam``` generado por ```STAR```) y los cuentas en relación a los genes o exones anotados en un archivo ```*.gtf```.

FeatureCounts toma tus lecturas alineadas y les asigna una etiqueta de gen, transcript o exón con base en una anotación de referencia. Básicamente, responde a la pregunta: ¿Cuántos reads caen sobre cada gen?

```bash
$  ./04_featurecounts.sh /home/proyectos/Protocolos/RNA-seq/outputs/03_star /home/proyectos/Protocolos/RNA-seq/outputs/04_counts 1
```

La ultima opción (1 en el ejemplo) corresponde al parametro -s en las tareas de featureCounts y depende del protocolo de librería usado: 0 corresponde a no stranded, 1 corresponde a stranded (sentido original) y 2 reverso (común en librerías tipo dUTP o TruSeq), al elegir cada una de estas opciones (por separado) se crea su propio directorio: ```../outputs/04_featurecounts/opt_0```, ```../outputs/04_featurecounts/opt_1``` y ```../outputs/04_featurecounts/opt_2```. Los archivos creados en este proceso son: ```*.counts``` es la tabla de conteo por gen (para DESeq2, edgeR, etc.), ```*.counts.summary``` es el resumen de cuántos reads se asignaron, descartaron, etc. y finalmente ```*_sartools.counts ``` es una versión simplificada del archivo ```*.counts```, pensada para ser directamente leída por ```SARTools``` o scripts de R, y contiene una tabla simple: una fila por gen, una columna por muestra, Encabezado limpio, directamente usable en R.

+ 05 Sartools

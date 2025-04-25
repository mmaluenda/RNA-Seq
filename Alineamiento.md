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

Los inputs que toma ```featureCounts``` provienen del directorio ```../outputs/03_star```:

```bash
$  ./04_featurecounts.sh /home/proyectos/Protocolos/RNA-seq/outputs/03_star /home/proyectos/Protocolos/RNA-seq/outputs/04_counts 1
```

La ultima opción (1 en el ejemplo) corresponde al parametro -s en las tareas de featureCounts y depende del protocolo de librería usado: 0 corresponde a no stranded, 1 corresponde a stranded (sentido original) y 2 reverso (común en librerías tipo dUTP o TruSeq), al elegir cada una de estas opciones (por separado) se crea su propio directorio: ```../outputs/04_featurecounts/opt_0```, ```../outputs/04_featurecounts/opt_1``` y ```../outputs/04_featurecounts/opt_2```. Los archivos creados en este proceso son: ```*.counts``` es la tabla de conteo por gen (para DESeq2, edgeR, etc.), ```*.counts.summary``` es el resumen de cuántos reads se asignaron, descartaron, etc. y finalmente ```*_sartools.counts ``` es una versión simplificada del archivo ```*.counts```, pensada para ser directamente leída por ```SARTools``` o scripts de R, y contiene una tabla simple: una fila por gen, una columna por muestra, Encabezado limpio, directamente usable en R.

+ 05 Sartools

Este quinto paso corresponde al análisis de expresión diferencial usando el paquete ```SARTools``` en R, que funciona como una interfaz amigable para ```DESeq2``` o ```edgeR```, en nuestro caso lo usamos para ```DESeq2```.

```SARTools``` (Statistical Analysis of RNA-seq data Tools) es un paquete en R que automatiza: Importar datos de featureCounts, normalizar la expresión, realizar análisis de expresión diferencial, y lo más importante generar informes ```HTML``` con gráficos y resultados.

Se necesitan inputs especiales para correr este código que se construyen a partir de los archivos ```*.counts``` y la manera de lanzarlo se especifica a continuación:
```bash
$  Rscript 05_SarTools.r   --projectName "IPS_NPC_iso"   --author "rcelis"   --targetFile target.txt   --rawDir raw   --varInt group   --condRef IPS   --typeTrans VST --forceCairoGraph
```
donde ```05_SarTools.r``` es el script en R, ```target.txt``` corresponde a una tabla con los dos genes especificos que se van a comparar (este archivo tiene formato de label, group y files), ```raw``` el directorio donde se encuentra la data en formato *.counts.summary, ```group``` corresponde a los dos grupos al que pertenece cada gen (por ejemplo IPS o NPC), ```--condRef IPS``` es el gen que toma de referencia, en este caso el grupo ```IPS```. 

Formato del archivo ```target.txt```
```bash
label	group	files
IPS_WT1	IPS	IPS_WT1.counts.summary
IPS_WT2	IPS	IPS_WT2.counts.summary
IPS_WT3	IPS	IPS_WT3.counts.summary
NPC_WT1	NPC	NPC_WT1.counts.summary
NPC_WT2	NPC	NPC_WT2.counts.summary
NPC_Iso	NPC	NPC_Iso.counts.summary
```

Formato del archivo ```*.counts.summary``` ubicados en el directorio ```raw``` (son muy grandes).
```bash
ENSG00000284662	0
ENSG00000186827	3
ENSG00000186891	2
ENSG00000160072	1625
ENSG00000041988	590
ENSG00000142611	439
ENSG00000067606	277
ENSG00000131584	2111
ENSG00000169972	570
ENSG00000157911	1227
ENSG00000224051	778
.
.
.
```
Estos archivos ```*.counts.summary``` se generan a partir de los ```*.counts``` usando el comando: 

```bash
$ cat *.counts| cut -f1,7 > *.counts.summary
```
donde del archivo ```*.counts``` toma solo la columna 1 y la 7 (```cut -f1,7```) que es el nombre del gen y el numero de cuentas.

Importante: estos archivos ```*.counts.summary``` se deben crear para que SARTools genere un analisis de expresion diferencial con todas las cuentas de los genes de la muestra, si se utilizan los generedos por el paso anterior ```04_featurecounts/opt_*/*.counts.summary``` hará el analisis pero con archivos mucho más resumidos y queremos que tome todos los genes.

+ 05 TPM Normalization

El paso ```05_TPM-normalizatio```n es paralelo a ```05_SARTools```, y cumple un objetivo complementario, pero distinto. Es ideal para comparar expresión absoluta entre genes o muestras, permite comparar entre genes, no hace test estadístico. TPM (Transcripts Per Million) es una forma de normalizar la expresión que corrige por la longitud del gen (genes más largos corresponden a más reads), corrige por la profundidad total de secuenciación y se expresa como "de cada millón de transcritos, cuántos vienen de este gen". 

Para correrlo se necesitan solo los archivos del directorio de ```../outputs/04_counts/``` como entrada y su respectivo directorio de salida:
```bash
$ ./05_TPM-normalization.sh ../outputs/04_counts/ ../outputs/05_tpm
```



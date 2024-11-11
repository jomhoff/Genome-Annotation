## **[Funannotate](https://funannotate.readthedocs.io/en/latest/index.html)
Funannotate is a software package that developed for functional annotation of genomes from multiple lines of evidence. 

# Train Step
Since we have a softmasked genome from Earlgrey, we start the funannotate pipeline at the train step. Here, I initiate the step with untrimmed RNA-seq data--trinity output fastas can also be used. PASA is then used to align the transcriptomes to the genomes and find putative gene-regions.

```
#!/bin/sh
#SBATCH --job-name funtrain
#SBATCH --nodes=1
#SBATCH --tasks-per-node=48
#SBATCH --mem=100gb
#SBATCH --time=100:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=jhoffman1@amnh.org

mamba init
mamba activate funannotate

cd /home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate

export FUNANNOTATE_DB=/mendel-nas1/jhoffman1/fasciatus_genome/funannotate/funannotate_db
export GENEMARK_PATH=/home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate/gmes_linux_64_4

#Train to align RNA-seq data, run Trinity, and then run PASA
funannotate train -i plestiodonFasciatus_2.softmasked.fasta -o fun_train_out \
    --left Plestiodon-fasciatus-AMNHFS20591-heart_R1_001.fastq.gz Plestiodon-fasciatus-AMNHFS20591-kidney_R1_001.fastq.gz Plestiodon-fasciatus-AMNHFS20591-liver_R1_001.fastq.gz Plestiodon-fasciatus-AMNHFS20591-lung_R1_001.fastq.gz Plestiodon-fasciatus-AMNHFS20591-muscle_R1_001.fastq.gz Plestiodon-fasciatus-AMNHFS20591-skin_R1_001.fastq.gz \
    --right Plestiodon-fasciatus-AMNHFS20591-heart_R2_001.fastq.gz Plestiodon-fasciatus-AMNHFS20591-kidney_R2_001.fastq.gz Plestiodon-fasciatus-AMNHFS20591-liver_R2_001.fastq.gz Plestiodon-fasciatus-AMNHFS20591-lung_R2_001.fastq.gz Plestiodon-fasciatus-AMNHFS20591-muscle_R2_001.fastq.gz Plestiodon-fasciatus-AMNHFS20591-skin_R2_001.fastq.gz \
    --species "Plestiodon fasciatus" \
    --max_intronlen 6000 --stranded RF
```

# Predict Step
The prediction step takes advantage of GENEMARK-ES to predict genes from functional gene databases and the output from the predict step.

```
#!/bin/sh
#SBATCH --job-name funpredict
#SBATCH --nodes=1
#SBATCH --tasks-per-node=48
#SBATCH --mem=300gb
#SBATCH --time=200:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=jhoffman1@amnh.org

mamba init
mamba activate funannotate

cd /home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate

export FUNANNOTATE_DB=/mendel-nas1/jhoffman1/fasciatus_genome/funannotate/funannotate_db
export GENEMARK_PATH=/home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate/gmes_linux_64_4

#Predict
funannotate predict -i plestiodonFasciatus_2.softmasked.fasta -o fun_train_out --species "Plestiodon fasciatus" \
    --busco_db tetrapoda \
    --organism other \
    --busco_seed_species Taeniopygia_guttata \
    --repeats2evm \
    --cpus $SLURM_NTASKS_PER_NODE
```



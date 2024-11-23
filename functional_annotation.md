# **[Funannotate](https://funannotate.readthedocs.io/en/latest/index.html)**
Funannotate is a software package that developed for functional annotation of genomes from multiple lines of evidence. 
Install tutorial [here](https://funannotate.readthedocs.io/en/latest/install.html). I used the bioconda install through mamba.

## Train Step
Since we have a softmasked genome from Earlgrey, we start the funannotate pipeline at the train step. Here, I initiate the step with untrimmed RNA-seq data--trinity output fastas can also be used. PASA is then used to align the transcriptomes to the genomes and find putative gene-regions.
```
#!/bin/sh
#SBATCH --job-name funtrain
#SBATCH --nodes=1
#SBATCH --tasks-per-node=48
#SBATCH --mem=100gb
#SBATCH --time=50:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=jhoffman1@amnh.org

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

## Predict Step
The prediction step takes advantage of GENEMARK-ES to predict genes from functional gene databases and the output from the predict step. You have to install GeneMark seperately; you can do so [here](https://topaz.gatech.edu/GeneMark)
```
#!/bin/sh
#SBATCH --job-name funpredict
#SBATCH --nodes=1
#SBATCH --tasks-per-node=48
#SBATCH --mem=200gb
#SBATCH --time=20:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=jhoffman1@amnh.org

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

## Update Step
Since I am using RNA-seq data, the update step is used to add UnTranslated Regions (UTRs) to the gene predictions and fix gene models that disagree with the RNA-seq data.
```
#!/bin/sh
#SBATCH --job-name funupdate
#SBATCH --nodes=1
#SBATCH --tasks-per-node=12
#SBATCH --mem=100gb
#SBATCH --time=100:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=jhoffman1@amnh.org

mamba activate funannotate

cd /home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate

export FUNANNOTATE_DB=/mendel-nas1/jhoffman1/fasciatus_genome/funannotate/funannotate_db
export GENEMARK_PATH=/home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate/gmes_linux_64_4

funannotate update -i fun_train_out --cpus $SLURM_NTASKS_PER_NODE
```

## Fix Step
If there funannotate-update.log file mentions any issues with predicted gene-models, you may need to remove them manually .tbl file and run the following code:
```
#!/bin/sh
#SBATCH --job-name funfix
#SBATCH --nodes=1
#SBATCH --tasks-per-node=12
#SBATCH --mem=5gb
#SBATCH --time=2:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=jhoffman1@amnh.org

mamba activate funannotate

cd /home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate

export FUNANNOTATE_DB=/mendel-nas1/jhoffman1/fasciatus_genome/funannotate/funannotate_db
export GENEMARK_PATH=/home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate/gmes_linux_64_4

funannotate fix -i fun_train_out/update_results/Plestiodon_fasciatus.gbk -t fun_train_out/update_results/archive_01582eca-5ec0-45ae-86bc-a17384b981b7/Plestiodon_fasciatus.tbl
```

## InterProScan 
Next, I used InterProScan to run the predicted genes against the InterPro database for gene families. InterProScan needs to be downloaded indepentely, which can be done by following the intructions [here](https://interproscan-docs.readthedocs.io/en/latest/InstallationRequirements.html). To run on a cluster, you have to edit the interproscan.properties file to match the system you're running on. For slurm, see mine [here]()
```
#!/bin/sh
#SBATCH --job-name funIntprscn
#SBATCH --nodes=1
#SBATCH --tasks-per-node=10
#SBATCH --mem=50gb
#SBATCH --time=50:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=jhoffman1@amnh.org

mamba activate funannotate

cd /home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate

funannotate iprscan -i fun_train_out --cpus $SLURM_NTASKS_PER_NODE -m local --iprscan_path /home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate/my_interproscan/interproscan-5.71-102.0/interproscan.sh
```

## Annotate
This is the final step of the annotation pipeline--it assigns functional annotations to the protein coding gene models. 
```
#!/bin/sh
#SBATCH --job-name funAnnotate
#SBATCH --nodes=1
#SBATCH --tasks-per-node=40
#SBATCH --mem=300gb
#SBATCH --time=100:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=jhoffman1@amnh.org

mamba activate funannotate

cd /home/jhoffman1/mendel-nas1/fasciatus_genome/funannotate

funannotate annotate -i fun_train_out --busco_db tetrapoda --cpus 40
```


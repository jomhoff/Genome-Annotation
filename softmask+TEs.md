## **[Earlgrey](https://github.com/TobyBaril/EarlGrey)**
Earlgrey is a wrapper that runs Repeat Modeler, Repeat Masker, and then uses a Blast, Extent, Align, and Trim process to identify TEs from TE databases. It outputs a softmasked genome, TE annotations, TE quanitifcations, and nifty figures.

```
#!/bin/sh
#SBATCH --job-name earlgrey
#SBATCH --nodes=1
#SBATCH --tasks-per-node=48
#SBATCH --mem=300gb
#SBATCH --time=200:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=jhoffman1@amnh.org

#activate earlgrey with mamba before submitting
mamba init
source ~/.bash_profile
mamba activate earlgrey

genome="/home/jhoffman1/mendel-nas1/fasciatus_genome/KY_LR/assembly/hoff_pfas_total.fasta"

earlGrey -g $genome -s plestiodonFasciatus_2 -t $SLURM_NTASKS_PER_NODE -d yes -o ./earlgreyOutputs
```


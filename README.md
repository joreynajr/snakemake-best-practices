# snakemake-best-practices
Personal best practices with Snakemake

Description: As I've become more familiar with snakemake there are a few tips, tricks, practices that I found extremely helpful. I always evolving my practices so this document is a statement of my current Snakemake understanding. Currently I only intend it for personal use but if this is helpeful to other then feel free to take a look. I would add that I try to come up with the good data management practices so these best-practices are not only limited to Snakemake. 

Table of contents:
- Data Provenance & Freezing
- Reproducibility 


## Data Provenance & Freezing
This section is meant to capture situation where you want to keep/freeze your data. 


## Other
- Don't use hardlinking between rules. If fileA has timestamp Jan 1 00:00 and fileB is a hardlink made 2 weeks later later then its timestamp will be Jan 15 00:00, however since fileA and fileB are pointing to the same inode fileA's timestamp will be updated to Jan 15 00:00 which will result in Snakemake thinking fileA has been updated and the whole pipeline will be re-run. Use a symlink instead or copy the entire file. If you really need to make a hardlink then there are two options: 1) limit hardlinks to rules whose output is not used in later steps or 2) use the Snakemake `ancient()` function for ALL rules which use this hardlinked file as input.  

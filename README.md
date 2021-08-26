# snakemake-best-practices
Personal best practices with Snakemake

Description: As I've become more familiar with snakemake there are a few tips, tricks, practices that I found extremely helpful. I always evolving my practices so this document is a statement of my current Snakemake understanding. Currently I only intend it for personal use but if this is helpeful to other then feel free to take a look. I would add that I try to come up with the good data management practices so these best-practices are not only limited to Snakemake. 

Table of contents:
- Data Provenance & Freezing
- Reproducibility 

## Data Freezing & Provenance
This section is meant to capture situation where you want to keep/freeze your data. Motivation, there are a lot of times where you are asked to re-run a pipeline where several of the inputs are the same but a few of them are changed. A good example I can give for bioinformatics would be running a pipeline on GRCh37, which means using a reference file somewhere at the VERY beginning of a pipeline, and then wanting to see the results with GRCh38. To handle this situation I have created the `results/freeze`. All previously run data can be saved/moved within `results/freeze` under some appropriate name and now the new GRCh38 reference can be run within the main results directory. 

However, there is another situation, it's is often the case that you need to change some input within an intermiedate rule, to handle this situation you can either copy all the data over to the datafreeze or, if you have very large files you can use softlinks, however you should make sure the files are protected so files are not easily overwritten. 

## Reproducibility
Coming soon...

## Other
- Don't use hardlinking between rules. If fileA has timestamp Jan 1 00:00 and fileB is a hardlink made 2 weeks later later then its timestamp will be Jan 15 00:00, however since fileA and fileB are pointing to the same inode fileA's timestamp will be updated to Jan 15 00:00 which will result in Snakemake thinking fileA has been updated and the whole pipeline will be re-run. Use a symlink instead or copy the entire file. If you really need to make a hardlink then there are two options: 1) limit hardlinks to rules whose output is not used in later steps or 2) use the Snakemake `ancient()` function for ALL rules which use this hardlinked file as input.  

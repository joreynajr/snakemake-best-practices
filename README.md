# snakemake-best-practices
Personal best practices with Snakemake 🐍

Description: As I've become more familiar with snakemake there are a few tips, tricks, practices that I found extremely helpful. I always evolving my practices so this document is a statement of my current Snakemake understanding. Currently I only intend it for personal use but if this is helpeful to other then feel free to take a look. I would add that I try to come up with the good data management practices so these best-practices are not only limited to Snakemake. 

Table of contents:
- Developing a Pipeline 
- Data Freezing, Provenance and Re-running the Pipeline 
- Reproducibility 

## Developing a Pipeline
When you first start using Snakemake it's quite easy to build rules on top of one another but pretty soon you have a web of rules that you have to handle. Before running your pipeline on a large set of samples it is best to run it on a single sample and using `--dry` and personally I think it's best to run each sample independently, things can go wrong within any rule. I also find it very important to produce the dag and visualize it. The best way I have found is to use: 1) `snakemake --jobs 1 --dag <target> | dot -Tsvg > YYYY.MM.DD.<nameOfJob>.svg` and 2) open the svg within Inkscape. There are a few dags you can visualize actually and I will list then below:
- dag:
  -  best for debugging
  -  shows where the wildcards are first assigned
  -  uses solid borders for out-dated/unrun rules/files and dashed borders for up-to-date rules/files
- rulegraph:
  -  best for final pipeline visualization with **rules only**
  -  DOES NOT shows where the wildcards are first assigned (makes for a cleaner looking dag)
  -  DOES NOT use solid/dashed borders for rule/file status
- filegraph:
  - best for final pipeline visualization with **file paths**
  - shows wildcards in all rules/files (makes a messier looking dag)
  - can also be used for debugging however you have to replace the wildcards 
  - DOES NOT use solid/dashed borders for rule/file status

New idea: When running a new sample make a new dag image (i.e. results/main/<sample>/dags/<date>.<target rule>.svg).  

## Data Freezing, Provenance and Re-running the Pipeline 
This section is meant to capture situation where you want to keep/freeze your data. Motivation, there are a lot of times where you are asked to re-run a pipeline where several of the inputs are the same but a few of them are changed. A good example I can give for bioinformatics would be running a pipeline on GRCh37, which means using a reference file somewhere at the VERY beginning of a pipeline, and then wanting to see the results with GRCh38. To handle this situation I have created the `results/freeze`. All previously run data can be saved/moved within `results/freeze` under some appropriate name and now the new GRCh38 reference can be run within the main results directory. 

Summary:
1) Run the pipeline on the old data (or already have it from a previous run)
2) Make a new folder within `results/freeze/<data freeze version XX>`
3) Move the old data into this data freeze folder 
4) [optional] Copy with softlinking the directory branches you want to 
keep the same between the frozen data and the new data. Best to use 
`cp -r -s --preserve=timestamps <absPath to frozen branch> <new path>`
5) Run the pipeline on the new data

However, there is another situation, it's is often the case that you need to change some input within an intermediate rule, to handle this situation you can either copy all the data over to the data freeze or, if you have very large files you can copy the whole directory tree using softlinks for the files, make sure your large files are write-protected. To do this I will follow these steps:
1) Run the pipeline on the old data (or already have it from a previous run)
2) Make a new folder within `results/freeze/<data freeze version XX>`
3) Move the old data into this data freeze folder 
4) Copy the directory tree from part 3 using `cp -r -s {old tree absolute path} {new tree relative path}`
5) Remove the symlink to the file you want to replace (IMPORTANT)
6) Copy/add the new input file accordingly 
7) Run `snakemake --jobs 1 --detailed-summary {target}`
8) Search the detailed summary for files that will be updated as a result of adding the new input file
9) Remove all symlinked files which are listed in step 8
10) Run `snakemake --profile {profile} {target}` to generate your new set of results 

Other:
- I'm trying to understand how to best protect files especially frozen ones where you still want to share some files. `chmod 444` can make a file write protected. I was also thinking of `chattr i` which makes a file immutable but you also can't make any kind of link (soft nor hard) to this file as well as chattr's limitation to certain file systems. 

## Reproducibility
### Conda Environments
I have struggled with this one for a long time. Here are a few caveats with conda, you can always Google a packages and software with conda and you'll get the code that would make this possible. Now, if your project is older then the packages have all updated and conda stops supporting them so making yaml files of old environments can be problematic, causing you to remove versions of packages in favor of the latest release that conda will automatically install. I just want to add some concreteness to what I'm trying to say, let's say we have the following yaml file:
channel:
  - bioconda
 pkgs:
  - qt=5.5.0

If we go to https://anaconda.org/conda-forge/qt/files this qt version is not possible so installing this environment will fail. 

The best thing to remedy this issue is to just create a new Python environment and try to re-run the whole pipeline with the lastest version of all packages. Make sure to install  packages in a very minimal way so that you can create an clean and portable yaml file.

Another thing, some linux based tools are also made available through conda but they seem to fail when installing via Snakemake. You tend to run into a Multidownload error. 

## Other
- Don't use hardlinking between rules. If fileA has timestamp Jan 1 00:00 and fileB is a hardlink made 2 weeks later later then its timestamp will be Jan 15 00:00, however since fileA and fileB are pointing to the same inode fileA's timestamp will be updated to Jan 15 00:00 which will result in Snakemake thinking fileA has been updated and the whole pipeline will be re-run. Use a symlink instead or copy the entire file. If you really need to make a hardlink then there are two options: 1) limit hardlinks to rules whose output is not used in later steps or 2) use the Snakemake `ancient()` function for ALL rules which use this hardlinked file as input.  
- It's not possible to have a file with read/write but no delete. The best option would be to change a files permission to read only and when you want to modify it you change the permssion temporary. 
- Cooler files are hard to integrate into snakemake since the cooler commandline tools modifies the file inplace, the best solution for proper snakemake integration is to copy the file and modify this copied file. 

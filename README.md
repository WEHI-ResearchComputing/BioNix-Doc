# WEHI-Workflow-Onboarding
This repo will contain introductory documentation to workflow management system (WMS) used at WEHI, namely <a href="https://github.com/PapenfussLab/bionix">BioNix</a> and <a href="https://snakemake.readthedocs.io/en/stable/">Snakemake</a>. Both are tools that are able to create highly reproducible pipelines in bioinformatics and can be executed via HPC, there are also other WMS such as <a href="https://www.nextflow.io/">Nextflow</a> and <a href="https://galaxyproject.org/">Galaxy</a>.

Suggestions and useful resources to BioNix and Snakemake will be detailed below respectively, particularly dedicated for concurrent and future WEHI student interns to gain understanding of the workflows or to get a head start on working with either workflows for their project.

For context, here is a description of my <a href="https://wehi-researchcomputing.github.io/gsoc">project's</a> scope.
Note for students: dev work is preferred to be done on your local machine unless you need to run big datasets which may require HPC, i.e. via SLURM.

## BioNix
<a href="https://github.com/PapenfussLab/bionix">BioNix</a> provides computational reproducibility, where it unifies workflow engines, package managers, and containers. It is implemented as a library on top of <a href="https://nixos.org/">Nix</a> and does not require any other dependencies.

Must read: <a href="https://doi.org/10.1093/gigascience/giaa121">Article about BioNix Workflow</a>

### Installation
Follow https://github.com/PapenfussLab/bionix installation steps for Nix, or from the official site: https://nixos.org/download.html. Provided root access, it is simply 
```
curl -L https://nixos.org/nix/install | sh
```

### On WEHI Slurm Shell Access
```
module load rc-tools
```
Note: However, you need to load rc-tools everytime and Nix is only able to be run in vc7-shared node. Run ```ssh vc7-shared``` and enter your WEHI password to switch nodes.

### Learning Nix

### Starter Examples
The example in <a href="https://github.com/PapenfussLab/bionix">BioNix</a>'s repo is a good way to run your first workflow.

### Configuration Settings


## Snakemake
<a href="https://snakemake.readthedocs.io/en/stable/">Snakemake</a> is another popular tool for reproducible and scalable data analyses, where its workflows are more easily readable for users familiar with Python since Snakemake is a Python based language. There are many tutorials and documentation on Snakemake's website to learn the language.

### Installation
Follow https://github.com/WEHIGenomicsRnD/qc-pipe for installation steps via a conda-based Python distribution, which suggests to use Mambaforge as it is faster.
```
mamba create -c conda-forge -c bioconda -n snakemake snakemake
```
Notice that once installed and restarted shell as prompted, you will see that (base) which denotes the environment you are in. Activating Snakemake via ```conda activate snakemake``` as shown in the <a href="https://github.com/WEHIGenomicsRnD/qc-pipe">guide</a> will change environments from (base) to (snakemake) and you can start using snakemake commands.
Another easier way is with nixpkgs:
```
nix run 'github:nixos/nixpkgs/release-22.05#snakemake' -- --help
```
To activate Snakemake in Nix, instead run
```
nix shell nixpkgs/release-22.05#snakemake
```
Similarly as mentioned, you may need to load rc-tools every login to the vc7-shared node.

### On WEHI Slurm Shell Access
```
module load snakemake
```
or using nixpkgs like mentioned:
```
module load rc-tools
nix run 'github:nixos/nixpkgs/release-22.05#snakemake' -- --help
```

### Learning Snakemake
You should start with going through this doc: https://snakemake.readthedocs.io/en/stable/tutorial/basics.html

### Starter Examples

# WEHI-BioNix-Introduction
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
Here are some useful links to start learning about Nix as a functional programming language.  
https://github.com/tazjin/nix-1p  
https://nix.dev/  
https://nixos.wiki/  
<a href=https://github.com/WEHI-ResearchComputing/BioNix-Training>Justin's Workshop</a>

### BioNix Workflow Examples
The example in <a href="https://github.com/PapenfussLab/bionix">BioNix</a>'s repo is a good way to run your first workflow. Another more complicated example of BioNix pipeline can be found at https://github.com/jbedo/malaria-variant-calling.

### Configuration Settings
For the BioNix project, we would like to utilise the nix flakes, this allows us to create builds in a strictly determined environment and create same outputs on different machines. Once you have installed Nix, add the following line to ``/etc/nix/nix.conf``
```
experimental-features = nix-command flakes
```
If you're operating on WSL like me, add to `~/.config/nix/nix.conf` (where the current shell user is a nix trusted user).  
To check if the installation is successful, input the following command line
```
$ nix flake --help
```

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
You can start with going through this doc: https://snakemake.readthedocs.io/en/stable/tutorial/basics.html


# Kickstart to Bionix
From this section onwards, the focus will be placed on BioNix, containing my notes on certain important topics to be able to start wrapping software tools in BioNix. On top of that, please refer to the official manual, code from nixpkgs GitHub repository or written guides for technical details and explanations that are available online for a better understanding if necessary.

## Set Up prior to Development
Before coding in Nix, you would first need an editor. Of course, choosing editors always comes down to preference, any popular editors out there may be sufficient i.e. Vim. Personally, I use VS Code as my editor because of the available nix extension like code formatter. Other than that, I find it useful that viewing derivation log files or content of the outputs is easy in VS Code, for example clicking the `/nix/store/example-output.drv` opens a tab to view the .drv derivation file, or `/nix/store/example-output-directory` can open the directory in your explorer, this comes in handy when you're debugging or checking your file outputs.

## nix build
When you want to test your workflow, the ``nix build`` command should be utilised, where a symlink to the result will be created in the directory called "result" by default if the build is successful (no errors in your build). Usually, if no paths are specified, ``nix build`` will use default.nix in the current directory, i.e. when running ``$ nix build`` in the CLI. Hence, when you want to write a workflow, it should be orchestrated in a default.nix file at the top level of that directory. Additionally, you can run ``nix build -o foo-output-name`` to give the output directory a specific name rather than "result".

## default.nix
### Importing BioNix
Usually, you would start a workflow with ``{ bionix ? import <bionix> { } }:`` This would ensure that bionix is imported, otherwise if it's not already imported it would default to `import <bionix> {}`. If you receive some errors, it could be that calling the bionix package is missing in the flake.nix 

### With expression
The `with` expression includes symbols into the scope of the inner expression. If you're using any function or tools from BioNix multiple times, you may include `with bionix;`, otherwise you need to write `bionix.foo-function` multiple times in your code. It behaves like `from module import *` from Python.

### Fetching files
Let's say you need to copy a file that is available online for your code, a direct way is to use `fetchurl`, which will require arguments of a URL and a hash. In Nix, you can define a variable `foo` for the file you are copying, for example:
```
foo = fetchurl {
    url = "http://www.example.org/hello-1.0.tar.gz";
    sha256 = "0v6r3wwnsk5pdjr188nip3pjgn1jrn5pc5ajpcfy6had6b3v4dwm";
}
``` 
Simply paste the url to the file as the value to the attribute `url`. To obtain the hash, I usually use `nix-prefetch-url --type sha256 yourUrlHere` to get the hash, generally sha256 is recommended. Note that the utilisation of hash here ensures consistency and specific versions of your file sources that are being fetched.  
You may check this <a href=https://ryantm.github.io/nixpkgs/builders/fetchers/>page</a> or the manual out for alternate fetchers and their examples.

### Fetching file of specific types in BioNix
There are some specific file types that are common in bioninformatics. There already are functions written in Bionix to check and fetch the specific file types, and they should be used when fetching files of those types, namely ``fetchFastQ``, ``fetchFastA``, ``fetchFastQGZ``, and ``fetchFastAGZ``. (Refer to <a href=https://github.com/PapenfussLab/bionix/blob/81cfa50e6b345942951b68eac0e184ea025f2ae4/default.nix#L121-133>code</a>.)

### Let expressions
Now, you will need to write Nix's <a href=https://nixos.org/guides/nix-pills/basics-of-language.html>let expressions</a> to run your workflow. Where you can define all the required sample files, and also pipeline of the workflow. Here is a simplified example workflow written in default.nix in BioNix:
```
{ bionix ? import <bionix> { } }:

with bionix;
with lib;

let
  # fetch an ecoli sample FastA file using fetchFastA rather than fetchurl
  ecoli = fetchFastA {
    url = "https://github.com/WEHIGenomicsRnD/qc-pipe/raw/main/.test/data/GCF_013166975.1_ASM1316697v1_genomic.fna";
    sha256 = "0rcph75mczwsn6q7aqcpdpj75vjd9v2insmhnf8dmcyyldz25dqi";
  };
in
exampleSoftware.tool { } ecoli
```

## nix flake
As mentioned earlier, configuration was made to enable Nix's experimental feature of using flakes. This Nix's feature is one of the significant feature for BioNix, which contributes to guaranteeing reproducibility within a workflow. There are a couple of useful subcommands, where I will mention a few, you can check out <a href=https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake.html>manual</a> for more.  
`nix flake init` - creates a template flake in your current directory (an alternate template specific to BioNix framework will be presented later)  
`nix flake show` - shows you the outputs of a flake (may be quite useful IMO)

## flake.nix
Flake is also important for reproducibility as it keeps track of all versions of the dependencies when a build is executed. If you have a `flake.nix` file in your directory, when you first run `nix build`, it will generate a `flake.lock` file. With the lock file, building the same directory in different machines guarantees high reproducibility of the same output in the future, as flake.lock contains the records of all versions of packages used at the time of build. If you want to use the latest version of certain packages imported via your flake, where the `flake.lock` was generated a while ago and the package versions were not specified in your `flake.nix`, you can run `nix build --recreate-lock-file` when building the derivation. This creates a new lock file specifying the latest versions of the `inputs` (again, if they are not specified in your flake).

### Template
For BioNix, here is an example `flake.nix` template that I suggest which utilises flake utility functions by <a href=https://github.com/numtide/flake-utils>numtide/flake-utils</a>. Also, check out <a href=https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake.html#flake-format>manual</a> for descriptions of each attributes and the format of `flake.nix`. Note that there are many alternate packages developed to enhance flakes other than numtide/flake-utils. There are also many well written guides that can be found online, such as this <a href=https://blog.kubukoz.com/flakes-first-steps/>page</a>.
```
{
  description = "Flake for yourSoftware";

  inputs = {
    bionix.url = "github:papenfusslab/bionix";
    nixpkgs.url = "github:nixos/nixpkgs";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, bionix, flake-utils }:
    flake-utils.lib.eachDefaultSystem
      (system: with bionix.lib
        {
          overlays = [ (self: super: { example = self.callBionix ./example.nix { }; }) ];
          nixpkgs = import nixpkgs { inherit system; };
        };
      {
        defaultPackage = callBionix ./. { };
      }
      );
}
```
For the above example, the versions of the `inputs` were not specified, hence `nix build` takes the latest versions and running the build at different times may not reproduce the outcome. This is where the `flake.lock` file comes into play.

### overlays
Here, take note on the line `overlays = [ (self: super: { example = self.callBionix ./example.nix { }; }) ];`, we are using `overlays` to include new files for wrapping a new software. For instance, you can make a new directory, and include the `overlays` in your flake.nix so that you can call the wrapped software in your workflow(default.nix) through BioNix, i.e. `bionix.example { }`.  

In other words, without overlays, if you want to call the tools through BioNix in your workflow, you would have to clone the directory and the wrapper files would need to be placed in the bionix/tools directory. Moreover, when specifying the BioNix in `inputs` in your flake, you would now need to specify it to the cloned BioNix directory where the new files are located, rather than like the example above which refers to the online repository. Without doubt, this method is not smart and may introduce complexity such as during version control, i.e. when making pull requests. Thus, it is recommended to use overlay to make wrapping software and testing easier.

## Wrapping Software
Now that we have some basics about workflow and flakes, we can move on to the process of wrapping new software to make it available in bionix. 

## Derivations
`derivation` is one of the most important built-in function from Nix, it describes a build action. However, we will look at `stdenv.mkDerivation` because most packages uses `mkDerivation` as the base wrapper rather than basic `derivation` to package software and it also pulls in all the `stdenv` toolchain and basic tools needed to compile packages. It takes a set of attributes that defines the input of a build, where compulsory attributes are `name`, `buildInputs`, etc. and optional attributes such as `outputs`. This <a href=https://ryantm.github.io/nixpkgs/stdenv/stdenv/>guide</a> is a good read about stdenv.mkDerivation. Below is an example code for building a derivation.

### example-app.nix (Example BioNix Derivation)
```
{ bionix, fetchurl, stdenv }:

with bionix;

stdenv.mkDerivation rec {
  name = "foo-software-${version}";
  version = "1.0";
  src = fetchurl {
    url = "https://example.com.tar.gz";
    sha256 = "sha256-AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=";
  };
  buildInputs = with pkgs; [ unzip ];
}
```
This derivation creates an environment for a software package, and can be used as inputs to other derivations. Note that software versions for the derivations need to be specified and any hashes need to match in order to reproduce the same derivation path. Moreover, the default builder for stdenv.mkDerivation is bash

Explanations from <a href=https://nixos.org/guides/nix-pills/fundamentals-of-stdenv.html>Chapter 19</a> and <a href=https://nixos.org/guides/nix-pills/basic-dependencies-and-hooks.html>Chapter 20</a> of Nix Pills will be very helpful for this topic.

## Specifying Stage
As mentioned in the BioNix's article, stages in a BioNix workflow specifies the entire computation environment. It tracks dependencies down to kernel level and the stage is executed in its own sandbox, which is one of the main factors to guaranteeing reproducibility. Below is an example where an example stage "tool" is specified.

### example.nix
```
{ bionix }:

with bionix;

rec {
  appName = pkgs.callPackage ./example-app.nix;
  tool = callBionixE ./example-tool.nix;
}

```
The example.nix file will be accessible via an overlay as shown in the example flake.nix earlier. This file contains some variables defined to call the other packages/derivations or stages.

### example-tool.nix
```
{ bionix 
, ref
}:

{ input1
, input2
}:

with bionix;

stage {
  name = "foo";
  buildInputs = with bionix; [ example.appName ];
  outputs = [ "out" ];
  buildCommand = ''
    mkdir -p $out/example-tool
    tool --input ${input1} ${input2} --ref ${ref}
  '';
}
```
Using the derivation of the target software (example-app.nix), we can parse that derivation as our buildInputs. For demonstration purposes, the line `tool --input ${input1} ${input2} --ref ${ref}` was made up. Depending on how the software can be ran through their CLI, your buildCommand will always vary. Note that `${}` is used for parameter substitution or variable expansion in a shell script. `$()` for command substitution might also be used.

Typically, BioNix functions are written in a way as `config -> inputs -> drv`. In this case, `ref` which may be a reference genome (for example), may be considered as configuring the software, then followed by parsing in the inputs (input1 and input2) before giving us an output. Hence, executing this stage (through <a href=https://github.com/PapenfussLab/bionix/blob/81cfa50e6b345942951b68eac0e184ea025f2ae4/default.nix#L7-L15>callBionixE</a>) in a workflow would look like: 
```
with bionix;
tool { inherit ref; } { inherit input1 input2; }
```
<!-- ## Additional Notes -->


<!-- ### Debugging/Testing
Depending on the error you get, you might need to debug and find the root cause using different methods
`nix shell`  
`nix develop` -->

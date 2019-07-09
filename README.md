# MAGotator

MAGotator (MAG (Metagenome Assembled Genome) annotator) is a tool for annotating metagenomic assembled genomes that 
outputs predicted metabolisms. It will annotated MAG's using [KEGG](https://www.kegg.jp/) (if provided by the user), 
[UniRef90](https://www.uniprot.org/), [PFAM](https://pfam.xfam.org/), [dbCAN](http://bcb.unl.edu/dbCAN2/), 
[RefSeq viral](https://www.ncbi.nlm.nih.gov/genome/viruses/) and the [MEROPS](https://www.ebi.ac.uk/merops/) peptidase 
database. MAGotator is ran in two stages. First the annotation is done to give a full summary of the provided MAGs with 
all annotations from these databases. Second the annotations are summarized to give an idea of what metabolisms each 
genome is capable of. These summarizations are given in the form of tables. One table is counts of genes organized by
the metabolisms they are a part of and the other is a measure of completeness of KEGG modules.

## Installation
To install MAGotator some dependencies need to be installed first then MAGotator can be installed from this repository. 
In the future MAGotator will be available via both pip and conda.

0. Install Dependencies
    
    Dependencies can be installed via conda or by manually installing all dependencies.
    
    _Conda Installation_
    
    Installed MAGotator into a new [conda](https://docs.conda.io/en/latest/) environment using the provided 
enviornment.yaml file.
    ```bash
    wget https://raw.githubusercontent.com/shafferm/checkMetab/master/environment.yaml
    conda env create -f environment.yml -n MAGotator
    ```
    If this installation method is used then all further steps should be ran 

    _Manual Installation_
    
    If you do not install via a conda enviornment the dependencies [pandas](https://pandas.pydata.org/), 
    [networkx](https://networkx.github.io/), [scikit-bio](http://scikit-bio.org/),
    [prodigal](https://github.com/hyattpd/Prodigal), [mmseqs2](https://github.com/soedinglab/mmseqs2), 
    [hmmer](http://hmmer.org/) and [tRNAscan-SE](http://lowelab.ucsc.edu/tRNAscan-SE/) manually.

1. Download this repository using `git clone https://github.com/shafferm/checkMetab.git`
2. Change directory into the MAGotator directory and install MAGotator using `pip install -e .`

You have now installed MAGotator.

## Setup

To run MAGotator you need to set up the databases it needs in order to get annotations from those databases. All 
databases but KEGG can be downloaded and set up for use with MAGotator for you automatically. To get KEGG annotations 
and fully take advantage of the genome summarization capabilities of MAGotator you must have access to the KEGG 
database. KEGG is a paid subscription service to download the protein files used by this annotator. If you do not have 
access to KEGG then your data can be annotated with all other databases. Genome summarization will summarize the tRNAs, 
peptidases and CAZy's in your data set but not the primary metabolisms.

_I have access to KEGG_

Then set up MAGotator using the following command:

```bash
MAGotator.py prepare_databases --output_dir MAGotator_data --kegg_loc kegg.pep
```

`kegg.pep` is the path to the amino acid FASTA file downloaded from KEGG. This can be any of the single files 
provided by the KEGG FTP server or a concatenated version of the multiple provided files. `MAGotator_data` is the path 
to the processed databases used by MAGotator. If you already have any of the databases downloaded to your server and 
don't want to download them again then you can give them to the `prepare_databases` command by use the `--{db_name}_loc`
 flags such as `--uniref_loc` and `--viral_loc`.

_I don't have access to KEGG_

Not a problem. Then use this command:

```bash
MAGotator.py prepare_databases --output_dir MAGotator_data --kegg_loc kegg.pep
```

Similar to above you can still provide locations of databases you have already downloaded so you don't have to do it
again.

To see that your set up worked use the command `MAGotator.py print_config` and the location of all databases provided 
will be shown as well as the presence of additional annotation information.

*NOTE:* Setting up MAGotator can take a long time (up to 5 hours) depending on the number of processors which you tell 
it to use (using the `--threads` argument) and the speed of your internet connection. On my university server using 10 
processors it takes about 2 hours to process the data when databases do not need to be downloaded.

## Usage

Once MAGotator is set up you are ready to annotate some MAGs. The following command will generate your full annotation: 

```bash
MAGotator.py annotate -i 'my_bins/*.fa' -o annotation
```

`my_bins` should be replaced with the path to a directory which contains all of your bins you would like to annotated. 
If you only need to annotated a single genome (or an entire assembly) a direct path to a nucleotide fasta should be 
provided. Using 20 processors MAGotator.py takes about 17 hours to annotate ~80 MAGs of medium quality or higher from a 
mouse gut metagenome.

In the output `annotation` folder there will be various files. `genes.faa` and `genes.fna` are fasta files with all 
genes called by prodigal with additional header information gained from the annotation as nucleotide and amino acid 
records respectively. `genes.gff` is a GFF3 with the same annotation information as well as gene locations.
`scaffolds.fna` is a collection of all scaffolds/contigs given as input to `MAGotator.py annotate` with added bin 
information in the headers. `annotations.tsv` is the most important output of the annotation. This includes all 
annotation information about every gene from all MAGs. Each line is a different gene and each column contains annotation
 information. `trnas.tsv` contains a summary of the tRNAs found in each MAG.

Then after your annotation is finished you can summarize these anntotations with the following command:

```bash
MAGotator.py summarize_genomes -i annotation/annotations.tsv -o genome_summaries --trna_path annotation/trnas.tsv
```

This command will generate two files. The first is called `genome_summary.tsv` this contains a summary of  metabolisms 
present in each genome. It gives gene by gene information across various metabolisms for every genome in your dataset. 
The `module_summary.tsv` file contains all KEGG modules and will give completion statistics in terms of the number of 
steps in each module that have been covered by at least one gene in each genome.

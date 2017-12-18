# metaWRAP - Comprehensive Metagenomic Bin Analysis in One Place
## metaWRAP v=0.5

 MetaWRAP aims to be an easy-to-use inclusive wrapper program that accomplishes the most basic tasks in metagenomic analysis: QC, assembly, binning, visualization, and taxonomic profiling. While there is no single best approach for processing metagenomic data, metaWRAP is meant to be a fast and simple first pass program before you delve deeper into parameterization of your approach. Each individual component of the pipeline is also a standalone module. This modularity allows the users to use only the modules they are interested in. 
 
![General walkthrough of metaWRAP modules](https://i.imgur.com/LcC09ym.png)
   
 
 In addition to being a tool wrapper, MetaWRAP offers a innovative hybrid pipeline for extracting high-quality draft genomes (bins) from metagenomic data. By using a variety of software (metaBAT2, CONCOCT, MaxBin2) and utilizing their individual strengths and minimizing their weaknesses, the [bin refinement module](https://i.imgur.com/JL665Qo.png) will always produce stronger results than individual approaches. Additionally, due to its diverse binning approach, this pipeline shows promise to produce robust binning results in a variety of microbial communities. 

 MetaWRAP also includes a [bin reassembly module](https://i.imgur.com/GUSMXl8.png), which allows to drastically improve the quality of a set of bins by extracting the reads belonging to that draft genome, and reassembling it with a more permissive, non-metagenomic assembler. In addition to improving the N50 of the bins, this modestly increases the compleiton of the bins, and drastically reduces contamination.
  
 If you already have your metagenomic data assembled and binned with two or more software (or the same software with different parameters), try using the BIN_REFINEMENT and REASSEMBLE_BINS modules to see how you can further improve your bin predictions! 
  

## OVERVIEW OF METAWRAP MODULES:
  
#### Metagemonic data pre-processing modules:
		1) Read QC (trimming and human read removal)
    	2) Assembly (with metaSPAdes or MegaHit, plust assembly QC)
		3) Kraken (taxonomy profiling and visualization)
    	4) Binning (MaxBin2, metaBAT2, CONCOCT)
	
#### Bin processing modules:
		1) Bin refinement and consolidation of multiple bin sets
		2) Bin reassembly (reassemble bins to improve completiona and reduce contamination)
		3) Bin quantitation (bin abundance estimation across samples)
    	5) Blobology (visualize bin success with blobplots)
		6) Classify bins (asign taxonomy to draft genomes)
  

  
## INSTALLATION

 Clone or download the metaWRAP directory into a semi-permanent location, then go into the metaWRAP/bin folder and edit the metaWRAP/bin/contig-metawrap file. Make sure that all the paths are correct, especially the paths pointing to the "scripts" and "pipelines" folders in the main metaWRAP directory. Once that is configured, simply copy the contents of metaWRAP/bin into your local bin folder, or simply add metaWRAP/bin/ to your path:
 
 ```
 echo "export PATH="/full/path/to/metaWRAP/bin:$PATH"" >> ~/.bash_profile
 source ~/.bash_profile
 ```
 
 No try running ```metaWRAP -h``` to see if everything works!



## DEPENDENCIES

  Since this is a wrapper program, the biggest challenge in installing metaWRAP will likely be configuring all the dependencies correctly. A complete list of dependancies can be viewed in "dependancies.txt". Feel free to install them one by one. However, to make this as painless as possible, I HIGHLY recommend you use Conda, which automatically installs programs and handles dependancies. To start, download [miniconda2](https://conda.io/miniconda.html) and install it. Then create a new environment and install all the required packages in metawrap-environment.yml. To do so, run:
  
  ``` bash
  conda env create -f metawrap-environment.yml
  ```
  
  To enter and exit the newly created envornment, run:
  
  ``` bash
  source activate metawrap
  metaWRAP -h
  ```

  I want to emphasize that because metaWRAP is written in BASH scripts, it is realitively easy to understand why some part of the program is failing, or if there is a dependancy issue. Just to to that line in the code and see how metaWRAP is calling that software. There is not magic here - metaWRAP is simply calling programs from your current environment in a specific sequence just like you would if you were following a pipeline. If it cannot find them, an error pops up. Feel free to dive in the code to see where things went wrong. If you find a bug, please post in the "issues" section!


  After this, there are still a few software that you will need to install and place in yout PATH, becuase they are not currently in conda. Note that these software are only required only for specific modules. 

|    Software     | Tested version  |  Used in module 			|
|:---------------:|:---------------:|:---------------------------------:| 
|    kronatools   |    v=2.7        |  kraken				|
|    metabat2     |    v=2.9.1      |  binning				|
|    maxbin2      |    v=2.2.4      |  binning				|
|    pplacer      |    v=1.1        |  bin_refinement, reassemble_bins  |
|    taxator-kt   |    v=1.3.3      |  classify_bins                    |


## DATABASES

Finally, you will need to download several databases and configure their paths in the config.sh file. This may be the longest step of the installation. Again, you may not need all of these if you intend to use specific parts of the pipeline. Don't forget to configure the paths to them in the metaWRAP/bin/config-metawrap file! Here is a full list of the databases:

|    Database     | Size  | Source |  Used in module
|:---------------:|:---------------:|:---------------:|:-----:| 
|Checkm_DB	 |1.4GB| CheckM should prompt you to download this during first use | binning, bin_refinement, reassemble_bins |
|KRAKEN standard database|161GB | Look at the official KRAKEN support website for download instructions | kraken |
|RefSeq NCBI_nt |71GB | Look at the metaWRAP/config.sh for download instructions | blobology |
|RefSeq NCBI_tax |283MB | Look at the metaWRAP/config.sh for download instructions | blobology |
|Indexed hg38  	|  20GB | You will need to download the human genome and index it with bmtagger. Look at the bmtagger manual for instructions. | read_qc |


  ![Detailed pipeline walkthrough](https://i.imgur.com/iNa6oUF.png)



## USAGE

Once all the dependencies are in place, running metaWRAP is relatively simple. The main metaWRAP script wraps around all of its individual modules, which you can call independently.

```
metaWRAP -h
	Usage: metaWRAP [module] --help
	Options:

	read_qc		Raw read QC module
	assembly	Assembly module
	binning		Binning module
	bin_refinement	Refinement of bins from binning module
	reassemble_bins Reassemble bins using metagenomic reads
	quant_bins	Quantify the abundance of each bin across samples
	blobology	Blobology module
	kraken		KRAKEN module
```

Each module is run separately. For example, to run the assembly module:

```
metaWRAP assembly -h

Usage: metaWRAP assembly [options] -1 reads_1.fastq -2 reads_2.fastq -o output_dir
Options:

	-1 STR          forward fastq reads
	-2 STR          reverse fastq reads
	-o STR          output directory
	-m INT          memory in GB (default=10)
	-t INT          number of threads (defualt=1)

	--use-megahit		assemble with megahit (default)
	--use-metaspades	assemble with metaspades instead of megahit
```


Here is an example of running the binning module with multiple samples as input for better binning:
```bash
metaWRAP binning -t 48 -m 500 --checkm-best-bins --checkm-good-bins -a coassembly.fa -o binning_out sampleA_1.fastq sampleA_2.fastq sampleB_1.fastq sampleB_2.fastq sampleC_1.fastq sampleC_2.fastq
```



###  System requirements
 The resource requirements for this pipeline will vary greatly based on the number of reads you are processing, but I would advise against attempting to run it on anything less than 10 cores and 100GB RAM. With the help of conda, installing metaWRAP and its dependencies on a cluster (even without sudo privileges) should be relatively easy even for beginners.



### Acknowledgements
Author of pipeline: German Uritskiy.

Principal Investigators: [James Taylor](http://bio.jhu.edu/directory/james-taylor/) and [Jocelyne DiRuggiero](http://bio.jhu.edu/directory/jocelyne-diruggiero/)

Institution: Johns Hopkins, [Department of Cell, Molecular, Developmental Biology, and Biophysics](http://cmdb.jhu.edu/) 

I do not claim to have any authorship of the many programs this pipeline uses. For questions, bugs, and suggestions, contact me at guritsk1@jhu.edu, or leave a comment on this github page.


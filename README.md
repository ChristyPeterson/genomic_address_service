[![PyPI](https://img.shields.io/badge/Install%20with-PyPI-blue)](https://pypi.org/project/genomic_address_service/#description)
[![Bioconda](https://img.shields.io/badge/Install%20with-bioconda-green)](https://anaconda.org/bioconda/genomic_address_service)
[![Conda](https://img.shields.io/conda/dn/bioconda/profile_dists?color=green)](https://anaconda.org/bioconda/genomic_address_service)
[![License: Apache-2.0](https://img.shields.io/github/license/phac-nml/genomic_address_service)](https://www.apache.org/licenses/LICENSE-2.0)

<img src="https://github.com/phac-nml/genomic_address_service/blob/main/logo.png?raw=true" width = "150" height="189">

# Genomic Address Service
# Introduction

Surveillance and outbreak analysis of pathogens has been operationalized by multiple public health laboratories around the world using gene-by-gene, SNP and k-mer approaches to produce estimates of genetic distance between sets of samples. Standard phylogentic or heirarchal clustering approaches group samples based on distances but need to be recalculated each time a new sample is added. The lack of repeatable genetic units or nomenclature between runs of clustering makes communication between different groups difficult, especially with the poor scaling of these approaches to larger sample sizes. 

A number of different software/pipelines have been published to address the issues mentioned above. Firstly, Enterobase implements [HeirCC](https://github.com/zheminzhou/pHierCC) as an algorithm approach to assign new cgMLST profiles into existing multi-level cluster nomenclature from a minimum spanning tree, using single linkage in real time, along with some tools for evaluating the clusters when run in de novo mode. [SnapperDB](https://github.com/ukhsa-collaboration/snapperdb) used by the UKHSA utilizes single linkage clustering based on SNPs to produce a multi-level cluster nomenclature for outbreak and surveillance activities. [ReportTree](https://github.com/insapathogenomics/ReporTreer) will perform de novo clustering based on [SciPy](https://scipy.org/) linkage methods from a sequence alignment, VCF, allele profile, or distance matrix and so provides significant flexibility in inputs compared to the other two methods. Like Enterobase, ReportTree also provides tools for evaluating regions of cluster stability and can maintain cluster nomenclature between runs with the caveat that ReportTree can split and merge previous cluster assignments, a significant issue for Public Health where it is desirable to maintain existing cluster designations. HeirCC can be run as a standalone piece of software and applied to new cgMLST schemes and organisms readily as well as incorporated into other bioinformatic pipelines. Conversely, SnapperDB is a relatively complex pipeline where establishing databases for new organisms is rather complex resulting in an inability to incorporate this tool into other workflows.  While HeirCC provides customization and flexibility in terms of thresholds and schemes, it is limited to single-linkage clustering which is known to have issues with stability and be prone to producing "scraggly" clusters due to chaining. ReportTree provides customization and reporting functionality which can be of use to public health labs, however the potential for changing cluster assignments poses issues for distributed Public Health surveillance and outbreak activities due to the potential for becoming out of synch with each other. 

Within our public health partners, there is a need for a clustering service which can perform de novo clustering based on average, complete, and single linkage that can then be partitioned into clusters based on multiple thresholds. Additionally, there is a need to assign new samples into an existing clustering to provide stable nomenclature for communication between different partners and stakeholders. To address needs of users within our team we have designed an integrated solution for calculating distance matrices and querying genetically similar samples, within a defined threshold, to support outbreak and surveillance activities. We provide the flexibility to have standard text based outputs and have included [parquet](https://parquet.apache.org/) format for highthrough put needs. It is implemented in pure python and currently is only available in a single threaded version but later refinements may include the support for multiprocessing. To facilitate integration of the tool into larger workflows it will also be implemented as a nextflow workflow.

## Citation

Robertson, James, Wells, Matthew, Schonfeld, Justin, Reimer, Aleisha. Genomic Address Service: Convenient package for de novo clustering and sample assignment to existing clusters. 2023. [https://github.com/phac-nml/genomic_address_service](https://github.com/phac-nml/genomic_address_service)
## Contact

For any questions, issues or comments please make a Github issue or reach out to [**James Robertson**](james.robertson@phac-aspc.gc.ca).

# Install

Install the latest released version from conda:

        conda create -c bioconda -c conda-forge -n genomic_address_service genomic_address_service

Install using pip:

        pip install genomic_address_service

Install the latest master branch version directly from Github:

        pip install git+https://github.com/phac-nml/genomic_address_service.git
        
### Compatibility
List out Dependencies and/or packages as appropriate

# Getting Started
## Usage

`gas command args`

### Commands
There are three commands that gas uses:

1. **mcluster** - de novo nested multi-level clustering
2. **call** - call genomic address based on existing clusterings
4. **test** - test functionality on a small dataset

### Args
There are a number of arguments that are specific for each command. They can be found directly by adding `--help` after each command. The following are common arguments:

- `-o`, `--outdir` - output directory to put cluster results
- `-m`, `--method` - cluster method [single, complete, average (default)]
- `-t`, `--thresholds` - thresholds delimited by ',' columns will be treated in sequential order
- `-V`, `--version` - print installed tool version
- `-f`, `--force` - overwrite existing out directory

#### mcluster specific args
- `-i`, `--matrix` - TSV formatted distance matrix or parquet
- `-d`, `--delimiter` - delimiter desired for nomenclature code [default="."]

#### call specific args
- `-d`, `--dists` - a 3 column file [query_id, ref_id, dist] in TSV or parquet format
- `-r`, `--rclusters` - existing cluster file in TSV or parquet format
- `-j`, `--thresh_map` - Json file of [colname:threshold]
- `-u`, `--outfmt` - output format for assignments [text (default), parquet]
- `-l`, `--delimiter` - delimiter desired for nomenclature code [default="."]


## Configuration and Settings:
Thresholds must be configured when using GAS. These threshold must be determined manually through testing and establishment of practical criteria for each pathogen of interest. 

For instance, in PulseNet Canada they have determined the use of '10,5,0' to be the threshold of choice for their pathogen surveillance program. [Publication on going]

## Data Input/formats

### Square distance matrix

| id  | S1  | S2  | S3  | S4  | S5  | S6  |
| --- | --- | --- | --- | --- | --- | --- |
| S1  | 0   | 0   | 3   | 3   | 9   | 9   |
| S2  | 0   | 0   | 3   | 3   | 9   | 9   |
| S3  | 3   | 3   | 0   | 0   | 9   | 9   |
| S4  | 3   | 3   | 0   | 0   | 9   | 9   |
| S5  | 9   | 9   | 9   | 9   | 0   | 0   |
| S6  | 9   | 9   | 9   | 9   | 0   | 0   |

- Distance matrix units can be of float, or integer type with the constrain that the diagonal must be 0 and the first line must be a header with all of the samples

### Apache parquet
- More information on the open-source column-oriented data format can be found [here](https://parquet.apache.org/).
## Output/Results

```
{Output folder name}
├── distances.{text|parquet} - Three column file of [query_id, ref_if, distance]
├── thresholds.json - JSON formated mapping of columns to distance thresholds
├── clusters.{text|parquet} - Either symmetric distance matrix or three column file of [query_id, ref_if, distance]
├── tree.newick - Newick formatted dendrogram of the linkage matrix produced by SciPy
└── run.json - Contains logging information for the run including parameters, newick tree, and threshold mapping info
```

# Troubleshooting and FAQs:

1. Mcluster fails due to missing scipy, with the following error:
```
import scipy
ModuleNotFoundError: No module named 'scipy'
```
- This dependency is currently missing in the pip install. Use the following command to install scipy separately: `pip install scipy`

# Benchmarking

Coming soon.

# Legal and Compliance Information:

Copyright Government of Canada 2023

Written by: National Microbiology Laboratory, Public Health Agency of Canada

Licensed under the Apache License, Version 2.0 (the "License"); you may not use
this work except in compliance with the License. You may obtain a copy of the
License at:

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed
under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
CONDITIONS OF ANY KIND, either express or implied. See the License for the
specific language governing permissions and limitations under the License.

# Updates and Release Notes:
Please see the `CHANGELOG.md`

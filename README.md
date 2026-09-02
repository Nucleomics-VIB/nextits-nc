# NextITS <img src='images/NextITS_logo.svg' align="right" height="70" />

# Original repo (forked at Nucleomics and maintained in the fork: [NC_NextITS](https://github.com/Nucleomics-VIB/NC_NextITS))

![GitHub (latest release)](https://img.shields.io/github/v/release/vmikk/NextITS?label=GitHub%20release&color=23aa62)
[![Nextflow](https://img.shields.io/badge/Nextflow%20DSL2-%E2%89%A524.04-23aa62.svg)](https://www.nextflow.io/)
[![GitHub license](https://img.shields.io/github/license/vmikk/NextITS)](https://github.com/vmikk/NextITS/blob/main/LICENSE)  

[![Runs with singularity](https://img.shields.io/badge/Runs%20with-Singularity-blue?style=flat&logo=singularity)](https://sylabs.io/docs/)
[![Runs with Docker](https://img.shields.io/badge/Runs%20with-Docker-blue?style=flat&logo=docker)](https://hub.docker.com/r/vmikk/nextits/tags)  

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.15074881.svg)](https://doi.org/10.5281/zenodo.15074881)

NextITS is an automated pipeline for metabarcoding fungi and other eukaryotes with full-length ITS sequenced with PacBio.
Amplicons obtained with Illumina are also supported.

## Introduction

The most widely used genetic markers for metabarcoding fungal communities are highly variable rRNA ITS1 and ITS2 sub-regions of the internal transcribed spacer. High-throughput metabarcoding has greatly improved our understanding of fungal community ecology. Here, we present NextITS, an automated pipeline for analyzing full-length ITS sequences (ITS1-5.8S-ITS2) from the Pacific Biosciences (PacBio) third-generation sequencing platform. Although the PacBio HiFi reads are highly accurate, the primary type of sequencing error is insertions or deletions in homopolymeric sites, which are also naturally common in fungal ITS. In the pipeline, we implemented correction of homopolymer errors, detection of tag-switching artefacts, and recovery of sequences false-positively annotated as chimeric. The pipeline is built using Nextflow workflow manager, with all the software dependencies packaged into Docker and Singularity containers.

## User Documentation

User documentation: https://Next-ITS.github.io/  

## custom primers

Default primers are ITS9munNgs_Fwd and ITS4ngsUni_Rev. Teh forward primer of the developer has an extra 'G' absent from the NC primers.
We also have an alternative forward primer ITS1catta_Fwd which is recommendced by PacBio.

* ITS9munNgs_Fwd*: (G)TACACACCGCCCGTCG # no 5' G in the NC primer
* ITS1catta_Fwd: ACCWGCGGARGGATCATTA
* ITS4ngsUni_Rev*: CCTSCSCTTANTDATATGC

## Quick Start

The pipeline is ran twice in sequence:
* the first run is done on each sequencing run batch
* then comes the second run to aggregate the results of multiple runs into a final report and dataset

### Run1

```
#! /bin/bash

# run NextITS from the singularity image
# from:= https://next-its.github.io/usage/
# and: https://next-its.github.io/parameters/
# SP@NC; 2025-06-11; v1.0

infolder="/data/splaisan/NC_projects/5001_SWeckx_NextITS/fastq_ITS"
outfolder="/data/splaisan/NC_projects/5001_SWeckx_NextITS/NextITS_analysis_noITSx"
runid="run1"

profile="/data/splaisan/NC_projects/5001_SWeckx_NextITS/NextITS/conf/wallace.config"

# create folder and contained workdir
mkdir -p ${outfolder}/workdir

# set to the current version of the nextflow pipeline
ver="main"

# primers adapted to the project sequences
forp="TCCGTAGGTGAACCTGC"
revp="CCTSCSCTTANTDATATGC"

# compute length -2
forl=$((${#forp} - 2))
revl=$((${#revp} - 2))

# database for chimera
# downloaded from https://owncloud.ut.ee/owncloud/s/PkasGDNDimNssm3
chimeraDB="/data/splaisan/biodata/NextITS_chimeraDB/UN95_chimera.udb"

# -with-singularity /opt/sif_files/vmiks_nextits_1-1-0.sif \
cmd="nextflow run vmikk/NextITS -r ${ver} \
  -with-singularity /opt/sif_files/vmiks_nextits_1-1-0.sif \
  -c ${profile} \
  --step "Step1" \
  --input ${infolder} \
  --demultiplexed true \
  --primer_forward ${forp} \
  --primer_reverse ${revp} \
  --primer_mismatches 2 \
  --primer_foverlap ${forl} \
  --primer_roverlap ${revl} \
  --chimera_db ${chimeraDB} \
  --its_region 'none' \
  --ITSx_tax 'all' \
  --ITSx_evalue 0.1 \
  --ITSx_partial 0 \
  --outdir ${outfolder}/nextits_Step1/${runid} \
  -work-dir ${outfolder}/workdir \
  --storagemode 'copy'"

# copy mode is not yet working, need to make a complete copy
echo "# ${cmd}"
eval ${cmd} && \
  mkdir -p ${outfolder}/final_results && \
  cp -rL ${outfolder}/nextits_Step1/${runid} ${outfolder}/final_results/Step1_results

echo "# Step1 completed, complete results are in ${outfolder}/final_results/Step1_results"
```

### Run2

```
#! /bin/bash

# run NextITS from the singularity image
# from:= https://next-its.github.io/usage/
# and: https://next-its.github.io/parameters/
# SP@NC; 2025-06-11; v1.0

outfolder="/data/splaisan/NC_projects/5001_SWeckx_NextITS/NextITS_analysis_noITSx"

# set to the current version of the nextflow pipeline
ver="main"

profile="/data/splaisan/NC_projects/5001_SWeckx_NextITS/NextITS/conf/wallace.config"

cmd="nextflow run vmikk/NextITS -r ${ver} \
  -with-singularity /opt/sif_files/vmiks_nextits_1-1-0.sif \
  -c ${profile} \
  --step 'Step2' \
  --data_path ${outfolder}/nextits_Step1\
  --outdir ${outfolder}/nextits_Step2 \
  --clustering_method 'vsearch' \
  --otu_id 0.98 \
  --storagemode 'copy'"

echo "# ${cmd}"
eval ${cmd} && \
  mkdir -p ${outfolder}/final_results && \
  cp -rL ${outfolder}/nextits_Step2 ${outfolder}/final_results/Step2_results

echo "# Step2 completed, complete results are in ${outfolder}/final_results/Step2_results"
```

## Citation

Mikryukov V., Anslan S., Tedersoo L. NextITS: a pipeline for metabarcoding fungi and other eukaryotes with full-length ITS sequenced with PacBio. [https://github.com/vmikk/NextITS](https://github.com/vmikk/NextITS)




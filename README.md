Species Distribution Models for Prioritization in Denmark
================
Derek Corcoran
16/12, 2022

- <a href="#1-taxonomic-cleaning" id="toc-1-taxonomic-cleaning">1
  Taxonomic cleaning</a>
- <a href="#2-presence-download" id="toc-2-presence-download">2 Presence
  download</a>
- <a href="#3-presence-cleaning" id="toc-3-presence-cleaning">3 Presence
  cleaning</a>

<!-- README.md is generated from README.Rmd. Please edit that file -->
<!-- badges: start -->
<!-- badges: end -->

The goal of SpeciesDistributionModelsDanmark is to explore and generate
Species distribution models for de prioritization of Denmark based on
its biological diversity and landuse.

# 1 Taxonomic cleaning

First we will read a file with all the taxa found in
[arter.dk](https://arter.dk/landing-page), on the 21st of September of
2022.

for this we will need the follwing packages (Wickham and Bryan 2022;
Chamberlain et al. 2020; Chamberlain and Boettiger 2017):

<details style="\&quot;margin-bottom:10px;\&quot;">
<summary>

Load packages

</summary>

``` r
library(readxl)
library(taxize)
library(rgbif)
library(janitor)
```

</details>

We first read the file with all the presences:

``` r
Taxa <- readxl::read_xlsx("2022-09-21.xlsx") |> 
  janitor::clean_names()
```

This file has 60958 entries, however it only has
`length(unique(Taxa$videnskabeligt_navn))` unique entries in the
attribute *videnskabeligt_navn*

# 2 Presence download

# 3 Presence cleaning

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-Chamberlain2017RGBIF" class="csl-entry">

Chamberlain, Scott, and Carl Boettiger. 2017. “R Python, and Ruby
Clients for GBIF Species Occurrence Data.” *PeerJ PrePrints*.
<https://doi.org/10.7287/peerj.preprints.3304v1>.

</div>

<div id="ref-Chamberlain2020Taxize" class="csl-entry">

Chamberlain, Scott, Eduard Szoecs, Zachary Foster, Zebulun Arendsee,
Carl Boettiger, Karthik Ram, Ignasi Bartomeus, et al. 2020. *Taxize:
Taxonomic Information from Around the Web*.
<https://github.com/ropensci/taxize>.

</div>

<div id="ref-Wickham2022Readxl" class="csl-entry">

Wickham, Hadley, and Jennifer Bryan. 2022. *Readxl: Read Excel Files*.
<https://CRAN.R-project.org/package=readxl>.

</div>

</div>
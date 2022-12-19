Species Distribution Models for Prioritization in Denmark
================
Derek Corcoran
19/12, 2022

-   [1 Taxonomic cleaning](#1-taxonomic-cleaning)
    -   [1.1 Cleaning using Taxize](#11-cleaning-using-taxize)
    -   [1.2 Cleaning using RGBIF](#12-cleaning-using-rgbif)
-   [2 Presence download](#2-presence-download)
-   [3 Presence cleaning](#3-presence-cleaning)

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
library(dplyr)
library(stringr)
```

</details>

We first read the file with all the presences:

``` r
Taxa <- readxl::read_xlsx("2022-09-21.xlsx") |> 
  janitor::clean_names() |> 
  dplyr::select(videnskabeligt_navn)
```

This file has 60958 entries, however it only has
`length(unique(Taxa$videnskabeligt_navn))` unique entries in the
attribute *videnskabeligt_navn*

## 1.1 Cleaning using Taxize

First we generate a new data frame considering only the unique
*videnskabeligt_navn*:

``` r
NewTaxa <- data.frame(Taxa = sort(unique(Taxa$videnskabeligt_navn)), score = NA, matched_name2 = NA) |> 
  tibble::rowid_to_column(var = "TaxaID")
```

and then we clean it using taxize first

<details style="\&quot;margin-bottom:10px;\&quot;">
<summary>

Taxize clean

</summary>

``` r
dir.create("Results")

for(i in 1:nrow(NewTaxa)){
  try({
    Temp <- taxize::gnr_resolve(NewTaxa$Taxa[i],
                                         data_source_ids = "11", canonical = TRUE, best_match_only = T) |> 
      dplyr::select(score, matched_name2)
    NewTaxa[i,3:4] <- Temp
      if((i %% 50) == 0){
      message(paste(i, "of", nrow(NewTaxa), "Ready!", Sys.time()))
      readr::write_csv(NewTaxa, "Results/Cleaned_Taxa_Taxize.csv")
    }
    gc()
  }, silent = T)
  
}
```

</details>

This cleaning ends up eliminating 1313 taxa which are mostly Families,
subfamilies or hybrid species, as seen in table
<a href="#tab:tableout1">1.1</a>

| TaxaID | Taxa                            | score |
|-------:|:--------------------------------|------:|
|     51 | Abraeinae                       |    NA |
|     52 | Abraeini                        |    NA |
|     89 | Acaenitinae                     |    NA |
|    154 | Acanthocinini                   |    NA |
|    168 | Acanthoderini                   |    NA |
|    210 | Acari                           |    NA |
|    381 | Achelata                        |    NA |
|    406 | Achillea ptarmica × salicifolia |    NA |
|    452 | Aciculata                       |    NA |
|    463 | Aciliini                        |    NA |

Table 1.1: First 10 taxa eliminated by taxize

Of the reminding species that were identidied by taxize there are still
some unique species in out initial file that ended up being identified
as duplicate species some examples can be seen in
<a href="#tab:FindDuplicates">1.2</a>

| TaxaID | Taxa                                                        | score | matched_name2                   |
|-------:|:------------------------------------------------------------|------:|:--------------------------------|
|   1829 | Alisma plantago-aquatica                                    | 0.988 | Alisma plantago-aquatica        |
|   1830 | Alisma plantago-aquatica f. submersa                        | 0.988 | Alisma plantago-aquatica        |
|   2451 | Ammophila arenaria                                          | 0.988 | Ammophila arenaria              |
|   2453 | Ammophila arenaria × Calamagrostis epigejos nm. epigeioidea | 0.988 | Ammophila arenaria              |
|   2454 | Ammophila arenaria × Calamagrostis epigejos nm. intermedia  | 0.988 | Ammophila arenaria              |
|   2455 | Ammophila arenaria × Calamagrostis epigejos nm. subarenaria | 0.988 | Ammophila arenaria              |
|   3047 | Anemone apennina                                            | 0.988 | Anemone apennina                |
|   3048 | Anemone apennina var. apennina                              | 0.988 | Anemone apennina                |
|   3607 | Anthyllis vulneraria subsp. vulneraria                      | 0.999 | Anthyllis vulneraria vulneraria |
|   3611 | Anthyllis vulneraria var. vulneraria                        | 0.999 | Anthyllis vulneraria vulneraria |
|   4963 | Arrhenia acerosa                                            | 0.988 | Arrhenia acerosa                |
|   4964 | Arrhenia acerosa var. acerosa                               | 0.988 | Arrhenia acerosa                |

Table 1.2: First 12 duplicate species

All and all, we started with 60,915 unique taxa and ended up with 59,197
unique taxa

## 1.2 Cleaning using RGBIF

In order to do this cleanly we will just get one observation of each
taxa found by Taxize in its column matched_name2

<details style="\&quot;margin-bottom:10px;\&quot;">
<summary>

Unique taxize names

</summary>

``` r
Cleaned_Taxize <- NewTaxa |> 
  dplyr::filter(!is.na(matched_name2)) |> 
  dplyr::group_by(matched_name2) |> 
  dplyr::filter(TaxaID == min(TaxaID)) |> 
  ungroup()
```

</details>

and then we will pass this through rgbif, change the input name
(vertbatim_name), to matched_name2, so that it is the same as in
cleaned_Taxize, then we kept the matched, name, the confidence on the
finding for RGBIF, and all the taxonomic groups

<details style="\&quot;margin-bottom:10px;\&quot;">
<summary>

rgbif call

</summary>

``` r
rgbif_find <- rgbif::name_backbone_checklist(Cleaned_Taxize$matched_name2) |>
  # Change name to match the cleaned_taxize dataset
  dplyr::rename(matched_name2 = verbatim_name) |> 
  dplyr::relocate(matched_name2, .before = everything()) |> 
  dplyr::select(matched_name2, confidence, kingdom, phylum, order, family, genus, species)

readr::write_csv(rgbif_find, "Results/Cleaned_Taxa_rgbif.csv")
```

</details>

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

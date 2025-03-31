# Biodiversity of European macrobenthos by EUNIS seabed habitat types

## Introduction

This product is built from two existing EMODnet data products. It uses the [EUSeaMap product created by EMODNet Seabed Habitats](https://emodnet.ec.europa.eu/en/seabed-habitats#seabed-habitat-products) and the [Presence/absence data of macrozoobenthos in the European Seas product created by EMODnet Biology](https://github.com/EMODnet/EMODnet-Biology-Benthos-European-Seas) to obtain lists of all macrobenthic species occurring in all [EUNIS seabed habitat types](https://doi.org/10.1016/j.marpolbul.2012.10.010). For the purposes of this product the seabed habitat map is summarised on a grid at approx. 500m resolution, with each grid cell assigned to the dominant EUNIS code present within it. Summaries of benthic diversity are created for each grid cell, as well as more synthetic summaries by EUNIS category.

## Directory structure

```         
{{directory_name}}/
├── analysis
├── data/
│   ├── derived_data/
│   └── raw_data/
│               └── EMODnet_bio_benthos_processed
│               └── EUSeaMap_2023
├── docs/
├── product/
└── scripts/
```

-   **analysis** - Markdown or Jupyter notebooks
-   **data** - Raw and derived data
-   **docs** - Rendered reports
-   **product** - Output product files
-   **scripts** - Reusable code

## Data series

### Macrobenthic presence/absence data

The data product provided by [Herman 2022](https://www.vliz.be/nl/imis?dasid=8216) includes a full presence-absence matrix of \>10,800 distinct taxa over c. 290,000 unique sampling events throughout European seas - see <https://github.com/EMODnet/EMODnet-Biology-Benthos-European-Seas> for documentation of the data processing and filtering workflow that was used to generate this product. The final product is served as a single large netcdf file. A full zipped version of the product can be obtained directly from <https://mda.vliz.be/download.php?file=VLIZ_00000727_640f2ae069c07615997234>, with the full \~3.2GB netcdf file available (zipped) within this in as product/maps/Macrobenthos_Eur_Seas_Pres_Abs_v0-4.nc.

In a previous EMODnet biology product, [Webb 2023](https://github.com/EMODnet/EMODnet-Biology-benthos-trends) created processed versions of Herman’s data product with reduced file size to facilitate further analysis. A full description of this data processing, which is all done in R, is provided as part of the previous product and can be downloaded from <https://github.com/EMODnet/EMODnet-Biology-benthos-trends/blob/master/docs/benthos-trends-dataprep.html>. The key steps were:

1.  Assembling a table of unique survey events
2.  Assembling a taxonomic table with unique taxon IDs and associated taxonomic ranks (all following [WoRMS](https://marinespecies.org/) standards)
3.  Assembling a dataset with taxon presence by survey event

These three separate tables were saved as csv files and can be downloaded from the ‘derived_data’ folder provided as part of [Webb 2023](https://github.com/EMODnet/EMODnet-Biology-benthos-trends) here: <https://github.com/EMODnet/EMODnet-Biology-benthos-trends/tree/master/data/derived_data>

These three datasets have been added to the data folder of the current project, in subfolder raw_data/EMODnet_bio_benthos_processed. Some additional processing steps are then performed, documented in full in the file Eurobenthos_habitat_species.html in the docs folder of this project. In brief this involved:

1.  Restrict data to surveys which report presence / absence of *all* macrobenthic species - a few surveys sample only a subset of the macrobenthos
2.  Filtering out any incorrect dates (year is after the data of product creation) and limiting to surveys conducted after 1980 (this includes 95% of surveys) to reduce the impact of historical range changes
3.  Retaining only taxa identified to species level (n = 8149)
4.  Writing two fully processed datasets, benthos_events (all unique survey events) and benthos_full (all unique species presences) to the derived_data folder in parquet format with gzip compression to limit file size

### **European seabed habitat data**

Seabed habitat information comes from the European broad-scale seabed habitat map [EUSeaMap 2021](https://archimer.ifremer.fr/doc/00723/83528/) (hereafter EUSM). While this product can be accessed directly through EMODnet Web Feature Services using the [emodnet.wfs package](https://github.com/EMODnet/emodnet.wfs), obtaining the full EUSM via this route is prohibitively slow. For the purposes of the current product, we therefore use a local download of the [full EUSeaMap product](https://emodnet.ec.europa.eu/geonetwork/emodnet/eng/catalog.search#/metadata/0a1cb988-22de-48b2-8cda-d90947ef77d1) - the direct download is available from <https://files.emodnet-seabedhabitats.eu/data/EUSeaMap_2023.zip>. Given its large size (1.75GB) it is not made available as part of the current product, so users wishing to access the full resolution map will need to download it themselves from the link above.

For the product developed here, we focus on EUNIS (European Union Nature Information System) habitat classifications (see [Galparsoro et al. 2012](https://doi.org/10.1016/j.marpolbul.2012.10.010)). Specifically, and following consultation with Helen Lillis of the EMODnet Seabed Habitats team, we include the fields Eunis2019c - this is the code for the habitats, in version 2019 (which now called version 2022) of the EUNIS habitat classification, and Eunis2019d, which is a concatenation of the code and the full name of the habitats. The document Eurobenthos_habitat_species.html in the docs folder of this project details the processing performed on this dataset, but in brief this involves:

1.  Excluding polygons with no EUNIS classification in EUSM
2.  Rasterising EUSM onto a regular grid of approx 500m resulution (0.052 decimal degrees), assigning each grid cell the dominant EUNIS classification if it is covered by more than one
3.  Writing the resulting raster file to the derived_data folder in geotiff format (using the `terra` package)
4.  Two coarser-resolution rasters (approx 2km and 12km resolution) are also created to facilitate matching of benthic surveys to habitats

### **Matching benthic community data to seabed habitat data**

The derived benthic community and seabed habitat datasets described above were then used to match benthic community data to seabed habitats. The full resolution (\~500m) seabed habitat raster was used first; surveys that were not matched to this were frequently coastal and fell just into the land mask of the seabed habitat raster. In these cases, the coarser resolution habitat maps were used in an attempt to match the surveys to nearby habitats. Details of this process are provided in Eurobenthos_habitat_species.html in the docs folder of this project. Following the matching process, the full matched benthos community data was written to file in the derived data folder as benthos_full_matched in parquet format with gzip compression.

## Data product

A summary product includes three gridded spatial layers covering European seas. The layers are:

-   eunis2019 - this is the EUNIS code for the dominant habitat type in each grid cell

-   n_surveys - this is the number of distinct sampling events recorded in each grid cell

-   n_species - this is the number of distinct macrobenthic species recorded as present in each grid cell

The product is available as a geoTiff in the product folder of this project, or as a NetCDF file from ####.

In addition, the following synthetic tables are available in the product folder:

| Table filename | Table description | Variables included |
|------------------|--------------------|-----------------------------------|
| eunis_summaries.csv | Summary of benthic diversity for each EUNIS habitat category | *EUNIS2019C:* EUNIS habitat code, *EUNIS2019D:* EUNIS habitat code + full description, *n_surveys:* Number of benthic surveys, *n_species:* Number of benthic species, *total_area:* total area of each EUNIS code in EUSeaMap, *n_cells:* total number of \~500m grid cells assigned to each EUNIS category in the rasterised map |
| species_by_eunis.csv | Benthic species lists for each EUNIS habitat category | *aphia_id:* WoRMS Aphia ID for the accepted species name, *scientificname:* species name, *EUNIS2019C:* EUNIS habitat code, *EUNIS2019D:* EUNIS habitat code + full description |

These summary tables are derived from the benthos_full_matched dataset. Example code for producing these tables, as well as other data structures that may be of interest, is included in the *Code and methodology* section below.

## More information:

### References

### Code and methodology

All data processing is described in the document Eurobenthos_habitat_species.html within the docs folder. Creation of the final product is described in ####script/additional file.

To create the summary data tables in R from the 'benthos_full_matched.parquet' file requires the `tidyverse` and `arrow` packages:

```
library(tidyverse)
library(arrow)
```
Then the full dataset can be read in directly from this repo:

```
benthos_full_matched <- read_parquet("https://github.com/tomjwebb/EUNIS_benthos_species/raw/refs/heads/master/data/derived_data/benthos_full_matched.parquet")
```
From this it is straightforward to derive the list of unique species per EUNIS habitat:

```
species_by_eunis <- benthos_full_matched %>%
  select(aphia_id, scientificname, EUNIS2019C, EUNIS2019D) %>% 
  distinct() %>% 
  arrange(EUNIS2019C)
```
This is the dataset provided in the product folder as species_by_eunis.csv. From this dataset you can easily get a list of species for a particular EUNIS code of interest, for instance for MB12 (Atlantic infralittoral rock):

```
MB12_species <- species_by_eunis %>% 
  filter(EUNIS2019C == "MB12")
```

Or to get all the habitats in which a species of interest - e.g. _Mytilus edulis_ - has been recorded:

```
m_edulis_habs <- species_by_eunis %>% 
  filter(scientificname == "Mytilus edulis")
```

NB - because the version of the habitat map used here is at a coarser resolution than the full EUSeaMap, and also the precision of the benthos survey data locations is variable, these outputs should be considered as a broad scale guide rather than as a definitive measure of habitat affinities of benthic species. For individual species of interest it may be more informative to get the frequency of occurrence across different habitat types from the main dataset, for instance for _M. edulis_:

```
m_edulis_habs_freq <- benthos_full_matched %>% 
  filter(scientificname == "Mytilus edulis") %>% 
<<<<<<< HEAD
  group_by(EUNIS2019C, EUNIS2019D) %>% 
  summarise(freq_by_hab = n(),
            .groups = "drop")
```
To put these frequencies in context, it is useful to know the total number of surveys per habitat type. Fortunately that is available in the eunis_summaries dataset, which can be read in using:

```
eunis_summaries_file <- read_csv("https://github.com/tomjwebb/EUNIS_benthos_species/raw/refs/heads/master/product/eunis_summaries.csv")
```

Then a 'p_present' variable (i.e., the proportion of surveys in each habitat type in which a species was found) can be added like this:

```
m_edulis_habs_freq <- m_edulis_habs_freq %>% 
  left_join(eunis_summaries, join_by(EUNIS2019C, EUNIS2019D)) %>% 
  mutate(p_present = freq_by_hab / n_surveys) %>% 
  select(EUNIS2019C, EUNIS2019D, freq_by_hab, p_present, everything()) %>% 
  arrange(desc(p_present))
```

To get proportion of surveys in each habitat classification a species is present in, across all species, you can use:

```
species_by_eunis <- benthos_full_matched %>% 
  group_by(aphia_id, EUNIS2019C, EUNIS2019D) %>% 
  summarise(n_present = n(), .groups = "drop") %>% 
  left_join(eunis_summaries, join_by(EUNIS2019C, EUNIS2019D)) %>% 
  mutate(p_present = n_present / n_surveys,
         eunis_full = paste(EUNIS2019C, EUNIS2019D, sep = "_")) %>% 
  select(aphia_id, eunis_full, p_present)

```
Note the creation of a composite eunis_full variable - this is due to some EUNIS codes having more than one description (e.g. EUNIS code MB61 is present in the dataset with the long name 'MB61: Arctic infralittoral mud' and also with just 'Infralittoral'). Pasting codes and descriptions together is an inelegant way to fix this. The resulting dataset can be turned into wide format (species x habitat) if needed, e.g. for diversity calculations:

```
species_by_eunis_mat <- species_by_eunis %>% 
  pivot_wider(names_from = eunis_full, values_from = p_present) %>% 
  mutate(across(everything(), ~replace_na(.x, 0)))
```

### Citation and download link

This product should be cited as:

Webb, T.J. 2025. Seabed habitat and macrobenthos occurrences in European seas. Integrated data products created under the European Marine Observation Data Network (EMODnet) Biology project CINEA/EMFAF/2022/3.5.2/SI2.895681, funded by the European Union under Regulation (EU) No 508/2014 of the European Parliament and of the Council of 15 May 2014 on the European Maritime and Fisheries Fund.

Available to download in:

{{link_download}}

### Authors

Tom Webb

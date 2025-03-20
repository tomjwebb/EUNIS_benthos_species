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

The product includes three gridded spatial layers covering European seas. The layers are:

-   eunis2019 - this is the EUNIS code for the dominant habitat type in each grid cell

-   n_surveys - this is the number of distinct sampling events recorded in each grid cell

-   n_species - this is the number of distinct macrobenthic species recorded as present in each grid cell

The product is available as a geoTiff in the product folder of this project, or as a NetCDF file from ####.

In addition, the following synthetic tables are available:

## More information:

### References

### Code and methodology

{{link_code}}

### Citation and download link

This product should be cited as:

Webb, T.J. 2025. Seabed habitat and macrobenthos occurrences in European seas. Integrated data products created under the European Marine Observation Data Network (EMODnet) Biology project CINEA/EMFAF/2022/3.5.2/SI2.895681, funded by the European Union under Regulation (EU) No 508/2014 of the European Parliament and of the Council of 15 May 2014 on the European Maritime and Fisheries Fund.

Available to download in:

{{link_download}}

### Authors

Tom Webb

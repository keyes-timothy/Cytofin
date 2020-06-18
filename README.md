# Cytofin

Cytofin (CyTOF integration) is an R package for homogenizing and integrating hetereogenous Cytof data from diverse data source.

Before CyTOF data inegration, all cytof files need to be homogenized to have consistent channels. Cytofin requires that all input cytof be homogenized based on a user provided standarized panel with user defined search pattern. To normalize the cytof data, Cytofin use a novel generalized anhor strategy that define the based line of the signal between batch to correct for batch effects. One anchor need to be identified by the user from each plate (batch). A reference anchor is generated based on the mean expression of all identified anchors from each plate (batch). Next, different transformation functions is applied to fit each plate-specific anchor to the reference data distribution and the same transformation is then appleid to correct the sample data signal on each plate.  

Cytofin provided three functions for Cytof data integration:

1. homogenize-this function performs batch homogenization of cytof data based on a user-defined panel and search pattern. 
2. anprep-this function generates reference statistics from anchors identified from each plate (batch).
3. annorm-this function performs signal normalization using transformation function based on the anchor from anprep.
4. annorm_nrs-this function performs signal normalization using stabilized channels as internal anchors. 

1. CyTOF data homogenization

![Alt text](./images/Slide1.png?raw=true "Title")

Function Definition: _homogenize(metadata_filename, panel_filename, input_file_dir, output_file_dir)_

Input: raw CyTOF files (.fcs), standardized antigen panel table file (.xlsx/.csv), metadata table (.xlsx/.csv). 

Output: The homognize function take a user input antigen panel table, which include standardized antigen name and associated antigen search pattern. Given two Cytof files with distinct antigen naming, the program perform a regular expression search to match the synomymous term in the panel and correct the antigen name with standardized name in the panel.This function generated homogenized CyTOF file with user defined channel defined by the standardized antigen table.  

2. CyTOF data normalization

![Alt text](./images/Slide2.PNG?raw=true "Title")

The normalization include two steps: 1. preparation of external anchors and 2. application of transformation function.

External anchor normalization:

1. Anchors preparation:

Function definition: _anprep(metadata_filename, panel_filename, input_file_dir)_

Input: homogenized CyTOF files (.fcs), metadata table file (.xlsx/.csv), and anchor table file (.xlsx/.csv).

Output: the anprep function concatenated the identified anchor file, one file per plate/batch, and subsequently generated summary statistics including mean and variacne which will be used for batch correction. The values will be store in the RData object. The function also outputs the concatenated anchor FCS files.

2. Data Transformation:

Input: homogenized CyTOF files (.fcs), validation CyTOF files (.fcs, optional), anchor table file (.xlsx/.csv), metadata table file (.xlsx/.csv), and anchor table file (.xlsx/.csv).

Output: the annorm function applied different transformation functions (modes) to normalize each anchor to the referenece summary statistcs generated by the anprep function.

Internal anchor normalization:

Input: homogenized CyTOF files (.fcs), validation CyTOF files (.fcs, optional),metadata table file (.xlsx/.csv).

Output:In the event that the external references are not available, internal anchors can be used. Here, we identifed the most stable channels as internal anchors using a PCA-based non-redundnacy score. A minimal of three channels should be selected to establish an internal refernece from which signal can be calibrated between CyTOF files.

#Computational Pipeline for CyTOF data integration

Below is an demo Rscript using Cytofin package for CyTOF data integration.

```{r}
#import cytofin R package
library(cytofin)

#homogenization antigen panel, use the demo data supplied with the package
metadata_filename <- paste0(path.package("cytofin"),"/extdata/test_metadata_raw.csv")
panel_filename <- paste0(path.package("cytofin"),"/extdata/test_panel.csv")
input_file_dir <- paste0(path.package("cytofin"),"/extdata/test_raw_fcs_files/")
output_file_dir <- "out_test/"
homogenize(metadata_filename, panel_filename, input_file_dir, output_file_dir)

#prep external anchor 
anchor_metadata_filename <- paste0(path.package("cytofin"),"/extdata/test_anchor_metadata_raw.csv")
input_file_dir <- output_file_dir #use the homogenized files
anprep(anchor_metadata_filename, panel_filename, input_file_dir)

#data normalization using external anchors and meanshift transofmration function
val_file_dir <- paste0(path.package("cytofin"),"/extdata/test_batch_fcs_files/")
anchor_data_filename <- "./Prep_control.RData"
output_file_dir <- "norm_test/"
mode <- "meanshift"
annorm(anchor_metadata_filename, anchor_data_filename, metadata_filename, panel_filename, input_file_dir, val_file_dir, output_file_dir, mode)
annorm(anchor_metadata_filename, anchor_data_filename, metadata_filename, panel_filename, input_file_dir, "none", output_file_dir, mode)

#data normalization using 4 internal channels and meanshift_bulk transformation function
nchannels <- 4
output_file_dir <- "norm_test2/"
annorm_nrs(metadata_filename, panel_filename, input_file_dir, val_file_dir, output_file_dir, nchannels)
annorm_nrs(metadata_filename, panel_filename, input_file_dir, "none", output_file_dir, nchannels)

```

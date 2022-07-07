---
layout: tutorial_hands_on

title: 'Mass spectrometry: GC-MS untargeted analysis'
# zenodo_link: 'https://zenodo.org/record/3244991'
questions:
- What are the main steps of untargeted GC-MS data processing for metabolomic analysis?
- How to conduct metabolomic data analysis from preprocessing to annotation using Galaxy?
objectives:
- To comprehend the diversity of GC-MS metabolomic data analysis.
- To get familiar with the main steps constituting a metabolomic workflow for untargeted GC-MS analysis.
- To evaluate the potential of a workflow approach when dealing with GC-MS metabolomic data.
time_estimation: ''
key_points:
- To process untargeted GC-MS metabolomic data, you need a large variety of steps and tools.
- Although main steps are standard, various ways to combined tools exist, depending on your data.
- Resources are available in Galaxy, but do not forget that you need appropriate knowledge to perform a relevant analysis.
contributors:
- martenson


---

# Introduction
{:.no_toc}

GC-MS untargetted analysis based on RECETOX-adapted tools

This is a training walking through the RECETOX GC prototype workflow. It runs on the Seminal plasma dilution series training dataset available in the RECETOX [data library](https://umsa.cerit-sc.cz/libraries/folders/F7f9c6096b8fa4528/page/1). The dilution series dataset consists of...

During the tutorial we will perform initial peak detection and feature grouping after retention time-alignment across the samples, deconvolute those peaks into complete spectra, assigns retention indices and finally matches those spectra against the reference library.

> ### Agenda
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Importing training data into Galaxy

## The main GC/MS dataset
The main data input is a collection of sample data in the .mzML format, converted from the .RAW format by the `ThermoRawFileConverter` tool. The samples should be in profile mode (so no centroiding during conversion) which improves the actual peak detection and distinguishing peaks from noise over centroided data.

The training dataset is in the UMSA Galaxy Data Library available [here](https://umsa.cerit-sc.cz/libraries/folders/F7f9c6096b8fa4528/page/1)

> ### {% icon hands_on %} Hands-on: Import the data
> 1. Create a new history for this tutorial and give it a proper name
>
>    {% snippet faqs/galaxy/histories_create_new.md %}
>    {% snippet faqs/galaxy/histories_rename.md %}
>
> 2. Import the 9 `mzml` files into a collection
>    - The Data Library with these files is called `gc_training`
>    - Link to the library is [here](https://umsa.cerit-sc.cz/libraries/folders/F7f9c6096b8fa4528/page/1)
>
>    {% snippet faqs/galaxy/datasets_import_from_data_library.md %}
{: .hands_on}

## Reference data
There are two more reference dataset we are going to need: the `alkanes` and `spectral library`.

The list of `alkanes` with retention time and carbon number or retention index is used to compute the retention index of the deconvoluted peaks. The alkanes should be measured ideally in the same batch as the input sample collection. The file is in the data library [here](https://umsa.cerit-sc.cz/library/list#folders/F1c84aa7fc4490e6d/datasets/4582c858b46c378e)

The `reference spectral library` is used for identification via [matchms](https://github.com/matchms/matchms). The spectral library contains the recorded and annotated mass spectra of compounds which can be detected in the sample and confirmed via comparison with this library. We are going to use the RECETOX [in-house](https://www.recetox.muni.cz/sluzby/centralni-laboratore-recetox/laboratore-analyzy-biomarkeru/recetox-mass-spectrum-reference-library) library of metabolite standards. The file is available [here](https://umsa.cerit-sc.cz/library/list#folders/F1c84aa7fc4490e6d/datasets/b592e391ebe7cadd).

## Getting an overview of your samples' chromatograms
You may be interested in getting an overview of what your samples' chromatograms look like, for example to see if some of
your samples have distinct overall characteristics, *e.g.* unexpected chromatographic peaks or huge overall intensity.

> ### {% icon hands_on %} Hands-on: xcms plot chromatogram
>
> 1. Run {% tool [MSnbase readMSData](msnbase_readmsdata) %} with the following parameters:
>    - *Files from your history containing your chromatograms*: `seminal_plasma_list` (the collection with .mzml files)
>
> 2. Run {% tool [xcms plot chromatogram](xcms_plot_chromatogram) %} with the following parameters:
>    - *RData file*: `seminal_plasma_list.raw.RData` (the output of the readMSData step)
{: .hands_on}

This tool generates Base Peak Intensity Chromatograms (BPIs) and Total Ion Chromatograms (TICs).

## Peak Detection
The first four steps in the workflow are to detect the peaks in the `.mzml` data using [recetox-apLCMS](https://github.com/RECETOX/recetox-aplcms). In contrast to XCMS, apLCMS can process profile mode data and fits a bi-Gaussian peak shape model to the data, resulting in better peak detection than XCMS. Drifts in retention time are also corrected in recetox-apLCMS, outputting an aligned feature table. The four tools we are going to use are:

  1. extract features
  1. adjust time
  1. align features
  1. recover weaker signals

> ### {% icon hands_on %} Extract Features
> 1. Run the {% tool [RECETOX apLCMS - extract features](recetox_aplcms_extract_features) %} tool with the following parameters:
>   - *Input file*: `seminal_plasma_list` (the collection with .mzml files)
>   - *min_pres*: `0.5`
>   - *min_run*: `12.0`
>   - *mz_tol*: `1e-05`
>   - *baseline_correct*: `0.0`
>   - *baseline_correct_noise_percentile*: `0.0`
>   - *intensity_weighted*: `Yes`
>   - *shape_model*: `bi-Gaussian`
>   - *shape_model*: `bi-Gaussian`
{: .hands_on}

> ### {% icon hands_on %} Adjust Time
> 1. Run the {% tool [RECETOX apLCMS - adjust time](recetox_aplcms_adjust_time) %} tool with the following parameters:
>   - *Input data*: `RECETOX apLCMS - extract features on collection...` (the collection with output of the extract feature step)
>   - *mz_tol*: `1e-05`
>   - *max_align_mz_diff*: `0.01`
{: .hands_on}

> ### {% icon hands_on %} Align Features
> 1. Run the {% tool [RECETOX apLCMS - align features](recetox_aplcms_align_features) %} tool with the following parameters:
>   - *Input data collection*: `seminal_plasma_list` (the collection with .mzml files)
>   - *Input corrected feature samples collection*: `RECETOX apLCMS - adjust time corrected_feature_tables on data...` (the collection with output of the adjust time step)
>   - *min_exp*: `2`
>   - *mz_tol*: `1e-05`
>   - *max_align_mz_diff*: `0.01`
{: .hands_on}

> ### {% icon hands_on %} Recover Weaker Signals
> 1. Run the {% tool [RECETOX apLCMS - recover weaker signals](recetox_aplcms_recover_weaker_signals) %} tool with the following parameters:
>   - *Input data collection*: `seminal_plasma_list` (the collection with .mzml files)
>   - *Input data*: `RECETOX apLCMS - extract features on collection...` (the collection with output of the extract feature step)
>   - *Input corrected feature samples collection*: `RECETOX apLCMS - adjust time corrected_feature_tables on data...` (the collection with output of the adjust time step)
>   - *Input tolerances*: `RECETOX apLCMS - align features on data 45, data 44, and others (tolerances)`
>   - *Input rt cross table*: `RECETOX apLCMS - align features on data 45, data 44, and others (rt cross table)`
>   - *Input int cross table*: `RECETOX apLCMS - align features on data 45, data 44, and others (int cross table)`
>   - *mz_tol*: `1e-05`
>   - *use_observed_range*: `Yes`
>   - *recover_min_count*: `3`
{: .hands_on}

## Peak Deconvolution
The next step is deconvoluting the detected peaks in order to reconstruct the full spectra of the analysed compound. [RAMClustR](https://github.com/cbroeckl/RAMClustR) is used to group features based on correlations across samples in a hierarchy, focusing on consistency across samples.

> ### {% icon hands_on %} Data conversion
> Since the apLCMS and RamClustR use different data formats we will run a tool to convert our output from the previous step to format RamClustR takes as input.
> 1. Run the {% tool [apLCMS to RamClustR converter](aplcms_to_ramclustr_converter) %} tool with the following parameters:
>   - *apLCMS Dataframe*: `RECETOX apLCMS - recover weaker signals recovered_feature_sample_table on...` 
{: .hands_on}

> ### {% icon hands_on %} RAMClustR
> 2. Run the {% tool [RAMClustR](ramclustr) %} tool with the following parameters:
>   - *Choose input format*: `CSV`
>   - *Input CSV*: ` Feature-by-sample RECETOX apLCMS - recover weaker signals recovered_feature_sample_table on...`
{: .hands_on}

The deconvoluted features are outputted in the spectral library `.msp` format with unique identifiers and the list of peaks, as well as retention time values. The intensities of those respective features are also computed for each sample and stored in a tabular `.csv` file.

## Retention Index Calculation
We developed a new package [RIAssigner](https://github.com/RECETOX/RIAssigner) to compute retention indices for files in the .msp format using an indexed reference list in .csv or .msp format.

> ### {% icon hands_on %} RIAssigner
> 2. Run the {% tool [RIAssigner](riassigner) %} tool with the following parameters:
>   - *Query compound list*: `Mass spectra from RAMClustR on...` (one of the outputs of the previous step)
>   - *query_rt_units*: `seconds`
>   - *Reference compound list*: `SeminalPlasmaALKANES_KC_batch_1.csv` from the [data library](https://umsa.cerit-sc.cz/library/list#folders/F1c84aa7fc4490e6d/datasets/4582c858b46c378e)
>   - *reference_rt_units*: `minutes`
{: .hands_on}

The output follows the same format as the input, but with added retention index values. These will be used at a later stage to improve compound identification with an additional filtering step. Multiple computation methods (piecewise-linear & cubic spline) are supported.

## Identification
The deconvoluted spectra are annotated for identification by comparing them with a reference spectral library. This library contains spectra of standards measured on the same instrument for optimal comparability. The matchms package is used for spectral matching. The cosine score with a greedy peak pairing heuristic was used to compute the number of matching ions with a given tolerance and the cosine scores for the matched peaks.


> ### {% icon hands_on %}  matchMS similarity 
> 2. Run the {% tool [ matchMS similarity ](matchms) %} tool with the following parameters:
>   - *Queries spectra*: `RI using kovats of Mass spectra from RAMClustR on...` this is the output of the RIAssigner from the previous step
>   - *Reference spectra*: `rcx_gc-orbitrap_metabolites_20210817.msp` from the [data library](https://umsa.cerit-sc.cz/library/list#folders/F1c84aa7fc4490e6d/datasets/b592e391ebe7cadd)
>   - *Similarity metric*: `CosineHungarian`
{: .hands_on}

## Scores and Matches
The output table contains the scores and number of matched ions of the deconvoluted spectra with the spectra in the reference library. The raw output is filtered to only contain the top matches (3 by default) and is then further filtered to contain only pairs with a score and number of matched ions larger than provided thresholds (0.65 & 3 by default). The columns are ordered as `query/reference/matches/score`.

# The full Workflow
All steps within this training can be run as a single Galaxy workflow which is available as reference [here](https://github.com/RECETOX/workflow-testing/tree/main/gc-training-material).

---
layout: tutorial_hands_on

title: 'Mass Spectrometry: Untargeted Metabolomics with Molecular Networking'
zenodo_link: ''
questions:
- How molecular networking can help in dereplicating unidentified
  compounds from mass spectra?
- What similarity metrics can be used to compare a pair of mass spectra and what are the differences between them?
- What tools to use for molecular network visualizations?
objectives:
- Get to know the most common mass spectra similarity metrics
- Learn molecular networking basics
- Create a molecular network using Galaxy
time_estimation: 3H
level: Introductory
key_points:
- The take-home messages
- They will appear at the end of the tutorial
contributors:
- maximskorik

---


# Introduction

The study of metabolites in biological samples is routinely defined as metabolomics and provides the capability to investigate metabolism on a global and relatively unbiased scale in comparison to traditional targeted studies focused on specific pathways of metabolism and a small number of metabolites. The untargeted approach enables to detect thousands of metabolites in hypothesis-generating studies and to link previously unknown mettabolites with biologically important roles. There are two major issues in contemporary metabolomics: the first is enormous loads of signal generated during the experiments, and the second is the fact that some metabolites in the studied samples may not be known to us. These obstacles make the task of processing and interpreting the metabolomics data a cumbersome and time-consuming process ({% cite Nash2019 %}).

Molecular Networking is a powerful technique for visualizing chemical space of untargeted chromatography-mass spectrometry experiments. This method is based on plotting spectral data as networks of mass spectral nodes interconnected via edges that represent the similarity between corresponding pairs of spectra. Such an approach simplifies interpretation and discovery of the related molecules in your dataset even when some of them have not been matched to any known compound in the spectral libraries ({% cite Aksenov2017 %}).

In this tutorial you will learn how to prepare your data for a molecular networking analysis and use the networks to propagate spectra annotations. You will also learn how to compute similarity between a pair of spectra, and the differences between various similarity metrics. During the tutorial we will use several modules of [MatchMS](https://github.com/matchms/matchms) ({% cite Huber2020 %}) – a package for computational mass spectrometry, [MSMetaEnhancer](https://github.com/RECETOX/MSMetaEnhancer) ({% cite Troják2022 %}) – a tool for `.msp` files annotation, and explore graph visualization tools such as [Cytoscape](https://cytoscape.org/) ({% cite Shannon2003 %}) and [MetGem](https://metgem.github.io/) ({% cite Olivon2018 %}).

> <agenda-title></agenda-title>
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Dataset

Some background on the data.

## Load the Data

> <hands-on-title> Data Upload </hands-on-title>
>
> 1. Create a new history for this tutorial
> 2. Import the `.msp` file from [Zenodo]({{ page.zenodo_link }}) either by downloading it to your PC and uploading to Galaxy or directly via url link:
>
>    ```
>    # add url link to the dataset
>    ```
>
>    {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
>
> 3. Rename the dataset to `seminal_plasma_spectra`
> 4. Check that the file format in the Galaxy history is set to `msp`, set it to `msp` manually if it is different
>
>    {% snippet faqs/galaxy/datasets_change_datatype.md datatype="datatypes" %}
>
{: .hands_on}

## Explore the Data

This spectral data comes as MSP file, which is a text file structured according to the **NIST Search** spectra format.
MSP is one of the generally accepted formats for mass spectra representations and it is compatible with lots of spectra processing programms (MS-DIAL, NIST MS Search, AMDIS, etc.).

Because MSP files are text-based, they can be viewed as simple `txt` files. You can use any text editor that you have on your computer or use Galaxy built-in editor. In this tutorial we will use the Galaxy editor to check the contents of the file:
> <hands-on-title> Data Exploration </hands-on-title>
>
> 1. Click *"Visualize this data"* {% icon galaxy-barchart %} icon next to the dataset in the Galaxy history
> 2. Select the *"Editor"* {% icon galaxy-eye %} tool. The first spectra would look like this:
>
>    ```
>     1 NAME: UnknownID=2|RT=2.035|RI=1090.5
>     2 SCANNUMBER: 13
>     3 RETENTIONTIME: 2.035461
>     4 RETENTIONINDEX: 1090.48
>     5 MODELION: 154.0318
>     6 MODELIONHEIGHT: 8473462
>     7 MODELIONAREA: 8473462
>     8 INTEGRATEDHEIGHT: 8473462
>     9 INTEGRATEDAREA: 8473462
>    10 COMMENT: ID:2|RT:2.035461|RI:1090.48
>    11 Num Peaks: 78
>    12 70.02345	10915
>    13 71.04917	357446
>       ...
>    89 283.07889	56537
>    90
>       ...
>    ```
>
>    > <comment-title> Omitted lines </comment-title>
>    >
>    > The lines 14 to 88 have the same format as 12, 13, and 89, and are omitted for the sake of simplicity.
>    {: .comment}
>
{: .hands_on}

MSP files can contain one or more mass spectra, these are split by an empty line as the line **90** in the example above. The individual spectra essentialy consist of two sections: metadata and peaks. 

The metadata consists of compound name, retention time and index, base ion *m/z* and its intensity, and number of *m/z* peaks. In the example above this corresponds to the lines **1** to **11**. If the compound has not been identified as in our example spectrum, the *NAME* can be any arbitrary string. It is best however, if that string is unique within that `.msp` file. The *COMMENT* field stores some additional information about a spectrum; the number of mass spectrum peaks says how many *m/z* peaks is detected in that spectrum. The metadata fields are usually unordered, so it is quite common for one `.msp` file to contain **NAME** as the first metadata key, and for another `.msp` to have this key somewhere in the middle. The keys themselves also aren't rigid and can have different names (e.g., **compound_name** instead of **NAME**) or store information not present in this `.msp`, such as ionization mode.

The section with *m/z* peaks is quite simple. It consists of two tab separated columns: first tells us the *m/z* value of a peak and the second the intensity of that peak.

# Molecular Networking Workflow

It comes first a description of the step: some background and some theory.
Some image can be added there to support the theory explanation:

The idea is to keep the theory description before quite simple to focus more on the practical part.

***TODO***: *Consider adding a detail box to expand the theory*

> <details-title> More details about the theory </details-title>
>
> But to describe more details, it is possible to use the detail boxes which are expandable
>
{: .details}

A big step can have several subsections or sub steps:


## Clean and Normalize the Data

Before we dive into processing and analyzing the data it is useful to make sure all the spectra are normalized to a single format, which the upstream tools in our workflow can work with. We can achieve this with **matchMS filtering**. This tool has a variety a of options to normalize the data. It can process spectrums peaks like normalizing intensities if our spectra are combined from different sources or apply windows on *m/z* or intensity ranges; it will clean, correct, and harmonize metadata, and more. For a complete list of steps see [matchMS](https://matchms.readthedocs.io/en/latest/?badge=latest) documentation, in particular [filtering package](https://matchms.readthedocs.io/en/latest/api/matchms.filtering.html). The tool also includes helpful annotations under each parameter to get you started.

> <hands-on-title> Data filtering and normalization </hands-on-title>
>
> 1. Run {% tool [matchMS filtering](toolshed.g2.bx.psu.edu/repos/recetox/matchms_filtering/matchms_filtering/0.17.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Spectra file"*: `input.msp` (Input spectra in MSP format)
>    - *"Normalize intensities"*: `Yes`
>    - *"Apply default filters"*: `Yes`
>    - *"Clean metadata"*: `Yes`
>    - *"Filter relative intensity"*: `No`
>    - *"Filter m/z range"*: `No`
>
>    ***TODO***: Check parameter descriptions to see what filtering and normalization the tool can do.
>
>    > <comment-title> Output </comment-title>
>    >
>    > This will output an `msp` file with normalized intensities (from 0 to 1) and harmonized metadata. There is no need to apply intensity or
*m/z* windows since we want to keep all the peaks.
>    {: .comment}
>
{: .hands_on}

> <question-title></question-title>
>
> Check the first spectrum of the newly created dataset the same way as we did before an try to answer these questions:
> 1. Are there any changes in peaks intensities or metadata-field names?
> 2. Are there any new metadata fields?
>
> > <solution-title></solution-title>
> >
> > 1. All intensity values are now mapped to range from 0 to 1. Some metadata fields have been swapped in places and some of them renamed, e.g.,
`RETENTIONTIME` -> `RETENTION_TIME` and `NAME` -> `COMPOUND_NAME`. This will make this data compatible with the tools in the next steps of the workflow.
> > 2. The tool also tried to conclude the charge of precursor based on the ion mode. But since it couldn't find the ion mode for the spectra, it
created a new field `ION_MODE` and set it to `n/a` as well as added a `CHARGE` field with a default value of `0`.
> >
> {: .solution}
>
{: .question}

## Enhance Metadata with **MSMetaEnhancer**

Now that our data is clean and normalized we can query some public chemical databases to extend the information of annotated compounds. **MSMetaEnhancer** ({% cite Troják2022 %}) can perform various tasks and conversions, such as retrieve *SMILES*, *InChi* or *molecular weight* of annotated spectra. These additional metadata will be useful for us later, when we will be exploring a molecular network of this spectra.

> <hands-on-title> Metadata Retrieval </hands-on-title>
>
> 1. Run {% tool [MSMetaEnhancer](toolshed.g2.bx.psu.edu/repos/recetox/msmetaenhancer/msmetaenhancer/0.2.5+galaxy1) %} with the following parameters:
>    - {% icon param-file %} *"Input spectra dataset"*: `filtered msp spectra` (output of **matchMS filtering** {% icon tool %})
>    - In *"Ordered conversions"*:
>        - {% icon param-repeat %} *"Insert Ordered conversions"*
>            - *"Available conversions"*: `PubChem: compound_name -> canonical_smiles`
>        - {% icon param-repeat %} *"Insert Ordered conversions"*
>            - *"Available conversions"*: `PubChem: compound_name -> formula`
>        - {% icon param-repeat %} *"Insert Ordered conversions"*
>            - *"Available conversions"*: `PubChem: compound_name -> inchi`
>
>    ***TODO***: *Expand "Available conversions" dropdown menu on the tool's page and check the available conversions.*
>
>    > <comment-title> Output </comment-title>
>    >
>    > **MSMEtaEnhancer** will output a new `.msp` file with new metadata fields correspoding to the conversions that we selected. Also, notice how the conversions that we selected in this run use `compound_name` as a source. This is one of the cases where the metadata harmonization from the previous step is crutial.
>    {: .comment}
>
{: .hands_on}

> <question-title></question-title>
>
> How many services can **MSMetaEnhancer** query to fetch the metadata?
>
> > <solution-title></solution-title>
> >
> > Right now there are **7** different services: RDKit, IDSM, CTS, CIR, NLM, PubChem, and BridgeDB, but more are on the way!
> >
> {: .solution}
>
{: .question}

## Compute Spectral Similairity Scores

After having preprocessed the spectral data we can compute spectral similarity scores. To do this we use **matchMS similarity** {% icon tool %} tool, which can compute similarity scores not only for molecular networking, but also to query mass spectra against a library of known compounds.

> <hands-on-title> Task description </hands-on-title>
>
> 1. {% tool [matchMS similarity](toolshed.g2.bx.psu.edu/repos/recetox/matchms/matchms/0.17.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Queries spectra"*: `annotated msp spectra` (output of **MSMetaEnhancer** {% icon tool %})
>    - *"Symmetric"*: `Yes` (if we were to query our spectra agains a spectral library we would select `No`)
>    - *"Similarity metric"*: `CosineGreedy`
>
>    ***TODO***: *Check descriptions of "Algorithm Parameters" section. If you want to know more what they are for expand the "Details" section below.*
>
>    > <comment-title> Output </comment-title>
>    >
>    > The output of the tool is a `json` file. This format is very simple to read for our computers, but not so much for us.
If you would like to check the scores besides the molecular networking, you can use **matchMS output formatter** {% icon tool %}.
Just pass the `json` output to this tool and it will convert the data to a tab-separated file with a scores matrix. 
>    {: .comment}
>
{: .hands_on}

> <details-title> Overview of the spectral similarity scores </details-title>
>
> **Cosine Greedy**
> > foo
> >
>
> **Cosine Hungarian**
> > bar
> >
>
> **Modified Cosine**
> > foo
> >
>
> **Neutral Losses Cosine**
> > bar
> >
>
{: .details}

> <question-title></question-title>
>
> 1. Why, when computing similarity scores for molecular networking, we run **matchMS similarity** {% icon tool %} with *"Symmetric"* parameter set to `Yes`?
> 2. What similarity metrics can only be used for *MS<sup>n</sup>* spectra and not for *MS<sup>1</sup>* (you may need to go through *Details* section to find that out)?
>
> > <solution-title></solution-title>
> >
> > 1. Setting *"Symmetric"* to `No` is useful when we want to compute similarity between two different sets of spectra.
In this case Galaxy will let you pass a second file to the tool, which will be used as a reference.
A possible scenario is if we want to compute similarities between our experimental data and spectra from a spectral library.
During the molecular networking workflow however, we want to compute similarity scores between the spectra from within our experimental data and not another set of spectra.
> > 2. Answer for question2
> >
> {: .solution}
>
{: .question}

## Generate Molecular Network Graph

Finally, after we have computed the similarity scores, it is time to generate a molecular network file, which can be used by graph visualization tools such as [Cytoscape](https://cytoscape.org/) or [MetGem](https://metgem.github.io/). This can be done by **matchMS networking** tool.

> <hands-on-title> Task description </hands-on-title>
>
> 1. {% tool [matchMS networking](toolshed.g2.bx.psu.edu/repos/recetox/matchms_networking/matchms_networking/0.17.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Similarity scores"*: `json file with similarity scores` (output of **matchMS similarity** {% icon tool %})
>    - *"Network-graph format"*: `graphml`
>    - *"Identifier key"*: `compound_name` (a unique metadata identifier of each spectrum)
>
>    > <comment-title> Output </comment-title>
>    >
>    > **matchMS networking** {% icon tool %} can generate network files in various formats. You can select a format which is accepted by a
graph visualization tool of your liking. In this tutorial we will be using [Cytoscape](https://cytoscape.org/) and [MetGem](https://metgem.github.io/), which can import data in `graphml` format.
>    {: .comment}
>
{: .hands_on}

> <question-title></question-title>
>
> 1. Question1?
> 2. Question2?
>
> > <solution-title></solution-title>
> >
> > 1. Answer for question1
> > 2. Answer for question2
> >
> {: .solution}
>
{: .question}

# Visualize Molecular Network

# Conclusion

Sum up the tutorial and the key takeaways here. We encourage adding an overview image of the
pipeline used.
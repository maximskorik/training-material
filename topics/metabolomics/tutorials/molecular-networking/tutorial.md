---
layout: tutorial_hands_on

title: 'Mass Spectrometry: Untargeted Data Analysis with Molecular Networking'
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

Add section about untargeted mass spec.

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
> 2. Import the files from [Zenodo]({{ page.zenodo_link }}) or from
>    the shared data library (`GTN - Material` -> `{{ page.topic_name }}`
>     -> `{{ page.title }}`):
>
>    ```
>    
>    ```
>    ***TODO***: *Add the files by the ones on Zenodo here (if not added)*
>
>    ***TODO***: *Remove the useless files (if added)*
>
>    {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
>    {% snippet faqs/galaxy/datasets_import_from_data_library.md %}
>
> 3. Rename the datasets
> 4. Check that the datatype
>
>    {% snippet faqs/galaxy/datasets_change_datatype.md datatype="datatypes" %}
>
> 5. Add to each database a tag corresponding to ...
>
>    {% snippet faqs/galaxy/datasets_add_tag.md %}
>
{: .hands_on}

## Explore the Data

* Visualize the data with Galaxy built-in editor: {% icon galaxy-barchart %} **vis icon**
* Explain the fields

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

After having preprocessed the spectral data we can compute spectral similarity scores. The spectral similairity between each pair of spectra in the dataset will enable us to build a spectral similarity graph, better known as a Molecular Network.

> <hands-on-title> Task description </hands-on-title>
>
> 1. {% tool [matchMS similarity](toolshed.g2.bx.psu.edu/repos/recetox/matchms/matchms/0.17.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Queries spectra"*: `output_file` (output of **MSMetaEnhancer** {% icon tool %})
>    - *"Symmetric"*: `Yes`
>    - *"Apply RI filtering"*: `Yes`
>
>    ***TODO***: *Check parameter descriptions*
>
>    ***TODO***: *Consider adding a comment or tip box*
>
>    > <comment-title> short description </comment-title>
>    >
>    > A comment about the tool or something else. This box can also be in the main text
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

***TODO***: *Consider adding a question to test the learners understanding of the previous exercise*

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

## Generate Molecular Network Graph

> <hands-on-title> Task description </hands-on-title>
>
> 1. {% tool [matchMS networking](toolshed.g2.bx.psu.edu/repos/recetox/matchms_networking/matchms_networking/0.17.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Similarity scores"*: `similarity_scores` (output of **matchMS similarity** {% icon tool %})
>    - *"Identifier key"*: `compound_name`
>
>    ***TODO***: *Check parameter descriptions*
>
>    ***TODO***: *Consider adding a comment or tip box*
>
>    > <comment-title> short description </comment-title>
>    >
>    > A comment about the tool or something else. This box can also be in the main text
>    {: .comment}
>
{: .hands_on}

***TODO***: *Consider adding a question to test the learners understanding of the previous exercise*

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
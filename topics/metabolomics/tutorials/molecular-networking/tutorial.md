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

Molecular Networking (MN) is a powerful technique for visualizing chemical space of untargeted chromatography-mass spectrometry experiments. This approach makes it easier to discover the related molecules in your dataset even when some of them have not been matched to any known compound in the spectral libraries {% cite Aksenov2017 %}.

In this tutorial you will learn how to prepare your data for a molecular networking analysis and use the networks to propagate spectra annotations. You will also learn how to compute similarity between a pair of spectra, and the differences between various similarity metrics.

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

* Visualize the data with Galaxy built-in editor: **vis icon**
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


## Clean and Normalize Metadata

Before we dive into processing and analyzing the data it is useful to make sure all the spectra are normalized to a single format, which the upstream tools in our workflow can work with. We can achieve this with **matchMS filtering**. This tool has a variety a of options to normalize the data. It can process spectrums peaks like normalizing intensities if our spectra are combined from different sources or apply windows on *m/z* or intensity ranges; it will clean, correct, and harmonize metadata, and more. For a complete list of steps see [matchMS](https://matchms.readthedocs.io/en/latest/?badge=latest) documentation, in particular [filtering package](https://matchms.readthedocs.io/en/latest/api/matchms.filtering.html). The tool also includes helpful annotations under each parameter to get you started.

> <hands-on-title> Task description </hands-on-title>
>
> 1. Run {% tool [matchMS filtering](toolshed.g2.bx.psu.edu/repos/recetox/matchms_filtering/matchms_filtering/0.17.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Spectra file"*: `input.msp` (Input spectra in MSP format)
>    - *"Normalize intensities"*: `Yes`
>    - *"Apply default filters"*: `Yes`
>    - *"Clean metadata"*: `Yes`
>    - *"Filter relative intensity"*: `No`
>    - *"Filter m/z range"*: `No`
>
>    ***TODO***: Check parameter descriptions to see what the tool can do.
>
>    > <comment-title> short description </comment-title>
>    >
>    > There is no need to norma
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

## Enhance Metadata with **MSMetaEnhancer**

> <hands-on-title> Task description </hands-on-title>
>
> 1. {% tool [MSMetaEnhancer](toolshed.g2.bx.psu.edu/repos/recetox/msmetaenhancer/msmetaenhancer/0.2.5+galaxy1) %} with the following parameters:
>    - {% icon param-file %} *"Input spectra dataset"*: `output` (output of **matchMS filtering** {% icon tool %})
>    - In *"Ordered conversions"*:
>        - {% icon param-repeat %} *"Insert Ordered conversions"*
>            - *"Available conversions"*: `PubChem: compound_name -> canonical_smiles`
>        - {% icon param-repeat %} *"Insert Ordered conversions"*
>            - *"Available conversions"*: `PubChem: compound_name -> formula`
>        - {% icon param-repeat %} *"Insert Ordered conversions"*
>            - *"Available conversions"*: `PubChem: compound_name -> inchi`
>    - In *"Options"*:
>        - *"Save the log file"*: `Yes`
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

## Compute Spectral Similairity

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


## Re-arrange

To create the template, each step of the workflow had its own subsection.

***TODO***: *Re-arrange the generated subsections into sections or other subsections.
Consider merging some hands-on boxes to have a meaningful flow of the analyses*

# Visualize Molecular Network

# Conclusion

Sum up the tutorial and the key takeaways here. We encourage adding an overview image of the
pipeline used.
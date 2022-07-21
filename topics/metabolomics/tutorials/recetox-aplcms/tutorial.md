---
layout: tutorial_hands_on

title: 'Mass Spectrometry: Peak detection in profile mode HRMS data with recetox-aplcms'
zenodo_link: https://zenodo.org/record/6878356
questions: null
objectives: null
time_estimation: ''
key_points: null
contributors:
- hechth

---


# Introduction
{:.no_toc}

<!-- This is a comment. -->

General introduction about the topic and then an introduction of the
tutorial (the questions and the objectives). It is nice also to have a
scheme to sum up the pipeline used during the tutorial. The idea is to
give to trainees insight into the content of the tutorial and the (theoretical
and technical) key concepts they will learn.

You may want to cite some publications; this can be done by adding citations to the
bibliography file (`tutorial.bib` file next to your `tutorial.md` file). These citations
must be in bibtex format. If you have the DOI for the paper you wish to cite, you can
get the corresponding bibtex entry using [doi2bib.org](https://doi2bib.org).

With the example you will find in the `tutorial.bib` file, you can add a citation to
this article here in your tutorial like this:
{% raw %} `{% cite Batut2018 %}`{% endraw %}.
This will be rendered like this: {% cite Batut2018 %}, and links to a
[bibliography section](#bibliography) which will automatically be created at the end of the
tutorial.


**Please follow our
[tutorial to learn how to fill the Markdown]({{ site.baseurl }}/topics/contributing/tutorials/create-new-tutorial-content/tutorial.html)**

> ### Agenda
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Title for your first section

Give some background about what the trainees will be doing in the section.
Remember that many people reading your materials will likely be novices,
so make sure to explain all the relevant concepts.

## Title for a subsection
Section and subsection titles will be displayed in the tutorial index on the left side of
the page, so try to make them informative and concise!

# Hands-on Sections
Below are a series of hand-on boxes, one for each tool in your workflow file.
Often you may wish to combine several boxes into one or make other adjustments such
as breaking the tutorial into sections, we encourage you to make such changes as you
see fit, this is just a starting point :)

Anywhere you find the word "***TODO***", there is something that needs to be changed
depending on the specifics of your tutorial.

have fun!

## Get data

> ### {% icon hands_on %} Hands-on: Data upload
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

# Title of the section usually corresponding to a big step in the analysis

It comes first a description of the step: some background and some theory.
Some image can be added there to support the theory explanation:

![Alternative text](../../images/image_name "Legend of the image")

The idea is to keep the theory description before quite simple to focus more on the practical part.

***TODO***: *Consider adding a detail box to expand the theory*

> ### {% icon details %} More details about the theory
>
> But to describe more details, it is possible to use the detail boxes which are expandable
>
{: .details}

A big step can have several subsections or sub steps:


## Sub-step with **Thermo**

> ### {% icon hands_on %} Hands-on: Task description
>
> 1. {% tool [Thermo](toolshed.g2.bx.psu.edu/repos/galaxyp/thermo_raw_file_converter/thermo_raw_file_converter/1.3.4+galaxy0) %} with the following parameters:
>    - {% icon param-collection %} *"Thermo RAW file"*: `output` (Input dataset collection)
>    - *"Output format"*: `mzml`
>        - *"Use the peak picking provided by the native thermo library"*: `Yes`
>    - *"Extract additional detector data"*: `Yes`
>    - *"Include reference and exception data"*: `Yes`
>    - *"Select MS levels "*: `1`
>
>    ***TODO***: *Check parameter descriptions*
>
>    ***TODO***: *Consider adding a comment or tip box*
>
>    > ### {% icon comment %} Comment
>    >
>    > A comment about the tool or something else. This box can also be in the main text
>    {: .comment}
>
{: .hands_on}

***TODO***: *Consider adding a question to test the learners understanding of the previous exercise*

> ### {% icon question %} Questions
>
> 1. Question1?
> 2. Question2?
>
> > ### {% icon solution %} Solution
> >
> > 1. Answer for question1
> > 2. Answer for question2
> >
> {: .solution}
>
{: .question}

## Sub-step with **RECETOX apLCMS - extract features**

> ### {% icon hands_on %} Hands-on: Task description
>
> 1. {% tool [RECETOX apLCMS - extract features](toolshed.g2.bx.psu.edu/repos/recetox/recetox_aplcms_extract_features/recetox_aplcms_extract_features/0.9.4+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file"*: `output` (output of **Thermo** {% icon tool %})
>    - In *"Noise filtering and peak detection"*:
>        - *"min_run"*: `4.0`
>        - *"baseline_correct_noise_percentile"*: `0.001`
>
>    ***TODO***: *Check parameter descriptions*
>
>    ***TODO***: *Consider adding a comment or tip box*
>
>    > ### {% icon comment %} Comment
>    >
>    > A comment about the tool or something else. This box can also be in the main text
>    {: .comment}
>
{: .hands_on}

***TODO***: *Consider adding a question to test the learners understanding of the previous exercise*

> ### {% icon question %} Questions
>
> 1. Question1?
> 2. Question2?
>
> > ### {% icon solution %} Solution
> >
> > 1. Answer for question1
> > 2. Answer for question2
> >
> {: .solution}
>
{: .question}

## Sub-step with **RECETOX apLCMS - adjust time**

> ### {% icon hands_on %} Hands-on: Task description
>
> 1. {% tool [RECETOX apLCMS - adjust time](toolshed.g2.bx.psu.edu/repos/recetox/recetox_aplcms_adjust_time/recetox_aplcms_adjust_time/0.9.4+galaxy1) %} with the following parameters:
>    - {% icon param-file %} *"Input data"*: `feature_sample_table` (output of **RECETOX apLCMS - extract features** {% icon tool %})
>
>    ***TODO***: *Check parameter descriptions*
>
>    ***TODO***: *Consider adding a comment or tip box*
>
>    > ### {% icon comment %} Comment
>    >
>    > A comment about the tool or something else. This box can also be in the main text
>    {: .comment}
>
{: .hands_on}

***TODO***: *Consider adding a question to test the learners understanding of the previous exercise*

> ### {% icon question %} Questions
>
> 1. Question1?
> 2. Question2?
>
> > ### {% icon solution %} Solution
> >
> > 1. Answer for question1
> > 2. Answer for question2
> >
> {: .solution}
>
{: .question}

## Sub-step with **RECETOX apLCMS - align features**

> ### {% icon hands_on %} Hands-on: Task description
>
> 1. {% tool [RECETOX apLCMS - align features](toolshed.g2.bx.psu.edu/repos/recetox/recetox_aplcms_align_features/recetox_aplcms_align_features/0.9.4+galaxy1) %} with the following parameters:
>    - {% icon param-file %} *"Input data collection"*: `output` (output of **Thermo** {% icon tool %})
>    - {% icon param-file %} *"Input corrected feature samples collection"*: `corrected_feature_tables` (output of **RECETOX apLCMS - adjust time** {% icon tool %})
>    - *"min_exp"*: `1`
>
>    ***TODO***: *Check parameter descriptions*
>
>    ***TODO***: *Consider adding a comment or tip box*
>
>    > ### {% icon comment %} Comment
>    >
>    > A comment about the tool or something else. This box can also be in the main text
>    {: .comment}
>
{: .hands_on}

***TODO***: *Consider adding a question to test the learners understanding of the previous exercise*

> ### {% icon question %} Questions
>
> 1. Question1?
> 2. Question2?
>
> > ### {% icon solution %} Solution
> >
> > 1. Answer for question1
> > 2. Answer for question2
> >
> {: .solution}
>
{: .question}

## Sub-step with **RECETOX apLCMS - recover weaker signals**

> ### {% icon hands_on %} Hands-on: Task description
>
> 1. {% tool [RECETOX apLCMS - recover weaker signals](toolshed.g2.bx.psu.edu/repos/recetox/recetox_aplcms_recover_weaker_signals/recetox_aplcms_recover_weaker_signals/0.9.4+galaxy1) %} with the following parameters:
>    - {% icon param-file %} *"Input data collection"*: `output` (output of **Thermo** {% icon tool %})
>    - {% icon param-file %} *"Input extracted feature samples collection"*: `feature_sample_table` (output of **RECETOX apLCMS - extract features** {% icon tool %})
>    - {% icon param-file %} *"Input corrected feature samples collection"*: `corrected_feature_tables` (output of **RECETOX apLCMS - adjust time** {% icon tool %})
>    - {% icon param-file %} *"Input tolerances"*: `tolerances` (output of **RECETOX apLCMS - align features** {% icon tool %})
>    - {% icon param-file %} *"Input rt cross table"*: `rt_cross_table` (output of **RECETOX apLCMS - align features** {% icon tool %})
>    - {% icon param-file %} *"Input int cross table"*: `int_cross_table` (output of **RECETOX apLCMS - align features** {% icon tool %})
>
>    ***TODO***: *Check parameter descriptions*
>
>    ***TODO***: *Consider adding a comment or tip box*
>
>    > ### {% icon comment %} Comment
>    >
>    > A comment about the tool or something else. This box can also be in the main text
>    {: .comment}
>
{: .hands_on}

***TODO***: *Consider adding a question to test the learners understanding of the previous exercise*

> ### {% icon question %} Questions
>
> 1. Question1?
> 2. Question2?
>
> > ### {% icon solution %} Solution
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

# Conclusion
{:.no_toc}

Sum up the tutorial and the key takeaways here. We encourage adding an overview image of the
pipeline used.
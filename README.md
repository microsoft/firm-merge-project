# Firm Merge Project: A model to merge firm data across datasets utilizing exact & non-exact firm identifiers

> Please see the SUPPORT.md file for anything support related

By Rakan AlZagha, Mert Demirer, and Sida Peng at Microsoft Research

## Introduction
The formation of this model was primarily created to aid economists in the process of merging internal and external datasets for future research purposes. However, it can be utilized for any type of 'non-exact' data-matching (not limited to just firm-level data).

To accomplish this type of merge, string-matching algorithms were tested for their accuracy in matching records correctly. Ultimately, the **Jaro-Winkler distance** string metric was the most promising of all the algorithms tests. It is an edit-distance between two strings (much like other string-matching algorithms), but it utilizes a prefix scale that favors strings that match in the beginning. With this scaling factor, and given that most string variation occurs at the end of a string, we are able to normalize scores and choose the lowest result as an approximate match. 

*To note, a result of 0 is an exact match, while a result of 1 indicates no similarity.* 

Building on this, after we receive a Jaro-Winkler similarity score that is not 0 (non-exact), we need to classify the match as definite or approximate based on exact **record identifiers**. In the case of matching firm-level data, we can accomplish this multiple ways. Currently, the algorithm supports this **classification** utilizing geographical and industry variables. 

For example, say we take these two firm names. Our Jaro-Winkler score will not return a perfect match due to abbreviations, however, intuitevly we know that this is a correct matching. 

> "Some Random Company Incorporated"

> "Some Rand. Co. Inc."

Suppose we have **industry level** information on both of these records. For the sake of this example, "Some Random Company Incorporated" is in the Consumer Products Industry and is located in Scranton, PA. With the help of the industry identifier and a Jaro-Winkler threshold, we can then classify this as a definite match. The same logic applies to **geographical information** and any other unique identifiers.

Another way to merge 'fuzzy' data is via **domain name**. The benefit of this is that domain is mostly a binary way of classifiying a match as definite or approximate (given that other identifiers are corrupt/unreliable). A matched domain name indicates a definite match and is more reliable than name matching.

However, there is the underlying issue of corporations and other entities having multiple domain names, migrating their domain, or even getting acquired by another company (domain now linked to the parent company). Take the example below of a consumer products company with multiple domains linked to the same HTML page. 

> http://www.nike.com

> http://www.nikeshoes.com

With this issue, it was imperative to have a way of linking these two domains to the same record. To accomplish this, we added a **web-scraping algorithm** to extract the HTML webpage's title, which is an identifying factor to be shared by any domain pointed to the same HTML page. 

The script **merge_algorithm.R** incorporates all of these methods in the same script. As a user, you will be able to select whichever combination of the algorithm you desire to run on your datasets via a caller function (see-below). 

## How To Use

### Clone the Repository
To start, clone the repository to your local machine (or fork to your GitHub).
```
git clone https://github.com/microsoft/firm-merge-project
```
### Open the Script
Open RStudio or R on your machine of choice. Locate the repo and open the **merge_algorithm.R** file.

### Installing Dependencies
At the top of the script, there is a list of libraries that the algorithm depends on. Running the script will automatically install and open the necessary libraries, but just for reference, here is a list of the dependencies:
- "stringr", "bit64","data.table", "dplyr", "foreach", "stringdist", "doParallel", "openxlsx", "comparator", "compare", "rvest", "httr", "ggplot2", "doSNOW", "parallel", "progress", "svMisc", "fastmatch"

To note, the version of R utilized was **1.4.1717**.

### Global Options
There are four global options to set at the top of the script. 

```
toggle         <- TRUE or FALSE

nthread        <- getOption("sd_num_thread") or custome number

load_directory <- ""

save_directory <- ""
```

**toggle**: used for the _save_object()_ function to save objects when algorithm terminates, toggle to TRUE to save.

**nthread**: the algorithm utilizes parallelization to decrease run-time. As it is, the default option takes the total number of cores available on your machine and subtracts by one. Leaving one core is imperative to ensure your machine does not crash (need one core available for system processes). If you are running multiple scripts concurrently, utilizing parallel processing will slow down other tasks. Set to your desired number of cores. Please be aware that more cores usually means a faster run-time, but there is computational overhead (meaning algorithm may take longer to start or finish).

**load_directory**: location of where data should be imported from

**save_directory**: location of where data should be exported to

### Preparing Data
Depending on what facets of the algorithm your specific use case my entail, the following data requirments should be met after loading in the data. 

> Note: the general design behind the algorithm is that there is an external (denoted by 'ex') dataset and an internal dataset (denoted by 'in'). This is mearly to logically seperate two datasets that are to be merged. Merge-wise, the external data is being matched to the internal data **(ex: name in external dataset being mapped to a name in the internal dataset)**.

#### Column Names
- Internal Domain Name: 'in_domain'
- External Domain Name: 'ex_domain'
- Internal City: 'in_city'
- External City: 'ex_city'
- Internal Industry: 'in_industry'
- External Industry: 'ex_industry'

#### Name Formatting & Cleaning
Any firm names, or record name, should conform to the following cleaning and formatting principles to eliminate any inconsitencies in data. This should be applied to **BOTH** datasets. Utilize the _clean_name_data()_ function to accomplish this/further clean your data to your needs. 

- Make all records uppercase
- Remove entity identifiers & abbreviations (ex: INC., INCORPORATED, LLP, CORP., CORPORATION, etc...)
- Standardize either 'AND' or '&' across both datasets
- Either eliminate or complete abbreviations (& CO. versus AND COMPANY)
- Remove any punctuation ('-', ',', '.', etc...)
- Remove 'THE' from the beginning of strings
- **Optionally:** remove spaces...some benefits of doing so, although not a major algorithm boost


#### Domain Name Formatting & Cleaning
All domain names should conform to the following principles in order to be utilized by the html-title matching algorithm and for direct domain name matching. You may utilize the _clean_domain_name_data()_ function to clean domain name (please add/modify to your data needs).

- Make all records lowercase
- Domain names should be formatted as 'http://www.random_website.com'
- Remove anything past the '.com, .org, .edu, .gov, etc...) identifier (no '/specific_tab/')

### Running Algorithm
To run the algorithm, utilize the _algorithm_selection()_ menu of options function.

**An example of running the domain name algorithm:**

```
algorithm_selection(nthread, iterations = nrow(external_data), internal_data, external_data, domain_name_match = TRUE, in_domain_vector = internal_data$in_domain, ex_domain_vector = external_data$ex_domain)
```

**An example of running the Jaro-Winkler name algorithm:**

```
algorithm_selection(nthread, iterations = nrow(external_data), internal_data, external_data, name_match = TRUE, in_name_vector = internal_data$formatted_name, ex_name_vector = external_data$formatted_name)
```

**An example of running the html-title matching algorithm:**

```
algorithm_selection(nthread, iterations = nrow(external_data), internal_data, external_data, html_title_match = TRUE, in_domain_vector = internal_data$in_domain, ex_domain_vector = external_data$ex_domain)
```
In terms of run-time, the algorithm is fairly quick (depending on input size). A progress bar will show up for the Jaro-Winkler name matching and html-title matching. For larger datasets (~3 million entries), the algorithm can run in 24 hours depending on the number of records that need to be cross-checked. This, of course, is contingent upon the number of cores on your machine. The html-title matching faces a bottle-neck by the number of web-requests possible (system timer between requests). 

> You can also run a combination of these together! For example, all three can be toggled to TRUE, or domain name and Jaro-Winkler can be run together. 

### Generating Results
After results are output, you will have a variety of columns generated with results:
- **is_match**: if any record match was found, will be toggled to TRUE, if no results, FALSE
- **jw_match**: Jaro-Winkler match score for the closest match
- **def_match**: based on algorithm output, is the record that we have a definite match? TRUE if definite, FALSE if approximate.
- **internal_index**: final index of the matched record (external_index maps to the internal record index), domain internal index is prioritized over name if it is a definite match
- **internal_index_(algorithm name)**: internal index record link given by one of the algorithms

Utilizing the _match_thresholds()_ function, you can generate confidence thresholds for approximate matches. The thresholds are based on a hard cutoff from the Jaro-Winkler score. These were decided on via random sampling and manually checking the data, however, it is recommended for you to cross check your data results and set these thresholds based on your output. These thresholds can be defined striclty by the Jaro-Winkler score, or a combination of geographical data and industry data. This will also sort records as definite matches based on the variables passed in.

Finally, if needed, you can generate plots to visualize definite versus approximate matches generally or based on decile/thresholds using the _results_output()_ function. 

## Future of Algorithm
Currently, there are plans to develop this into an R library to make the use of these functions more accessible. Alongside this, converting this to Python is a possibility in the near future. 

Adding more features and improving the results is a priority.

## Acknowledgement
A huge thanks to Mert Demirer and Sida Peng for their guidance and support. This would not have been possible without their mentorship. 

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.

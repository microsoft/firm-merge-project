# Firm Merge Project: A model to merge firm data across datasets utilizing exact & non-exact firm identifiers

> Please see the SUPPORT.md file for anything support related

By Rakan AlZagha, Mert Demirer, and Sida Peng at Microsoft Research

## Introduction
The formation of this model was primarily created to aid economists in the process of merging internal and external datasets for future research purposes. However, it can be utilized for any type of 'non-exact' data-matching (not limited to just firm-level data).

To accomplish this type of merge, string-matching algorithms were tested for their accuracy in matching records correctly. Ultimately, the Jaro-Winkler distance string metric was the most promising of all the algorithms tests. It is an edit-distance between two strings (much like other string-matching algorithms), but it utilizes a prefix scale that favors strings that match in the beginning. With this scaling factor, and given that most string variation occurs at the end of a string, we are able to normalize scores and choose the lowest result as an approximate match. To note, a result of 0 is an exact match, while a result of 1 indicates no similarity. 

Building on this, after we receive a Jaro-Winkler similarity score that is not 0 (non-exact), we need to classify the match as definite or approximate based on exact record identifiers. In the case of matching firm-level data, we can accomplish this multiple ways. Currently, the algorithm supports this classification utilizing geographical and industry variables. 

For example, say we take these two firm names. Our Jaro-Winkler score will not return a perfect match due to abbreviations, however, intuitevly we know that this is a correct matching. 
> "Some Random Company Incorporated"
> "Some Rand. Co. Inc."
Suppose we have industry level information on both of these records. For the sake of this example, "Some Random Company Incorporated" is in the Consumer Products Industry and is located in Scranton, PA. With the help of the industry identifier and a Jaro-Winkler threshold, we can then classify this as a definite match. The same logic applies to geographical information and any other unique identifiers.

Another way to merge 'fuzzy' data is via domain name. The benefit of this is that domain is mostly a binary way of classifiying a match as definite or approximate (given that other identifiers are corrupt/unreliable). A matched domain name indicates a definite match and is more reliable than name matching.

However, there is the underlying issue of corporations and other entities having multiple domain names, migrating their domain, or even getting acquired by another company (domain now linked to the parent company). Take the example below of a consumer products company with multiple domains linked to the same HTML page. 
> http://www.nike.com
> http://www.nikeshoes.com
With this issue, it was imperative to have a way of linking these two domains to the same record. To accomplish this, we added a web-scraping algorithm to extract the HTML webpage's title, which is an identifying factor to be shared by any domain pointed to the same HTML page. 

The script **merge_algorithm.R** incorporates all of these methods in the same script. As a user, you will be able to select whichever combination of the algorithm you desire to run on your datasets via a caller function (see-below). 

## How To Use

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

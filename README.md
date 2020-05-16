# CitationNetworkCrawler
![](./user-documentation/System_Structure_Diagram.png)


# How to install

1. Python 3.5 or newer should be installed (installed by default on Ubuntu 16.04 or newer). 
2. MongoDB should be installed as the reference crawler depends on it. Refer to [./user-documentation/MongoDB_README.docx](./user-documentation/MongoDB_README.docx)
3. Run [./installation/python_install_script.sh](./installation/python_install_script.sh)

For installation of NodeJS for real-time visualization, refer to [./user-documentation/NodeJS_Visualization_README.docx](./user-documentation/NodeJS_Visualization_README.docx)

# Usage for data gathering

1. Put a starting paper in the database collection you want to use to store the data.
It needs to contain at least the "pure_title", "validation_method", "authors",
"year" and "source" fields. "include" should be set to true, "processed" to false,
"paper_id" to 0.
2. In crawlers.py, run recursive_crawler(db_collection) 

# Verifying accuracy

1. In crawlers.py, run initialize_dbs().
2. In crawlers.py, run crawl_ground_papers().
3. In data_verification.py, run verify_all_ground_papers(“sources3.json”).
4. When this completes, the results will be stored in the resources/statistics folder.

# crawlers.py

Interacts with the MongoDB database, contains the reference gathering crawlers.

**Methods**

> **recursive_crawler(db_collection)**

Endlessly crawls papers, i.e. it gathers the references of papers and puts them into the database.
Continues until the program is manually stopped. It needs a starting point, so the database must contain
at least one unprocessed paper when the function is called. The inner workings are as follows: Get unprocessed
paper from DB -> Get its references from IEEE, ACM or Cermine -> Put those references in the DB -> Get next
unprocessed paper from DB.

*db_collection* is the name of the MongoDB collection to use.

>**single_paper_crawler(title, validation_methods)**

Crawls a single paper's references. Puts them into a database collection with
its name in the format of {validation_method}_{alphanumeric underscore version of the title},
e.g. 'acm_pocketelasticephermeralstorageforserverlessanalytics'.

*title* is the original title of the paper to be crawled.

*validation_methods* can be one of the strings 'all', 'pdf', 'ieee' and 'acm'. The given method is used when finding the references.
As with the recursive crawler, it needs a starting point. The paper that is to be crawled is assumed to be in the db collection
named {validation_methods}_{sanitized_title}, where the latter is the title in lower-case with all non-alphanumeric
characters (including whitespace) removed. So if the paper "Paper about cats!" is to be crawled using only ACM to find references,
the DB collection 'acm_paperaboutcats' should contain the paper in question.

>**initialize_one_db(title_substring, validation_method)**

Used only when verifying the crawler's accuracy. Used to set up database collections for crawl_ground_papers().
It scans the ground truths folder for a file with a name containing the given *title_substring*. It then attempts
to locate the corresponding paper using the given validation_method and put it (**i.e. the paper itself, not its references**) into a database collection with
its name in the format of {validation_method}_{ground truth file name}, e.g. 'acm_pocketelasticephermeralstorageforserverlessanalytics'.

>**initialize_dbs()**

Similar to initialize_one_db, but does so for all files in the ground truths folder and all verification methods ('all', 'ieee',
'acm', 'pdf').

>**crawl_ground_papers()**

Looks for files in resources/ground_truths and gathers their references, once using each validation method
('all', 'ieee', 'acm', 'pdf').

>**validate_paper_all_methods(reference, validation_methods)**

Attempts to locate a referenced paper using the given validation methods. It returns if any of the attempts succeed,
returning a dictionary containing the succesful validation method and other info that makes it easy to locate the
paper in the future.

*reference* is a dictionary corresponding to a parsed reference from a paper as gathered by the crawler. Its keys correspond
to the MongoDB document fields.

*validation_methods* can be one of the strings 'all', 'pdf', 'ieee' and 'acm'. The given method is used when locating
the referenced paper. When using validation method 'all', the priority is
IEEE > ACM > PDF. I.e. it first attempts to locate the paper through IEEE, then ACM and lastly by PDF download.

# data_verification.py
Used when trying to assess the performance of the crawler by using a ground truth to compare results to.

# references_searcher.py
Get references and info of a given paper directly from IEEE, ACM, Aminer websites. Makes extensive use of the
Requests and BeautifulSoup libraries to access and scrape webpages.

# cermine_pdf_parsing.py
Uses the Cermine library to parse a PDF and return its references

# pdf_downloader.py
Given the title of a paper, try to download the pdf by using links from StartPage

# data_standardization.py
Utility methods to standardize database data

# common_lib.py
Utility methods used in several places, e.g. string sanitization and some common reference parsing/sanitization

# MongoDB document fields:

| Key                  | Description                                                                                                         | Example value                                                                               |
|----------------------|---------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| pure_title           | The actual title of the paper                                                                                       | Paper about CATS 3: A simulation                                                            |
| sanitized_title      | No whitespace, lowercase alphanumeric version                                                                       | paperaboutcats3asimulation                                                                  |
| source               | Venue/conference/journal/..                                                                                         | Proc. IPDPS '19                                                                             |
| year                 | Year of publishing (4-digit integer)                                                                                | 2015                                                                                        |
| paper_id             | Starting from 1, order of insertion into DB                                                                         | 3                                                                                           |
| parents_id           | list of paper_id's of the papers that cite the paper                                                                | [1, 5, 34]                                                                                  |
| children_id          | list of paper_id's of the paper's references                                                                        | [4, 36, 39]                                                                                 |
| processed            | The paper's references have been crawled                                                                            | false                                                                                       |
| include              | It is a paper whose references can and should be gathered  (i.e. not a Technical report, Thesis, website, book etc) | true                                                                                        |
| validation_method    | "ACM", "IEEE" or "PDF". The method that was used to locate the paper. Only for papers that have 'include' == true.  | ACM                                                                                         |
| ieee_id              | The paper's id on the IEEEExplore website. Only for papers with IEEE as validation_method.                          | 6006036                                                                                     |
| acm_id               | The paper's id on the ACM website. Only for papers with ACM as validation_method.                                   | 1950369                                                                                     |
| raw_reference_string | Only used in certain cases. Contains the entire reference string from IEEE or ACM, unparsed.                        | 7. R. O. Duda P. E. Hart and D. G. Stork. Pattern Classification. Wiley- Interscience 2001. |
| ENTRYTYPE            | Not reliable, can't be depended on. Attempts to classify the reference. Lowercase.                                  | website, unknown, book, inproceedings, article, ...                                         |                                                                                                           



Lab 06 - Regular Expressions and Web Scraping
================
Yufei Liu

# Learning goals

- Use a real world API to make queries and process the data.
- Use regular expressions to parse the information.
- Practice your GitHub skills.

# Lab description

In this lab, we will be working with the [NCBI
API](https://www.ncbi.nlm.nih.gov/home/develop/api/) to make queries and
extract information using XML and regular expressions. For this lab, we
will be using the `httr`, `xml2`, and `stringr` R packages.

This markdown document should be rendered using `github_document`
document ONLY and pushed to your *JSC370-labs* repository in
`lab06/README.md`.

## Question 1: How many sars-cov-2 papers?

Build an automatic counter of sars-cov-2 papers using PubMed. You will
need to apply XPath as we did during the lecture to extract the number
of results returned by PubMed in the following web address:

    https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2

Complete the lines of code:

``` r
# Downloading the website
website <- xml2::read_html("https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2")

# Finding the counts
counts <- xml2::xml_find_first(website, "//span[@class='value']")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
stringr::str_extract(counts, "[0-9,]{7}")
```

    ## [1] "218,713"

- How many sars-cov-2 papers are there?

*There are 218713 sars-cov-2 papers.*

Don’t forget to commit your work!

## Question 2: Academic publications on COVID19 related to Toronto

Use the function `httr::GET()` to make the following query:

1.  Baseline URL:
    <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi>

2.  Query parameters:

    - db: pubmed
    - term: covid19 toronto
    - retmax: 300

The parameters passed to the query are documented
[here](https://www.ncbi.nlm.nih.gov/books/NBK25499/).

``` r
library(httr)
query_ids <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(db = "pubmed",
               term = "covid19 toronto",
               retmax = 300)
)

# Extracting the content of the response of GET
ids <- httr::content(query_ids)
```

The query will return an XML object, we can turn it into a character
list to analyze the text directly with `as.character()`. Another way of
processing the data could be using lists with the function
`xml2::as_list()`. We will skip the latter for now.

Take a look at the data, and continue with the next question (don’t
forget to commit and push your results to your GitHub repo!).

``` r
# Take a look at the data
ids_text <- as.character(ids)
substr(ids_text, 0, 998)
```

    ## [1] "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<!DOCTYPE eSearchResult PUBLIC \"-//NLM//DTD esearch 20060628//EN\" \"https://eutils.ncbi.nlm.nih.gov/eutils/dtd/20060628/esearch.dtd\">\n<eSearchResult>\n  <Count>6863</Count>\n  <RetMax>300</RetMax>\n  <RetStart>0</RetStart>\n  <IdList>\n    <Id>38350768</Id>\n    <Id>38350292</Id>\n    <Id>38350210</Id>\n    <Id>38350016</Id>\n    <Id>38349519</Id>\n    <Id>38348598</Id>\n    <Id>38348438</Id>\n    <Id>38348131</Id>\n    <Id>38346919</Id>\n    <Id>38346896</Id>\n    <Id>38346886</Id>\n    <Id>38344865</Id>\n    <Id>38342293</Id>\n    <Id>38340955</Id>\n    <Id>38336708</Id>\n    <Id>38336492</Id>\n    <Id>38320778</Id>\n    <Id>38334940</Id>\n    <Id>38334706</Id>\n    <Id>38334695</Id>\n    <Id>38333890</Id>\n    <Id>38332736</Id>\n    <Id>38331861</Id>\n    <Id>38331098</Id>\n    <Id>38330987</Id>\n    <Id>38329905</Id>\n    <Id>38324970</Id>\n    <Id>38323703</Id>\n    <Id>38323501</Id>\n    <Id>38322144</Id>\n    <Id>38321333</Id>\n    <Id>38320044</Id>\n    <Id>38318236</Id>\n   "

## Question 3: Get details about the articles

The Ids are wrapped around text in the following way:
`<Id>... id number ...</Id>`. we can use a regular expression that
extract that information. Fill out the following lines of code:

``` r
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, "<Id>[0-9]+</Id>")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "<Id>|</Id>")
```

With the ids in hand, we can now try to get the abstracts of the papers.
As before, we will need to coerce the contents (results) to a list
using:

1.  Baseline url:
    <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi>

2.  Query parameters:

    - db: pubmed
    - id: A character with all the ids separated by comma, e.g.,
      “1232131,546464,13131”
    - retmax: 300
    - rettype: abstract

**Pro-tip**: If you want `GET()` to take some element literal, wrap it
around `I()` (as you would do in a formula in R). For example, the text
`"123,456"` is replaced with `"123%2C456"`. If you don’t want that
behavior, you would need to do the following `I("123,456")`.

``` r
publications <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi",
  query = list(
    db = "pubmed",
    id = paste(ids, collapse = ","),
    retmax = 300,
    rettype = "abstract"
    )
)

# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```

With this in hand, we can now analyze the data. This is also a good time
for committing and pushing your work!

## Question 4: Distribution of universities, schools, and departments

Using the function `stringr::str_extract_all()` applied on
`publications_txt`, capture all the terms of the form:

1.  University of …
2.  … Institute of …

Write a regular expression that captures all such instances

``` r
institution <- str_extract_all(
  publications_txt,
  "\\bUniversity of \\w+|\\w+ Institute of \\w+"
  ) 
institution <- unlist(institution)
as.data.frame(table(institution))
```

    ##                               institution Freq
    ## 1                 and Institute of Health    1
    ## 2             Caledon Institute of Social    1
    ## 3      California Institute of Technology    2
    ## 4            Canadian Institute of Health    3
    ## 5           Catalan Institute of Oncology    1
    ## 6          Chinese Institute of Engineers    1
    ## 7              CIHR Institute of Genetics    1
    ## 8                CIHR Institute of Health    1
    ## 9       College Institute of Neuroscience    1
    ## 10           Gordon Institute of Business    1
    ## 11      Graduate Institute of Acupuncture    1
    ## 12         Heidelberg Institute of Global    2
    ## 13                In Institute of Network    1
    ## 14             India Institute of Medical    1
    ## 15                 IZA Institute of Labor    1
    ## 16              Knowledge Institute of St    1
    ## 17              Leeds Institute of Health    1
    ## 18  Massachusetts Institute of Technology    3
    ## 19             Meghe Institute of Medical    1
    ## 20        National Institute of Arthritis    1
    ## 21         National Institute of Diabetes    1
    ## 22           National Institute of Health    1
    ## 23           National Institute of Mental    1
    ## 24     National Institute of Neurological    1
    ## 25          National Institute of Science    1
    ## 26      Postgraduate Institute of Medical    1
    ## 27         Research Institute of Genetics    1
    ## 28         Research Institute of Manitoba    1
    ## 29               Research Institute of St    3
    ## 30              Research Institute of the    5
    ## 31          Saveetha Institute of Medical    1
    ## 32            Sinai Institute of Critical    1
    ## 33                the Institute of Health    1
    ## 34                 University of Aberdeen    1
    ## 35                 University of Adelaide    1
    ## 36                  University of alberta    1
    ## 37                  University of Alberta   64
    ## 38                University of Amsterdam    2
    ## 39                University of Antioquia    1
    ## 40                  University of Applied    1
    ## 41                  University of Arizona    4
    ## 42                 University of Auckland    7
    ## 43                University of Barcelona    2
    ## 44                     University of Bari    5
    ## 45                    University of Basel    2
    ## 46                   University of Beirut    1
    ## 47                 University of Belgrade    3
    ## 48                   University of Bergen    1
    ## 49                   University of Berlin    2
    ## 50                     University of Bern    2
    ## 51               University of Birmingham    3
    ## 52                     University of Bonn    1
    ## 53                  University of Brescia   18
    ## 54                  University of Bristol    1
    ## 55                  University of British  115
    ## 56                  University of Calgary   54
    ## 57               University of California   12
    ## 58                University of Cambridge    5
    ## 59                 University of Campinas    1
    ## 60                     University of Cape    1
    ## 61                University of Cartagena    1
    ## 62                  University of Chicago    3
    ## 63                  University of Cologne    4
    ## 64                 University of Colorado    1
    ## 65              University of Connecticut    7
    ## 66               University of Copenhagen    1
    ## 67                     University of Doha    3
    ## 68                  University of Eastern    2
    ## 69                   University of Exeter    1
    ## 70                  University of Florida    1
    ## 71                   University of Foggia    2
    ## 72                   University of Gdansk    1
    ## 73                   University of Geneva    2
    ## 74                  University of Granada    3
    ## 75                University of Groningen    1
    ## 76                   University of Guelph    9
    ## 77                    University of Halle    1
    ## 78                   University of Hawaii    1
    ## 79                   University of Health    1
    ## 80                 University of Helsinki    2
    ## 81            University of Hertfordshire    1
    ## 82                     University of Hong   16
    ## 83                 University of Illinois    6
    ## 84                   University of Kansas    4
    ## 85                     University of Kent    2
    ## 86                  University of Koblenz    2
    ## 87                     University of Kyiv    2
    ## 88                        University of L    1
    ## 89                    University of Leeds    2
    ## 90                University of Ljubljana    1
    ## 91                   University of London    1
    ## 92                 University of Manitoba   21
    ## 93                 University of Mannheim    1
    ## 94                 University of Maryland    1
    ## 95                  University of Mashhad    2
    ## 96                  University of Medical   22
    ## 97                 University of Medicine    2
    ## 98                University of Melbourne    4
    ## 99                  University of Messina    2
    ## 100                  University of Mexico    1
    ## 101                University of Michigan    7
    ## 102                   University of Milan    1
    ## 103                  University of Milano    5
    ## 104                 University of Mineiro    1
    ## 105                  University of Modena    1
    ## 106             University of Montpellier    1
    ## 107                University of Montreal    3
    ## 108                University of Montréal    4
    ## 109                 University of Navarra    1
    ## 110                   University of Negev    1
    ## 111                     University of New    3
    ## 112               University of Newcastle    1
    ## 113            University of Newfoundland    2
    ## 114                 University of Nigeria    1
    ## 115                   University of North    4
    ## 116                University of Northern    1
    ## 117                  University of Norway    1
    ## 118                   University of Notre    1
    ## 119              University of Nottingham    2
    ## 120                 University of Ontario    1
    ## 121                  University of Oregon    2
    ## 122                    University of Oslo    1
    ## 123                  University of Ottawa   61
    ## 124                  University of Oxford   11
    ## 125                   University of Padua    1
    ## 126                 University of Paraiba   13
    ## 127                 University of Pelotas    3
    ## 128            University of Pennsylvania    7
    ## 129              University of Pittsburgh   19
    ## 130                University of Plymouth    1
    ## 131                University of Pretoria    1
    ## 132                  University of Public    1
    ## 133                  University of Punjab    1
    ## 134              University of Queensland    6
    ## 135                  University of Regina    2
    ## 136                     University of Rio    2
    ## 137               University of Rochester    1
    ## 138                    University of Rome    7
    ## 139                     University of São    1
    ## 140            University of Saskatchewan    3
    ## 141                 University of Seville    4
    ## 142              University of Sherbrooke    1
    ## 143               University of Singapore    5
    ## 144                   University of South    1
    ## 145                University of Southern    7
    ## 146                  University of Sydney   46
    ## 147                  University of Tehran    1
    ## 148                   University of Texas    3
    ## 149                     University of the    2
    ## 150               University of Timisoara    3
    ## 151                 University of Toronto  848
    ## 152                  University of Trento    1
    ## 153                    University of Utah    4
    ## 154                 University of Vermont    2
    ## 155                University of Victoria   14
    ## 156                  University of Vienna    3
    ## 157                University of Virginia    1
    ## 158                 University of Waikato    1
    ## 159                  University of Warsaw    2
    ## 160              University of Washington    7
    ## 161                University of Waterloo   16
    ## 162                 University of Western   16
    ## 163                 University of Windsor    1
    ## 164               University of Wisconsin    1
    ## 165           University of Witwatersrand    1
    ## 166               University of Wuppertal    2
    ## 167                    University of York    2
    ## 168                  University of Zurich    4

Repeat the exercise and this time focus on schools and departments in
the form of

1.  School of …
2.  Department of …

And tabulate the results

``` r
schools_and_deps <- str_extract_all(
  publications_txt,
  "\\bSchool of \\w+|\\bDepartment of \\w+"
  )
as.data.frame(table(schools_and_deps))
```

    ##                      schools_and_deps Freq
    ## 1              Department of Academic    1
    ## 2          Department of Agricultural    1
    ## 3           Department of Agriculture    1
    ## 4           Department of Anaesthesia    2
    ## 5       Department of Anaesthesiology    2
    ## 6               Department of Anatomy    1
    ## 7            Department of Anesthesia   19
    ## 8        Department of Anesthesiology   11
    ## 9          Department of Anthropology    1
    ## 10              Department of Applied    4
    ## 11         Department of Biochemistry   14
    ## 12            Department of Bioethics    1
    ## 13           Department of Biological    3
    ## 14              Department of Biology    8
    ## 15           Department of Biomedical    7
    ## 16        Department of Biostatistics    5
    ## 17               Department of Cancer    3
    ## 18           Department of Cardiology    2
    ## 19       Department of Cardiovascular    1
    ## 20                 Department of Cell    3
    ## 21            Department of Chemistry    1
    ## 22             Department of Clinical   25
    ## 23            Department of Cognition    1
    ## 24            Department of Cognitive    1
    ## 25            Department of Community   21
    ## 26             Department of Computer    6
    ## 27           Department of Continuity    1
    ## 28          Department of Criminology    1
    ## 29             Department of Critical   29
    ## 30              Department of Defence    1
    ## 31          Department of Dermatology   20
    ## 32        Department of Developmental    1
    ## 33           Department of Diagnostic    6
    ## 34                 Department of Drug    1
    ## 35              Department of Dynamic    2
    ## 36            Department of Economics    1
    ## 37            Department of Education    8
    ## 38           Department of Electrical    4
    ## 39            Department of Emergency   39
    ## 40        Department of Environmental    3
    ## 41         Department of Epidemiology   27
    ## 42         Department of Experimental    7
    ## 43               Department of Family  115
    ## 44              Department of General    1
    ## 45            Department of Geography    3
    ## 46           Department of Geriatrics    2
    ## 47               Department of Global    6
    ## 48           Department of Government    1
    ## 49           Department of Gynecology    1
    ## 50               Department of Health   75
    ## 51           Department of Healthcare    1
    ## 52                Department of Heath    1
    ## 53             Department of Homeland    1
    ## 54                Department of Human    2
    ## 55           Department of Humanities    2
    ## 56        Department of Immunobiology    2
    ## 57           Department of Immunology   20
    ## 58           Department of Infectious    2
    ## 59          Department of Information    1
    ## 60           Department of Integrated    1
    ## 61             Department of Internal   26
    ## 62        Department of International    3
    ## 63              Department of Justice    1
    ## 64          Department of Kinesiology    3
    ## 65           Department of Laboratory    9
    ## 66                 Department of Life    1
    ## 67          Department of Mathematics   10
    ## 68              Department of Medical   27
    ## 69             Department of Medicine  176
    ## 70               Department of Mental    1
    ## 71         Department of Microbiology   12
    ## 72            Department of Molecular   13
    ## 73             Department of National    4
    ## 74           Department of Nephrology    9
    ## 75      Department of Neuroimmunology    1
    ## 76            Department of Neurology   29
    ## 77         Department of Neuroscience    1
    ## 78         Department of Neurosurgery    3
    ## 79                  Department of Non    1
    ## 80              Department of Nursing    2
    ## 81            Department of Nutrition    1
    ## 82          Department of Nutritional   11
    ## 83           Department of Obstetrics   31
    ## 84         Department of Occupational    9
    ## 85             Department of Oncology    9
    ## 86         Department of Orthopaedics    2
    ## 87       Department of Otolaryngology    2
    ## 88  Department of Otorhinolaryngology    1
    ## 89           Department of Paediatric    2
    ## 90          Department of Paediatrics   69
    ## 91           Department of Palliative    1
    ## 92         Department of Paramedicine    1
    ## 93            Department of Pathology   47
    ## 94            Department of Pediatric    5
    ## 95           Department of Pediatrics   80
    ## 96         Department of Periodontics    2
    ## 97       Department of Pharmaceutical    2
    ## 98         Department of Pharmacology   13
    ## 99             Department of Pharmacy    1
    ## 100          Department of Philosophy    2
    ## 101            Department of Physical   20
    ## 102             Department of Physics    1
    ## 103          Department of Physiology    2
    ## 104       Department of Physiotherapy    3
    ## 105           Department of Political    7
    ## 106            Department of Politics    1
    ## 107          Department of Population    2
    ## 108          Department of Preventive    6
    ## 109          Department of Psychiatry  157
    ## 110       Department of Psychological    5
    ## 111          Department of Psychology   58
    ## 112        Department of Psychosocial    1
    ## 113              Department of Public   10
    ## 114           Department of Radiation   20
    ## 115           Department of Radiology    3
    ## 116      Department of Rehabilitation    1
    ## 117        Department of Reproductive    1
    ## 118            Department of Research    7
    ## 119         Department of Respiratory    1
    ## 120        Department of Rheumatology    1
    ## 121             Department of Service    1
    ## 122              Department of Social    6
    ## 123           Department of Sociology    4
    ## 124           Department of Somnology    1
    ## 125               Department of South    1
    ## 126               Department of Spine    4
    ## 127          Department of Statistics    3
    ## 128          Department of Supportive    6
    ## 129             Department of Surgery   20
    ## 130             Department of Systems    4
    ## 131          Department of Toxicology    1
    ## 132       Department of Translational    1
    ## 133            Department of Veterans    1
    ## 134          Department of Veterinary    1
    ## 135                  School of Allied   12
    ## 136                   School of Basic   10
    ## 137              School of Biomedical    1
    ## 138                School of Business    5
    ## 139                  School of Cancer    2
    ## 140                School of Clinical   10
    ## 141               School of Community    1
    ## 142                    School of Data    1
    ## 143               School of Dentistry    6
    ## 144               School of Economics    9
    ## 145             School of Educational    2
    ## 146             School of Engineering    5
    ## 147            School of Epidemiology   11
    ## 148               School of Geography    2
    ## 149                  School of Global    6
    ## 150                  School of Health   21
    ## 151                 School of Hygiene    8
    ## 152           School of International    1
    ## 153              School of Journalism    1
    ## 154             School of Kinesiology    4
    ## 155                    School of Life   11
    ## 156              School of Management    6
    ## 157                 School of Medical    1
    ## 158                School of Medicine  161
    ## 159                 School of Nursing   17
    ## 160            School of Occupational    8
    ## 161                School of Pharmacy    1
    ## 162                School of Physical    5
    ## 163           School of Physiotherapy    1
    ## 164              School of Population   24
    ## 165                 School of Primary    2
    ## 166              School of Psychiatry    1
    ## 167              School of Psychology    5
    ## 168                  School of Public  175
    ## 169          School of Rehabilitation   14
    ## 170                  School of Social    8
    ## 171                     School of the    2

## Question 5: Form a database

We want to build a dataset which includes the title and the abstract of
the paper. The title of all records is enclosed by the HTML tag
`ArticleTitle`, and the abstract by `AbstractText`.

Before applying the functions to extract text directly, it will help to
process the XML a bit. We will use the `xml2::xml_children()` function
to keep one element per id. This way, if a paper is missing the
abstract, or something else, we will be able to properly match PUBMED
IDS with their corresponding records.

``` r
pub_char_list <- xml2::xml_children(publications)
pub_char_list <- sapply(pub_char_list, as.character)
```

Now, extract the abstract and article title for each one of the elements
of `pub_char_list`. You can either use `sapply()` as we just did, or
simply take advantage of vectorization of `stringr::str_extract`

``` r
# Extract the abstract
# abstracts <- str_extract(pub_char_list, "<AbstractText>.+?</AbstractText>")
abstracts <- str_extract(pub_char_list, "<Abstract>(.|\n)+?</Abstract>")

# Clean all the HTML tags
abstracts <- str_remove_all(abstracts, "<.*?>")

# Clean all extra white space and new lines with an empty space.
abstracts <- str_replace_all(abstracts, "\\s+", " ")
# alternatively, you can also use str_replace_all(abstracts, "[ORIGINAL]", "[REPLACEMENT]")

# Find the number of articles without an abstract
count_na <- sum(is.na(abstracts))
count_na
```

    ## [1] 13

- How many of these don’t have an abstract?

*13 of these articles don’t have an abstract.*

- **Note:** By taking a look at some of the texts, I find out that there
  are some articles with abstract not enclosed by \<AbstractText\> sth.
  \</AbstractText\> but rather \<AbstractText sth.\> sth.
  \</AbstractText\>. Also, there are several labels of the abstract like
  BACKGROUND, METHOD, RESULTS, and CONCLUSIONS. The common structure of
  the abstract is enclosed by \<Abstract\> sth. \</Abstract\>.
  Therefore, I use this structure to find the abstract of the article.

Now, the title

``` r
# Extract the title
titles <- str_extract(pub_char_list, "<ArticleTitle>.+?</ArticleTitle>")

# Clean all the HTML tags
titles <- str_remove_all(titles, "<.*?>")

# Clean all extra white space and new lines with an empty space.
titles <- str_replace_all(titles, "\\s+", " ")

# Find the number of articles without a title
count_na <- sum(is.na(titles))
count_na
```

    ## [1] 0

- How many of these don’t have a title ?

*None of these articles don’t have an title.*

Finally, put everything together into a single `data.frame` and use
`knitr::kable` to print the results

``` r
database <- data.frame(
  ID = ids,
  Title = titles,
  Abstract = abstracts
)

#knitr::kable(database)
# Make the output look better
kableExtra::scroll_box(knitr::kable(database)) %>%
  kable_styling()
```

<div style="border: 1px solid #ddd; padding: 5px; ">

<table class="table" style="margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
ID
</th>
<th style="text-align:left;">
Title
</th>
<th style="text-align:left;">
Abstract
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
38350768
</td>
<td style="text-align:left;">
COVID-19 vaccines and adverse events of special interest: A
multinational Global Vaccine Data Network (GVDN) cohort study of 99
million vaccinated individuals.
</td>
<td style="text-align:left;">
The Global COVID Vaccine Safety (GCoVS) Project, established in 2021
under the multinational Global Vaccine Data Network™ (GVDN®),
facilitates comprehensive assessment of vaccine safety. This study aimed
to evaluate the risk of adverse events of special interest (AESI)
following COVID-19 vaccination from 10 sites across eight countries.
Using a common protocol, this observational cohort study compared
observed with expected rates of 13 selected AESI across neurological,
haematological, and cardiac outcomes. Expected rates were obtained by
participating sites using pre-COVID-19 vaccination healthcare data
stratified by age and sex. Observed rates were reported from the same
healthcare datasets since COVID-19 vaccination program rollout. AESI
occurring up to 42 days following vaccination with mRNA (BNT162b2 and
mRNA-1273) and adenovirus-vector (ChAdOx1) vaccines were included in the
primary analysis. Risks were assessed using observed versus expected
(OE) ratios with 95 % confidence intervals. Prioritised potential safety
signals were those with lower bound of the 95 % confidence interval
(LBCI) greater than 1.5. Participants included 99,068,901 vaccinated
individuals. In total, 183,559,462 doses of BNT162b2, 36,178,442 doses
of mRNA-1273, and 23,093,399 doses of ChAdOx1 were administered across
participating sites in the study period. Risk periods following
homologous vaccination schedules contributed 23,168,335 person-years of
follow-up. OE ratios with LBCI &gt; 1.5 were observed for Guillain-Barré
syndrome (2.49, 95 % CI: 2.15, 2.87) and cerebral venous sinus
thrombosis (3.23, 95 % CI: 2.51, 4.09) following the first dose of
ChAdOx1 vaccine. Acute disseminated encephalomyelitis showed an OE ratio
of 3.78 (95 % CI: 1.52, 7.78) following the first dose of mRNA-1273
vaccine. The OE ratios for myocarditis and pericarditis following
BNT162b2, mRNA-1273, and ChAdOx1 were significantly increased with LBCIs
&gt; 1.5. This multi-country analysis confirmed pre-established safety
signals for myocarditis, pericarditis, Guillain-Barré syndrome, and
cerebral venous sinus thrombosis. Other potential safety signals that
require further investigation were identified. Copyright © 2024 The
Author(s). Published by Elsevier Ltd.. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38350292
</td>
<td style="text-align:left;">
Sex differences in plasma proteomic markers in late-life depression.
</td>
<td style="text-align:left;">
Previous studies have shown significant sex-specific differences in
major depressive disorder (MDD) in multiple biological parameters. Most
studies focused on young and middle-aged adults, and there is a paucity
of information about sex-specific biological differences in older adults
with depression (aka, late-life depression (LLD)). To address this gap,
this study aimed to evaluate sex-specific biological abnormalities in a
large group of individuals with LLD using an untargeted proteomic
analysis. We quantified 344 plasma proteins using a multiplex assay in
430 individuals with LLD and 140 healthy comparisons (HC) (age range
between 60 and 85 years old for both groups). Sixty-six signaling
proteins were differentially expressed in LLD (both sexes). Thirty-three
proteins were uniquely associated with LLD in females, while six
proteins were uniquely associated with LLD in males. The main biological
processes affected by these proteins in females were related to
immunoinflammatory control. In contrast, despite the smaller number of
associated proteins, males showed dysregulations in a broader range of
biological pathways, including immune regulation pathways, cell cycle
control, and metabolic control. Sex has a significant impact on
biomarker changes in LLD. Despite some overlap in differentially
expressed biomarkers, males and females show different patterns of
biomarkers changes, and males with LLD exhibit abnormalities in a larger
set of biological processes compared to females. Our findings can
provide novel targets for sex-specific interventions in LLD. Copyright ©
2024 Elsevier B.V. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38350210
</td>
<td style="text-align:left;">
A global comparative analysis of the the inclusion of priority setting
in national COVID-19 pandemic plans: A reflection on the methods and the
accessibility of the plans.
</td>
<td style="text-align:left;">
Despite the swift governments’ response to the COVID-19 pandemic, there
remains a paucity of literature assessing the degree to which; priority
setting (PS) was included in the pandemic plans and the pandemic plans
were publicly accessible. This paper reflects on the methods employed in
a global comparative analysis of the degree to which countries
integrated PS into their COVID-19 pandemic plans based on Kapiriri &amp;
Martin’s framework. We also assessed if the accessibility of the plans
was related to the country’s transparency index. Through a three stage
search strategy, we accessed and reviewed 86 national COVID-19 pandemic
plans (and 11 Canadian provinces and territories). Secondary analysis
assessed any alignment between the readily accessible plans and the
country’s transparency index. 71 national plans were readily accessible
while 43 were not. There were no systematic differences between the
countries whose plans were readily available and those whose plans were
‘missing’. However, most of the countries with ‘missing’ plans tended to
have a low transparency index. The framework was adapted to the pandemic
context by adding a parameter on the need to plan for continuity of
priority routine services. While document review may be the most
feasible and appropriate approach to conducting policy analysis during
health emergencies, interviews and follow up document review would
assess policy implementation. Copyright © 2024. Published by Elsevier
B.V.
</td>
</tr>
<tr>
<td style="text-align:left;">
38350016
</td>
<td style="text-align:left;">
Impact of Delayed Dental Treatment during the COVID-19 Pandemic in an
Undergraduate Dental Clinic in Southwestern Ontario, Canada - A
Retrospective Chart Review.
</td>
<td style="text-align:left;">
To investigate the impact of a COVID-19 mandated lockdown on the type
and frequency of dental services accessed at an undergraduate dental
clinic in southwestern Ontario. We retrieved anonymized sociodemographic
(n = 4791) and billing data (n = 11616) of patients for 2 periods of 199
days, before (T1) and after (T2) lockdown. We applied descriptive
statistics and used Student’s t test to compare the type and frequency
of dental services provided between the 2 periods. We mapped forward
sortation area (FSA) codes of each patient. Of the 4791 patients seen
collectively in T1 and T2, most (67%) sought care before the lockdown.
In both periods, most patients were ≥ 60 years of age (51.8%), female
(33.9%) and residing in an urban area (88.6%). Compared with T1, there
was a significant increase in middle-aged adults (p = 0.002) and
significantly fewer patients earning over CAD 100 000 (p = 0.021) in T2.
A total of 11616 billable procedures were carried out during T1 and T2:
in T1, most procedures were preventative, whereas in T2, most were
related to urgent care. Significantly fewer males than females sought
urgent care, regardless of time. Finally, mapping showed a decrease in
patients from Toronto, central and northern Ontario and clustering of
patients in southwestern Ontario. We noted an overall reduction in
billed services following the COVID-19 lockdown. The decrease in both
billed services and patients seen during T2 demonstrates the impact of
COVID-19 on access to timely and definitive dental care during the first
2 years of the pandemic.
</td>
</tr>
<tr>
<td style="text-align:left;">
38349519
</td>
<td style="text-align:left;">
Pre- and post-COVID-19 gender trends in authorship for paediatric
radiology articles worldwide: a systematic review.
</td>
<td style="text-align:left;">
Gender inequalities in academic medicine persist despite progress over
the past decade. Evidence-based targeted interventions are needed to
reduce gender inequalities. This systematic review aimed to investigate
the impact of COVID-19 on gender trends in authorship of paediatric
radiology research worldwide. This prospectively registered,
PRISMA-compliant systematic review searched the following databases:
PubMed, MEDLINE, Web of Science, and Scopus from January 1, 2018, to May
29, 2023, with no restrictions on country of origin. Screening and data
extraction occurred independently and in duplicate. Gender of first,
last, and corresponding authors were determined using an artificial
intelligence-powered, validated, multinational database (
www.genderize.io ). Two time periods were categorised according to the
Johns Hopkins Center for Systems Science and Engineering: pre-COVID
(prior to March 2020) and peak and post-COVID (March 2020 onwards).
One-sample binomial testing was used to analyse proportion of authorship
based on gender. Categorical variables were described as frequencies and
percentages, and compared using testing chi-square or Fisher exact
testing, with a threshold of P&lt;0.05 representing statistical
significance. In total, 922 articles were included with 39 countries
represented. A statistically significant difference in authorship based
on gender persisted during the peak and post-COVID time period (March
2020 onwards) where women represented a statistically significant lower
proportion of last (35.5%) and corresponding (42.7%) authors
(P&lt;0.001, P=0.001, respectively). Statistically significant
differences for first authors were not found in either period (P=0.08
and P=0.48). This study identifies differences in gender trends for
authorship in paediatric radiology research worldwide. Future efforts to
increase authorship by women are needed. © 2024. The Author(s), under
exclusive licence to Springer-Verlag GmbH Germany, part of Springer
Nature.
</td>
</tr>
<tr>
<td style="text-align:left;">
38348598
</td>
<td style="text-align:left;">
Impact of COVID-19 pandemic on sleep parameters and characteristics in
individuals living with overweight and obesity.
</td>
<td style="text-align:left;">
Coronavirus disease 2019 (COVID-19) has been very challenging for those
living with overweight and obesity. The magnitude of this impact on
sleep requires further attention to optimise patient care and outcomes.
This study assessed the impact of the COVID-19 lockdown on sleep
duration and quality as well as identify predictors of poor sleep
quality in individuals with reported diagnoses of obstructive sleep
apnoea and those without sleep apnoea. An online survey (June-October
2020) was conducted with two samples; one representative of Canadians
living with overweight and obesity (n = 1089) and a second of
individuals recruited through obesity clinical services or patient
organisations (n = 980). While overall sleep duration did not decline
much, there were identifiable groups with reduced or increased sleep.
Those with changed sleep habits, especially reduced sleep, had much
poorer sleep quality, were younger, gained more weight and were more
likely to be female. Poor sleep quality was associated with medical,
social and eating concerns as well as mood disturbance. Those with sleep
apnoea had poorer quality sleep although this was offset to some degree
by use of CPAP. Sleep quality and quantity has been significantly
impacted during the early part of the COVID-19 pandemic in those living
with overweight and obesity. Predictors of poor sleep and the impact of
sleep apnoea with and without CPAP therapy on sleep parameters has been
evaluated. Identifying those at increased risk of sleep alterations and
its impact requires further clinical consideration. © 2024 World Obesity
Federation.
</td>
</tr>
<tr>
<td style="text-align:left;">
38348438
</td>
<td style="text-align:left;">
The global impact of the COVID-19 pandemic on pediatric spinal care: A
multi-centric study.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has affected healthcare worldwide since December
2019. We aimed to identify the effect of the COVID-19 pandemic on
outpatient clinic and surgical volumes and peri-operative complications
for pediatric spinal deformities patients. In this multi-center
retrospective study, outpatient visits (in-person and virtual care) and
pediatric spine surgeries volumes in four high-volume pediatric spine
centers were compared between March and December 2019 and the same
period in 2020. Peri-operative complications were collected and compared
in the same periods. Descriptive statistics were calculated, and
comparative analyses were performed. During the 2020 study period, the
outpatient visit (in-person and virtual care) volume decreased during
local lockdown periods by 71% for new patients (p &lt; 0.001) and 53%
for returning patients (p = 0.03). Overall, for 2020, there was a 20%
reduction in new patients (p = 0.001) and 21% decrease in returning
patients (p &lt; 0.001). During the pandemic, there was also 20% less
overall surgical volume of adolescent idiopathic scoliosis (AIS)
patients undergoing primary posterior spinal fusion, with a 70%
reduction during lockdown times (p &lt; 0.001). Complication rate and
profile were similar between periods. There was a significant decrease
in outpatient pediatric spine outpatient visits, particularly new
patients, which may increase the proportion of pediatric patients with
spinal deformities that present late, meeting surgical indication. This,
in combination with the reduction in surgical volume of AIS over the
first year of the pandemic, could result in an extended waitlist for
surgeries during years to come. Complication rate was similar for both
periods, suggesting it is safe to continue elective pediatric spine
surgery even in a time of a pandemic. level IV. © The Author(s) 2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38348131
</td>
<td style="text-align:left;">
The Intersections of COVID-19 Global Health Governance and Population
Health Priorities: Equity-Related Lessons Learned From Canada and
Selected G20 Countries.
</td>
<td style="text-align:left;">
Background: COVID-19-related global health governance (GHG) processes
and public health measures taken influenced population health priorities
worldwide. We investigated the intersection between COVID-19-related GHG
and how it redefined population health priorities in Canada and other
G20 countries. We analysed a Canada-related multilevel qualitative study
and a scoping review of selected G20 countries. Findings show the
importance of linking equity considerations to funding and
accountability when responding to COVID-19. Nationalism and limited
coordination among governance actors contributed to fragmented COVID-19
public health responses. COVID-19-related consequences were not
systematically negative, but when they were, they affected more
population groups living and working in conditions of vulnerability and
marginalisation. Policy options and recommendations: Six policy options
are proposed addressing upstream determinants of health, such as
providing sufficient funding for equitable and accountable global and
public health outcomes and implementing gender-focused policies to
reduce COVID-19 response-related inequities and negative consequences
downstream. Specific programmatic (e.g., assessing the needs of the
community early) and research recommendations are also suggested to
redress identified gaps. Conclusion: Despite the consequences of the
COVID-19 pandemic, programmatic and research opportunities along with
concrete policy options must be mobilised and implemented without
further delay. We collectively share the duty to act upon global health
justice. Copyright © 2024 Mac-Seing and Di Ruggiero.
</td>
</tr>
<tr>
<td style="text-align:left;">
38346919
</td>
<td style="text-align:left;">
“Going vaccine hunting”: Multilevel influences on COVID-19 vaccination
among racialized sexual and gender minority adults-a qualitative study.
</td>
<td style="text-align:left;">
High levels of COVID-19 vaccine hesitancy have been reported among Black
and Latinx populations, with lower vaccination coverage among racialized
versus White sexual and gender minorities. We examined multilevel
contexts that influence COVID-19 vaccine uptake, barriers to
vaccination, and vaccine hesitancy among predominantly racialized sexual
and gender minority individuals. Semi-structured online interviews
explored perspectives and experiences around COVID-19 vaccination.
Interviews were recorded, transcribed, uploaded into ATLAS.ti, and
reviewed using thematic analysis. Among 40 participants (mean age, 29.0
years \[SD, 9.6\]), all identified as sexual and/or gender minority,
82.5% of whom were racialized. COVID-19 vaccination experiences were
dominated by structural barriers: systemic racism, transphobia and
homophobia in healthcare and government/public health institutions;
limited availability of vaccination/appointments in vulnerable
neighborhoods; absence of culturally-tailored and multi-language
information; lack of digital/internet access; and prohibitive indirect
costs of vaccination. Vaccine hesitancy reflected in uncertainties about
a novel vaccine amid conflicting information and institutional mistrust
was integrally linked to structural factors. Findings suggest that the
uncritical application of “vaccine hesitancy” to unilaterally explain
undervaccination among marginalized populations risks conflating
structural and institutional barriers with individual-level
psychological factors, in effect placing the onus on those most
disenfranchised to overcome societal and institutional processes of
marginalization. Rather, disaggregating structural determinants of
vaccination availability, access, and institutional stigma and mistrust
from individual attitudes and decision-making that reflect vaccine
hesitancy, may support 1) evidence-informed interventions to mitigate
structural barriers in access to vaccination, and 2) culturally-informed
approaches to address decisional ambivalence in the context of
structural homophobia, transphobia, and racism.
</td>
</tr>
<tr>
<td style="text-align:left;">
38346896
</td>
<td style="text-align:left;">
Aerosol Generating Procedures and Associated Control/Mitigation
Measures: A position paper from the Canadian Dental Hygienists
Association and the American Dental Hygienists’ Association.
</td>
<td style="text-align:left;">
Background Since the outbreak of COVID-19, how to reduce the risk of
spreading viruses and other microorganisms while performing aerosol
generating procedures (AGPs) has become a challenging question within
the dental and dental hygiene communities. The purpose of this position
paper is to summarize the existing evidence about the effectiveness of
various mitigation methods used to reduce the risk of infection
transmission during AGPs in dentistry.Methods The authors searched six
databases, MEDLINE, EMBASE, Scopus, Web of Science, Cochrane Library,
and Google Scholar, for relevant scientific evidence published in the
last ten years (January 2012 to December 2022) to answer six research
questions about the the aspects of risk of transmission, methods,
devices, and personal protective equipment (PPE) used to reduce contact
with microbial pathogens and limit the spread of aerosols.Results A
total of 78 studies fulfilled the eligibility criteria. There was
limited literature to indicate the risk of infection transmission of
SARS-CoV-2 between dental hygienists and their patients. A number of
mouthrinses are effective in reducing bacterial contaminations in
aerosols; however, their effectiveness against SARS-CoV-2 was limited.
The combined use of eyewear, masks, and face shields are effective for
the prevention of contamination of the facial and nasal region, while
performing AGPs. High volume evacuation with or without an intraoral
suction, low volume evacuation, saliva ejector, and rubber dam (when
appropriate) have shown effectiveness in reducing aerosol transmission
beyond the generation site. Finally, the appropriate combination of
ventilation and filtration in dental operatories are effective in
limiting the spread of aerosols.Conclusion Aerosols produced during
clinical procedures can potentially pose a risk of infection
transmission between dental hygienists and their patients. The
implementation of practices supported by available evidence are best
practices to ensure patient and provider safety in oral health settings.
More studies in dental clinical environment would shape future practices
and protocols, ultimately to ensure safe clinical care delivery.
Copyright © 2024 The American Dental Hygienists’ Association.
</td>
</tr>
<tr>
<td style="text-align:left;">
38346886
</td>
<td style="text-align:left;">
Prevalence and drivers of nurse and physician distress in cardiovascular
and oncology programmes at a Canadian quaternary hospital network during
the COVID-19 pandemic: a quality improvement initiative.
</td>
<td style="text-align:left;">
To assess the prevalence and drivers of distress, a composite of
burnout, decreased meaning in work, severe fatigue, poor work-life
integration and quality of life, and suicidal ideation, among nurses and
physicians during the COVID-19 pandemic. Cross-sectional design to
evaluate distress levels of nurses and physicians during the COVID-19
pandemic between June and August 2021. Cardiovascular and oncology care
settings at a Canadian quaternary hospital network. 261 nurses and 167
physicians working in cardiovascular or oncology care. Response rate was
29% (428 of 1480). Survey tool to measure clinician distress using the
Well-Being Index (WBI) and additional questions about workplace-related
and COVID-19 pandemic-related factors. Among 428 respondents, nurses
(82%, 214 of 261) and physicians (62%, 104 of 167) reported high
distress on the WBI survey. Higher WBI scores (≥2) in nurses were
associated with perceived inadequate staffing (174 (86%) vs 28 (64%),
p=0.003), unfair treatment, (105 (52%) vs 11 (25%), p=0.005), and
pandemic-related impact at work (162 (80%) vs 22 (50%), p&lt;0.001) and
in their personal life (135 (67%) vs 11 (25%), p&lt;0.001), interfering
with job performance. Higher WBI scores (≥3) in physicians were
associated with perceived inadequate staffing (81 (79%) vs 32 (52%),
p=0.001), unfair treatment (44 (43%) vs 13 (21%), p=0.02), professional
dissatisfaction (29 (28%) vs 5 (8%), p=0.008), and pandemic-related
impact at work (84 (82%) vs 35 (56%), p=0.001) and in their personal
life (56 (54%) vs 24 (39%), p=0.014), interfering with job performance.
High distress was common among nurses and physicians working in
cardiovascular and oncology care settings during the pandemic and linked
to factors within and beyond the workplace. These results underscore the
complex and contextual aspects of clinician distress, and the need to
develop targeted approaches to effectively address this problem. ©
Author(s) (or their employer(s)) 2024. Re-use permitted under CC BY-NC.
No commercial re-use. See rights and permissions. Published by BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38344865
</td>
<td style="text-align:left;">
Recommendations Related to Visitor and Movement Restrictions in
Long-Term Care and Retirement Homes in Ontario during the COVID-19
Pandemic: Perspectives of Residents, Families, and Staff.
</td>
<td style="text-align:left;">
In Canada, long-term care and retirement home residents have experienced
high rates of COVID-19 infection and death. Early efforts to protect
residents included restricting all visitors as well as movement inside
homes. These restrictions, however, had significant implications for
residents’ health and well-being. Engaging with those most affected by
such restrictions can help us to better understand their experiences and
address their needs. In this qualitative study, 43 residents of
long-term care or retirement homes, family members and staff were
interviewed and offered recommendations related to infection control,
communication, social contact and connection, care needs, and policy and
planning. The recommendations were examined using an ethical framework,
providing potential relevance in policy development for public health
crises. Our results highlight the harms of movement and visiting
restrictions and call for effective, equitable, and transparent
measures. The design of long-term care and retirement policies requires
ongoing, meaningful engagement with those most affected.
</td>
</tr>
<tr>
<td style="text-align:left;">
38342293
</td>
<td style="text-align:left;">
fRace and Ethnicity Research in Cardiovascular Disease in Canada:
Challenges and Opportunities.
</td>
<td style="text-align:left;">
Even though Canada is one of the most culturally diverse countries in
the world, offering universal healthcare to all its citizens, the recent
COVID-19 pandemic has exposed significant health inequities in our
healthcare system. We continue to face challenges ensuring health equity
in cardiovascular diseases. Persistent outcome disparities may be driven
by race and ethnicity-based differences in healthcare delivery,
treatment, follow-up, and outcomes. However, we lack data about these
processes because they are not routinely collected. There are
significant opportunities to implement sustainable processes to collect
data that can generate evidence to inform and deliver equitable
healthcare and address racial and ethnic disparities in cardiovascular
disease. Copyright © 2024. Published by Elsevier Inc. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38340955
</td>
<td style="text-align:left;">
Impact of Telehealth Post-operative Care on Early Outcomes Following
Esophagectomy.
</td>
<td style="text-align:left;">
This study aimed to address the short-term clinical outcomes of
post-esophagectomy patients who underwent telehealth care following
surgery. The primary objective was to compare the frequency of emergency
department admission between telehealth and in-person cohorts. Secondary
objectives included comparing the frequency of endoscopies and clinic
visits, as well as reasons for emergency department admission. We
conducted a retrospective cohort study to assess the clinical outcomes
of esophagectomy patients between March 2018 and May 2022. Patients
attending telehealth (phone or video call) surgical follow-up visits,
largely due to the COVID-19 pandemic, were compared to a pre-COVID
cohort of patients attending standard in person care. Demographic data,
clinical and disease characteristics, and hospital visit data within 6
months of operation were collected. This included surgical clinic
visits, endoscopies, and emergency department admissions. There were 168
esophagectomy patients who underwent follow-up care between March 2018
to May 2022; 76 telehealth and 92 in-person. Patients attending
telehealth appointments had significantly fewer emergency department
admissions (0.45 vs. 0.79, p = 0.037) and more endoscopy visits (1.37
vs. 0.91, p = 0.020) compared to patients attending in-person visits.
The number of follow-up surgical clinic visits did not differ between
the groups. The most frequent reasons for emergency visits for the
telehealth cohort included dysphagia, feeding tube problems, and failure
to thrive. For the in-person cohort, feeding tube complications,
inflammation/infection, and failure to thrive were the most common
reasons. A program of virtual follow-up, with integrated in person
visits and endoscopy as required, is feasible and safe for following
patients post esophagectomy. Copyright © 2024. Published by Elsevier
Inc. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38336708
</td>
<td style="text-align:left;">
Evaluation of an automated matching system of children and families to
virtual mental health resources during COVID-19.
</td>
<td style="text-align:left;">
Children and their families often face obstacles in accessing mental
health (MH) services. The purpose of this study was to develop and pilot
test an electronic matching process to match children with virtual MH
resources and increase access to treatment for children and their
families during COVID-19. Within a large observational child cohort, a
random sample of 292 families with children ages 6-12 years were invited
to participate. Latent profile analysis indicated five MH profiles using
parent-reported symptom scores from validated depression, anxiety,
hyperactivity, and inattention measures: (1) Average Symptoms, (2) Low
Symptoms, (3) High Symptoms, (4) Internalizing, and (5) Externalizing.
Children were matched with virtual MH resources according to their
profile; parents received surveys at Time 1 (matching process
explanation), Time 2 (match delivery) and Time 3 (resource uptake). Data
on demographics, parent MH history, and process interest were collected.
128/292 families (44%) completed surveys at Time 1, 80/128 families
(63%) at Time 2, and a final 67/80 families (84%) at Time 3, yielding an
overall uptake of 67/292 (23%). Families of European-descent and those
with children assigned to the Low Symptoms profile were most likely to
express interest in the process. No other factors were associated with
continued interest or uptake of the electronic matching process. Most
participating parents were satisfied with the process. The electronic
matching process delivered virtual MH resources to families in a
time-efficient manner. Further research examining the effectiveness of
electronically matched resources in improving children’s MH symptoms is
needed. © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38336492
</td>
<td style="text-align:left;">
Serologic Response to Vaccine for COVID-19 in Patients with Hematologic
Malignancy: A Prospective Cohort Study.
</td>
<td style="text-align:left;">
Patients with hematological cancers have increased COVID-19 morbidity
and mortality, and these patients show attenuated vaccine responses.
This study aimed to characterize the longitudinal humoral immune
responses to COVID-19 vaccination in patients with hematological
malignancies. We conducted a prospective cohort study, collecting
samples from March 2021 to July 2022, from patients seen at a cancer
treatment center in London, Ontario, Canada, who met the following
eligibility criteria: age ≥18 years, diagnosed with a hematological
malignancy, recipient of a COVID-19 vaccine during the study period, and
able to provide informed consent. Median anti-S titers (MST) were 0.0,
64.0, and 680.5 U/mL following first (V1), second (V2), and third (V3)
vaccine doses, respectively. Patients with lymphoid malignancies’
response to vaccination was attenuated compared to myeloid malignancy
patients after V2 and V3 (P &lt; .001, P &lt; .01). Active treatment was
associated with lower antibody titers (MST 10) compared to treatment
12-24 months (MST 465, P = .04367) and &gt;24 months (MST 1660.5, P =
.0025) prior to vaccination. V3 significantly increased antibody titers
compared to V2 for patients less than 3 months from treatment.
Increasing age was associated with smaller antibody response following
V2 (P &lt; .05), but not following V3. Patients receiving anti-CD20
therapy did not demonstrate increased antibody titer levels after V3 (V2
MST 0, V3 MST 0; P &gt; .05). We report an attenuated serologic response
to COVID-19 vaccination in our study population of patients with
hematological malignancy. The immune response to vaccination was
affected by patient age, diagnosis, treatment, and timing of treatment
exposure. Copyright © 2024 The Author(s). Published by Elsevier Inc. All
rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38320778
</td>
<td style="text-align:left;">
The impact of public health lockdown measures during the COVID-19
pandemic on the epidemiology of children’s orthopedic injuries requiring
operative intervention.
</td>
<td style="text-align:left;">
In March 2020, Ontario instituted a lockdown to reduce spread of the
SARS-CoV-2 virus. Schools, recreational facilities, and nonessential
businesses were closed. Restrictions were eased through 3 distinct
stages over a 6-month period (March to September 2020). We aimed to
determine the impact of each stage of the COVID-19 public health
lockdown on the epidemiology of operative pediatric orthopedic trauma. A
retrospective cohort study was performed comparing emergency department
(ED) visits for orthopedic injuries and operatively treated orthopedic
injuries at a level 1 pediatric trauma centre during each lockdown stage
of the pandemic with caseloads during the same date ranges in 2019
(prepandemic). Further analyses were based on patients’ demographic
characteristics, injury severity, mechanism of injury, and anatomic
location of injury. Compared with the prepandemic period, ED visits
decreased by 20% (1356 v. 1698, p &lt; 0.001) and operative cases by 29%
(262 v. 371, p &lt; 0.001). There was a significant decrease in the
number of operative cases per day in stage 1 of the lockdown (1.3 v.
2.0, p &lt; 0.001) and in stage 2 (1.7 v. 3.0; p &lt; 0.001), but there
was no significant difference in stage 3 (2.4 v. 2.2, p = 0.35). A
significant reduction in the number of playground injuries was seen in
stage 1 (1 v. 62, p &lt; 0.001) and stage 2 (6 v. 35, p &lt; 0.001), and
there was an increase in the number of self-propelled transit injuries
(31 v. 10, p = 0.002) during stage 1. In stage 3, all patient
demographic characteristics and all characteristics of operatively
treated injuries resumed their prepandemic distributions. Provincial
lockdown measures designed to limit the spread of SARS-CoV-2
significantly altered the volume and demographic characteristics of
pediatric orthopedic injuries that required operative management. The
findings from this study will serve to inform health system planning for
future emergency lockdowns. © 2024 CMA Impact Inc. or its licensors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38334940
</td>
<td style="text-align:left;">
Virtual urgent care is here to stay: driving toward safe, equitable, and
sustainable integration within emergency medicine.
</td>
<td style="text-align:left;">
Virtual care in Canada rapidly expanded during the COVID-19 pandemic in
a low-rules environment in response to pressing needs for ongoing access
to care amid public health restrictions. Emergency medicine specialists
now face the challenge of advising on which virtual urgent care services
ought to remain as part of comprehensive emergency care. Consideration
must be given to safe, quality, and appropriate care as well as issues
of equitable access, public demand, and sustainability (financial and
otherwise). The aim of this project was to summarize current literature
and expert opinion and formulate recommendations on the path forward for
virtual care in emergency medicine. We formed a working group of
emergency medicine physicians from across Canada working in a variety of
practice settings. The virtual care working group conducted a scoping
review of the literature and met monthly to discuss themes and develop
recommendations. The final recommendations were circulated to
stakeholders for input and subsequently presented at the 2023 Canadian
Association of Emergency Physicians (CAEP) Academic Symposium for
discussion, feedback, and refinement. The working group developed and
reached unanimity on nine recommendations addressing the themes of
system design, equity and accessibility, quality and patient safety,
education and curriculum, financial models, and sustainability of
virtual urgent care services in Canada. Virtual urgent care has become
an established service in the Canadian health care system. Emergency
medicine specialists are uniquely suited to provide leadership and
guidance on the optimal delivery of these services to enhance and
complement emergency care in Canada. © 2024. The Author(s), under
exclusive licence to Canadian Association of Emergency Physicians
(CAEP)/ Association Canadienne de Médecine d’Urgence (ACMU).
</td>
</tr>
<tr>
<td style="text-align:left;">
38334706
</td>
<td style="text-align:left;">
The independent and combined impact of moral injury and moral distress
on post-traumatic stress disorder symptoms among healthcare workers
during the COVID-19 pandemic.
</td>
<td style="text-align:left;">
Background: Healthcare workers (HCWs) across the globe have reported
symptoms of Post-Traumatic Stress Disorder (PTSD) during the COVID-19
pandemic. Moral Injury (MI) has been associated with PTSD in military
populations, but is not well studied in healthcare contexts. Moral
Distress (MD), a related concept, may enhance understandings of MI and
its relation to PTSD among HCWs. This study examined the independent and
combined impact of MI and MD on PTSD symptoms in Canadian HCWs during
the pandemic.Methods: HCWs participated in an online survey between
February and December 2021, with questions regarding sociodemographics,
mental health and trauma history (e.g. MI, MD, PTSD, dissociation,
depression, anxiety, stress, childhood adversity). Structural equation
modelling was used to analyze the independent and combined impact of MI
and MD on PTSD symptoms (including dissociation) among the sample when
controlling for sex, age, depression, anxiety, stress, and childhood
adversity.Results: A structural equation model independently regressing
both MI and MD onto PTSD accounted for 74.4% of the variance in PTSD
symptoms. Here, MI was strongly and significantly associated with PTSD
symptoms (β = .412, p &lt; .0001) to a higher degree than MD (β = .187,
p &lt; .0001), after controlling for age, sex, depression, anxiety,
stress and childhood adversity. A model regressing a combined MD and MI
construct onto PTSD predicted approximately 87% of the variance in PTSD
symptoms (r2 = .87, p &lt; .0001), with MD/MI strongly and significantly
associated with PTSD (β = .813, p &lt; .0001), after controlling for
age, sex, depression, anxiety, stress, and childhood
adversity.Conclusion: Our results support a relation between MI and PTSD
among HCWs and suggest that a combined MD and MI construct is most
strongly associated with PTSD symptoms. Further research is needed
better understand the mechanisms through which MD/MI are associated with
PTSD.
</td>
</tr>
<tr>
<td style="text-align:left;">
38334695
</td>
<td style="text-align:left;">
Exposure to moral stressors and associated outcomes in healthcare
workers: prevalence, correlates, and impact on job attrition.
</td>
<td style="text-align:left;">
Introduction: Healthcare workers (HCWs) often experience morally
challenging situations in their workplaces that may contribute to job
turnover and compromised well-being. This study aimed to characterize
the nature and frequency of moral stressors experienced by HCWs during
the COVID-19 pandemic, examine their influence on psychosocial-spiritual
factors, and capture the impact of such factors and related moral
stressors on HCWs’ self-reported job attrition intentions.Methods: A
sample of 1204 Canadian HCWs were included in the analysis through a
web-based survey platform whereby work-related factors (e.g. years spent
working as HCW, providing care to COVID-19 patients), moral distress
(captured by MMD-HP), moral injury (captured by MIOS), mental health
symptomatology, and job turnover due to moral distress were
assessed.Results: Moral stressors with the highest reported frequency
and distress ratings included patient care requirements that exceeded
the capacity HCWs felt safe/comfortable managing, reported lack of
resource availability, and belief that administration was not addressing
issues that compromised patient care. Participants who considered
leaving their jobs (44%; N = 517) demonstrated greater moral distress
and injury scores. Logistic regression highlighted burnout (AOR = 1.59;
p &lt; .001), moral distress (AOR = 1.83; p &lt; .001), and moral injury
due to trust violation (AOR = 1.30; p = .022) as significant predictors
of the intention to leave one’s job.Conclusion: While it is impossible
to fully eliminate moral stressors from healthcare, especially during
exceptional and critical scenarios like a global pandemic, it is crucial
to recognize the detrimental impacts on HCWs. This underscores the
urgent need for additional research to identify protective factors that
can mitigate the impact of these stressors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38333890
</td>
<td style="text-align:left;">
A systematic review of studies on stress during the COVID-19 pandemic by
visualizing their structure through COOC, VOS viewer, and Cite Space
software.
</td>
<td style="text-align:left;">
The COVID-19 epidemic generated different forms of stress. From this
period, there has been a remarkable increase in the quantity of studies
on stress conducted by scholars. However, few used bibliometric analyses
to focus on overall trends in the field. This study sought to understand
the current status and trends in stress development during COVID-19, as
well as the main research drives and themes in this field. 2719
publications from the Web of Science(WOS) core repository on stress
during COVID-19 were analyzed by utilizing Co-Occurrence (COOC), VOS
viewer, and Cite Space bibliometric software. The overall features of
research on stress during COVID-19 were concluded by analyzing the
quantity of publications, keywords, countries, and institutions. The
results indicated that the United States had the largest number of
publications and collaborated closely with other countries with each
other. University of Toronto was the most prolific institution
worldwide. Visualization and analysis demonstrated that the influence of
stress during COVID-19 on the work, life, mental and spiritual
dimensions is a hot research topic. Among other things, the frequency of
each keyword in research on stress during COVID-19 increased from 2021
to 2022, and the researchers expanded their scope and study population;
the range of subjects included children, nurses, and college students,
as well as studies focusing on different types of stress, and
emphasizing the handling of stress. Our findings reveal that the heat of
stress research during COVID-19 has declined, and the main research
forces come from the United States and China. Additionally, subsequent
research should concern more on coping methods with stress, while using
more quantitative and qualitative studies in the future. Copyright ©
2024 Lu, Liu, Xu, Jiang and Wei.
</td>
</tr>
<tr>
<td style="text-align:left;">
38332736
</td>
<td style="text-align:left;">
Anti-Black discrimination in primary health care: a qualitative study
exploring internalized racism in a Canadian context.
</td>
<td style="text-align:left;">
A growing body of evidence points to persistent health inequities within
racialized minority communities, and the effects of racial
discrimination on health outcomes and health care experiences. While
much work has considered how anti-Black racism operates at the
interpersonal and institutional levels, limited attention has focused on
internalized racism and its consequences for health care. This study
explores patients’ attitudes towards anti-Black racism in a Canadian
health care system, with a particular focus on internalized racism in
primary health care. This qualitative study employed purposive maximal
variation and snowball sampling to recruit and interview self-identified
Black persons aged 18 years and older who: (1) lived in Montréal during
the COVID-19 pandemic, (2) could speak English or French, and (3) were
registered with the Québec health insurance program. Adopting a
phenomenological approach, in-depth interviews took place from October
2021 to July 2022. Following transcription, data were analyzed
thematically. Thirty-two participants were interviewed spanning an age
range from 22 years to 79 years (mean: 42 years). Fifty-nine percent of
the sample identified as women, 38% identified as men, and 3% identified
as non-binary. Diversity was also reflected in terms of immigration
experience, financial situation, and educational attainment. We
identified three major themes that describe mechanisms through which
internalized racism may manifest in health care to impact experiences:
(1) the internalization of anti-Black racism by Black providers and
patients, (2) the expression of anti-Black prejudice and discrimination
by non-Black racialized minority providers, and (3) an insensitivity
towards racial discrimination. Our study suggests that multiple levels
of racism, including internalized racism, must be addressed in efforts
to promote health and health care equity among racialized minority
groups, and particularly within Black communities.
</td>
</tr>
<tr>
<td style="text-align:left;">
38331861
</td>
<td style="text-align:left;">
Role of modifiable organisational factors in decreasing barriers to
mental healthcare: a longitudinal study of mission meaningfulness, team
relatedness and leadership trust among Canadian military personnel
deployed on Operation LASER.
</td>
<td style="text-align:left;">
The literature presents complex inter-relationships among
individual-factors and organisational-factors and barriers to seeking
mental health support after deployment. This study aims to quantify
longitudinal associations between such factors and barriers to mental
health support. A longitudinal online survey of Canadian Armed Forces
(CAF) personnel collected data at 3 months post-deployment (T1), 6
months post-deployment (T2) and 1 year post-deployment (T3). In 2020, as
part of Canada’s response to the COVID-19 pandemic, 2595 CAF personnel
deployed on Operation LASER to support civilian long-term care
facilities in Québec and Ontario. All Operation LASER personnel were
invited to participate: 1088, 582 and 497 responded at T1, T2 and T3,
respectively. Most respondents were young, male, non-commissioned
members. Barriers to mental health support were measured using 25
self-reported items and grouped into theory-based factors, including
eight factors exploring care-seeking capabilities, opportunities and
motivations; and two factors exploring moral issues. Logistic
regressions estimated the crude and adjusted associations of individual
and organisational characteristics (T1) with barriers (T2 and T3). When
adjusting for sex, military rank and mental health status, increased
meaningfulness of deployment was associated with lower probability of
endorsing barriers related to conflicts with career goals and moral
discomfort in accessing support at T2. Higher scores in trust in
leadership were associated with lower probability of endorsing four
barriers at T2, and five barriers at T3. We identified several
modifiable organisational-level characteristics that may help reduce
perceived barriers to mental health support in military and other
high-risk occupational populations. Results suggest that promoting
individuals’ sense of purpose, instilling trust in leadership and
promoting relatedness among team members may improve perceptions of
access to mental health supports in the months following a domestic
deployment or comparable occupational exposure. © Author(s) (or their
employer(s)) 2024. Re-use permitted under CC BY-NC. No commercial
re-use. See rights and permissions. Published by BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38331098
</td>
<td style="text-align:left;">
Vaccination Recommendations for Adults Receiving Biologics and Oral
Therapies for Psoriasis and Psoriatic Arthritis: Delphi Consensus from
the Medical Board of the National Psoriasis Foundation.
</td>
<td style="text-align:left;">
For psoriatic patients who need to receive non-live or live vaccines,
evidence-based recommendations are needed regarding whether to pause or
continue systemic therapies for psoriasis and/or psoriatic arthritis. To
evaluate literature regarding vaccine efficacy and safety and to
generate consensus-based recommendations for adults receiving systemic
therapies for psoriasis and/or psoriatic arthritis receiving non-live or
live vaccines. Using a modified Delphi process, 22 consensus statements
were developed by the National Psoriasis Foundation Medical Board and
COVID-19 Task Force, and infectious disease experts. Key recommendations
include continuing most oral and biologic therapies without modification
for patients receiving non-live vaccines; consider interruption of
methotrexate for non-live vaccines. For patients receiving live
vaccines, discontinue most oral and biologic medications before and
after administration of live vaccine. Specific recommendations include
discontinuing most biologic therapies, except for abatacept, for 2-3
half-lives before live vaccine administration and deferring next dose
2-4 weeks after live vaccination. Studies regarding infection rates
after vaccination are lacking. Interruption of anti-psoriatic oral and
biologic therapies is generally not necessary for patients receiving
non-live vaccines. Temporary interruption of oral and biologic therapies
before and after administration of live vaccines is recommended in most
cases. Copyright © 2024. Published by Elsevier Inc. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38330987
</td>
<td style="text-align:left;">
Glucagon and GLP-1 receptor dual agonist survodutide for obesity: a
randomised, double-blind, placebo-controlled, dose-finding phase 2
trial.
</td>
<td style="text-align:left;">
Obesity is a widespread and chronic condition that requires long-term
management; research into additional targets to improve treatment
outcomes remains a priority. This study aimed to investigate the safety,
tolerability, and efficacy of glucagon receptor-GLP-1 receptor dual
agonist survodutide (BI 456906) in obesity management. In this
randomised, double-blind, placebo-controlled, dose-finding phase 2 trial
conducted in 43 centres in 12 countries, we enrolled participants (aged
18-75 years, BMI ≥27 kg/m2, without diabetes) and randomly assigned them
by interactive response technology (1:1:1:1:1; stratified by sex) to
subcutaneous survodutide (0·6, 2·4, 3·6, or 4·8 mg) or placebo
once-weekly for 46 weeks (20 weeks dose escalation; 26 weeks dose
maintenance). The primary endpoint was the percentage change in
bodyweight from baseline to week 46. Primary analysis included the
modified intention-to-treat population (defined as all randomly assigned
patients who received at least one dose of trial medication and who had
analysable data for at least one efficacy endpoint) and was based on the
dose assigned at randomisation (planned treatment), including all data
censored for COVID-19-related discontinuations; the sensitivity analysis
was based on the actual dose received during maintenance phase (actual
treatment) and included on-treatment data. Safety analysis included all
participants who received at least one dose of study drug. The trial is
registered with ClinicalTrials.gov (NCT04667377) and EudraCT
(2020-002479-37). Between March 30, 2021, and Nov 11, 2021, we enrolled
387 participants; 386 (100%) participants were treated (0·6 mg, n=77;
2·4 mg, n=78; 3·6 mg, n=77; 4·8 mg, n=77; placebo n=77) and 233 (60·4%)
of 386 completed the 46-week treatment period (187 \[61%\] of 309
receiving survodutide; 46 \[60%\] of 77 receiving placebo). When
analysed according to planned treatment, mean (95% CI) changes in
bodyweight from baseline to week 46 were -6·2% (-8·3 to -4·1; 0·6 mg);
-12·5% (-14·5 to -10·5; 2·4 mg); -13·2% (-15·3 to -11·2; 3·6 mg); -14·9%
(-16·9 to -13·0; 4·8 mg); -2·8% (-4·9 to -0·7; placebo). Adverse events
occurred in 281 (91%) of 309 survodutide recipients and 58 (75%) of 77
placebo recipients; these were primarily gastrointestinal in 232 (75%)
of 309 survodutide recipients and 32 (42%) of 77 placebo recipients. All
tested survodutide doses were tolerated, and dose-dependently reduced
bodyweight. Boehringer Ingelheim. Copyright © 2024 Elsevier Ltd. All
rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38329905
</td>
<td style="text-align:left;">
A longitudinal study examining the associations between prenatal and
postnatal maternal distress and toddler socioemotional developmental
during the COVID-19 pandemic.
</td>
<td style="text-align:left;">
Elevated psychological distress, experienced by pregnant women and
parents, has been well-documented during the COVID-19 pandemic. Most
research focuses on the first 6-months postpartum, with single or
limited repeated measures of perinatal distress. The present
longitudinal study examined how perinatal distress, experienced over
nearly 2 years of the COVID-19 pandemic, impacted toddler socioemotional
development. A sample of 304 participants participated during pregnancy,
6-weeks, 6-months, and 15-months postpartum. Mothers reported their
depressive, anxiety, and stress symptoms, at each timepoint.
Mother-reported toddler socioemotional functioning (using the Brief
Infant-Toddler Social and Emotional Assessment) was measured at
15-months. Results of structural equation mediation models indicated
that (1) higher prenatal distress was associated with elevated
postpartum distress, from 6-weeks to 15-months postpartum; (2)
associations between prenatal distress and toddler socioemotional
problems became nonsignificant after accounting for postpartum distress;
and (3) higher prenatal distress was indirectly associated with greater
socioemotional problems, and specifically elevated externalizing
problems, through higher maternal distress at 6 weeks and 15 months
postpartum. Findings suggest that the continued experience of distress
during the postpartum period plays an important role in child
socioemotional development during the COVID-19 pandemic. © 2024 The
Authors. Infancy published by Wiley Periodicals LLC on behalf of
International Congress of Infant Studies.
</td>
</tr>
<tr>
<td style="text-align:left;">
38324970
</td>
<td style="text-align:left;">
Estimating the population effectiveness of interventions against
COVID-19 in France: A modelling study.
</td>
<td style="text-align:left;">
Non-pharmaceutical interventions (NPIs) and vaccines have been widely
used to manage the COVID-19 pandemic. However, uncertainty persists
regarding the effectiveness of these interventions due to data quality
issues, methodological challenges, and differing contextual factors.
Accurate estimation of their effects is crucial for future epidemic
preparedness. To address this, we developed a population-based
mechanistic model that includes the impact of NPIs and vaccines on
SARS-CoV-2 transmission and hospitalization rates. Our statistical
approach estimated all parameters in one step, accurately propagating
uncertainty. We fitted the model to comprehensive epidemiological data
in France from March 2020 to October 2021. With the same model, we
simulated scenarios of vaccine rollout. The first lockdown was the most
effective, reducing transmission by 84 % (95 % confidence interval (CI)
83-85). Subsequent lockdowns had diminished effectiveness (reduction of
74 % (69-77) and 11 % (9-18), respectively). A 6 pm curfew was more
effective than one at 8 pm (68 % (66-69) vs. 48 % (45-49) reduction),
while school closures reduced transmission by 15 % (12-18). In a
scenario without vaccines before November 2021, we predicted 159,000 or
168 % (95 % prediction interval (PI) 70-315) more deaths and 1,488,000
or 300 % (133-492) more hospitalizations. If a vaccine had been
available after 100 days, over 71,000 deaths (16,507-204,249) and
384,000 (88,579-1,020,386) hospitalizations could have been averted. Our
results highlight the substantial impact of NPIs, including lockdowns
and curfews, in controlling the COVID-19 pandemic. We also demonstrate
the value of the 100 days objective of the Coalition for Epidemic
Preparedness Innovations (CEPI) initiative for vaccine availability.
Copyright © 2024. Published by Elsevier B.V.
</td>
</tr>
<tr>
<td style="text-align:left;">
38323703
</td>
<td style="text-align:left;">
Characteristics of the sexual networks of gay, bisexual, and other men
who have sex with men in Montréal, Toronto, and Vancouver: implications
for the transmission and control of mpox in Canada.
</td>
<td style="text-align:left;">
The 2022-2023 global mpox outbreak disproportionately affected gay,
bisexual, and other men who have sex with men (GBM). In Canada, almost
all cases occurred among GBM and &gt;70% of them were from the country’s
three largest cities: Montréal, Toronto, and Vancouver. We examined how
the distributions of sexual partners 1) varied by city and over time
(2017-2023) and 2) were associated with mpox transmission. The Engage
Cohort Study (2017-2023) recruited GBM via respondent-driven sampling in
Montréal, Toronto, and Vancouver (n=2,449). We compared reported numbers
of sexual partners in the past 6 months across cities and three time
periods: pre-COVID-19 pandemic (2017-2019), pandemic (2020-2021), and
post-restrictions (2021-2023). We modeled the distribution of sexual
partners using Bayesian negative binomial regressions and
post-stratification, adjusting for sampling design and attrition. We
estimated mpox’s basic reproduction number (ℛ0) using a risk-stratified
compartmental model. The pre-COVID-19 pandemic distributions of sexual
partner numbers were similar across cities: participants’ mean number of
partners over the last 6 months was 10.4 (95%CrI: 9.4-11.5) in Montréal,
13.1 (11.3-15.1) in Toronto, and 10.7 (9.5-12.1) in Vancouver. Partner
numbers decreased during the pandemic in all cities. Post-restrictions,
sexual activity increased but remained below pre-pandemic levels. Based
on reported cases and post-restrictions distributions of sexual
partners, the estimated ℛ0 for mpox varied from 2.4-2.7 between cities.
The estimated mpox per-partnership transmission probability was 84%
(uncertainty ranging from 51-98%). Cumulative incidences (0.7-0.9%) were
similar across cities. GBM sexual activity after restrictions were
lifted remained below pre-pandemic levels. Comparable sexual partner
distributions may explain similarities in mpox ℛ0 and cumulative
incidence across cities. With potential for further recovery in sexual
activity, mpox vaccination and surveillance strategies should be
maintained. © The Author(s) 2024. Published by Oxford University Press
on behalf of Infectious Diseases Society of America. All rights
reserved. For permissions, please e-mail:
<journals.permissions@oup.com>.
</td>
</tr>
<tr>
<td style="text-align:left;">
38323501
</td>
<td style="text-align:left;">
Economic evaluations of treatment of depressive disorders in
adolescents: Protocol for a scoping review.
</td>
<td style="text-align:left;">
Depressive disorders in adolescents are common and impairing.
Evidence-based treatments are available; however, at a cost. In the
context of the COVID-19 pandemic, we anticipate increased demand for
treatment services for adolescents with depression. We also anticipate
that economic resources will be strained. Identifying cost-effective
strategies to optimally treat depression in adolescents is imperative.
This protocol for a scoping review aims to describe the literature with
respect to economic evaluations of treatments for depression in
adolescents. We will conduct a scoping review using established methods
and reporting guidelines. MEDLINE, Embase, PsyclNFO, Econlit, and the
International HTA Database will be searched from inception to June 13,
2023, with an update closer to time of manuscript submission, while the
NHS Economic Evaluation Database archives will be searched from
inception to December 2014. Publications that contain economic
evaluations, in the context of a clinical trial or a model-based study,
testing a treatment of depression in adolescents will be selected for
inclusion. Extracted data items will include: economic evaluation
perspectives, health outcome variables and costs used in economic
evaluations, types of analyses performed, as well as quality of
reporting and methodology. A narrative synthesis with summary tables
will be used to describe our findings. Our findings will help identify
gaps in the literature with respect to economic analyses for the
treatment of depression such that these gaps can be filled with future
research. Policy-makers, funders and administrators may also use our
findings to inform their decisions around provision of various
treatments for depression in adolescents. osf.io/5fteb (note that
information on this link will be updated upon acceptance for publication
based on reviewer comments). © 2024 John Wiley &amp; Sons Australia,
Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38322144
</td>
<td style="text-align:left;">
Prevalence and factors associated with depression and anxiety among
COVID-19 survivors in Dhaka city.
</td>
<td style="text-align:left;">
Coronavirus disease 2019 (COVID-19) is a global public health concern.
Evidence shows that depression and anxiety are common among patients
with COVID-19 after recovery. About one-third of the total COVID-19
cases in Bangladesh have been reported in Dhaka city. Therefore, the
study aimed to evaluate the prevalence of depression and anxiety among
COVID-19 survivors in Dhaka city as well as to identify the factors
associated with these mental health conditions. A cross-sectional study
was carried out among a total of 384 COVID-19 survivors aged 18 years or
older. Data collection was done through face-to-face and telephone
interviews using a semi-structured questionnaire. Patient Health
Questionnaire (PHQ-9) and Generalized Anxiety Disorder (GAD-7) scales
were used to assess depression and anxiety, respectively. Binary
logistic regression analysis was performed to identify the predictors of
depression and anxiety among patients recovered from COVID-19. The
overall prevalence of depression and anxiety was 26.0% and 23.2%,
respectively among COVID-19 survivors. The respondents who were ≥60
years were 2.62 and 3.02 times more likely to report depressive and
anxiety symptoms, respectively than those aged 18 to 39 years.
Hospitalised patients recovered from COVID-19 had a 2.18 times higher
chance of developing anxiety than their non-hospitalised counterparts.
COVID-19 recovered patients with comorbidities were at 3.35 and 2.97
times higher risk of depression and anxiety, respectively compared to
those without comorbidities. Similarly, the respondents who had already
passed a period of 15 days to 3 months after recovery showed 3.06 and
1.85 times higher odds of depression and anxiety, respectively than
those who had already passed a period of above 3 to 6 months after
recovery. The study reported a high prevalence of depression and anxiety
among COVID-19 survivors living in Dhaka city. The findings suggest the
need for appropriate interventions to reduce mental health complications
in COVID-19 survivors. Copyright © 2024 Kibria, Kabir, Rahman, Ahmed,
Amin, Rahman and Arafat.
</td>
</tr>
<tr>
<td style="text-align:left;">
38321333
</td>
<td style="text-align:left;">
Impact of vortioxetine on psychosocial functioning moderated by symptoms
of fatigue in post-COVID-19 condition: a secondary analysis.
</td>
<td style="text-align:left;">
Fatigue is a prominent symptom in post-COVID condition (PCC) sequelae,
termed “long COVID.” Herein, we aim to ascertain the effect of fatigue
on psychosocial function in persons living with PCC. This post hoc
analysis evaluated the effects of vortioxetine on measures of fatigue as
assessed by the Fatigue Severity Scale (FSS) in psychosocial function as
measured by the Sheehan Disability Scale (SDS) in persons with PCC. We
also evaluated the change in FSS on psychosocial functioning as measured
by the Sheehan Disability Scale (SDS). This post hoc analysis obtained
data from a recently published placebo-controlled study evaluating
vortioxetine’s effect on objective cognitive functions in persons living
with PCC. One hundred forty-four participants meeting World Health
Organization (WHO) criteria for PCC were included in this analysis. At
the end of 8 weeks of vortioxetine treatment, significant improvement of
all domains was observed for psychosocial functioning. There was a
significant between-group difference at treatment endpoint in the
family, social, and work SDS subcategories (p &lt; 0.001). There was a
statistically significant interaction effect between the treatment
condition time point and FSS effect on the SDS social (χ2 = 10.640, p =
0.014) and work (χ2 = 9.342, p = 0.025) categories but a statistically
insignificant effect on the family categories ((χ2 = 5.201, p = 0.158)).
This post hoc analysis suggests that vortioxetine treatment
significantly improves psychosocial function in persons with PCC. Our
results also indicate that the improvement in psychosocial function was
significantly mediated by improvement in measures of fatigue. Our
results provide empirical support for recommendations to identify
therapeutics for fatigue in persons living with PCC with a broader aim
to improve psychosocial function in this common and severely impaired
population. © 2024. Fondazione Società Italiana di Neurologia.
</td>
</tr>
<tr>
<td style="text-align:left;">
38320044
</td>
<td style="text-align:left;">
Detection of Covid-19 Outbreaks Using Built Environment Testing for
SARS-CoV-2.
</td>
<td style="text-align:left;">
Built Environment Testing for SARS-CoV-2Wastewater testing has proven to
be a valuable tool for forecasting Covid-19 outbreaks. Fralick et
al. now report that swabbing of surfaces (i.e., floors) for SARS-CoV-2
may provide a similar benefit for predicting outbreaks in long-term care
homes.
</td>
</tr>
<tr>
<td style="text-align:left;">
38318236
</td>
<td style="text-align:left;">
Long-term safety, tolerability, and efficacy of efgartigimod (ADAPT+):
interim results from a phase 3 open-label extension study in
participants with generalized myasthenia gravis.
</td>
<td style="text-align:left;">
ADAPT+ assessed the long-term safety, tolerability, and efficacy of
efgartigimod in adult participants with generalized myasthenia gravis
(gMG). ADAPT+ was an open-label, single-arm, multicenter, up to 3-year
extension of the pivotal phase 3 ADAPT study. Efgartigimod was
administered in treatment cycles of 4 intravenous infusions (one 10
mg/kg infusion per week). Initiation of subsequent treatment cycles was
individualized based on clinical evaluation. Safety endpoints included
incidence and severity of adverse events. Efficacy endpoints assessed
disease severity using Myasthenia Gravis-Activities of Daily Living
(MG-ADL) and Quantitative Myasthenia Gravis (QMG) scores. As of January
2022, 151 participants had rolled over to ADAPT+ and 145 had received ≥1
dose of efgartigimod, of whom, 111 (76.6%) were AChR-Ab+ and 34 (23.4%)
were AChR-Ab-. Mean study duration (treatment plus follow-up) was 548
days, and participants received up to 17 treatment cycles, corresponding
to 217.6 participant-years of exposure. In the overall population, 123
(84.8%) participants reported ≥1 treatment-emergent adverse event; most
frequent were headache (36 \[24.8%\]), COVID-19 (22 \[15.2%\]), and
nasopharyngitis (20 \[13.8%\]). Clinically meaningful improvement (CMI)
in mean MG-ADL and QMG scores was seen as early as 1 week following the
first infusion across multiple cycles in AChR-Ab+ and AChR-Ab-
participants. Maximal MG-ADL and QMG improvements aligned with onset and
magnitude of total IgG and AChR-Ab reductions. For AChR-Ab+ participants
at any time point in each of the first 10 treatment cycles, more than
90% had a maximum reduction of ≥2 points (CMI) in MG-ADL total score;
across the 7 cycles in which QMG was measured, 69.4% to 91.3% of
participants demonstrated a maximum reduction of ≥3 points (CMI) in QMG
total score. Many participants demonstrated improvements well beyond CMI
thresholds. In AChR-Ab+ participants with ≥1 year of combined follow-up
between ADAPT and ADAPT+, mean number of annualized cycles was 4.7 per
year (median \[range\] 5.0 \[0.5-7.6\]). Results of ADAPT+ corroborate
the substantial clinical improvements seen with efgartigimod in ADAPT
and support its long-term safety, tolerability, and efficacy, as well as
an individualized dosing regimen for treatment of gMG.
<https://classic.clinicaltrials.gov/ct2/show/NCT03770403>, NCT03770403.
Copyright © 2024 Howard, Bril, Vu, Karam, Peric, De Bleecker, Murai,
Meisel, Beydoun, Pasnoor, Guglietta, Van Hoorick, Steeland, T’joen,
Utsugisawa, Verschuuren, Mantegazza, the ADAPT+ Study Group.
</td>
</tr>
<tr>
<td style="text-align:left;">
38317176
</td>
<td style="text-align:left;">
Changes in breakfast and water consumption among adolescents in Canada:
examining the impact of COVID-19 in worsening inequity.
</td>
<td style="text-align:left;">
To assess whether changes in breakfast and water consumption during the
first full school year after the emergence of the COVID-19 pandemic
varied based on sex/gender, race/ethnicity, and socioeconomic status
among Canadian adolescents. Prospective annual survey data collected
pre- (October 2019-March 2020) and post-COVID-19 onset (November
2020-June 2021) the Cannabis, Obesity, Mental health, Physical activity,
Alcohol, Smoking, and Sedentary behaviour (COMPASS) study. The sample
consisted of 8,128 students; mean (SD) age = 14.2 (1.3) years from a
convenience sample of 41 Canadian secondary schools. At both timepoints
self-reported breakfast and water consumption were dichotomized as daily
or not. Multivariable logistic generalized estimating equations with
school clustering were used to estimate differences in
maintenance/adoption of daily consumption post-COVID-19 based on
demographic factors, while controlling for pre-COVID-19 behaviour.
Adjusted odds ratios (AOR) with 95% confidence intervals are reported.
Females (AOR = 0.71 \[0.63, 0.79\]) and lower socioeconomic status
individuals (AORLowest:Highest=0.41 \[0.16, 1.00\]) were less likely to
maintain/adopt daily breakfast consumption than male and higher
socioeconomic status peers in the 2020-2021 school year. Black
identifying individuals were less likely than all other racial/ethnic
identities to maintain/adopt plain water consumption every day of the
week (AOR = 0.33 \[0.15, 0.75\], p &lt; 0.001). No significant
interaction effects were detected. Results support the hypothesis that
changes in nutritional behaviours were not equal across demographic
groups. Female, lower socioeconomic status, and Black adolescents
reported greater declines in healthy nutritional behaviours. Public
health interventions to improve adherence to daily breakfast and water
consumption should target these segments of the population. Not a trial.
© 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38316515
</td>
<td style="text-align:left;">
Variation in occupational exposure risk for COVID-19 workers’
compensation claims across pandemic waves in Ontario.
</td>
<td style="text-align:left;">
To understand rates of work-related COVID-19 (WR-C19) infection by
occupational exposures across waves of the COVID-19 pandemic in Ontario,
Canada. We combined workers’ compensation claims for COVID-19 with data
from Statistics Canada’s Labour Force Survey, to estimate rates of
WR-C19 among workers spending the majority of their working time at the
workplace between 1 April 2020 and 30 April 2022. Occupational
exposures, imputed using a job exposure matrix, were whether the
occupation was public facing, proximity to others at work, location of
work and a summary measure of low, medium and high occupational
exposure. Negative binomial regression models examined the relationship
between occupational exposures and risk of WR-C19, adjusting for
covariates. Trends in rates of WR-C19 differed from overall COVID-19
cases among the working-aged population. All occupational exposures were
associated with increased risk of WR-C19, with risk ratios for medium
and high summary exposures being 1.30 (95% CI 1.09 to 1.55) and 2.46
(95% CI 2.10 to 2.88), respectively, in fully adjusted models. The
magnitude of associations between occupational exposures and risk of
WR-C19 differed across waves of the pandemic, being weakest for most
exposures in period March 2021 to June 2021, and highest at the start of
the pandemic and during the Omicron wave (December 2021 to April 2022).
Occupational exposures were consistently associated with increased risk
of WR-C19, although the magnitude of this relationship differed across
pandemic waves in Ontario. Preparation for future pandemics should
consider more accurate reporting of WR-C19 infections and the potential
dynamic nature of occupational exposures. © Author(s) (or their
employer(s)) 2024. No commercial re-use. See rights and permissions.
Published by BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38315731
</td>
<td style="text-align:left;">
The flashbulb-like nature of memory for the first COVID-19 case and the
impact of the emergency. A cross-national survey.
</td>
<td style="text-align:left;">
Flashbulb memories (FBMs) refer to vivid and long-lasting
autobiographical memories for the circumstances in which people learned
of a shocking and consequential public event. A cross-national study
across eleven countries aimed to investigate FBM formation following the
first COVID-19 case news in each country and test the effect of
pandemic-related variables on FBM. Participants had detailed memories of
the date and others present when they heard the news, and had partially
detailed memories of the place, activity, and news source. China had the
highest FBM specificity. All countries considered the COVID-19 emergency
as highly significant at both the individual and global level. The
Classification and Regression Tree Analysis revealed that FBM
specificity might be influenced by participants’ age, subjective
severity (assessment of COVID-19 impact in each country and relative to
others), residing in an area with stringent COVID-19 protection
measures, and expecting the pandemic effects. Hierarchical regression
models demonstrated that age and subjective severity negatively
predicted FBM specificity, whereas sex, pandemic impact expectedness,
and rehearsal showed positive associations in the total sample.
Subjective severity negatively affected FBM specificity in Turkey,
whereas pandemic impact expectedness positively influenced FBM
specificity in China and negatively in Denmark.
</td>
</tr>
<tr>
<td style="text-align:left;">
38315512
</td>
<td style="text-align:left;">
Physical Activity, Heart Rate Variability, and Ventricular Arrhythmia
During the COVID-19 Lockdown: Retrospective Cohort Study.
</td>
<td style="text-align:left;">
Ventricular arrhythmias (VAs) increase with stress and national
disasters. Prior research has reported that VA did not increase during
the onset of the COVID-19 lockdown in March 2020, and the mechanism for
this is unknown. This study aimed to report the presence of VA and
changes in 2 factors associated with VA (physical activity and heart
rate variability \[HRV\]) at the onset of COVID-19 lockdown measures in
Ontario, Canada. Patients with implantable cardioverter defibrillator
(ICD) followed at a regional cardiac center in Ontario, Canada with data
available for both HRV and physical activity between March 1 and 31,
2020, were included. HRV, physical activity, and the presence of VA were
determined during the pre- (March 1-10, 2020) and immediate postlockdown
(March 11-31) period. When available, these data were determined for the
same period in 2019. In total, 68 patients had complete data for 2020,
and 40 patients had complete data for 2019. Three (7.5%) patients had VA
in March 2019, whereas none had VA in March 2020 (P=.048). Physical
activity was reduced during the postlockdown period (mean 2.3, SD 1.6
hours vs mean 2.1, SD 1.6 hours; P=.003). HRV was unchanged during the
pre- and postlockdown period (mean 91, SD 30 ms vs mean 92, SD 28 ms;
P=.84). VA was infrequent during the COVID-19 pandemic. A reduction in
physical activity with lockdown maneuvers may explain this observation.
©Sikander Z Texiwala, Russell J de Souza, Suzette Turner, Sheldon M
Singh. Originally published in JMIR Cardio (<https://cardio.jmir.org>),
05.02.2024.
</td>
</tr>
<tr>
<td style="text-align:left;">
38315149
</td>
<td style="text-align:left;">
Investigating adaptive sport participation for adults aged 50 years or
older with spinal cord injury or disease: A descriptive cross-sectional
survey.
</td>
<td style="text-align:left;">
Spinal cord injury or disease (SCI/D) can lead to health challenges that
are exacerbated with aging. Adaptive sport is understood to provide
health benefits for the SCI/D population. Prior literature investigating
adaptive sport in this population pertains to adults with SCI/D who are
&lt;50 years of age. However, most Canadians with SCI/D are &gt;50 years
of age. This study aimed to: (1) Compare demographics of those who do
and do not participate in adaptive sport; (2) Describe the
characteristics of adaptive sport that adults aged ≥50 years with SCI/D
participate in; and (3) Identify barriers and facilitators to adaptive
sport participation in this age group. This descriptive, cross-sectional
survey was carried out using an online survey. Analytical statistics
were used to address objective one, while descriptive statistics were
employed for objectives two and three. Responses from 72 adults aged ≥50
years, residing in Canada, living with a SCI/D for &gt;6 months were
included in the analysis. Findings revealed that adaptive sport
participants aged ≥50 years with SCI/D were more likely to identify as
men, be younger individuals (50-59 years), and report greater
satisfaction with physical health (P &lt; 0.05). Adaptive sport
participants most commonly played individual sports at the recreational
level. Common barriers pertained to physical capacity, travel, and
COVID-19; common facilitators included social support, desire to improve
health, and having friends/peers who also participate. Future research
should investigate strategies to enhance facilitators and mitigate
barriers to adaptive sport participation in order to improve access.
</td>
</tr>
<tr>
<td style="text-align:left;">
38313681
</td>
<td style="text-align:left;">
What might working from home mean for the geography of work and
commuting in the Greater Golden Horseshoe, Canada?
</td>
<td style="text-align:left;">
The Covid-19 pandemic has highlighted the precarity of urban society,
illustrating both opportunities and challenges. Teleworking rates
increased dramatically during the pandemic and may be sustained over the
long term. For transportation planners, these changes belie the broader
questions of how the geography of work and commuting will change based
on pandemic-induced shifts in teleworking and what this will mean for
society and policymaking. This study focuses on these questions by using
survey data (n = 2580) gathered in the autumn of 2021 to explore the
geography of current and prospective telework. The study focuses on the
Greater Golden Horseshoe, the mega-region in Southern Ontario,
representing a fifth of Canadians. Survey data document telework
practices before and during the pandemic, including prospective future
telework practices. Inferential models are used to develop
working-from-home scenarios which are allocated spatially based on
respondents’ locations of work and residence. Findings indicate that
telework appears to be poised to increase most relative to pre-pandemic
levels around downtown Toronto based on locations of work, but increases
in teleworking are more dispersed based on employees’ locations of
residence. Contrary to expectations by many, teleworking is not
significantly linked to home-work disconnect - suggesting that telework
is poised to weaken the commute-housing trade-off embedded in bid rent
theory. Together, these results portend a poor outlook for downtown
urban agglomeration economies but also more nuanced impacts than simply
inducing sprawl. © Urban Studies Journal Limited 2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38309831
</td>
<td style="text-align:left;">
SARS-CoV-2 antibodies and their neutralizing capacity against live virus
in human milk after COVID-19 infection and vaccination: prospective
cohort studies.
</td>
<td style="text-align:left;">
There is limited understanding of the impact of coronavirus disease 2019
(COVID-19) infection and vaccination type and interval on severe acute
respiratory syndrome coronavirus 2 (SARS-CoV-2) human milk antibodies
and their neutralizing capacity. These cohort studies aimed to determine
the presence of antibodies and live virus neutralizing capacity in milk
from females infected with COVID-19, unexposed milk bank donors, and
vaccinated females and examine impacts of vaccine interval and type.
Milk was collected from participants infected with COVID-19 during
pregnancy or lactation (Cohort-1) and milk bank donors (Cohort-2) from
March 2020-July 2021 at 3 sequential 4-wk intervals and COVID-19
vaccinated participants with varying dose intervals (Cohort-3)
(January-October 2021). Cohort-1 and Cohort-3 were recruited from Sinai
Health (patients) and through social media. Cohort-2 included Ontario
Milk Bank donors. Milk was examined for SARS-CoV-2 antibodies and live
virus neutralization. Of females with COVID-19, 53% (Cohort-1, n = 55)
had anti-SARS-CoV-2 IgA antibodies in ≥1 milk sample. IgA+ samples (40%)
were more likely neutralizing than IgA- samples (odds ratio \[OR\]:
2.18; 95% confidence interval \[CI\]: 1.03, 4.60; P = 0.04); however,
25% of IgA- samples were neutralizing. Both IgA positivity and
neutralization decreased ∼6 mo after symptom onset (0-100 compared with
201+ d: IgA OR: 14.30; 95% CI: 1.08, 189.89; P = 0.04; neutralizing OR:
4.30; 95% CI: 1.55, 11.89; P = 0.005). Among milk bank donors (Cohort-2,
n = 373), 4.3% had IgA antibodies; 23% of IgA+ samples were
neutralizing. Vaccination (Cohort-3, n = 60) with mRNA-1273 and shorter
vaccine intervals (3 to &lt;6 wk) resulted in higher IgA and IgG than
BNT162b2 (P &lt; 0.04) and longer intervals (6 to &lt;16 wk) (P≤0.02),
respectively. Neutralizing capacity increased postvaccination (P = 0.04)
but was not associated with antibody positivity. SARS-CoV-2 infection
and vaccination (type and interval) impacted milk antibodies; however,
antibody presence did not consistently predict live virus
neutralization. Although human milk is unequivocally the best way to
nourish infants, guidance on protection to infants following maternal
infection/vaccination may require more nuanced messaging. This study was
registered at clinicaltrials.gov as NCT04453969 and NCT04453982.
Copyright © 2023 American Society for Nutrition. Published by Elsevier
Inc. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38309479
</td>
<td style="text-align:left;">
Cannabis-involvement in emergency department visits for self-harm
following medical and non-medical cannabis legalization.
</td>
<td style="text-align:left;">
Cannabis use may increase the risk of self-harm, but whether
legalization of cannabis is associated with changes in self-harm is
unknown. We examined changes in cannabis-involvement in emergency
department (ED) visits for self-harm after the liberalization of medical
and legalization of non-medical cannabis in Canada. This repeated
cross-sectional study used health administrative data to identify all ED
visits for self-harm in individuals aged ten and older between January
2010 and December 2021. We identified self-harm ED visits with a
co-diagnosis of cannabis (main exposure) or alcohol (control condition)
and examined changes in rates of visits over four distinct policy
periods (pre-legalization, medical liberalization, non-medical
legalization with restrictions, and non-medical
commercialization/COVID-19) using Poisson models. The study included
158,912 individuals with one or more self-harm ED visits, of which 7810
(4.9 %) individuals had a co-diagnosis of cannabis use and 24,761 (15.6
%) had a co-diagnosis of alcohol use. Between 2010 and 2021, the annual
rate of ED visits for self-harm injuries involving cannabis per 100,000
individuals increased by 90.1 % (3.6 in 2010 to 6.9 in 2021 per 100,000
individuals), while the annual rate of self-harm injuries involving
alcohol decreased by 17.3 % (168.1 in 2010 to 153.1 in 2021 per 100,000
individuals). The entire increase in visits relative to pre-legalization
occurred after medical liberalization (seasonally adjusted Risk Ratio
\[asRR\] 1.71 95 % CI 1.09-1.15) with no further increases during the
legalization with restrictions (asRR 1.77 95%CI 1.62-1.93) or
commercialization/COVID-19 periods (asRR 1.63 95%CI 1.50-176).
Cannabis-involvement in self-harm ED visits almost doubled over 12 years
and may have accelerated after medical cannabis liberalization. While
the results cannot determine whether cannabis is increasingly causing
self-harm ED visits or whether cannabis is increasingly being used by
individuals at high risk of self-harm, greater detection for cannabis
use in this population and intervention may be indicated. Copyright ©
2024 The Authors. Published by Elsevier B.V. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38309379
</td>
<td style="text-align:left;">
Evaluating a novel accelerated free-breathing late gadolinium
enhancement imaging sequence for assessment of myocardial injury.
</td>
<td style="text-align:left;">
Cardiac magnetic resonance imaging (MRI), including late gadolinium
enhancement (LGE), plays an important role in the diagnosis and
prognostication of ischemic and non-ischemic myocardial injury.
Conventional LGE sequences require patients to perform multiple
breath-holds and require long acquisition times. In this study, we
compare image quality and assessment of myocardial LGE using an
accelerated free-breathing sequence to the conventional standard-of-care
sequence. In this prospective cohort study, a total of 41 patients post
Coronavirus 2019 (COVID-19) infection were included. Studies were
performed on a 1.5 Tesla scanner with LGE imaging acquired using a
conventional inversion recovery rapid gradient echo (conventional LGE)
sequence followed by the novel accelerated free-breathing (FB-LGE)
sequence. Image quality was visually scored (ordinal scale from 1 to 5)
and compared between conventional and free-breathing sequences using the
Wilcoxon rank sum test. Presence of per-segment LGE was identified
according to the American Heart Association 16-segment myocardial model
and compared across both conventional LGE and FB-LGE sequences using a
two-sided chi-square test. The perpatient LGE extent was also evaluated
using both sequences and compared using the Wilcoxon rank sum test.
Interobserver variability in detection of per-segment LGE and
per-patient LGE extent was evaluated using Cohen’s kappa statistic and
interclass correlation (ICC), respectively. The mean acquisition time
for the FB-LGE sequence was 17 s compared to 413 s for the conventional
LGE sequence (P &lt; 0.001). Assessment of image quality was similar
between both sequences (P = 0.19). There were no statistically
significant differences in LGE assessed using the FB-LGE versus
conventional LGE on a per-segment (P = 0.42) and per-patient (P = 0.06)
basis. Interobserver variability in LGE assessment for FB-LGE was good
for per-segment (= 0.71) and per-patient extent (ICC = 0.92) analyses.
The accelerated FB-LGE sequence performed comparably to the conventional
standard-of-care LGE sequence in a cohort of patients post COVID-19
infection in a fraction of the time and without the need for
breath-holding. Such a sequence could impact clinical practice by
increasing cardiac MRI throughput and accessibility for frail or acutely
ill patients unable to perform breath-holding. Copyright © 2024 Elsevier
Inc. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38303635
</td>
<td style="text-align:left;">
Sexual and Reproductive Health Outcomes Among Adolescent Females During
the COVID-19 Pandemic.
</td>
<td style="text-align:left;">
Coronavirus disease 2019 (COVID-19) posed a significant threat to
adolescents’ sexual and reproductive health. In this study, we examined
population-level pregnancy and sexual health-related care utilization
among adolescent females in Ontario, Canada during the pandemic and
evaluated relationships between these outcomes and key sociodemographic
characteristics. This was a population-based, repeated cross-sectional
study of &gt;630 000 female adolescents (12-19 years) during the
prepandemic (January 1, 2018-February 29, 2020) and COVID-19 pandemic
(March 1, 2020-December 31, 2022) periods. Primary outcome was
pregnancy; secondary outcomes were contraceptive management visits,
contraception prescription uptake, and sexually transmitted infection
(STI) management visits. Poisson models with generalized estimating
equations for clustered count data were used to model pre-COVID-19
trends and forecast expected rates during the COVID-19 period. Absolute
rate differences between observed and expected outcome rates for each
pandemic month were calculated overall and by urbanicity, neighborhood
income, immigration status, and region. During the pandemic,
lower-than-expected population-level rates of adolescent pregnancy (rate
ratio 0.87; 95% confidence interval \[CI\]:0.85-0.88), and encounters
for contraceptive (rate ratio 0.82; 95% CI:0.77-0.88) and STI management
(rate ratio 0.52; 95% CI:0.51-0.53) were observed. Encounter rates did
not return to pre-pandemic rates by study period end, despite health
system reopening. Pregnancy rates among adolescent subpopulations with
the highest pre-pandemic pregnancy rates changed least during the
pandemic. Population-level rates of adolescent pregnancy and sexual
health-related care utilization were lower than expected during the
COVID-19 pandemic, and below-expected care utilization rates persist.
Pregnancy rates among more structurally vulnerable adolescents
demonstrated less decline, suggesting exacerbation of preexisting
inequities. Copyright © 2024 by the American Academy of Pediatrics.
</td>
</tr>
<tr>
<td style="text-align:left;">
38302878
</td>
<td style="text-align:left;">
Enhancing detection of SARS-CoV-2 re-infections using longitudinal
sero-monitoring: demonstration of a methodology in a cohort of people
experiencing homelessness in Toronto, Canada.
</td>
<td style="text-align:left;">
Accurate estimation of SARS-CoV-2 re-infection is crucial to
understanding the connection between infection burden and adverse
outcomes. However, relying solely on PCR testing results in
underreporting. We present a novel approach that includes longitudinal
serologic data, and compared it against testing alone among people
experiencing homelessness. We recruited 736 individuals experiencing
homelessness in Toronto, Canada, between June and September 2021.
Participants completed surveys and provided saliva and blood serology
samples every three months over 12 months of follow-up. Re-infections
were defined as: positive PCR or rapid antigen test (RAT) results &gt;
90 days after initial infection; new serologic evidence of infection
among individuals with previous infection who sero-reverted; or
increases in anti-nucleocapsid in seropositive individuals whose levels
had begun to decrease. Among 381 participants at risk, we detected 37
re-infections through PCR/RAT and 98 re-infections through longitudinal
serology. The comprehensive method identified 37.4 re-infection events
per 100 person-years, more than four-fold more than the rate detected
through PCR/RAT alone (9.0 events/100 person-years). Almost all
test-confirmed re-infections (85%) were also detectable by longitudinal
serology. Longitudinal serology significantly enhances the detection of
SARS-CoV-2 re-infections. Our findings underscore the importance and
value of combining data sources for effective research and public health
surveillance. © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38301371
</td>
<td style="text-align:left;">
Diagnostic performance of deep learning models versus radiologists in
COVID-19 pneumonia: A systematic review and meta-analysis.
</td>
<td style="text-align:left;">
Although several studies have compared the performance of deep learning
(DL) models and radiologists for the diagnosis of COVID-19 pneumonia on
CT of the chest, these results have not been collectively evaluated. We
performed a meta-analysis of original articles comparing the performance
of DL models versus radiologists in detecting COVID-19 pneumonia. A
systematic search was conducted on the three main medical literature
databases, Scopus, Web of Science, and PubMed, for articles published as
of February 1st, 2023. We included original scientific articles that
compared DL models trained to detect COVID-19 pneumonia on CT to
radiologists. Meta-analysis was performed to determine DL versus
radiologist performance in terms of model sensitivity and specificity,
taking into account inter and intra-study heterogeneity. Twenty-two
articles met the inclusion criteria. Based on the meta-analytic
calculations, DL models had significantly higher pooled sensitivity
(0.933 vs. 0.829, p &lt; 0.001) compared to radiologists with similar
pooled specificity (0.905 vs. 0.897, p = 0.746). In the differentiation
of COVID-19 versus community-acquired pneumonia, the DL models had
significantly higher sensitivity compared to radiologists (0.915
vs. 0.836, p = 0.001). DL models have high performance for screening of
COVID-19 pneumonia on chest CT, offering the possibility of these models
for augmenting radiologists in clinical practice. Copyright © 2024 The
Author(s). Published by Elsevier Inc. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38300964
</td>
<td style="text-align:left;">
Physical activity and unexpected weight change in Ontario children and
youth during the COVID-19 pandemic: A cross-sectional analysis of the
Ontario Parent Survey 2.
</td>
<td style="text-align:left;">
The objective of this study was to investigate the association between
children’s parent-reported physical activity levels and weight changes
during the COVID-19 pandemic among children and youth in Ontario Canada.
A cross-sectional online survey was conducted in parents of children
5-17 years living in Ontario from May to July 2021. Parents recalled
their child’s physical activity and weight change during the year prior
to their completion of the survey. Odds ratios (OR) and 95% confidence
intervals (CI) were estimated using multinomial logistic regression for
the association between physical activity and weight gain or loss,
adjusted for child age and gender, parent ethnicity, current housing
type, method of school delivery, and financial stability. Overall, 86.8%
of children did not obtain 60 minutes of moderate-to-vigorous physical
activity per day and 75.4% of parents were somewhat or very concerned
about their child’s physical activity levels. For all physical activity
exposures (outdoor play, light physical activity, and
moderate-to-vigorous physical activity), lower physical activity was
consistently associated with increased odds of weight gain or loss. For
example, the adjusted OR for the association between 0-1 days of
moderate-to-vigorous physical activity versus 6-7 days and child weight
gain was 5.81 (95% CI 4.47, 7.56). Parent concern about their child’s
physical activity was also strongly associated with child weight gain
(OR 7.29; 95% CI 5.94, 8.94). No differences were observed between boys
and girls. This study concludes that a high proportion of children in
Ontario had low physical activity levels during the COVID-19 pandemic
and that low physical activity was strongly associated with parent
reports of both weight gain and loss among children. Copyright: © 2024
McQuillan et al. This is an open access article distributed under the
terms of the Creative Commons Attribution License, which permits
unrestricted use, distribution, and reproduction in any medium, provided
the original author and source are credited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38296543
</td>
<td style="text-align:left;">
The Efficacy and Safety of Nafamostat Mesylate in the Treatment of
COVID-19: A Meta-Analysis.
</td>
<td style="text-align:left;">
Nafamostat mesylate, a synthetic serine protease inhibitor, has
demonstrated early antiviral activity against SARS-CoV-2 and
anticoagulant properties that may be beneficial in COVID-19. We
conducted a meta-analysis evaluating the efficacy and safety of
nafamostat mesylate for COVID-19 treatment. PubMed, Embase, Cochrane
Library, Scopus, Web of Science, medRxiv and bioRxiv were searched up to
July 2023 for studies comparing outcomes between nafamostat mesylate
treatment and no nafamostat mesylate treatment in COVID-19 patients.
Mortality, disease progression and adverse events were analyzed. Six
studies involving 16,195 patients were included. Meta-analysis revealed
no significant difference in mortality (OR=0.88, 95%CI: 0.20-3.75,
P=0.86) or disease progression (OR=2.76, 95%CI: 0.31-24.68, P=0.36)
between groups. However, nafamostat mesylate was associated with
increased hyperkalemia risk (OR=7.15, 95%CI: 2.66 to 19.24,
P&lt;0.0001). Nafamostat mesylate does not improve mortality or
morbidity in hospitalized COVID-19 patients compared to no nafamostat
mesylate treatment. The significant hyperkalemia risk is a serious
concern requiring monitoring and preventative measures. Further research
is needed in different COVID-19 populations.
</td>
</tr>
<tr>
<td style="text-align:left;">
38295675
</td>
<td style="text-align:left;">
How did European countries set health priorities in response to the
COVID-19 threat? A comparative document analysis of 24 pandemic
preparedness plans across the EURO region.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has forced governments across the world to
consider how to prioritise the allocation of scarce resources. There are
many tools and frameworks that have been designed to assist with the
challenges of priority setting in health care. The purpose of this study
was to examine the extent to which formal priority setting was evident
in the pandemic plans produced by countries in the World Health
Organisation’s EURO region, during the first wave of the COVID-19
pandemic. This compliments analysis of similar plans produced in other
regions of the world. Twenty four pandemic preparedness plans were
obtained that had been published between March and September 2020. For
data extraction, we applied a framework for identifying and assessing
the elements of good priority setting to each plan, before conducting
comparative analysis across the sample. Our findings suggest that while
some pre-requisites for effective priority setting were present in many
cases - including political commitment and a recognition of the need for
allocation decisions - many other hallmarks were less evident, such as
explicit ethical criteria, decision making frameworks, and engagement
processes. This study provides a unique insight into the role of
priority setting in the European response to the onset of the COVID-19
pandemic. Copyright © 2024. Published by Elsevier B.V.
</td>
</tr>
<tr>
<td style="text-align:left;">
38292817
</td>
<td style="text-align:left;">
Humoral Response Following 3 Doses of mRNA COVID-19 Vaccines in Patients
With Non-Dialysis-Dependent CKD: An Observational Study.
</td>
<td style="text-align:left;">
Chronic kidney disease (CKD) is associated with a lower serologic
response to vaccination compared to the general population. There is
limited information regarding the serologic response to coronavirus
disease 2019 (COVID-19) vaccination in the non-dialysis-dependent CKD
(NDD-CKD) population, particularly after the third dose and whether this
response varies by estimated glomerular filtration rate (eGFR). The
NDD-CKD (G1-G5) patients who received 3 doses of mRNA COVID-19 vaccines
were recruited from renal clinics within British Columbia and Ontario,
Canada. Between August 27, 2021, and November 30, 2022, blood samples
were collected serially for serological testing every 3 months within a
9-month follow-up period. The severe acute respiratory syndrome
coronavirus 2 (SARS-CoV-2) anti-spike, anti-receptor binding domain
(RBD), and anti-nucleocapsid protein (NP) levels were determined by
enzyme-linked immunosorbent assay (ELISA). Among 285 NDD-CKD patients,
the median age was 67 (interquartile range \[IQR\], 52-77) years, 58%
were men, 48% received BNT162b2 as their third dose, 22% were on
immunosuppressive treatment, and COVID-19 infection by anti-NP
seropositivity was observed in 37 of 285 (13%) patients. Following the
third dose, anti-spike and anti-RBD levels peaked at 2 months, with
geometric mean levels at 1131 and 1672 binding antibody units per
milliliter (BAU/mL), respectively, and seropositivity rates above 93%
and 85%, respectively, over the 9-month follow-up period. There was no
association between eGFR or urine albumin-creatinine ratio (ACR) with
mounting a robust antibody response or in antibody levels over time. The
NDD-CKD patients on immunosuppressive treatment were less likely to
mount a robust anti-spike response in univariable (odds ratio \[OR\]
0.43, 95% confidence interval \[CI\]: 0.20, 0.93) and multivariable (OR
0.52, 95% CI: 0.25, 1.10) analyses. An interaction between age,
immunoglobulin G (IgG) antibody levels, and time was observed in both
unadjusted (anti-spike: P = .005; anti-RBD: P = .03) and adjusted
(anti-spike: P = .004; anti-RBD: P = .03) models, with older individuals
having a more pronounced decline in antibody levels over time. Most
NDD-CKD patients were seropositive for anti-spike and anti-RBD after 3
doses of mRNA COVID-19 vaccines and we did not observe any differences
in the antibody response by eGFR. © The Author(s) 2024.
</td>
</tr>
<tr>
<td style="text-align:left;">
38291617
</td>
<td style="text-align:left;">
Promoting Self-Care in Palliative Care: Through the Wisdom of My
Grandmother.
</td>
<td style="text-align:left;">
In the post COVID-19 pandemic period, targeted efforts are needed more
than ever to improve frontline nurses’ well-being. In the field of
palliative care, there is recognition of the importance of self-care,
but the concept itself remains nebulous, and proactive implementation of
self-care is lacking. Reflective writing has been noted to have positive
impacts on health care providers’ well-being. This piece brings to light
the author’s interest and work in reflective writing, sharing a personal
account that provides a source of happiness and an opportunity to better
understand her palliative care practice. Beyond the individual level,
organizations are also encouraged to invest in their nurses’ overall
well-being.
</td>
</tr>
<tr>
<td style="text-align:left;">
38291585
</td>
<td style="text-align:left;">
COVID-19 Reinfection Has Better Outcomes Than the First Infection in
Solid Organ Transplant Recipients.
</td>
<td style="text-align:left;">
Solid organ transplant recipients face an increased risk of severe
coronavirus disease 2019 (COVID-19) and are vulnerable to repeat severe
acute respiratory syndrome coronavirus 2 (SARS-CoV-2) infections. In
nonimmunocompromised individuals, SARS-CoV-2 reinfections are milder
likely because of cross-protective immunity. We sought to determine
whether SARS-CoV-2 reinfection exhibits milder manifestations than
primary infection in transplant recipients. Using a large, prospective
cohort of adult transplant patients with COVID-19, we identified
patients with SARS-CoV-2 reinfections. We performed a 1:1 nearest
neighbor propensity score matching to control potential confounders,
including the COVID-19 variant. We compared outcomes including oxygen
requirement, hospitalization, and intensive care unit admission within
30 d after diagnosis between patients with reinfection and those with
the first episode of COVID-19. Between 2020 and 2023, 103 reinfections
were identified in a cohort of 1869 transplant recipients infected with
SARS-CoV-2 (incidence of 2.7% per year). These included 50 kidney
(48.5%), 27 lung (26.2%), 7 heart (6.8%), 6 liver (5.8%), and 13
multiorgan (12.6%) transplants. The median age was 54.5 y (interquartile
range \[IQR\], 40.5-65.5) and the median time from transplant to first
infection was 6.6 y (IQR, 2.8-11.2). The time between the primary
COVID-19 and reinfection was 326 d (IQR, 226-434). Three doses or more
of SARS-CoV-2 vaccine are received by 87.4% of patients. After
propensity score matching, reinfections were associated with
significantly lower hospitalization (5.8% versus 19.4%; risk ratio, 0.3;
95% CI, 0.12-0.71) and oxygen requirement (3.9% versus 13.6%; risk
ratio, 0.29; 95% CI, 0.10-0.84). In a within-patient analysis only in
the reinfection group, the second infection was milder than the first
(3.9% required oxygen versus 19.4%, P &lt; 0.0001), and severe first
COVID-19 was the only predictor of severe reinfection. Transplant
recipients with COVID-19 reinfection present better outcomes than those
with the first infection, providing clinical evidence for the
development of cross-protective immunity. Copyright © 2024 Wolters
Kluwer Health, Inc. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38289666
</td>
<td style="text-align:left;">
Evaluating the Impact of Virtual Reality on the Behavioral and
Psychological Symptoms of Dementia and Quality of Life of Inpatients
With Dementia in Acute Care: Randomized Controlled Trial (VRCT).
</td>
<td style="text-align:left;">
Virtual reality (VR) is increasingly considered a valuable therapeutic
tool for people with dementia. However, rigorous studies are still
needed to evaluate its impact on behavioral and psychological symptoms
of dementia (BPSDs) and quality of life (QoL) across care settings. The
primary aim of this study was to evaluate the impact of VR therapy on
managing BPSDs, falls, length of stay, and QoL in inpatients with
dementia admitted to an acute care hospital. The secondary aim was to
evaluate the intervention’s feasibility in terms of acceptability,
safety, and patient experience. A prospective, open-label, mixed
methods, randomized controlled clinical trial was conducted between
April 2019 and March 2020. A total of 69 participants (aged ≥65 years
with a diagnosis of dementia and who did not meet the exclusion
criteria) were randomly assigned to either the control (n=35, 51%) or VR
(n=34, 49%) arm. Participants in the experimental (VR) arm were visited
by a researcher and watched 360° VR films on a head-mounted display for
up to 20 minutes every 1 to 3 days, whereas individuals in the control
arm received standard of care. Instances of daily BPSDs and falls were
collected from nurses’ daily notes. QoL was measured through
semistructured interviews and the Quality of Life in Late-Stage Dementia
scale. Structured observations and semistructured interviews were used
to measure treatment feasibility. The primary outcomes were analyzed at
a 95% significance level based on the intention-to-treat method. VR
therapy had a statistically significant effect on reducing
aggressiveness (ie, physical aggression and loud vociferation; P=.01).
Substantial impact of VR therapy was not found for other BPSDs (eg,
apathy), falls, length of stay, or QoL as measured using the Quality of
Life in Late-Stage Dementia scale. The average VR therapy session lasted
6.8 (SD 6.6; range 0-20) minutes, and the intervention was overall an
acceptable and enjoyable experience for participants. No adverse events
occurred as a result of VR therapy. Immersive VR therapy appears to have
an effect on aggressive behaviors in patients with dementia in acute
care. Although the randomized controlled trial was stopped before
reaching the intended sample size owing to COVID-19 restrictions, trends
in the results are promising. We suggest conducting future trials with
larger samples and, in some cases, more sensitive data collection
instruments. ClinicalTrials.gov NCT03941119;
<https://clinicaltrials.gov/study/NCT03941119>. RR2-10.2196/22406. ©Lora
Appel, Eva Appel, Erika Kisonas, Samantha Lewis-Fung, Susanna Pardini,
Jarred Rosenberg, Julian Appel, Christopher Smith. Originally published
in the Journal of Medical Internet Research (<https://www.jmir.org>),
30.01.2024.
</td>
</tr>
<tr>
<td style="text-align:left;">
38289642
</td>
<td style="text-align:left;">
Effectiveness of the Wellness Together Canada Portal as a Digital Mental
Health Intervention in Canada: Protocol for a Randomized Controlled
Trial.
</td>
<td style="text-align:left;">
The Wellness Together Canada (WTC) portal is a digital mental health
intervention that was developed in response to an unprecedented rise in
mental health and substance use concerns due to the COVID-19 pandemic,
with funding from the Government of Canada. It is a mental health and
substance use website to support people across Canada providing digital
interventions and services at no cost. Two million people have visited
the WTC portal over the course of 1 year since launching; however,
rigorous evaluation of this potential solution to access to mental
health care during and after the COVID-19 pandemic is urgently required.
This study aims to better understand the effectiveness of the existing
digital interventions in improving population mental health in Canada.
The Let’s Act on Mental Health study is designed as a longitudinal fully
remote, equally randomized (1:1), double-blind, alternative
intervention-controlled, parallel-group randomized controlled trial to
be conducted between October 2023 and April 2024 with a prospective
follow-up study period of 26 weeks. This trial will evaluate whether a
digital intervention such as the WTC improves population mental health
trajectories over time. The study was approved by the research ethics
board of CAMH (Centre for Addiction and Mental Health, Toronto, Ontario,
Canada). It is ongoing and participant recruitment is underway. As of
August 2023, a total of 453 participants in the age group of 18-72 years
have participated, of whom 70% (n=359) are female. This initiative
provides a unique opportunity to match people’s specific unmet mental
health and substance use needs to evidence-based digital interventions.
©Syaron Basnet, Michael Chaiton. Originally published in JMIR Research
Protocols (<https://www.researchprotocols.org>), 30.01.2024.
</td>
</tr>
<tr>
<td style="text-align:left;">
38288995
</td>
<td style="text-align:left;">
Recommendations for supporting healthcare workers’ psychological
well-being: Lessons learned during the COVID-19 pandemic.
</td>
<td style="text-align:left;">
Healthcare workers are at risk of adverse mental health outcomes due to
occupational stress. Many organizations introduced initiatives to
proactively support staff’s psychological well-being in the face of the
COVID-19 pandemic. One example is the STEADY wellness program, which was
implemented in a large trauma centre in Toronto, Canada. Program
implementors engaged teams in peer support sessions, psychoeducation
workshops, critical incident stress debriefing, and community-building
initiatives. As part of a project designed to illuminate the experiences
of STEADY program implementors, this article describes recommendations
for future hospital wellness programs. Participants described the
importance of having the hospital and its leaders engage in supporting
staff’s psychological well-being. They recommended ways of doing so
(e.g., incorporating conversations about wellness in staff onboarding
and routine meetings), along with ways to increase program uptake and
sustainability (e.g., using technology to increase accessibility).
Results may be useful in future efforts to bolster hospital wellness
programming.
</td>
</tr>
<tr>
<td style="text-align:left;">
38288986
</td>
<td style="text-align:left;">
The clinical application of traditional Chinese medicine NRICM101 in
hospitalized patients with COVID-19.
</td>
<td style="text-align:left;">
The aim of this study was to assess the efficacy and safety of NRICM101
in hospitalized patients with COVID-19. We conducted a retrospective
study from 20 April 2021 to 8 July 2021, and evaluated the safety and
outcomes (mortality, hospital stay, mechanical ventilation, oxygen
support, diarrhea, serum potassium) in COVID-19 patients. Propensity
score matching at a 1:2 ratio was performed to reduce confounding
factors. A total of 201 patients were analyzed. The experimental group
(n = 67) received NRICM101 and standard care, while the control group (n
= 134) received standard care alone. No significant differences were
observed in mortality (10.4% vs. 14.2%), intubation (13.8% vs. 11%),
time to intubation (10 vs. 11 days), mechanical ventilation days (0
vs. 9 days), or oxygen support duration (6 vs. 5 days). However, the
experimental group had a shorter length of hospitalization (odds ratio =
0.12, p = 0.043) and fewer mechanical ventilation days (odds ratio =
0.068, p = 0.008) in initially severe cases, along with an increased
diarrhea risk (p = 0.035). NRICM101 did not reduce in-hospital
mortality. However, it shortened the length of hospitalization and
reduced mechanical ventilation days in initially severe cases. Further
investigation is needed.
</td>
</tr>
<tr>
<td style="text-align:left;">
38286593
</td>
<td style="text-align:left;">
“There’s a little bit of mistrust”: Red River Métis experiences of the
H1N1 and COVID-19 pandemics.
</td>
<td style="text-align:left;">
We examined the perspectives of the Red River Métis citizens in
Manitoba, Canada, during the H1N1 and COVID-19 pandemics and how they
interpreted the communication of government/health authorities’ risk
management decisions. For Indigenous populations, pandemic response
strategies play out within the context of ongoing colonial relationships
with government institutions characterized by significant distrust. A
crucial difference between the two pandemics was that the Métis in
Manitoba were prioritized for early vaccine access during H1N1 but not
for COVID-19. Data collection involved 17 focus groups with Métis
citizens following the H1N1 outbreak and 17 focus groups during the
COVID-19 pandemic. Métis prioritization during H1N1 was met with some
apprehension and fear that Indigenous Peoples were vaccine-safety test
subjects before population-wide distribution occurred. By contrast, as
one of Canada’s three recognized Indigenous nations, the
non-prioritization of the Métis during COVID-19 was viewed as an
egregious sign of disrespect and indifference. Our research demonstrates
that both reactions were situated within claims that the government does
not care about the Métis, referencing past and ongoing colonial
motivations. Government and health institutions must anticipate this
overarching colonial context when making and communicating risk
management decisions with Indigenous Peoples. In this vein, government
authorities must work toward a praxis of decolonization in these
relationships, including, for example, working in partnership with
Indigenous nations to engage in collaborative risk mitigation and
communication that meets the unique needs of Indigenous populations and
limits the potential for less benign-though
understandable-interpretations. © 2024 The Authors. Risk Analysis
published by Wiley Periodicals LLC on behalf of Society for Risk
Analysis.
</td>
</tr>
<tr>
<td style="text-align:left;">
38285495
</td>
<td style="text-align:left;">
A Novel Approach for the Early Detection of Medical Resource Demand
Surges During Health Care Emergencies: Infodemiology Study of Tweets.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has highlighted gaps in the current handling of
medical resource demand surges and the need for prioritizing scarce
medical resources to mitigate the risk of health care facilities
becoming overwhelmed. During a health care emergency, such as the
COVID-19 pandemic, the public often uses social media to express
negative sentiment (eg, urgency, fear, and frustration) as a real-time
response to the evolving crisis. The sentiment expressed in COVID-19
posts may provide valuable real-time information about the relative
severity of medical resource demand in different regions of a country.
In this study, Twitter (subsequently rebranded as X) sentiment analysis
was used to investigate whether an increase in negative sentiment
COVID-19 tweets corresponded to a greater demand for hospital intensive
care unit (ICU) beds in specific regions of the United States, Brazil,
and India. Tweets were collected from a publicly available data set
containing COVID-19 tweets with sentiment labels and geolocation
information posted between February 1, 2020, and March 31, 2021.
Regional medical resource shortage data were gathered from publicly
available data sets reporting a time series of ICU bed demand across
each country. Negative sentiment tweets were analyzed using the Granger
causality test and convergent cross-mapping (CCM) analysis to assess the
utility of the time series of negative sentiment tweets in forecasting
ICU bed shortages. For the United States (30,742,934 negative sentiment
tweets), the results of the Granger causality test (for whether negative
sentiment COVID-19 tweets forecast ICU bed shortage, assuming a
stochastic system) were significant (P&lt;.05) for 14 (28%) of the 50
states that passed the augmented Dickey-Fuller test at lag 2, and the
results of the CCM analysis (for whether negative sentiment COVID-19
tweets forecast ICU bed shortage, assuming a dynamic system) were
significant (P&lt;.05) for 46 (92%) of the 50 states. For Brazil
(3,004,039 negative sentiment tweets), the results of the Granger
causality test were significant (P&lt;.05) for 6 (22%) of the 27
federative units, and the results of the CCM analysis were significant
(P&lt;.05) for 26 (96%) of the 27 federative units. For India (4,199,151
negative sentiment tweets), the results of the Granger causality test
were significant (P&lt;.05) for 6 (23%) of the 26 included regions (25
states and the national capital region of Delhi), and the results of the
CCM analysis were significant (P&lt;.05) for 26 (100%) of the 26
included regions. This study provides a novel approach for identifying
the regions of high hospital bed demand during a health care emergency
scenario by analyzing Twitter sentiment data. Leveraging analyses that
take advantage of natural language processing-driven tweet extraction
systems has the potential to be an effective method for the early
detection of medical resource demand surges. ©Mahakprit Kaur, Taylor
Cargill, Kevin Hui, Minh Vu, Nicola Luigi Bragazzi, Jude Dzevela Kong.
Originally published in JMIR Formative Research
(<https://formative.jmir.org>), 29.01.2024.
</td>
</tr>
<tr>
<td style="text-align:left;">
38284645
</td>
<td style="text-align:left;">
Did Descriptive and Prescriptive Norms About Gender Equality at Home
Change During the COVID-19 Pandemic? A Cross-National Investigation.
</td>
<td style="text-align:left;">
Using data from 15 countries, this article investigates whether
descriptive and prescriptive gender norms concerning housework and child
care (domestic work) changed after the onset of the COVID-19 pandemic.
Results of a total of 8,343 participants (M = 19.95, SD = 1.68) from two
comparable student samples suggest that descriptive norms about unpaid
domestic work have been affected by the pandemic, with individuals
seeing mothers’ relative to fathers’ share of housework and child care
as even larger. Moderation analyses revealed that the effect of the
pandemic on descriptive norms about child care decreased with countries’
increasing levels of gender equality; countries with stronger gender
inequality showed a larger difference between pre- and post-pandemic.
This study documents a shift in descriptive norms and discusses
implications for gender equality-emphasizing the importance of
addressing the additional challenges that mothers face during
health-related crises.
</td>
</tr>
<tr>
<td style="text-align:left;">
38282921
</td>
<td style="text-align:left;">
Acceptability of the Long-Term In-Home Ventilator Engagement virtual
intervention for home mechanical ventilation patients during the
COVID-19 pandemic: A qualitative evaluation.
</td>
<td style="text-align:left;">
Clinical management of ventilator-assisted individuals (VAIs) was
challenged by social distancing rules during the COVID-19 pandemic. In
May 2020, the Long-Term In-Home Ventilator Engagement (LIVE) Program was
launched in Ontario, Canada to provide intensive digital care case
management to VAIs. The purpose of this qualitative study was to explore
the acceptability of the LIVE Program hosted via a digital platform
during the COVID-19 pandemic from diverse perspectives. We conducted a
qualitative descriptive study (May 2020-April 2021) comprising
semi-structured interviews with participants from eight home ventilation
specialty centers in Ontario, Canada. We purposively recruited patients,
family caregivers, and providers enrolled in LIVE. Content analysis and
the theoretical concepts of acceptability, feasibility, and
appropriateness were used to interpret findings. A total of 40
individuals (2 VAIs, 18 family caregivers, 20 healthcare providers)
participated. Participants described LIVE as acceptable as it addressed
a longstanding imperative to improve care access, ease of use, and
training provided; feasible for triaging problems and sharing
information; and appropriate for timeliness of provider responses,
workflows, and perceived value. Negative perceptions of acceptability
among healthcare providers concerned digital workload and fit with
existing clinical workflows. Perceived benefits accorded to LIVE
included enhanced physical and psychological safety in the home,
patient-provider relations, and VAI engagement in their own care. Study
findings identify factors influencing the LIVE Program’s acceptability
by patients, family caregivers, and healthcare providers during pandemic
conditions including enhanced access to care, ease of case management
triage, and VAI safety. Findings may inform the implementation of
digital health services to VAIs in non-pandemic circumstances. © The
Author(s) 2024.
</td>
</tr>
<tr>
<td style="text-align:left;">
38281988
</td>
<td style="text-align:left;">
Joint angle estimation during shoulder abduction exercise using
contactless technology.
</td>
<td style="text-align:left;">
Tele-rehabilitation, also known as tele-rehab, uses communication
technologies to provide rehabilitation services from a distance. The
COVID-19 pandemic has highlighted the importance of tele-rehab, where
the in-person visits declined and the demand for remote healthcare
rises. Tele-rehab offers enhanced accessibility, convenience,
cost-effectiveness, flexibility, care quality, continuity, and
communication. However, the current systems are often not able to
perform a comprehensive movement analysis. To address this, we propose
and validate a novel approach using depth technology and skeleton
tracking algorithms. Our data involved 14 participants (8 females, 6
males) performing shoulder abduction exercises. We collected depth
videos from an LiDAR camera and motion data from a Motion Capture
(Mocap) system as our ground truth. The data were collected at distances
of 2 m, 2.5 m, and 3.5 m from the LiDAR sensor for both arms. Our
innovative approach integrates LiDAR with the Cubemos and Mediapipe
skeleton tracking frameworks, enabling the assessment of 3D joint
angles. We validated the system by comparing the estimated joint angles
versus Mocap outputs. Personalized calibration was applied using various
regression models to enhance the accuracy of the joint angle
calculations. The Cubemos skeleton tracking system outperformed
Mediapipe in joint angle estimation with higher accuracy and fewer
errors. The proposed system showed a strong correlation with Mocap
results, although some deviations were present due to noise. Precision
decreased as the distance from the camera increased. Calibration
significantly improved performance. Linear regression models
consistently outperformed nonlinear models, especially at shorter
distances. This study showcases the potential of a marker-less system,
to proficiently track body joints and upper-limb angles. Signals from
the proposed system and the Mocap system exhibited robust correlation,
with Mean Absolute Errors (MAEs) consistently below \[Formula: see
text\]. LiDAR’s depth feature enabled accurate computation of in-depth
angles beyond the reach of traditional RGB cameras. Altogether, this
emphasizes the depth-based system’s potential for precise joint tracking
and angle calculation in tele-rehab applications. © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38280984
</td>
<td style="text-align:left;">
A comparison between different patient groups for diabetes management
during phases of the COVID-19 pandemic: a retrospective cohort study in
Ontario, Canada.
</td>
<td style="text-align:left;">
With the onset of the COVID-19 pandemic and the large uptake in virtual
care in primary care in Canada, the care of patients with type 2
diabetes has been greatly affected. This includes decreased in-person
visits, laboratory testing and in-person assessments such as blood
pressure (BP). No studies have investigated if these changes persisted
with pandemic progression, and it is unclear if shifts impacted patient
groups uniformly. The purpose of this paper was to examine changes in
diabetes care pre, early, and later pandemic across different patient
groups. A repeated cross-sectional design with an open cohort was used
to investigate diabetes care in adults with type 2 diabetes for a
6-month interval from March 14 to September 13 over three consecutive
years: 2019 (pre-pandemic period), 2020 (early pandemic period), and
2021 (later pandemic period). Data for this study were abstracted from
the University of Toronto Practice-Based Research Network (UTOPIAN) Data
Safe Haven, a primary care electronic medical records database in
Ontario, Canada. Changes in diabetes care, which included primary care
total visits, in-person visits, hemoglobin A1c (HbA1c) testing, and BP
measurements were evaluated across the phases of the pandemic.
Difference in diabetes care across patient groups, including age, sex,
income quintile, prior HbA1c levels, and prior BP levels, were assessed.
A total of 39,401 adults with type 2 diabetes were included in the
study. Compared to the 6-month pre-pandemic period, having any in-person
visits decreased significantly early pandemic (OR = 0.079
(0.076-0.082)), with a partial recovery later pandemic (OR = 0.162 (95%
CI: 0.157-0.169). Compared to the pre-pandemic period, there was a
significant decrease early pandemic for total visits (OR = 0.486 (95%
CI: 0.470-0.503)), HbA1c testing (OR = 0.401 (95% CI: 0.389-0.413)), and
BP measurement (OR = 0.121 (95% CI: 0.116-0.125)), with partial recovery
later pandemic. All measures of diabetes care were substantially
decreased early pandemic, with a partial recovery later pandemic across
all patient groups. With the increase in virtual care due to the
COVID-19 pandemic, diabetes care has been negatively impacted over
1-year after pandemic onset. © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38279660
</td>
<td style="text-align:left;">
Facilitating virtual social connections for youth with disabilities:
lessons for post-COVID-19 programming.
</td>
<td style="text-align:left;">
Social connections are essential for the development of life skills for
youth. Youth with disabilities have long faced barriers to meaningful
social connections. The onset of COVID-19 increased barriers to social
connections for all youth, and also led to enhanced use of virtual
platforms in paediatric rehabilitation programming. Harnessing this
opportunity, service providers created a suite of online programs to
foster social connections and friendships. The current study explores
participant and service provider experiences of such programs. This
qualitative descriptive study used interviews and focus groups to
explore how youth with disabilities (n = 8), their parents (n = 7), and
service providers (n = 13) involved in program development and delivery
experienced the programs, the accessibility of the virtual platforms,
and their social connections in relation to program participation.
Participants were satisfied with the programs’ content, accessibility
and ability to meet their social needs. Qualitative themes included
facilitating social connections, accessibility of virtual spaces, and
recommendations for future virtual programming. For youth with
disabilities who have been historically marginalized in social spheres,
the newly ubiquitous infrastructure regarding virtual programming must
be supported and enhanced. A hybrid approach involving virtual/in-person
options in future programming is recommended.
</td>
</tr>
<tr>
<td style="text-align:left;">
38278628
</td>
<td style="text-align:left;">
Determinants of SARS-CoV-2 IgG response and decay in Canadian healthcare
workers: A prospective cohort study.
</td>
<td style="text-align:left;">
Healthcare workers (HCWs) from an interprovincial Canadian cohort gave
serial blood samples to identify factors associated with anti-receptor
binding domain (anti-RBD) IgG response to the SARS-CoV-2 virus. Members
of the HCW cohort donated blood samples four months after their first
SARS-CoV-2 immunization and again at 7, 10 and 13 months. Date and type
of immunizations and dates of SARS-CoV-2 infection were collected at
each of four contacts, together with information on
immunologically-compromising conditions and current therapies. Blood
samples were analyzed centrally for anti-RBD IgG and anti-nucleocapsid
IgG (Abbott Architect, Abbott Diagnostics). Records of immunization and
SARS-CoV-2 testing from public health agencies were used to assess the
impact of reporting errors on estimates from the random-effects
multivariable model fitted to the data. 2752 of 4567 vaccinated cohort
participants agreed to donate at least one blood sample. Modelling of
anti-RBD IgG titer from 8903 samples showed an increase in IgG with each
vaccine dose and with first infection. A decrease in IgG titer was found
with the number of months since vaccination or infection, with the
sharpest decline after the third dose. An immunization regime that
included mRNA1273 (Moderna) resulted in higher anti-RBD IgG.
Participants reporting multiple sclerosis, rheumatoid arthritis or
taking selective immunosuppressants, tumor necrosis factor inhibitors,
calcineurin inhibitors and antineoplastic agents had lower anti-RBD IgG.
Supplementary analyses showed higher anti-RBD IgG in those reporting
side-effects of vaccination, no relation of anti-RBD IgG to obesity and
lower titers in women immunized in early or mid-pregnancy. Sensitivity
analysis results suggested no important bias in the self-report data.
Creation of a prospective cohort was central to the credibility of
results presented here. Serial serology assessments, with longitudinal
analysis, provided effect estimates with enhanced accuracy and a clearer
understanding of medical and other factors affecting response to
vaccination. Copyright © 2024 The Author(s). Published by Elsevier Ltd..
All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38278415
</td>
<td style="text-align:left;">
Comparison of Omicron breakthrough infection versus monovalent
SARS-CoV-2 intramuscular booster reveals differences in mucosal and
systemic humoral immunity.
</td>
<td style="text-align:left;">
Our understanding of the quality of cellular and humoral immunity
conferred by COVID-19 vaccination alone versus vaccination plus
SARS-CoV-2 breakthrough (BT) infection remains incomplete. While the
current (2023) SARS-CoV-2 immune landscape of Canadians is complex, in
late 2021 most Canadians had either just received a third dose of
COVID-19 vaccine, or had received their two-dose primary series and then
experienced an Omicron BT. Herein we took advantage of this coincident
timing to contrast cellular and humoral immunity conferred by three
doses of vaccine versus two doses plus BT. Our results show thatBT
infection induces cell-mediated immune responses to variants comparable
to an intramuscular vaccine booster dose. In contrast, BT subjects had
higher salivary immunoglobulin (Ig)G and IgA levels against the Omicron
spike and enhanced reactivity to the ancestral spike for the IgA
isotype, which also reacted with SARS-CoV-1. Serumneutralizing antibody
levels against the ancestral strain and the variants were also higher
after BT infection. Our results support the need for the development of
intranasal vaccines that could emulate the enhanced mucosal and humoral
immunity induced by Omicron BT without exposing individuals to the risks
associated with SARS-CoV-2 infection. Copyright © 2024 The Author(s).
Published by Elsevier Inc. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38277622
</td>
<td style="text-align:left;">
School Attendance Among Pediatric Oncology Patients During the COVID-19
Pandemic in Ontario, Canada.
</td>
<td style="text-align:left;">
Supporting schooling for current and past pediatric oncology patients is
vital to their quality of life and psychosocial recovery. However, no
study has examined the perspectives toward in-person schooling among
pediatric oncology families during the COVID-19 pandemic. In this online
survey study, we determined the rate of and attitudes toward in-person
school attendance among current and past pediatric oncology patients
living in Ontario, Canada during the 2020-2021 school year. Of our
31-family cohort, 23 children (74%) did attend and 8 (26%) did not
attend any in-person school during this time. Fewer children within 2
years of treatment completion attended in-person school (5/8; 62%) than
those more than 2 years from treatment completion (13/15; 87%). Notably,
22 of 29 parents (76%) felt that speaking to their care team had the
greatest impact compared to other potential information sources when
deciding about school participation, yet 13 (45%) were unaware of their
physician’s specific recommendation regarding whether their child should
attend. This study highlights the range in parental comfort regarding
permitting in-person schooling during the COVID-19 pandemic. Pediatric
oncologists should continue to address parental concerns around
in-person school during times of high transmission of COVID-19 and
potentially other communicable diseases in the future. Copyright © 2024
Wolters Kluwer Health, Inc. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38274670
</td>
<td style="text-align:left;">
Planning with a gender lens: A gender analysis of pandemic preparedness
plans from eight countries in Africa.
</td>
<td style="text-align:left;">
Health planning and priority setting with a gender lens can help to
anticipate and mitigate vulnerabilities that women and girls may
experience in health systems, which is especially relevant during health
emergencies. This study examined how gender considerations were
accounted for in COVID-19 pandemic response planning in a subset of
countries in Africa. Multi-country document review of national pandemic
response plans (published before July 2020 and as of March 2022) from
Ethiopia, Ghana, Kenya, Nigeria, Rwanda, South Africa, Uganda, and
Zambia, supplemented with secondary data on gender representation on
planning committees. A gender analysis framework informed the study
design and the Morgan et al. matrix guided data extraction and analysis.
All plans reflected implicit and explicit considerations of the impacts
of the pandemic responses on women and girls. Through a gender lens, the
implicit considerations focused on ensuring safety and protections
(e.g., training, access to personal protective equipment) for community
and facility-based health care workers and broad engagement of the
community in risk communication. The explicit gender considerations,
reflected in a minority of plans, focused on addressing gender-based
violence and providing access to essential services (e.g., sexual and
reproductive health care, psychosocial supports), products (e.g.,
menstrual hygiene products) and social protection measures. Women were
underrepresented on the COVID-19 planning committees in all countries.
The plans reflected varying national efforts to develop pandemic
responses that anticipated and reflected unique vulnerabilities faced by
women, though subsequent plans reflected further consideration of
gender-relevant impacts compared to initial plans. Embedding a gender
lens in emergency preparedness planning furthers equity and could
support anticipation and timely mitigation of negative outcomes for
women and girls who are often further marginalized during health
emergencies. © 2023 The Authors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38274512
</td>
<td style="text-align:left;">
The impact of COVID-19 on nurses’ job satisfaction: a systematic review
and meta-analysis.
</td>
<td style="text-align:left;">
The global healthcare landscape was profoundly impacted by the COVID-19
pandemic placing nurses squarely at the heart of this emergency. This
review aimed to identify the factors correlated with nurses’ job
satisfaction, the impact of their job satisfaction on both themselves
and their patients, and to explore strategies that might have
counteracted their job dissatisfaction during the COVID-19 pandemic. The
Joanna Briggs Institute (JBI) methodology for systematic reviews of
prevalence and incidence was used in this review. The electronic
databases of CINAHL, MEDLINE, SCOPUS, PsycINFO and Academic Search
Complete were searched between January 2020 to February 2023. The
literature review identified 23 studies from 20 countries on nurses’ job
satisfaction during the COVID-19 pandemic. A pooled prevalence of 69.6%
of nurses were satisfied with personal, environmental, and psychological
factors influencing their job satisfaction. Job satisfaction improved
psychological wellbeing and quality of life, while dissatisfaction was
linked to turnover and mental health issues. This systematic review
elucidates key factors impacting nurses’ job satisfaction during the
COVID-19 pandemic, its effects on healthcare provision, and the
potential countermeasures for job dissatisfaction. Core influences
include working conditions, staff relationships, and career
opportunities. High job satisfaction correlates with improved patient
care, reduced burnout, and greater staff retention.
<https://www.crd.york.ac.uk/prospero/display_record.php?ID=CRD42023405947>,
the review title has been registered in PROSPERO and the registration
number is CRD42023405947. Copyright © 2024 Yasin, Alomari, Al-Hamad and
Kehyayan.
</td>
</tr>
<tr>
<td style="text-align:left;">
38273454
</td>
<td style="text-align:left;">
Establishing association between HLA-C\*04:01 and severe COVID-19.
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
38272559
</td>
<td style="text-align:left;">
Strategies to support maternal and early childhood wellness: insight
from parent and provider qualitative interviews during the COVID-19
pandemic.
</td>
<td style="text-align:left;">
The COVID-19 pandemic resulted in rapid changes to the delivery of
maternal and newborn care. Our aim was to gain an understanding from
parents and healthcare professionals (HCPs) of how the pandemic and
associated public health restrictions impacted the peripartum and
postpartum experience, as well as longer-term health and well-being of
families. Qualitative study through focus groups. Ontario, Canada. HCPs
and parents who had a child born during the COVID-19 pandemic.
Semistructured interview guide, with questions focused on how the
pandemic impacted their care/their ability to provide care, and
strategies to improve care and support now or in future situations with
similar healthcare restrictions. Thematic analysis was used to describe
participant experiences and recommendations. We included 11 HCPs and 15
parents in 6 focus groups. Participants described their experiences as
‘traumatic’, with difficulties in accessing prenatal and postpartum
services, and feelings of distress and isolation. They also noted delays
in speech and development in children born during the pandemic. Key
recommendations included the provision of partner accompaniment
throughout the course of care, expansion of available services for young
families (particularly postpartum), and special considerations for
marginalised groups, including access to technology for virtual care or
the option of in-person visits. Our findings may inform the development
of healthcare system and organisational policies to ensure the provision
of maternal and newborn care in the event of future public health
emergencies. Of primary importance to the participants was the
accommodation of antenatal, intrapartum and postpartum partner
accompaniment, and the provision of postpartum services. © Author(s) (or
their employer(s)) 2024. Re-use permitted under CC BY-NC. No commercial
re-use. See rights and permissions. Published by BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38267971
</td>
<td style="text-align:left;">
Evaluating in vivo effectiveness of sotrovimab for the treatment of
Omicron subvariant BA.2 versus BA.1: a multicentre, retrospective cohort
study.
</td>
<td style="text-align:left;">
In vitro data suggested reduced neutralizing capacity of sotrovimab, a
monoclonal antibody, against Omicron BA.2 subvariant. However, limited
in vivo data exist regarding clinical effectiveness of sotrovimab for
coronavirus disease 2019 (COVID-19) due to Omicron BA.2. A multicentre,
retrospective cohort study was conducted at three Canadian academic
tertiary centres. Electronic medical records were reviewed for patients
≥ 18 years with mild COVID-19 (sequencing-confirmed Omicron BA.1 or
BA.2) treated with sotrovimab between February 1 to April 1, 2022.
Thirty-day co-primary outcomes included hospitalization due to moderate
or severe COVID-19; all-cause intensive care unit (ICU) admission, and
all-cause mortality. Risk differences (BA.2 minus BA.1 group) for
co-primary outcomes were adjusted with propensity score matching (e.g.,
age, sex, vaccination, immunocompromised status). Eighty-five patients
were included (15 BA.2, 70 BA.1) with similar baseline characteristics
between groups. Adjusted risk differences were non-statistically
significant between groups for 30-day hospitalization (- 14.3%; 95%
confidence interval (CI): - 32.6 to 4.0%), ICU admission (- 7.1%;
95%CI: - 20.6 to 6.3%), and mortality (- 7.1%; 95%CI: - 20.6 to 6.3%).
No differences were demonstrated in hospitalization, ICU admission, or
mortality rates within 30 days between sotrovimab-treated patients with
BA.1 versus BA.2 infection. More real-world data may be helpful to
properly assess sotrovimab’s effectiveness against infections due to
specific emerging COVID-19 variants. © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38266201
</td>
<td style="text-align:left;">
Virtual Cancer Care Beyond the COVID-19 Pandemic: Patient and Staff
Perspectives and Recommendations.
</td>
<td style="text-align:left;">
COVID-19 catalyzed rapid implementation of virtual cancer care (VC);
however, work is needed to inform long-term adoption. We evaluated
patient and staff experiences with VC at a large urban, tertiary cancer
center to inform recommendations for postpandemic sustainment. All
physicians who had provided VC during the pandemic and all patients who
had a valid e-mail address on file and at least one visit to the
Princess Margaret Cancer Centre in Toronto, Canada, in the preceding
year were invited to complete a survey. Interviews and focus groups with
patients and staff across the cancer center were analyzed using
qualitative descriptive analysis and triangulated with survey findings.
Response rates for patients and physicians were 15% (2,343 of 15,169)
and 41% (100 of 246), respectively. A greater proportion of patients
than physicians were satisfied with VC (80.1 v 53.4%; P &lt; .01). In
addition, fewer patients than physicians felt that virtual visits were
worse than those conducted in person (28.0 v 43.4%; P &lt; .01) and that
telephone and video visits negatively affected the human interaction
that they valued (59.8% v 82.0%; P &lt; .01). Major barriers to VC for
patients were respect for care preferences and personal boundaries,
accessibility, and equitable access. For staff, major barriers included
a lack of role clarity, dedicated resources (space and technology),
integration of nursing and allied health, support (administrative,
clinical, and technical), and guidance on appropriateness of use.
Patient and staff perceptions and barriers to virtual care are
different. Moving forward, we need to pay attention to both staff and
patient experiences with virtual care since this will have major
implications for long-term adoption into clinical practice.
</td>
</tr>
<tr>
<td style="text-align:left;">
38266144
</td>
<td style="text-align:left;">
“Everything has changed since COVID”: Ongoing challenges faced by
Canadian adults with intellectual disabilities during waves 2 and 3 of
the COVID-19 pandemic.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has disrupted the lives of people with
intellectual disabilities in many ways, impacting their health and
wellbeing. Early in the pandemic, the research team delivered a six-week
virtual group-based program to help Canadian adults with intellectual
disabilities cope and better manage their mental health. The study’s
objective was to explore ongoing concerns among individuals with
intellectual disabilities following their participation in this
education and support program. Thematic analysis was used to analyze
participant feedback provided eight weeks after course completion.
Twenty-four participants were interviewed in January 2021 and May 2021
across two cycles of the course. Three themes emerged: 1) employment and
financial challenges; 2) navigating changes and ongoing restrictions;
and 3) vaccine anticipation and experience. These findings suggest that
despite benefiting from the program, participants continued to
experience pandemic-related challenges in 2021, emphasising the need to
continually engage people with intellectual disabilities.
</td>
</tr>
<tr>
<td style="text-align:left;">
38265051
</td>
<td style="text-align:left;">
Impact of the COVID-19 pandemic on perceived changes in responsibilities
for adult caregivers who support children and youth in Ontario, Canada.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has created long-lasting changes in caregiving
responsibilities, including but not limited to increased demands, loss
of support, worsening mental and physical health, and increased
financial worries. There is currently limited evidence regarding factors
associated with perceived changes in caregiving responsibilities. This
observational study aimed to investigate factors (sociodemographic
characteristics of caregivers and mental health and/or addiction
concerns of the caregiver and their youth) that predict perceived
negative changes in caregiving responsibilities among adult caregivers
(aged 18+ years) of children and youth (aged 0-25 years) in Ontario,
Canada, during the COVID-19 pandemic. Data were collected from 1381
caregivers of children and youth between January and March of 2022
through a representative cross-sectional survey completed online.
Logistic regression was conducted to determine predictors contributing
to perceived negative changes in caregiving responsibilities. Among the
sociodemographic characteristics, only ethnicity significantly predicted
outcome. Higher caregiver strain (odds ratio \[OR\] = 10.567, 95% CI =
6.614-16.882, P &lt; 0.001), worsened personal mental health (OR =
1.945, 95% CI = 1.474-2.567, P &lt; 0.001), a greater number of
children/youth cared for per caregiver (OR = 1.368, 95% CI =
1.180-1.587, P &lt; 0.001), dissatisfaction with the availability of
social supports (OR = 1.768, 95% CI = 1.297-2.409, P &lt; 0.001) and
negative changes in mental well-being in at least one child/youth (OR =
2.277, 95% CI = 1.660-3.123, P &lt; 0.001) predicted negative changes in
caregiving responsibilities. These results support further exploration
of the implications of negative perceptions of caregiving
responsibilities and what processes might be implemented to improve
these perceptions and the outcomes.
</td>
</tr>
<tr>
<td style="text-align:left;">
38263777
</td>
<td style="text-align:left;">
Economic precarity and changing levels of anxiety and stress among
Canadians with disabilities and chronic health conditions throughout the
COVID-19 pandemic.
</td>
<td style="text-align:left;">
Early in the COVID-19 pandemic, multiple event stressors converged to
exacerbate a growing mental health crisis in Canada with differing
effects across status groups. However, less is known about changing
mental health situations throughout the pandemic, especially among
individuals more likely to experience chronic stress because of their
disability and health status. Using data from two waves of a targeted
online survey of people with disabilities and chronic health conditions
in Canada (N = 563 individuals, June 2020 and July 2021), we find that
approximately 25% of respondents experienced additional increases in
stress and anxiety levels in 2021. These increases were partly explained
by worsening perceived financial insecurity and, in the case of stress,
additional negative financial effects tied to the pandemic. This paper
understands mental health disparities as a function of social status and
social group membership. By linking stress process models and a minority
stress framework with a social model of disability, we allude to how
structural and contextual barriers make functional limitations disabling
and in turn, life stressors. © 2024 Canadian Sociological Association/La
Société canadienne de sociologie.
</td>
</tr>
<tr>
<td style="text-align:left;">
38263725
</td>
<td style="text-align:left;">
Exploring the Lived Experiences of Individuals With Spinal Cord Injury
During the COVID-19 Pandemic.
</td>
<td style="text-align:left;">
The global spread of severe acute respiratory syndrome coronavirus 2019
(COVID-19) has affected over 100 countries and has led to the tragic
loss of life, overwhelmed health care systems and severely impacted the
global economy. Specifically, individuals living with spinal cord injury
(SCI) are particularly vulnerable during the COVID-19 pandemic as they
often face adverse impacts on their health, emotional well-being,
community participation, and life expectancy. The objective of this
study was to investigate the lived experience of individuals with SCI
during the COVID-19 pandemic in Ontario, Canada. An exploratory design
with a qualitative descriptive approach was used to address the study
objective. Nine semi-structured interviews were conducted with
individuals with traumatic and non-traumatic SCI (37-69 years, C3-L5,
AIS A-D, and 5-42 years post-injury). Using reflexive thematic analysis,
the following themes were created: (1) Caregiver exposure to COVID-19;
(2) Staying physically active in quarantine; (3) Living in social
isolation; (4) Difficulty obtaining necessary medical supplies; (5)
Access to health services and virtual care during COVID-19; and (6)
Fighting COVID-19 misinformation. This is one of the first studies to
explore the impact of COVID-19 on individuals living with SCI in
Ontario. This study contributes to a greater understanding of the
challenges faced by individuals living with SCI and provides insight
into how to better support and respond to the specific and unique needs
of individuals with SCI and their families during a national emergency
or pandemic.
</td>
</tr>
<tr>
<td style="text-align:left;">
38263191
</td>
<td style="text-align:left;">
Structural and biochemical rationale for Beta variant protein booster
vaccine broad cross-neutralization of SARS-CoV-2.
</td>
<td style="text-align:left;">
Severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2),
responsible for the COVID-19 pandemic, uses a surface expressed trimeric
spike glycoprotein for cell entry. This trimer is the primary target for
neutralizing antibodies making it a key candidate for vaccine
development. During the global pandemic circulating variants of concern
(VOC) caused several waves of infection, severe disease, and death. The
reduced efficacy of the ancestral trimer-based vaccines against emerging
VOC led to the need for booster vaccines. Here we present a detailed
characterization of the Sanofi Beta trimer, utilizing cryo-EM for
structural elucidation. We investigate the conformational dynamics and
stabilizing features using orthogonal SPR, SEC, nanoDSF, and HDX-MS
techniques to better understand how this antigen elicits superior broad
neutralizing antibodies as a variant booster vaccine. This structural
analysis confirms the Beta trimer preference for canonical quaternary
structure with two RBD in the up position and the reversible equilibrium
between the canonical spike and open trimer conformations. Moreover,
this report provides a better understanding of structural differences
between spike antigens contributing to differential vaccine efficacy. ©
2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38262079
</td>
<td style="text-align:left;">
Motivation to participate and attrition factors in a COVID-19 biobank: A
qualitative study.
</td>
<td style="text-align:left;">
The Biobanque québécoise de la COVID-19 (Quebec Biobank for COVID-19, or
BQC19) is a provincial initiative that aims to manage the longitudinal
collection, storage, and sharing of biological samples and clinical data
related to COVID-19. During the study, BQC19 investigators reported a
high loss-to-follow-up rate. The current study aimed to explore
motivational and attrition factors from the perspective of BQC19
participants and health care and research professionals. This was an
inductive exploratory qualitative study. Using a theoretical sampling
approach, a sample of BQC19 participants and professionals were invited
to participate via semi-structured interviews. Topics included
motivations to participate; participants’ fears, doubts, and barriers to
participation; and professionals’ experiences with biobanking during the
COVID-19 pandemic. Interviews were conducted with BQC19 participants (n
= 23) and professionals (n = 17) from 8 clinical data collection sites.
Motivations included the contribution to science and society in crisis,
self-worth, and interactions with medical professionals. Reasons for
attrition included logistical barriers, negative attitudes about public
health measures or genomic studies, fear of clinical settings, and a
desire to move on from COVID-19. Motivations and barriers seemed to
evolve over time and with COVID-19 trends and surges. Certain situations
were associated with attrition, such as when patients experienced
indirect verbal consent during hospitalization. Barriers related to
human and material resources and containment/prevention measures limited
the ability of research teams to recruit and retain participants,
especially in the ever-evolving context of crisis. The pandemic setting
impacted participation and attrition, either by influencing
participants’ motivations and barriers or by affecting research teams’
ability to recruit and retain participants. Longitudinal and/or
biobanking studies in a public health crisis setting should consider
these factors to limit attrition. Crown Copyright © 2024. Published by
Elsevier Ltd. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38258676
</td>
<td style="text-align:left;">
Effectiveness of a fourth mRNA dose among individuals with systemic
autoimmune rheumatic diseases during the Omicron era.
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
38254851
</td>
<td style="text-align:left;">
Hypofractionated Radiotherapy in Gynecologic Malignancies-A Peek into
the Upcoming Evidence.
</td>
<td style="text-align:left;">
Radiotherapy (RT) has a fundamental role in the treatment of gynecologic
malignancies, including cervical and uterine cancers. Hypofractionated
RT has gained popularity in many cancer sites, boosted by technological
advances in treatment delivery and image verification. Hypofractionated
RT uptake was intensified during the COVID-19 pandemic and has the
potential to improve universal access to radiotherapy worldwide,
especially in low-resource settings. This review summarizes the
rationale, the current challenges and investigation efforts, together
with the recent developments associated with hypofractionated RT in
gynecologic malignancies. A comprehensive search was undertaken using
multiple databases and ongoing trial registries. In the definitive
radiotherapy setting for cervical cancers, there are several ongoing
clinical trials from Canada, Mexico, Iran, the Philippines and Thailand
investigating the role of a moderate hypofractionated external beam RT
regimen in the low-risk locally advanced population. Likewise, there are
ongoing ultra and moderate hypofractionated RT trials in the uterine
cancer setting. One Canadian prospective trial of stereotactic
hypofractionated adjuvant RT for uterine cancer patients suggested a
good tolerance to this treatment strategy in the acute setting, with a
follow-up trial currently randomizing patients between conventional
fractionation and the hypofractionated dose regimen delivered in the
former trial. Although not yet ready for prime-time use,
hypofractionated RT could be a potential solution to several challenges
that limit access to and the utilization of radiotherapy for gynecologic
cancer patients worldwide.
</td>
</tr>
<tr>
<td style="text-align:left;">
38253978
</td>
<td style="text-align:left;">
Disproportionate Rates of COVID-19 Among Black Canadian Communities:
Lessons from a Cross-Sectional Study in the First Year of the Pandemic.
</td>
<td style="text-align:left;">
Racialized communities, including Black Canadians, have
disproportionately higher COVID-19 cases. We examined the extent to
which SARS-CoV-2 infection has affected the Black Canadian community and
the factors associated with the infection. We conducted a
cross-sectional survey in an area of Ontario (northwest Toronto/Peel
Region) with a high proportion of Black residents along with 2 areas
that have lower proportions of Black residents (Oakville and London,
Ontario). SARS-CoV-2 IgG antibodies were determined using the EUROIMMUN
assay. The study was conducted between August 15, 2020, and December 15,
2020. Among 387 evaluable subjects, the majority, 273 (70.5%), were
enrolled from northwest Toronto and adjoining suburban areas of Peel,
Ontario. The seropositivity values for Oakville and London were
comparable (3.3% (2/60; 95% CI 0.4-11.5) and 3.9% (2/51; 95% CI
0.5-13.5), respectively). Relative to these areas, the seropositivity
was higher for the northwest Toronto/Peel area at 12.1% (33/273),
relative risk (RR) 3.35 (1.22-9.25). Persons 19 years of age or less had
the highest seropositivity (10/50; 20.0%, 95% CI 10.3-33.7%), RR 2.27
(1.23-3.59). There was a trend for an interaction effect between race
and location of residence as this relates to the relative risk of
seropositivity. During the early phases of the pandemic, the
seropositivity within a COVID-19 high-prevalence zone was threefold
greater than lower prevalence areas of Ontario. Black individuals were
among those with the highest seroprevalence of SARS-CoV-2. © 2024. The
Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38252464
</td>
<td style="text-align:left;">
Effects of a Rice-Farming Simulation Video Game on Nature Relatedness,
Nutritional Status, and Psychological State in Urban-Dwelling Adults
During the COVID-19 Pandemic: Randomized Waitlist Controlled Trial.
</td>
<td style="text-align:left;">
During the COVID-19 pandemic, urban inhabitants faced significant
challenges in maintaining connections with nature, adhering to
nutritional guidelines, and managing mental well-being. Recognizing the
urgent need for innovative approaches, this study was designed to
explore the potential benefits of a specific digital intervention, the
rice-farming simulation game Sakuna: Of Rice and Ruin, for nature
relatedness, nutritional behaviors, and psychological well-being. A
total of 66 adults without any prior major psychiatric disorders
residing in an urban area were recruited for the study. They were
randomly assigned to 2 groups through block randomization: the immediate
intervention group (IIG; 34/66, 52%) and the waitlist group (32/66,
48%). Participants in the IIG were instructed to play the game for at
least 4 days per week for 3 weeks, with each session lasting from 30
minutes to 3 hours. Assessments were performed at baseline, week 1, and
week 3. The Nature Relatedness Scale (NR) and Nutrition Quotient Scale
were used to evaluate nature relatedness and nutritional state,
respectively. Furthermore, psychological state was assessed using the
World Health Organization Quality of Life-Brief Version (WHOQOL-BREF),
Brief Fear of Negative Evaluation Scale, Social Avoidance and Distress
Scale, Toronto Alexithymia Scale, State-Trait Anxiety Inventory, Center
for Epidemiologic Studies Depression Scale Revised, and Korean
Resilience Quotient. This study’s results revealed significant time
interactions between the IIG and waitlist group for both the total NR
score (P=.001) and the score of the self subdomain of NR (P&lt;.001),
indicating an impact of the game on nature relatedness. No group×time
interactions were found for the total Nutrition Quotient Scale and
subdomain scores, although both groups showed increases from baseline.
For psychological state, a significant group×time interaction was
observed in the total WHOQOL-BREF score (P=.049), suggesting an impact
of the game on quality of life. The psychological (P=.01), social
(P=.003), and environmental (P=.04) subdomains of the WHOQOL-BREF showed
only a significant time effect. Other psychological scales did not
display any significant changes (all P&gt;.05). Our findings suggest
that the rice-farming game intervention might have positive effects on
nature relatedness, nature-friendly dietary behaviors, quality of life,
anxiety, depression, interpersonal relationships, and resilience among
urban adults during the COVID-19 pandemic. The impact of pronature games
in confined urban environments provides valuable evidence of how digital
technologies can be used to enhance urban residents’ affinity for nature
and psychological well-being. This understanding can be extended in the
future to other digital platforms, such as metaverses. Clinical Research
Information Service (CRIS) KCT0007657; <http://tinyurl.com/yck7zxp7>.
©Seulki Lee, Chisung Yuh, Yu-Bin Shin, Heon-Jeong Lee, Young-Mee Lee,
Jungsil Lee, Chul-Hyun Cho. Originally published in the Journal of
Medical Internet Research (<https://www.jmir.org>), 22.01.2024.
</td>
</tr>
<tr>
<td style="text-align:left;">
38251473
</td>
<td style="text-align:left;">
The effect of dupilumab on clinical outcomes in patients with COVID-19:
A meta-analysis.
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
38250849
</td>
<td style="text-align:left;">
Predictors of Breakthrough SARS-CoV-2 Infection after Vaccination.
</td>
<td style="text-align:left;">
The initial two-dose vaccine series and subsequent booster vaccine doses
have been effective in modulating SARS-CoV-2 disease severity and death
but do not completely prevent infection. The correlates of infection
despite vaccination continue to be under investigation. In this
prospective decentralized study (n = 1286) comparing antibody responses
in an older- (≥70 years) to a younger-aged cohort (aged 30-50 years), we
explored the correlates of breakthrough infection in 983 eligible
subjects. Participants self-reported data on initial vaccine series,
subsequent booster doses and COVID-19 infections in an online portal and
provided self-collected dried blood spots for antibody testing by ELISA.
Multivariable survival analysis explored the correlates of breakthrough
infection. An association between higher antibody levels and protection
from breakthrough infection observed during the Delta and Omicron BA.1/2
waves of infection no longer existed during the Omicron BA.4/5 wave. The
older-aged cohort was less likely to have a breakthrough infection at
all time-points. Receipt of an original/Omicron vaccine and the presence
of hybrid immunity were associated with protection of infection during
the later Omicron BA.4/5 and XBB waves. We were unable to determine a
threshold antibody to define protection from infection or to guide
vaccine booster schedules.
</td>
</tr>
<tr>
<td style="text-align:left;">
38250614
</td>
<td style="text-align:left;">
Predictors of later COVID-19 test seeking.
</td>
<td style="text-align:left;">
Delays in COVID-19 testing may increase the risk of secondary household
and community transmission. Little is known about what patient
characteristics and symptom profiles are associated with delays in test
seeking. We conducted a retrospective cohort study of all symptomatic
patients diagnosed with COVID-19 and assessed in a COVID Expansion to
Outpatients (COVIDEO) virtual care program between March 2020 and June
2021. The primary outcome was later test seeking more than 3 days from
symptom onset. Multivariable logistic regression was used to examine
predictors of later testing including patient characteristics and
symptoms (30 individual symptoms or 7 symptom clusters). Of 5,363
COVIDEO patients, 4,607 were eligible and 2,155/4,607 (46.8%) underwent
later testing. Older age was associated with increased odds of late
testing (adjusted odds ratio \[aOR\] 1.007/year; 95% CI 1.00 to 1.01),
as was history of recent travel (aOR 1.4; 95% CI 1.01 to 1.95). Health
care workers had lower odds of late testing (aOR 0.50; 95% CI 0.39 to
0.62). Late testing was associated with symptoms in the
cardiorespiratory (aOR 1.2; 95% CI 1.05, 1.36), gastrointestinal (aOR =
1.2; 95% CI 1.04, 1.4), neurological (aOR 1.1; 95% CI 1.003, 1.3) and
psychiatric (aOR 1.3; 95% CI 1.1, 1.5) symptom clusters. Among
individual symptoms, dyspnea, anosmia, dysgeusia, sputum, and anorexia
were associated with late testing; pharyngitis, myalgia, and headache
were associated with early testing. Certain patient characteristics and
symptoms are associated with later testing, and warrant further efforts
to encourage earlier testing to minimize transmission. © Association of
Medical Microbiology and Infectious Disease Canada (AMMI Canada), 2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38250580
</td>
<td style="text-align:left;">
COVID-19 Pandemic, Physical Distancing Policies, and the Non-Profit
Sector Volunteer Force.
</td>
<td style="text-align:left;">
Although COVID-19-related physical distancing has had large economic
consequences, the impact on volunteerism is unclear. Using volunteer
position postings data from Canada’s largest volunteer center (Volunteer
Toronto) from February 3, 2020, to January 4, 2021, we evaluated the
impact of different levels of physical distancing on average views,
total views, and total number of posts. There was about a 50% decrease
in the total number of posts that was sustained throughout the pandemic.
Although a more restrictive physical distancing policy was generally
associated with fewer views, there was an initial increase in views
during the first lockdown where total views were elevated for the first
4 months of the pandemic. This was driven by interest in
COVID-19-related and remote work postings. This highlights the community
of volunteers may be quite flexible in terms of adapting to new ways of
volunteering, but substantial challenges remain for the continued
operations of many non-profit organizations. © The Author(s) 2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38247439
</td>
<td style="text-align:left;">
Effect of COVID-19 on the prevalence of bystanders performing
cardiopulmonary resuscitation: A systematic review and meta-analysis.
</td>
<td style="text-align:left;">
The importance of bystander cardiopulmonary resuscitation (CPR) during
out-of-hospital cardiac arrests is especially important in the context
of coronavirus disease 2029 (COVID-19) because it can significantly
influence survival outcomes. The objective of this meta-analysis was to
examine the primary outcomes of bystander CPR during the pandemic and
pre-pandemic periods. A search was conducted in the PubMed Central,
Scopus, and EMBASE databases, as well as the Cochrane Central Register
of Controlled Trials database, up to December 10, 2023. In cases where
the value of I² was greater than or equal to 50% or the Q-test indicated
that the p-value was less than or equal to 0.05, the studies were
considered to be heterogeneous. Sensitivity assessment was performed
using the leave-one-out methodology. The study protocol was registered
in PROSPERO with the ID number CRD42023494912. Twenty-five articles were
included in this meta-analysis. Pooled analysis showed that bystander
CPR frequency during the COVID-19 pandemic was 38.8%, compared to 44.8%
for the pre-pandemic period (odds ratio: 1.04; 95% confidence interval:
0.93-1.16; p = 0.48). The article’s conclusions indicate that the
COVID-19 pandemic influenced a reduction in bystander CPR compared to
the pre-pandemic period, but this difference was not statistically
significant. Further research is recommended to understand attitudes,
including the fears of witnesses, before performing CPR on patients with
suspected or confirmed infectious diseases. The study highlights the
importance of bystander intervention in emergency situations and the
impact of a pandemic on public health response behaviors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38246253
</td>
<td style="text-align:left;">
Mental Health Outcomes of Endometriosis Patients during the COVID-19
Pandemic: Impact of Pre-pandemic Central Nervous System Sensitization.
</td>
<td style="text-align:left;">
To correlate pain-related phenotyping for central nervous system
sensitization in endometriosis-associated pain with mental health
outcomes during the COVID-19 pandemic, the prospective Endometriosis and
Pelvic Pain Interdisciplinary Cohort (ClinicalTrials.gov \#NCT02911090)
was linked to the COVID-19 Rapid Evidence Study of a Provincial
Population-Based Cohort for Gender and Sex (RESPPONSE) dataset. The
primary outcomes were depression (PHQ-9) and anxiety (GAD-7) scores
during the pandemic. The explanatory variables of interest were the
Central Sensitization Inventory (CSI) score (0-100) and
endometriosis-associated chronic pain comorbidities/psychological
variables before the pandemic. The explanatory and response variables
were assessed for correlation, followed by multivariable regression
analyses adjusting for PHQ-9 and GAD-7 scores pre-pandemic as well as
age, body mass index, and parity. A higher CSI score and a greater
number of chronic pain comorbidities before the pandemic were both
positively correlated with PHQ-9 and GAD-7 scores during the pandemic.
These associations remained significant in adjusted analyses. Increasing
the CSI score by 10 was associated with an increase in pandemic PHQ-9 by
.74 points (P &lt; .0001) and GAD-7 by .73 points (P &lt; .0001) on
average. Each additional chronic pain comorbidity/psychological variable
was associated with an increase in pandemic PHQ-9 by an average of .63
points (P = .0004) and GAD-7 by .53 points (P = .0002). Endometriosis
patients with a history of central sensitization before the pandemic had
worse mental health outcomes during the COVID-19 pandemic. As a risk
factor for mental health symptoms in the face of major stressors,
clinical proxies for central sensitization can be used to identify
endometriosis patients who may need additional support. PERSPECTIVE:
This article adds to the growing literature of the clinical importance
of central sensitization in endometriosis patients, who had more
symptoms of depression and anxiety during the COVID-19 pandemic.
Clinical features of central sensitization may help clinicians identify
endometriosis patients needing additional support when facing major
stressors. Copyright © 2024 United States Association for the Study of
Pain, Inc. Published by Elsevier Inc. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38243403
</td>
<td style="text-align:left;">
Geriatric care physicians’ perspectives on providing virtual care: a
reflexive thematic synthesis of their online survey responses from
Ontario, Canada.
</td>
<td style="text-align:left;">
During the COVID-19 pandemic, telemedicine was widely implemented to
minimise viral spread. However, its use in the older adult patient
population was not well understood. To understand the perspectives of
geriatric care providers on using telemedicine with older adults through
telephone, videoconferencing and eConsults. Qualitative online survey
study. We recruited geriatric care physicians, defined as those
certified in Geriatric Medicine, Care of the Elderly (family physicians
with enhanced skills training) or who were the most responsible
physician in a long-term care home, in Ontario, Canada between 22
December 2020 and 30 April 2021. We collected participants’ perspectives
on using telemedicine with older adults in their practice using an
online survey. Two researchers jointly analysed free-text responses
using the 6-phase reflexive thematic analysis. We recruited 29
participants. Participants identified difficulty using technology,
patient sensory impairment, lack of hospital support and pre-existing
high patient volumes as barriers against using telemedicine, whereas the
presence of a caregiver and administrative support were facilitators.
Perceived benefits of telemedicine included improved time efficiency,
reduced travel, and provision of visual information through
videoconferencing. Ultimately, participants felt telemedicine served
various purposes in geriatric care, including improving accessibility of
care, providing follow-up and obtaining collateral history. Main
limitations are the absence of, or incomplete physical exams and
cognitive testing. Geriatric care physicians identify a role for virtual
care in their practice but acknowledge its limitations. Further work is
required to ensure equitable access to virtual care for older adults. ©
The Author(s) 2024. Published by Oxford University Press on behalf of
the British Geriatrics Society. All rights reserved. For permissions,
please email: <journals.permissions@oup.com>.
</td>
</tr>
<tr>
<td style="text-align:left;">
38243267
</td>
<td style="text-align:left;">
Substance use care innovations during COVID-19: barriers and
facilitators to the provision of safer supply at a toronto COVID-19
isolation and recovery site.
</td>
<td style="text-align:left;">
Early in the COVID-19 pandemic, there was an urgent need to establish
isolation spaces for people experiencing homelessness who were exposed
to or had COVID-19. In response, community agencies and the City of
Toronto opened COVID-19 isolation and recovery sites (CIRS) in March
2020. We sought to examine the provision of comprehensive substance use
services offered to clients on-site to facilitate isolation,
particularly the uptake of safer supply prescribing (prescription of
pharmaceutical opioids and/or stimulants) as part of a spectrum of
comprehensive harm reduction and addiction treatment interventions. We
conducted in-depth, semi-structured interviews with 25 clients and 25
staff (including peer, harm reduction, nursing and medical team members)
from the CIRS in April-July 2021. Iterative and thematic analytic
methods were used to identify key themes that emerged in the interview
discussions. At the time of implementation of the CIRS, the provision of
a safer supply of opioids and stimulants was a novel and somewhat
controversial practice. Prescribed safer supply was integrated to
address the high risk of overdose among clients needing to isolate due
to COVID-19. The impact of responding to on-site overdoses and presence
of harm reduction and peer teams helped clinical staff overcome
hesitation to prescribing safer supply. Site-specific clinical guidance
and substance use specialist consults were crucial tools in building
capacity to provide safer supply. Staff members had varied perspectives
on what constitutes ‘evidence-based’ practice in a rapidly changing,
crisis situation. The urgency involved in intervening during a crisis
enabled the adoption of prescribed safer supply, meeting the needs of
people who use substances and assisting them to complete isolation
periods, while also expanding what constitutes acceptable goals in the
care of people who use drugs to include harm reduction approaches. ©
2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38242974
</td>
<td style="text-align:left;">
Public support for more stringent vaccine policies increases with
vaccine effectiveness.
</td>
<td style="text-align:left;">
Under what conditions do citizens support coercive public policies?
Although recent research suggests that people prefer policies that
preserve freedom of choice, such as behavioural nudges, many citizens
accepted stringent policy interventions like fines and mandates to
promote vaccination during the COVID-19 pandemic-a pattern that may be
linked to the unusually high effectiveness of COVID-19 vaccines. We
conducted a large online survey experiment (N = 42,417) in the Group of
Seven (G-7) countries investigating the relationship between a policy’s
effectiveness and public support for stringent policies. Our results
indicate that public support for stringent vaccination policies
increases as vaccine effectiveness increases, but at a modest scale.
This relationship flattens at higher levels of vaccine effectiveness.
These results suggest that intervention effectiveness can be a
significant predictor of support for coercive policies but only up to
some threshold of effectiveness. © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38241804
</td>
<td style="text-align:left;">
Child abuse and neglect during the COVID-19 pandemic: An umbrella
review.
</td>
<td style="text-align:left;">
During the COVID-19 pandemic, multiple child health experts postulated
that the stay-at-home orders would negatively impact child abuse and
neglect. We aimed to examine the impact of the COVID-19 pandemic on
child abuse and neglect in children ages 18 and under; and review author
recommendations for future emergency lockdown procedures. We completed a
systematic search of articles across five databases. Review-level
studies were included if they examined any abuse or neglect related
outcomes in children and youth (e.g., injuries, case openings), and were
published in English. We completed quality appraisals of each included
article using the Health Evidence™ tool. We categorized the findings by
data source including administrative and survey data, or other data
sources. We also narratively summarized reported recommendations. In
total, 11 reviews were included. Two reviews were of strong quality, 7
moderate, and 2 were weak. Overall, studies within reviews that reported
from administrative data sources demonstrated decreased child abuse and
neglect outcomes compared to before the pandemic. Studies using
cross-sectional data demonstrated increases. Reviews with mixed results
often reported increases in emotional, neglect and psychological abuse
cases and decreases physical and sexual abuse cases. This study found
consistent results across reviews; depending on the data source and
study design, child abuse and neglect outcomes either increased or
decreased during the COVID-19 pandemic. Future work should enhance data
collection methods for surveillance and intervention of child abuse and
neglect during public health emergencies when traditional mechanisms are
limited, with an increased focus on the rigor of reporting. Copyright ©
2024. Published by Elsevier Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38241450
</td>
<td style="text-align:left;">
COVID-19 Pivoted Virtual Skills Teaching Model: Project ECHO Ontario
Skin and Wound Care Boot Camp.
</td>
<td style="text-align:left;">
To describe a virtual, competency-based skin and wound care (SWC) skills
training model. The ECHO (Extension for Community Healthcare Outcomes)
Ontario SWC pivoted from an in-person boot camp to a virtual format
because of the COVID-19 pandemic. An outcome-based program evaluation
was conducted. Participants first watched guided commentary and videos
of experts performing in nine SWC multiskills videos, then practiced and
video-recorded themselves performing those skills; these recordings were
assessed by facilitators. Data were collected using pre-post surveys and
rubric-based assessments. Descriptive statistics and thematic analysis
were applied to data analysis. Fifty-five healthcare professionals
participated in the virtual boot camp, measured by the submission of at
least one video. A total of 216 videos were submitted and 215 assessment
rubrics were completed. Twenty-nine participants completed the pre-boot
camp survey (53% response rate) and 26 responded to the post-boot camp
survey (47% response rate). The strengths of the boot camp included the
applicability of virtual learning to clinical settings, boot camp
supplies, tool kits, and teaching strategies. The analysis of survey
responses indicated that average proficiency scores were greater than
80% for three videos, 50% to 70% for three of the videos, and less than
50% for three of the videos. Participants received lower scores in local
wound care and hand washing points of contact. The barriers of the boot
camp included technical issues, time, level of knowledge required at
times, and lack of equipment and access to interprofessional teams. This
virtual ECHO SWC model expanded access to practical skills acquisition.
The professional development model presented here is generalizable to
other healthcare domains. Copyright © 2024 Wolters Kluwer Health,
Inc. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38241269
</td>
<td style="text-align:left;">
Enhancing Minds in Motion® as a virtual program delivery model for
people living with dementia and their care partners.
</td>
<td style="text-align:left;">
The Alzheimer Society of Ontario’s Minds in Motion (MiM) program
improves physical function and well-being of people living with dementia
(PLWD) and their care partners (CP) (Regan et al., 2019). With the
COVID-19 pandemic, there was an urgent need to transition to a virtual
MiM that was similarly safe and effective. The purpose of this mixed
methods study is to describe the standardized, virtual MiM and evaluate
its acceptability, and impact on quality of life, and physical and
cognitive activity of participants. Survey of ad hoc virtual MiM
practices and a literature review informed the design of the
standardized MiM program: 8 weeks of weekly 90-minute sessions that
included 45-minutes of physical activity and 45-minutes of cognitive
stimulation in each session. Participants completed a standardized,
virtual MiM at one of 6 participating Alzheimer Societies in Ontario, as
well as assessments of quality of life, physical and cognitive activity,
and program satisfaction pre- and post-program. In all, 111 PLWD and 90
CP participated in the evaluation (average age of 74.6±9.4 years, 61.2%
had a college/university degree or greater, 80.6% were married, 48.6% of
PLWD and 75.6% of CP were women). No adverse events occurred. MiM
participants rated the program highly (average score of 4.5/5). PLWD
reported improved quality of life post-MiM (p = &lt;0.01). Altogether,
participants reported increased physical activity levels (p = &lt;0.01)
and cognitive activity levels (p = &lt;0.01). The virtual MiM program is
acceptable, safe, and effective at improving quality of life, cognitive
and physical activity levels for PLWD, and cognitive and physical
activity levels among CP. Copyright: © 2024 Neudorf et al. This is an
open access article distributed under the terms of the Creative Commons
Attribution License, which permits unrestricted use, distribution, and
reproduction in any medium, provided the original author and source are
credited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38239826
</td>
<td style="text-align:left;">
Post-Viral Pain, Fatigue, and Sleep Disturbance Syndromes: Current
Knowledge and Future Directions.
</td>
<td style="text-align:left;">
Post-viral pain syndrome, also known as post-viral syndrome, is a
complex condition characterized by persistent pain, fatigue,
musculoskeletal pain, neuropathic pain, neurocognitive difficulties, and
sleep disturbances that can occur after an individual has recovered from
a viral infection. This narrative review provides a summary of the
sequelae of post-viral syndromes, viral agents that cause it, and the
pathophysiology, treatment, and future considerations for research and
targeted therapies. Medline, PubMed, and Embase databases were used to
search for studies on viruses associated with post-viral syndrome. Much
remains unknown regarding the pathophysiology of post-viral syndromes,
and few studies have provided a comprehensive summary of the condition,
agents that cause it, and successful treatment modalities. With the
COVID-19 pandemic continuing to affect millions of people worldwide, the
need for an understanding of the etiology of post-viral illness and how
to help individuals cope with the sequalae is paramount. © 2024 The
Author(s). Published with license by Taylor &amp; Francis Group, LLC.
</td>
</tr>
<tr>
<td style="text-align:left;">
38238177
</td>
<td style="text-align:left;">
Understanding attitudes and beliefs regarding COVID-19 vaccines among
transitional-aged youth with mental health concerns: a youth-led
qualitative study.
</td>
<td style="text-align:left;">
Transitional-aged youth (16-29 years) with mental health concerns have
experienced a disproportionate burden of the COVID-19 pandemic.
Vaccination is limited in this population; however, determinants of its
vaccine hesitancy are not yet thoroughly characterised. This study aimed
to answer the following research question: What are the beliefs and
attitudes of youth with mental illness about COVID-19 vaccines, and how
do these perspectives affect vaccine acceptance? The study aims to
generate findings to inform the development of vaccine resources
specific to youth with mental health concerns. A qualitative methodology
with a youth engagement focus was used to conduct in-depth
semistructured interviews with transitional-aged youth aged 16-29 years
with one or more self-reported mental health diagnoses or concerns.
Mental health concerns encompassed a wide range of symptoms and
diagnoses, including mood disorders, anxiety disorders,
neurodevelopmental disorders and personality disorders. Participants
were recruited from seven main mental health clinical and support
networks across Canada. Transcripts from 46 youth and 6 family member
interviews were analysed using thematic analysis. Two major themes were
generated: (1) factors affecting trust in COVID-19 vaccines and (2)
mental health influences and safety considerations in vaccine
decision-making. Subthemes included trust in vaccines, trust in
healthcare providers, trust in government and mistreatment towards
racialised populations, and direct and indirect influences of mental
health. Our analysis suggests how lived experiences of mental illness
affected vaccine decision-making and related factors that can be
targeted to increase vaccine uptake. Our findings provide new insights
into vaccine attitudes among youth with mental health concerns, which is
highly relevant to ongoing vaccination efforts for new COVID-19 strains
as well as other transmissible diseases and future pandemics. Next steps
include cocreating youth-specific public health and clinical resources
to encourage vaccination in this population. © Author(s) (or their
employer(s)) 2024. Re-use permitted under CC BY-NC. No commercial
re-use. See rights and permissions. Published by BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38238170
</td>
<td style="text-align:left;">
Evaluating the impact of a SIMPlified LaYered consent process on
recruitment of potential participants to the Staphylococcus aureus
Network Adaptive Platform trial: study protocol for a multicentre
pragmatic nested randomised clinical trial (SIMPLY-SNAP trial).
</td>
<td style="text-align:left;">
Informed consent forms (ICFs) for randomised clinical trials (RCTs) can
be onerous and lengthy. The process has the potential to overwhelm
patients with information, leading them to miss elements of the study
that are critical for an informed decision. Specifically, overly long
and complicated ICFs have the potential to increase barriers to trial
participation for patients with mild cognitive impairment, those who do
not speak English as a first language or among those with lower medical
literacy. In turn, this can influence trial recruitment, completion and
external validity. SIMPLY-SNAP is a pragmatic, multicentre, open-label,
two-arm parallel-group superiority RCT, nested within a larger trial,
the Staphylococcus aureus Network Adaptive Platform (SNAP) trial. We
will randomise potentially eligible participants of the SNAP trial 1:1
to a full-length ICF or a SIMPlified LaYered (SIMPLY) consent process
where basic information is summarised with embedded hyperlinks to
supplemental information and videos. The primary outcome is recruitment
into the SNAP trial. Secondary outcomes include patient understanding of
the clinical trial, patient and research staff satisfaction with the
consent process, and time taken for consent. As an exploratory outcome,
we will also compare measures of diversity (eg, gender, ethnicity),
according to the consent process randomised to. The planned sample size
will be 346 participants. The study has been approved by the ethics
review board (Sunnybrook Health Sciences Research Ethics Board) at sites
in Ontario. We will disseminate study results via the SNAP trial group
and other collaborating clinical trial networks. ClinicalTrials.gov
Registry (NCT06168474; www. gov). © Author(s) (or their employer(s))
2024. Re-use permitted under CC BY-NC. No commercial re-use. See rights
and permissions. Published by BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38237635
</td>
<td style="text-align:left;">
Noninvasive Ventilation Before Intubation and Mortality in Patients
Receiving Extracorporeal Membrane Oxygenation for COVID-19: An Analysis
of the Extracorporeal Life Support Organization Registry.
</td>
<td style="text-align:left;">
Bilevel-positive airway pressure (BiPAP) is a noninvasive respiratory
support modality which reduces effort in patients with respiratory
failure. However, it may increase tidal ventilation and transpulmonary
pressure, potentially aggravating lung injury. We aimed to assess if the
use of BiPAP before intubation was associated with increased mortality
in adult patients with coronavirus disease 2019 (COVID-19) who received
venovenous extracorporeal membrane oxygenation (ECMO). We used the
Extracorporeal Life Support Organization Registry to analyze adult
patients with COVID-19 supported with venovenous ECMO from January 1,
2020, to December 31, 2021. Patients treated with BiPAP were compared
with patients who received other modalities of respiratory support or no
respiratory support. A total of 9,819 patients from 421 centers were
included. A total of 3,882 of them (39.5%) were treated with BiPAP
before endotracheal intubation. Patients supported with BiPAP were
intubated later (4.3 vs. 3.3 days, p &lt; 0.001) and showed higher
unadjusted hospital mortality (51.7% vs. 44.9%, p &lt; 0.001). The use
of BiPAP before intubation and time from hospital admission to
intubation resulted as independently associated with increased hospital
mortality (odds ratio \[OR\], 1.32 \[95% confidence interval {CI},
1.08-1.61\] and 1.03 \[1-1.06\] per day increase). In ECMO patients with
severe acute respiratory failure due to COVID-19, the extended use of
BiPAP before intubation should be regarded as a risk factor for
mortality. Copyright © 2024 The Author(s). Published by Wolters Kluwer
Health, Inc. on behalf of the ASAIO.
</td>
</tr>
<tr>
<td style="text-align:left;">
38236838
</td>
<td style="text-align:left;">
Modelling disease mitigation at mass gatherings: A case study of
COVID-19 at the 2022 FIFA World Cup.
</td>
<td style="text-align:left;">
The 2022 FIFA World Cup was the first major multi-continental sporting
Mass Gathering Event (MGE) of the post COVID-19 era to allow foreign
spectators. Such large-scale MGEs can potentially lead to outbreaks of
infectious disease and contribute to the global dissemination of such
pathogens. Here we adapt previous work and create a generalisable model
framework for assessing the use of disease control strategies at such
events, in terms of reducing infections and hospitalisations. This
framework utilises a combination of meta-populations based on clusters
of people and their vaccination status, Ordinary Differential Equation
integration between fixed time events, and Latin Hypercube sampling. We
use the FIFA 2022 World Cup as a case study for this framework
(modelling each match as independent 7 day MGEs). Pre-travel screenings
of visitors were found to have little effect in reducing COVID-19
infections and hospitalisations. With pre-match screenings of spectators
and match staff being more effective. Rapid Antigen (RA) screenings 0.5
days before match day performed similarly to RT-PCR screenings 1.5 days
before match day. Combinations of pre-travel and pre-match testing led
to improvements. However, a policy of ensuring that all visitors had a
COVID-19 vaccination (second or booster dose) within a few months before
departure proved to be much more efficacious. The State of Qatar
abandoned all COVID-19 related travel testing and vaccination
requirements over the period of the World Cup. Our work suggests that
the State of Qatar may have been correct in abandoning the pre-travel
testing of visitors. However, there was a spike in COVID-19 cases and
hospitalisations within Qatar over the World Cup. Given our findings and
the spike in cases, we suggest a policy requiring visitors to have had a
recent COVID-19 vaccination should have been in place to reduce cases
and hospitalisations. Copyright: © 2024 Grunnill et al. This is an open
access article distributed under the terms of the Creative Commons
Attribution License, which permits unrestricted use, distribution, and
reproduction in any medium, provided the original author and source are
credited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38236689
</td>
<td style="text-align:left;">
Pivoting school health and nutrition programmes during COVID-19 in low-
and middle-income countries: A scoping review.
</td>
<td style="text-align:left;">
Preventive and promotive interventions delivered by schools can support
a healthy lifestyle, positive development, and well-being in children
and adolescents. The coronavirus disease 2019 (COVID-19) pandemic
presented unique challenges to school health and nutrition programmes
due to closures and mobility restrictions. We conducted a scoping review
to examine how school health and nutrition programmes pivoted during the
COVID-19 pandemic, and to provide summative guidance to stakeholders in
strategic immediate and long-term response efforts. We searched MEDLINE,
Embase, PsycINFO, and grey literature sources for primary
(observational, intervention, and programme evaluations) and secondary
(reviews, best practices, and recommendations) studies conducted in low-
and middle-income countries from January 2020 to June 2023. Programmes
that originated in schools, which included children and adolescents
(5-19.9 years) were eligible. We included 23 studies in this review.
They varied in their adaptation strategy and key programmatic focus,
including access to school meals (n = 8), health services, such as
immunisations, eye health, and water, sanitation, and hygiene-related
activities (n = 4), physical activity curriculum and exercise training
(n = 3), mental health counselling and curriculum (n = 3), or were
multi-component in nature (n = 5). While school meals, physical
activity, and mental health programmes were adapted by out-of-school
administration (either in the community, households, or virtually), all
health services were suspended indefinitely. Importantly, there was an
overwhelming lack of quantitative data regarding modified programme
coverage, utilisation, and the impact on children and adolescent health
and nutrition. We found limited evidence of successful adaptation of
school health and nutrition programme implementation during the
pandemic, especially from Asia and Africa. While the adoption of the
World Health Organization health-promoting school global standards and
indicators is necessary at the national and school level, future
research must prioritise the development of a school-based comprehensive
monitoring and evaluation framework to track key indicators related to
both health and nutrition of school-aged children and adolescents.
Copyright © 2024 by the Journal of Global Health. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38236072
</td>
<td style="text-align:left;">
Health Care Professional Distress and Mental Health: A Call to the
Continuing Professional Development Community.
</td>
<td style="text-align:left;">
COVID-19 unleashed a maelstrom of distress on health care professionals.
The pandemic contributed to a host of stressors for workers because of
the need for rapid acquisition of new knowledge and skills to provide
best treatment while simultaneously dealing with personal safety,
limited resources, staffing shortages, and access to care issues.
Concurrently, problems with systemic racial inequality and
discrimination became more apparent secondary to difficulties with
accessing health care for minorities and other marginalized groups.
These problems contributed to many health care professionals
experiencing severe moral injury and burnout as they struggled to uphold
core values and do their jobs professionally. Some left or disengaged.
Others died. As continuing professional development leaders focused on
all health professionals, we must act deliberately to address health
care professionals’ distress and mental health. We must incorporate
wellness and mental health as organizing principles in all we do. We
must adopt a new mental model that recognizes the importance of
learners’ biopsychosocial functioning and commit to learners’ wellness
by developing activities that embrace a biopsychosocial point of view.
As educators and influencers, we must demonstrate that the Institute for
Healthcare Improvement’s fourth aim to improve clinician well-being and
safety (2014) and fifth aim to address health equity and the social
determinants of health (2021) matter. It is crucial that continuing
professional development leaders globally use their resources and
relationships to accomplish this imperative call for action. Copyright ©
2024 The Alliance for Continuing Education in the Health Professions,
the Association for Hospital Medical Education, and the Society for
Academic Continuing Medical Education.
</td>
</tr>
<tr>
<td style="text-align:left;">
38235159
</td>
<td style="text-align:left;">
Impact of the COVID-19 pandemic on the adaptability and resiliency of
school food programs across Canada.
</td>
<td style="text-align:left;">
Following the sudden closure of schools due to the pandemic in 2020,
many school food program (SFP) operators lost their operating venues and
had to innovate to continue distributing meals to children. Our
objective was to assess the impact of the COVID-19 pandemic on the
delivery, adaptability, and resiliency of school food programs across
Canada by conducting a systematic rapid review. Systematic literature
searches identified newspaper articles and social media sources related
to the adaptations and challenges faced by school food programs across
Canada in response to the COVID-19 pandemic. Included sources were
assessed and thematically categorized according to the dimensions of the
Analysis Grid for Environments Linked to Obesity (ANGELO) and Getting To
Equity (GTE) frameworks to identify factors impacting the delivery,
adaptability, and resiliency of school food programs in Canada. School
food programs in Canada made various efforts to meet existing and new
challenges associated with the delivery of these programs to keep
feeding school children, particularly those most vulnerable, during the
pandemic. Distribution of food kits, prepared meals and gift
cards/coupons were successful pathways in ensuring support for food
accessibility to students and their families. Increased collaborations
between community members and organizations/stakeholders to help
maintain food delivery or collectively offer new modes to deliver foods
were most frequently cited as key to facilitating school food
programming. However, maintenance and sustainability related to
operating costs and funding were identified as key challenges to
successful school food programming. Our study highlights the swift and
substantial transformation school food programs,, underwent in response
to the pandemic, driven by the urgent need to ensure that students still
had access to nutritious meals and the importance of policy and resource
support to bolster the adaptability and resiliency of these programs.
Findings on facilitators and challenges to school food programs during
the early months of the COVID-19 pandemic can inform development of
guidelines to design a robust national Canadian school food program and
help make existing programs more sustainable, adaptable, and resilient.
Copyright © 2024 Ahmed, Richardson, Riad, McPherson, Sellen and Malik.
</td>
</tr>
<tr>
<td style="text-align:left;">
38235005
</td>
<td style="text-align:left;">
The rise of India’s global health diplomacy amid COVID-19 pandemic.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has highlighted the importance of global health
diplomacy (GHD), with India emerging as a key player. India’s commitment
to GHD is demonstrated by its active participation in regional and
multilateral projects, pharmaceutical expertise, and large-scale
manufacturing capabilities, which include the production and
distribution of COVID-19 vaccines and essential medicines. India has
supported nations in need through bilateral and multilateral platforms,
providing vaccines to countries experiencing shortages and offering
technical assistance and capacity-building programs to improve
healthcare infrastructure and response capabilities. India’s unique
approach to GHD, rooted in humanitarian diplomacy, emphasized
collaboration and empathy and stressed the well-being of humanity by
embracing the philosophy of “Vasudhaiva Kutumbakam,” which translates to
“the world is one family.” Against this background, this paper’s main
focus is to analyze the rise of India’s GHD amidst the COVID-19 pandemic
and its leadership in addressing various global challenges. India has
demonstrated its commitment to global solidarity by offering medical
supplies, equipment, and expertise to more than 100 countries. India’s
rising global leadership can be attributed to its proactive approach,
humanitarian diplomacy, and significant contributions to global health
initiatives. © 2023 The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38234418
</td>
<td style="text-align:left;">
Influenza outbreak management tabletop exercise for congregate living
settings.
</td>
<td style="text-align:left;">
We conducted a tabletop exercise on influenza outbreak preparedness that
engaged a large group of congregate living settings (CLS), with
improvements in self-reported knowledge and readiness. This proactive
approach to responding to communicable disease threats has potential to
build infection prevention and control capacity beyond COVID-19 in the
CLS sector. © The Author(s) 2024.
</td>
</tr>
<tr>
<td style="text-align:left;">
38234326
</td>
<td style="text-align:left;">
Protocol and statistical analysis plan for the identification and
treatment of hypoxemic respiratory failure and acute respiratory
distress syndrome with protection, paralysis, and proning: A type-1
hybrid stepped-wedge cluster randomised effectiveness-implementation
study.
</td>
<td style="text-align:left;">
To describe a study protocol and statistical analysis plan (SAP) for the
identification and treatment of hypoxemic respiratory failure (HRF) and
acute respiratory distress syndrome (ARDS) with protection, paralysis,
and proning (TheraPPP) study prior to completion of recruitment,
electronic data retrieval, and analysis of any data. TheraPPP is a
stepped-wedge cluster randomised study evaluating a care pathway for HRF
and ARDS patients. This is a type-1 hybrid effectiveness-implementation
study design evaluating both intervention effectiveness and
implementation; however primarily powered for the effectiveness outcome.
Seventeen adult intensive care units (ICUs) across Alberta, Canada. We
estimate a sample size of 18816 mechanically ventilated patients, with
11424 patients preimplementation and 7392 patients postimplementation.
We estimate 2688 sustained ARDS patients within our study cohort. An
evidence-based, stakeholder-informed, multidisciplinary care pathway
called Venting Wisely that standardises diagnosis and treatment of HRF
and ARDS patients. The primary outcome is 28-day ventilator-free days
(VFDs). The primary analysis will compare the mean 28-day VFDs
preimplementation and postimplementation using a mixed-effects linear
regression model. Prespecified subgroups include sex, age, HRF, ARDS,
COVID-19, cardiac surgery, body mass index, height, illness acuity, and
ICU volume. This protocol and SAP are reported using the Standard
Protocol Items: Recommendations for Interventional Trials guidance and
the Guidelines for the Content of Statistical Analysis Plans in Clinical
Trials. The study received ethics approval and was registered
(ClinicalTrials.gov-NCT04744298) prior to patient enrolment. TheraPPP
will evaluate the effectiveness and implementation of an HRF and ARDS
care pathway. © 2023 The Authors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38232982
</td>
<td style="text-align:left;">
Advancing language concordant care: a multimodal medical interpretation
intervention.
</td>
<td style="text-align:left;">
Ensuring language concordant care through medical interpretation
services (MIS) allows for accurate information sharing and positive
healthcare experiences. The COVID-19 pandemic led to a regional halt of
in-person interpreters, leaving only digital MIS options, such as phone
and video. Due to longstanding institutional practices, and lack of
accessibility and awareness of these options, digital MIS remained
underused. A Multimodal Medical Interpretation Intervention (MMII) was
developed and piloted to increase digital MIS usage by 25% over an
18-month intervention period for patients with limited English
proficiency. Applying quality improvement methodology, an intervention
comprised digital MIS technology and education was trialled for 18
months. To assess intervention impact, the number of digital MIS minutes
was measured monthly and compared before and after implementation. A
questionnaire was developed and administered to determine healthcare
providers’ awareness, technology accessibility and perception of MIS
integration in the clinical workflow. Digital MIS was used consistently
from the beginning of the COVID-19 pandemic (March 2020) and over the
subsequent 18 months. The total number of minutes of MIS use per month
increased by 44% following implementation of our intervention.
Healthcare providers indicated that digital MIS was vital in
facilitating transparent communication with patients, and the MMII
ensured awareness of and accessibility to the various MIS modalities.
Implementation of the MMII allowed for an increase in digital MIS use in
a hospital setting. Providing digital MIS access, education and training
is a means to advance patient-centred and equitable care by improving
accuracy of clinical assessments and communication. © Author(s) (or
their employer(s)) 2024. Re-use permitted under CC BY-NC. No commercial
re-use. See rights and permissions. Published by BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38231468
</td>
<td style="text-align:left;">
“I’m still not over feeling so isolated”: Métis women, Two-Spirit, and
gender-diverse people’s experiences of the COVID-19 pandemic.
</td>
<td style="text-align:left;">
The aim of this study was to explore and learn from the experiences of
Métis women, Two-Spirit, and gender-diverse people accessing health and
social services in Victoria, British Columbia, during the COVID-19
pandemic. This paper comes from a larger study exploring Métis women,
Two-Spirit, and gender-diverse people’s experiences accessing health and
social services in Victoria. Using a by-and-for Métis approach that
employed a conversational interview method, we conducted interviews with
Métis women, Two-Spirit, and gender-diverse people who lived in and/or
accessed services in Victoria in December 2020 and January 2021. This
paper focuses specifically on data addressing how COVID-19 impacted
these participants. A total of 24 Métis women, Two-Spirit, and
gender-diverse people participated in the study. Overall, three themes
specific to COVID-19 were identified. First, participants described the
detrimental impacts of COVID-19 on their ability to connect with their
Métis community and practice their culture, as well as their overall
feelings of isolation. Second, participants highlighted some of the ways
that COVID-19 has exacerbated existing barriers to culturally safe
healthcare. Last, participants spoke about the mixed economic impacts
that COVID-19 has had for them, sharing insight into the ways in which
gender, in particular, has shaped their financial instability. Improving
access to culturally safe health and social services by incorporating
the experiences and expertise of Métis women, Two-Spirit, and
gender-diverse people is crucial to mitigating the disproportional
negative impacts of the pandemic and improving overall health outcomes
within Métis communities across Canada. © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38230447
</td>
<td style="text-align:left;">
Biopsychosocial risk factors for intimate partner violence perpetration
and victimization in adolescents and adults reported after the COVID-19
pandemic onset: a scoping review protocol.
</td>
<td style="text-align:left;">
This scoping review aims to provide a comprehensive summary of the
biological, psychological, and sociological risk factors for intimate
partner violence (IPV) victimization and perpetration reported after the
onset of the COVID-19 pandemic. IPV is a significant public health
concern, characterized by various forms of violence inflicted by
intimate partners. The onset of the COVID-19 pandemic significantly
increased the global prevalence of IPV. While prior research has
identified factors linked to IPV, the risk factors reported in the
literature during this period have not been systematically mapped.
Additionally, the similarities and differences in risk factors between
perpetration and victimization have not been well delineated. This
review will focus on individuals aged 12 years or older involved in
dyadic romantic relationships. Primary studies and systematic reviews
published from the year 2020 will be included. Full-text papers,
preprints, theses, and dissertations published in English will be
included. Studies focusing on factors unrelated to IPV risk will be
excluded. Non-systematic reviews, opinion pieces, and protocols will
also be excluded. Following the JBI methodology for scoping reviews,
systematic searches will be conducted for both peer-reviewed and gray
literature. Independent reviewers will screen records, select eligible
studies, and extract data using a standardized form. Key risk factors
will be mapped to explore their interplay. <https://osf.io/c2hkm>.
Copyright © 2024 JBI.
</td>
</tr>
<tr>
<td style="text-align:left;">
38228342
</td>
<td style="text-align:left;">
Acute health care use among children during the first 2.5 years of the
COVID-19 pandemic in Ontario, Canada: a population-based repeated
cross-sectional study.
</td>
<td style="text-align:left;">
The effects of the decline in health care use at the start of the
COVID-19 pandemic on the health of children are unclear. We sought to
estimate changes in rates of severe and potentially preventable health
outcomes among children during the pandemic. We conducted a repeated
cross-sectional study of children aged 0-17 years using linked
population health administrative and disease registry data from January
2017 through August 2022 in Ontario, Canada. We compared observed rates
of emergency department visits and hospital admissions during the
pandemic to predicted rates based on the 3 years preceding the pandemic.
We evaluated outcomes among children and neonates overall, among
children with chronic health conditions and among children with specific
diseases sensitive to delays in care. All acute care use for children
decreased immediately at the onset of the pandemic, reaching its lowest
rate in April 2020 for emergency department visits (adjusted relative
rate \[RR\] 0.28, 95% confidence interval \[CI\] 0.28-0.29) and hospital
admissions (adjusted RR 0.43, 95% CI 0.42-0.44). These decreases were
sustained until September 2021 and May 2022, respectively. During the
pandemic overall, rates of all-cause mortality, admissions for
ambulatory care-sensitive conditions, newborn readmissions or emergency
department visits or hospital admissions among children with chronic
health conditions did not exceed predicted rates. However, after
declining significantly between March and May 2020, new presentations of
diabetes mellitus increased significantly during most of 2021 (peak
adjusted RR 1.49, 95% CI 1.28-1.74 in July 2021) and much of 2022. Among
these children, presentations for diabetic ketoacidosis were
significantly higher than expected during the pandemic overall (adjusted
RR 1.14, 95% CI 1.00-1.30). We observed similar time trends for new
presentations of cancer, but we observed no excess presentations of
severe cancer overall (adjusted RR 0.91, 95% CI 0.62-1.34). In the first
30 months of the pandemic, disruptions to care were associated with
important delays in new diagnoses of diabetes but not with other acute
presentations of select preventable conditions or with mortality.
Mitigation strategies in future pandemics or other health system
disruptions should include education campaigns around important symptoms
in children that require medical attention. © 2024 CMA Impact Inc. or
its licensors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38228031
</td>
<td style="text-align:left;">
A global comparative analysis of the criteria and equity considerations
included in eighty-six national COVID-19 plans.
</td>
<td style="text-align:left;">
Systematic priority setting (PS), based on explicit criteria, is thought
to improve the quality and consistency of the PS decisions. Among the PS
criteria, there is increased focus on the importance of equity
considerations and vulnerable populations. This paper discusses the PS
criteria that were included in the national COVID-19 pandemic plans,
with specific focus on equity and on the vulnerable populations
considered. Secondary synthesis of data, from a global comparative study
that examined the degree to which the COVID-19 plans included PS, was
conducted. Only 32 % of the plans identified explicit criteria. Severity
of the disease and/or disease burden were the commonly mentioned
criteria. With regards to equity considerations and prioritizing
vulnerable populations, 22 countries identified people with
co-morbidities others mentioned children, women etc. Low social-economic
status and internally displaced population were not identified in any of
the reviewed national plans. The limited inclusion of explicit criteria
and equity considerations highlight a need for policy makers, in all
contexts, to consider instituting and equipping PS institutions who can
engage diverse stakeholders in identifying the relevant PS criteria
during the post pandemic period. While vulnerability will vary with the
type of health emergency- awareness of this and having mechanisms for
identifying and prioritizing the most vulnerable will support equitable
pandemic responses. Copyright © 2023. Published by Elsevier B.V.
</td>
</tr>
<tr>
<td style="text-align:left;">
38227180
</td>
<td style="text-align:left;">
Excess risk of COVID-19 infection and mental distress in healthcare
workers during successive pandemic waves: Analysis of matched cohorts of
healthcare workers and community referents in Alberta, Canada.
</td>
<td style="text-align:left;">
To investigate changes in risk of infection and mental distress in
healthcare workers (HCWs) relative to the community as the COVID-19
pandemic progressed. HCWs in Alberta, Canada, recruited to an
interprovincial cohort, were asked consent to link to Alberta’s
administrative health database (AHDB) and to information on COVID-19
immunization and polymerase chain reaction (PCR) testing. Those
consenting were matched to records of up to five community referents
(CRs). Physician diagnoses of COVID-19 were identified in the AHDB from
the start of the pandemic to 31 March 2022. Physician consultations for
mental health (MH) conditions (anxiety, stress/adjustment reaction,
depressive) were identified from 1 April 2017 to 31 March 2022. Risks
for HCW relative to CR were estimated by fitting wave-specific hazard
ratios. Eighty percent (3050/3812) of HCWs consented to be linked to the
AHDB; 97% (2959/3050) were matched to 14,546 CRs. HCWs were at greater
risk of COVID-19 overall, with first infection defined from either PCR
tests (OR=1.96, 95%CI 1.76-2.17) or physician records (OR=1.33, 95%CI
1.21-1.45). They were also at increased risk for each of the three MH
diagnoses. In analyses adjusted for confounding, risk of COVID-19
infection was higher than for CRs early in the pandemic and during the
fifth (Omicron) wave. The excess risk of stress/adjustment reactions
(OR=1.52, 95%CI 1.35-1.71) and depressive conditions (OR=1.39, 95%CI
1.24-1.55) increased with successive waves during the epidemic, peaking
in the fourth wave. HCWs were at increased risk of both COVID-19 and
mental ill-health with the excess risk continuing late in the pandemic.
© 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38226311
</td>
<td style="text-align:left;">
Protection, freedom, stigma: a critical discourse analysis of face masks
in the first wave of the COVID-19 pandemic and implications for medical
education.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has spotlighted the face mask as an intricate
object constructed through the uptake of varied and sometimes competing
discourses. We investigated how the concept of face mask was
discursively deployed during the first phase of the COVID-19 pandemic.
By examining the different discourses surrounding the use of face masks
in public domain texts, we comment on important educational
opportunities for medical education. We applied critical discourse
methodology to look for key phrases related to face masks that can be
linked to specific socio-economic and educational practices. We created
an archive of 171 English and Mandarin texts spanning the period of
February to July 2020 to explore how discourses in Canada related to
discourses of mask use in China, where the pandemic was first observed.
We analyzed how the uptake of discourses related to masks was
rationalized during the first phase of the pandemic and identified
practices/processes that were made possible. While the face mask was
initially constructed as personal protective equipment, it quickly
became a discursive object for rights and freedoms, an icon for personal
expression of political views and social identities, and a symbol of
stigma that reinforced illness, deviance, anonymity, or fear. Discourses
related to face masks have been observed in public and institutional
responses to the pandemic in the first wave. Finding from this research
reinforce the need for medical schools to incorporate a broader
socio-political appreciation of the role of masks in healthcare when
training for pandemic responses. © 2023 Huo, Martimianakis; licensee
Synergies Partners.
</td>
</tr>
<tr>
<td style="text-align:left;">
38226308
</td>
<td style="text-align:left;">
Investigating the experiences of medical students quarantined due to
COVID-19 exposure.
</td>
<td style="text-align:left;">
The COVID-19 pandemic profoundly impacted medical education systems
worldwide. Between March 2020 and December 2021, 111 MD students at the
University of Toronto completed two-week quarantines due to hospital or
community exposures and experienced disrupted clinical instruction. We
explored the experiences, barriers, and supports of these quarantined
medical students to identify program development opportunities and
improve student supports. We used a qualitative descriptive approach to
explore experiences of clerkship students quarantined due to COVID-19
exposure. Methods included an online survey with open-ended questions
and an audio-recorded interview. We analysed the demographic survey
responses using descriptive statistics. Subsequently, we conducted
descriptive thematic analysis of the narrative survey responses and
transcribed interview recordings. Concerns reported in surveys (n = 23,
response rate 20.7%) and interviews (n = 5) included themes of illness
uncertainty, racial tensions, confidentiality of COVID-19 status,
unclear academic expectations, and financial burden. Supports included
friends, family, and MD program administration. Recommendations related
to communication, administration, equity considerations, supports,
confidentiality/privacy, and academics. Supporting student wellbeing and
learning is at the core of medical training. Enhanced understanding of
health profession trainee needs during COVID can improve institutional
supportive responses to students routinely and during times of crisis. ©
2023 Han, Kim, Rojas, Nyhof-Young; licensee Synergies Partners.
</td>
</tr>
<tr>
<td style="text-align:left;">
38225625
</td>
<td style="text-align:left;">
Factors associated with SARS-CoV-2 infection in unvaccinated children
and young adults.
</td>
<td style="text-align:left;">
Pediatric COVID-19 cases are often mild or asymptomatic, which has
complicated estimations of disease burden using existing testing
practices. We aimed to determine the age-specific population
seropositivity and risk factors of SARS-CoV-2 seropositivity among
children and young adults during the pandemic in British Columbia (BC).
We conducted two cross-sectional serosurveys: phase 1 enrolled children
and adults &lt; 25 years between November 2020-May 2021 and phase 2
enrolled children &lt; 10 years between June 2021-May 2022 in BC.
Participants completed electronic surveys and self-collected
finger-prick dried blood spot (DBS) samples. Samples were tested for
immunoglobulin G antibodies against ancestral spike protein (S).
Descriptive statistics from survey data were reported and two
multivariable analyses were conducted to evaluate factors associated
with seropositivity. A total of 2864 participants were enrolled, of
which 95/2167 (4.4%) participants were S-seropositive in phase 1 across
all ages, and 61/697 (8.8%) unvaccinated children aged under ten years
were S-seropositive in phase 2. Overall, South Asian participants had a
higher seropositivity than other ethnicities (13.5% vs. 5.2%). Of 156
seropositive participants in both phases, 120 had no prior positive
SARS-CoV-2 test. Young infants and young adults had the highest reported
seropositivity rates (7.0% and 7.2% respectively vs. 3.0-5.6% across
other age groups). SARS-CoV-2 seropositivity among unvaccinated children
and young adults was low in May 2022, and South Asians were
disproportionately infected. This work demonstrates the need for
improved diagnostics and reporting strategies that account for
age-specific differences in pandemic dynamics and acceptability of
testing mechanisms. © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38224986
</td>
<td style="text-align:left;">
Role of Creation of Plain Language Summaries to Disseminate COVID-19
Research Findings to Patients With Rheumatic Diseases.
</td>
<td style="text-align:left;">
Dissemination of accurate scientific and medical information was
critical during the coronavirus disease 2019 (COVID-19) pandemic,
particularly during the early phases when little was known about the
disease. Unfortunately, poor science literacy is a serious public health
problem, contributing to widespread difficulties in accurately
interpreting information and scientific data during the ongoing COVID-19
infodemic.1,2.
</td>
</tr>
<tr>
<td style="text-align:left;">
38222304
</td>
<td style="text-align:left;">
COVID-19 pandemic impact on the potential exacerbation of screening
mammography disparities: A population-based study in Ontario, Canada.
</td>
<td style="text-align:left;">
Strategies to ramp up breast cancer screening after COVID-19 require
data on the influence of the pandemic on groups of women with
historically low screening uptake. Using data from Ontario, Canada, our
objectives were to 1) quantify the overall pandemic impact on weekly
bilateral screening mammography rates (per 100,000) of average-risk
women aged 50-74 and 2) examine if COVID-19 has shifted any mammography
inequalities according to age, immigration status, rurality, and access
to material resources. Using a segmented negative binomial regression
model, we estimated the mean change in rate at the start of the pandemic
(the week of March 15, 2020) and changes in weekly trend of rates during
the pandemic period (March 15-December 26, 2020) compared to the
pre-pandemic period (January 3, 2016-March 14, 2020) for all women and
for each subgroup. A 3-way interaction term (COVID-19*week*subgroup
variable) was added to the model to detect any pandemic impact on
screening disparities. Of the 3,481,283 mammograms, 8.6 % (n = 300,064)
occurred during the pandemic period. Overall, the mean weekly rate
dropped by 93.4 % (95 % CI 91.7 % - 94.8 %) at the beginning of
COVID-19, followed by a weekly increase of 8.4 % (95 % CI 7.4 % - 9.4 %)
until December 26, 2020. The pandemic did not shift any disparities (all
interactions p &gt; 0.05) and that women who were under 60 or over 70,
immigrants, or with a limited access to material resources had
persistently low screening rate in both periods. Interventions should
proactively target these underserved populations with the goals of
reducing advanced-stage breast cancer presentations and mortality. ©
2024 The Authors. Published by Elsevier Inc. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38217482
</td>
<td style="text-align:left;">
Humanitarian-Development Nexus: strengthening health system
preparedness, response and resilience capacities to address COVID-19 in
Sudan-case study of repositioning external assistance model and focus.
</td>
<td style="text-align:left;">
The advent of the COVID-19 pandemic and the establishment of a new
transitional government in Sudan with rejuvenated relations with the
international community paved the way for external assistance to the EU
COVID-19 response project, a project with a pioneering design within the
region. The project sought to operationalize the
humanitarian-development-peace nexus, perceiving the nexus as a
continuum rather than sequential due to the protracted nature of
emergencies in Sudan and their multiplicity and contextual complexity.
It went further into enhancing peace through engaging with conflict and
post-conflict-affected states and communities and empowering local
actors. Learning from this experience, external assistance models to
low- or middle-income countries (LMICs) should apply principles of
flexibility and adaptability, while maintaining trust through
transparency in exchange, to ensure sustainable and responsive action to
domestic needs within changing contexts. Careful selection and diverse
project team skills, early and continuous engagement with stakeholders,
and robust planning, monitoring and evaluation processes were the
project highlights. Yet, the challenges of political turmoil, changing
Ministry of Health leadership, competing priorities and inactive
coordination mechanisms had to be dealt with. While applying such an
approach of a health system lens to health emergencies in LMICs is
thought to be a success factor in this case, more robust technical
guidance to the nexus implementation is crucial and can be best attained
through encouraging further case reports analysing context-specific
practices. © The Author(s) 2024. Published by Oxford University Press in
association with The London School of Hygiene and Tropical Medicine.
</td>
</tr>
<tr>
<td style="text-align:left;">
38205969
</td>
<td style="text-align:left;">
Pivoting Continuing Professional Development During the COVID-19
Pandemic: A Narrative Scoping Review of Adaptations and Innovations.
</td>
<td style="text-align:left;">
Most formal continuing professional development (CPD) opportunities were
offered in person until March 2020 when the COVID-19 pandemic disrupted
traditional structures of CPD offerings. The authors explored the
adaptations and innovations in CPD that were strengthened or newly
created during the first 16 months of the pandemic. The objectives of
the narrative review were to answer the following questions: (1) what
types of adaptations to CPD innovations are described? and (2) what may
shape future innovations in CPD? The following databases were searched:
Medline, Embase, CINAHL, and ERIC to identify the literature published
between March 2020 to July 2021. The authors conducted a comprehensive
search by including all study types that described adaptations and/or
innovations in CPD during the stated pandemic period. Of the 8295
citations retrieved from databases, 191 satisfied the inclusion
criteria. The authors found three categories to describe adaptations to
CPD innovations: (1) creation of new online resources, (2) increased use
of the existing online platforms/software to deliver CPD, and (3) use of
simulation for teaching and learning. Reported advantages and
disadvantages associated with these adaptations included logistical,
interactional, and capacity building elements. The review identified
five potential future CPD innovations: (1) empirical research on the
effectiveness of virtual learning; (2) novel roles and ways of thinking;
(3) learning from other disciplines beyond medicine; (4) formation of a
global perspective; and (5) emerging wellness initiatives. This review
provided an overview of the adaptations and innovations that may shape
the future of CPD beyond the pandemic. Copyright © 2024 The Author(s).
Published by Wolters Kluwer Health, Inc. on behalf of The Alliance for
Continuing Education in the Health Professions, the Association for
Hospital Medical Education, and the Society for Academic Continuing
Medical Education.
</td>
</tr>
<tr>
<td style="text-align:left;">
38204848
</td>
<td style="text-align:left;">
Assessing Primary Care Blood Pressure Documentation for Hypertension
Management During the COVID-19 Pandemic by Patient and Provider Groups.
</td>
<td style="text-align:left;">
Primary care electronic medical record (EMR) data can be used to
identify, manage, and screen hypertension cases. However, this approach
relies on completeness and accessibility of documented blood pressure
(BP) values. With the large switch to virtual care due to the COVID-19
pandemic, we assessed BP documentation in primary care EMRs during the
pandemic, across patient and physician groups. Hypertension-related
visits were identified during the pre-pandemic (January 2017 to February
2020) and pandemic (March 2020 to December 2021) periods from a primary
care EMR database in Ontario, Canada. Clustered logistic regression
models were used to analyze the relationship of physician and patient
characteristics with an outcome variable of documented BP. A chart
review of 3200 hypertension visits without a BP recorded in structured
data fields was conducted to determine if BP was recorded in progress
notes. Pre-pandemic, 75.7% of hypertension-related visits (113,966 of
150,511) had a BP recorded in structured documentation, but this
significantly decreased to 36.4% (26,660 of 73,239) during the pandemic
(odds ratio \[OR\] = 0.18, 95% confidence interval \[CI\]: 0.18-0.19).
For virtual visits, 14.3% (6357 of 44,572) had a documented BP, vs 74.0%
(20,056 of 27,089) for in-person visits. Chart review found that 55.9%
of hypertension visits had no associated BP in structured documentation,
but did have a BP recorded in the progress note. Male providers,
compared to female providers, were less likely to record BPs
pre-pandemic (OR = 0.45, 95% CI: 0.32-0.63) and during the pandemic, for
both virtual visits (OR = 0.48, 95% CI: 0.32-0.71) and in-person visits
(OR = 0.46, 95% CI: 0.33-0.64). BP documented in primary care EMRs
declined during the pandemic, most likely due to high rates of virtual
visits impacting hypertension detection and management. © 2023 The
Authors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38204556
</td>
<td style="text-align:left;">
The “Lightbulb” Sign: A Novel Echocardiographic Finding Using Ultrasound
Enhancing Agent in Fulminant COVID-19-Related Myocarditis.
</td>
<td style="text-align:left;">
We report a case of fulminant COVID-19-related myocarditis requiring
venoarterial extracorporeal membrane oxygenation where the use of an
ultrasound-enhancing agent demonstrated a previously undescribed
echocardiographic finding, the “lightbulb” sign. This sign potentially
represents a new area for the use of an ultrasound enhancing agent in
the echocardiographic diagnosis of myocarditis. © 2023 The Authors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38200686
</td>
<td style="text-align:left;">
Outcomes of SARS-CoV-2 infection in early pregnancy-A systematic review
and meta-analysis.
</td>
<td style="text-align:left;">
Available data on severe acute respiratory syndrome coronavirus 2
(SARS-CoV-2) infection and pregnancy outcomes mostly refer to women
contracting the infection during advanced pregnancy or close to
delivery. There is limited information on the association between
SARS-CoV-2 infection in early pregnancy and outcomes thereof. We aimed
to systematically review the maternal, fetal and neonatal outcomes
following SARS-CoV-2 infection in early pregnancy, defined as &lt;20
weeks of gestation (PROSPERO Registration 2020 CRD42020177673). Searches
were carried out in PubMed, Medline, EMBASE, and Scopus databases from
January 2020 until April 2023 and the WHO database of publications on
coronavirus disease 2019 (COVID-19) from December 2019 to April 2023.
Cohort and case-control studies on COVID-19 occurring in early pregnancy
that reported data on maternal, fetal, and neonatal outcomes were
included. Case reports and studies reporting only exposure to SARS-CoV-2
or not stratifying outcomes based on gestational age were excluded. Data
were extracted in duplicate. Meta-analyses were conducted when
appropriate, using R meta (R version 4.0.5). A total of 18 studies, 12
retrospective and six prospective, were included in this review,
reporting on 10 147 SARS-CoV-2-positive women infected in early
pregnancy, 9533 neonates, and 180 882 SARS-CoV-2 negative women. The
studies had low to moderate risk of bias according to the
Newcastle-Ottawa quality assessment Scale. The studies showed
significant clinical and methodological heterogeneity. A meta-analysis
could be performed only on the outcome miscarriage rate, with a pooled
random effect odds ratio of 1.44 (95% confidence interval 0.96-2.18),
showing no statistical difference in miscarriage in SARS-CoV-2-infected
women. Individual studies reported increased incidences of stillbirth,
low birthweight and preterm birth among neonates born to mothers
affected by COVID-19 in early pregnancy; however, these results were not
consistent among all studies. In this comprehensive systematic review of
available evidence, we identified no statistically significant adverse
association between SARS-CoV-2 infection in early pregnancy (before 20
weeks of gestation) and fetal, neonatal, or maternal outcomes. However,
a 44% increase in miscarriage rate is concerning and further studies of
larger sample size are needed to confirm or refute our findings. © 2024
The Authors. Acta Obstetricia et Gynecologica Scandinavica published by
John Wiley &amp; Sons Ltd on behalf of Nordic Federation of Societies of
Obstetrics and Gynecology (NFOG).
</td>
</tr>
<tr>
<td style="text-align:left;">
38199921
</td>
<td style="text-align:left;">
Prospective monitoring of adverse events following vaccination with
Modified vaccinia Ankara - Bavarian Nordic (MVA-BN) administered to a
Canadian population at risk of Mpox: A Canadian Immunization Research
Network study.
</td>
<td style="text-align:left;">
MVA-BN is an orthopoxvirus vaccine that provides protection against both
smallpox and mpox. In June 2022, Canada launched a publicly-funded
vaccination campaign to offer MVA-BN to at-risk populations including
men who have sex with men (MSM) and sex workers. The safety of MVA-BN
has not been assessed in this context. To address this, the Canadian
National Vaccine Safety Network (CANVAS) conducted prospective safety
surveillance during public health vaccination campaigns in Toronto,
Ontario and in Vancouver, British Columbia. Vaccinated participants
received a survey 7 and 30 days after each MVA-BN dose to elicit adverse
health events. Unvaccinated individuals from a concurrent vaccine safety
project evaluating COVID-19 vaccine safety were used as controls.
Vaccinated and unvaccinated participants that reported a medically
attended visit on their 7-day survey were interviewed. Vaccinated
participants and unvaccinated controls were matched 1:1 based on age
group, gender, sex and provincial study site. Overall, 1,173 vaccinated
participants completed a 7-day survey, of whom 75 % (n = 878) also
completed a 30-day survey. Mild to moderate injection site pain was
reported by 60 % of vaccinated participants. Among vaccinated
participants 8.4 % were HIV positive and when compared to HIV negative
vaccinated individuals, local injection sites were less frequent in
those with HIV (48 % vs 61 %, p = 0.021), but health events preventing
work/school or requiring medical assessment were more frequent (7.1 % vs
3.1 %, p = 0.040). Health events interfering with work/school, or
requiring medical assessment were less common in the vaccinated group
than controls (3.3 % vs. 7.1 %, p &lt; 0.010). No participants were
hospitalized within 7 or 30 days of vaccination. No cases of severe
neurological disease, skin disease, or myocarditis were identified. Our
results demonstrate that the MVA-BN vaccine appears safe when used for
mpox prevention, with a low frequency of severe adverse events and no
hospitalizations observed. Copyright © 2023 Elsevier Ltd. All rights
reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38199634
</td>
<td style="text-align:left;">
Impact of a peer-support programme to improve loneliness and social
isolation due to COVID-19: does adding a secure, user friendly
video-conference solution work better than telephone support alone?
Protocol for a three-arm randomised clinical trial.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has forced the implementation of physical
distancing and self-isolation strategies worldwide. However, these
measures have significant potential to increase social isolation and
loneliness. Among older people, loneliness has increased from 40% to 70%
during COVID-19. Previous research indicates loneliness is strongly
associated with increased mortality. Thus, strategies to mitigate the
unintended consequences of social isolation and loneliness are urgently
needed. Following the Obesity-Related Behavioural Intervention Trials
model for complex behavioural interventions, we describe a protocol for
a three-arm randomised clinical trial to reduce social isolation and
loneliness. A multicentre, outcome assessor blinded, three-arm
randomised controlled trial comparing 12 weeks of: (1) the HOspitals
WoRking in Unity (‘HOW R U?’) weekly volunteer-peer support telephone
intervention; (2) ‘HOW R U?’ deliver using a video-conferencing solution
and (3) a standard care group. The study will follow Consolidated
Standard of Reporting Trials guidelines.We will recruit 24-26 volunteers
who will receive a previously tested half day lay-training session that
emphasises a strength-based approach and safety procedures. We will
recruit 141 participants ≥70 years of age discharged from two
participating emergency departments or referred from hospital family
medicine, geriatric or geriatric psychiatry clinics. Eligible
participants will have probable baseline loneliness (score ≥2 on the de
Jong six-item loneliness scale). We will measure change in loneliness,
social isolation (Lubben social network scale), mood (Geriatric
Depression Score) and quality of life (EQ-5D-5L) at 12-14 weeks
postintervention initiation and again at 24-26 weeks. Approval has been
granted by the participating research ethics boards. Participants
randomised to standard care will be offered their choice of telephone or
video-conferencing interventions after 12 weeks. Results will be
disseminated through journal publications, conference presentations,
social media and through the International Federation of Emergency
Medicine. NCT05228782. © Author(s) (or their employer(s)) 2024. Re-use
permitted under CC BY-NC. No commercial re-use. See rights and
permissions. Published by BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38196240
</td>
<td style="text-align:left;">
Social support buffers the impact of pregnancy stress on perceptions of
parent-infant closeness during the COVID-19 pandemic.
</td>
<td style="text-align:left;">
Pregnant individuals and parents have experienced elevated mental health
problems and stress during COVID-19. Stress during pregnancy can be
harmful to the fetus and detrimental to the parent-child relationship.
However, social support is known to act as a protective factor,
buffering against the adverse effects of stress. The present study
examined whether (1) prenatal stress during COVID-19 was associated with
parent-infant closeness at 6 months postpartum, and (2) social support
moderated the effect of prenatal stress on the parent-infant
relationship. In total, 181 participants completed questionnaires during
pregnancy and at 6 months postpartum. A hierarchical linear regression
analysis was conducted to assess whether social support moderated the
effect of stress during pregnancy on parent-infant closeness at 6 months
postpartum. Results indicated a significant interaction between prenatal
stress and social support on parents’ perceptions of closeness with
their infants at 6 months postpartum (β = .805, p = .029); parents who
experienced high prenatal stress with high social support reported
greater parent-infant closeness, compared to those who reported high
levels of stress and low social support. Findings underscore the
importance of social support in protecting the parent-infant
relationship, particularly in times of high stress, such as during the
COVID-19 pandemic. © 2024 The Authors. Infant Mental Health Journal
published by Wiley Periodicals LLC on behalf of Michigan Association for
Infant Mental Health.
</td>
</tr>
<tr>
<td style="text-align:left;">
38194247
</td>
<td style="text-align:left;">
Digital Interventions for Stress Among Frontline Health Care Workers:
Results From a Pilot Feasibility Cohort Trial.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has challenged the mental health of health care
workers, increasing the rates of stress, moral distress (MD), and moral
injury (MI). Virtual reality (VR) is a useful tool for studying MD and
MI because it can effectively elicit psychophysiological responses, is
customizable, and permits the controlled study of participants in real
time. This study aims to investigate the feasibility of using an
intervention comprising a VR scenario and an educational video to
examine MD among health care workers during the COVID-19 pandemic and to
use our mobile app for longitudinal monitoring of stress, MD, and MI
after the intervention. We recruited 15 participants for a compound
intervention consisting of a VR scenario followed by an educational
video and a repetition of the VR scenario. The scenario portrayed a
morally challenging situation related to a shortage of life-saving
equipment. Physiological signals and scores of the Moral Injury Outcome
Scale (MIOS) and Perceived Stress Scale (PSS) were collected.
Participants underwent a debriefing session to provide their impressions
of the intervention, and content analysis was performed on the sessions.
Participants were also instructed to use a mobile app for 8 weeks after
the intervention to monitor stress, MD, and mental health symptoms. We
conducted Wilcoxon signed rank tests on the PSS and MIOS scores to
investigate whether the VR scenario could induce stress and MD. We also
evaluated user experience and the sense of presence after the
intervention through semi-open-ended feedback and the Igroup Presence
Questionnaire, respectively. Qualitative feedback was summarized and
categorized to offer an experiential perspective. All participants
completed the intervention. Mean pre- and postintervention scores were
respectively 10.4 (SD 9.9) and 13.5 (SD 9.1) for the MIOS and 17.3 (SD
7.5) and 19.1 (SD 8.1) for the PSS. Statistical analyses revealed no
significant pre- to postintervention difference in the MIOS and PSS
scores (P=.11 and P=.22, respectively), suggesting that the experiment
did not acutely induce significant levels of stress or MD. However,
content analysis revealed feelings of guilt, shame, and betrayal, which
relate to the experience of MD. On the basis of the Igroup Presence
Questionnaire results, the VR scenario achieved an above-average degree
of overall presence, spatial presence, and involvement, and slightly
below-average realism. Of the 15 participants, 8 (53%) did not answer
symptom surveys on the mobile app. Our study demonstrated VR to be a
feasible method to simulate morally challenging situations and elicit
genuine responses associated with MD with high acceptability and
tolerability. Future research could better define the efficacy of VR in
examining stress, MD, and MI both acutely and in the longer term. An
improved participant strategy for mobile data capture is needed for
future studies. ClinicalTrails.gov NCT05001542;
<https://clinicaltrials.gov/study/NCT05001542>. RR2-10.2196/32240.
©Caroline W Espinola, Binh Nguyen, Andrei Torres, Walter Sim, Alice
Rueda, Lindsay Beavers, Douglas M Campbell, Hyejung Jung, Wendy Lou,
Bill Kapralos, Elizabeth Peter, Adam Dubrowski, Sridhar Krishnan, Venkat
Bhat. Originally published in JMIR Serious Games
(<https://games.jmir.org>), 09.01.2024.
</td>
</tr>
<tr>
<td style="text-align:left;">
38192563
</td>
<td style="text-align:left;">
A time-course prediction model of global COVID-19 mortality.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has caused over 6 million deaths worldwide and is
a significant cause of mortality. Mortality dynamics vary significantly
by country due to pathogen, host, social and environmental factors, in
addition to vaccination and treatments. However, there is limited data
on the relative contribution of different explanatory variables, which
may explain changes in mortality over time. We, therefore, created a
predictive model using orthogonal machine learning techniques to attempt
to quantify the contribution of static and dynamic variables over time.
A model was created using Partial Least Squares Regression trained on
data from 2020 to rank order the significance and effect size of static
variables on mortality per country. This model enables the prediction of
mortality levels for countries based on demographics alone. Partial
Least Squares Regression was then used to quantify how dynamic
variables, including weather and non-pharmaceutical interventions,
contributed to the overall mortality in 2020. Finally, mortality levels
for the first 60 days of 2021 were predicted using rolling-window
Elastic Net regression. This model allowed prediction of deaths per day
and quantification of the degree of influence of included variables,
accounting for timing of occurrence or implementation. We found that the
most parsimonious model could be reduced to six variables; three
policy-related variables - COVID-19 testing policy, canceled public
events policy, workplace closing policy; in addition to three
environmental variables - maximum temperature per day, minimum
temperature per day, and the dewpoint temperature per day. Country and
population-level static and dynamic variables can be used to predict
COVID-19 mortality, providing an example of how broad temporal data can
inform a preparation and mitigation strategy for both COVID-19 and
future pandemics and assist decision-makers by identifying
population-level contributors, including interventions, that have the
greatest influence in mitigating mortality, and optimizing the health
and safety of populations. Copyright © 2023 Ciaccio, Schneiderman,
Pandey, Fowler, Chiou, Koeller, Hallett, Krueger and Raskin.
</td>
</tr>
<tr>
<td style="text-align:left;">
38192113
</td>
<td style="text-align:left;">
Operational status of mental health, substance use, and problem gambling
services: A system-level snapshot two years into the COVID-19 pandemic.
</td>
<td style="text-align:left;">
The aim of this paper is to provide a system-level snapshot of the
operational status of mental health, substance use, and problem gambling
services 2 years into the pandemic in Ontario, Canada, with a specific
focus on services that target individuals experiencing vulnerable
circumstances (e.g., homelessness and legal issues). We examined data
from 6038 publicly funded community services that provide mental health,
substance use, and problem gambling services in Ontario. We used
descriptive statistics to describe counts and percentages by service
type and specialisation of service delivery. We generated
cross-tabulations to analyse the relationship between the service status
and service type for each target population group. As of March 2022,
38.4% (n = 2321) of services were fully operational, including 36.0% (n
= 1492) of mental health, 44.1% (n = 1037) of substance use, and 23.4%
(n = 78) of problem gambling services. These service disruptions were
also apparent among services tailored to sexual/gender identity
(women/girls, men/boys, 2SLGBTQQIA + individuals), individuals with
legal issues, with acquired brain injury, and those experiencing
homelessness. Accessible community-based mental health, substance use
and problem gambling services are critical supports, particularly for
communities that have historically contended with higher needs and
greater barriers to care relative to the general population. We discuss
the public health implications of the findings for the ongoing pandemic
response and future emergency preparedness planning for community-based
mental health, substance use and problem gambling services. © 2024 The
Authors. The International Journal of Health Planning and Management
published by John Wiley &amp; Sons Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38191390
</td>
<td style="text-align:left;">
A command centre implementation before and during the COVID-19 pandemic
in a community hospital.
</td>
<td style="text-align:left;">
The objective of the study was to assess the effects of high-reliability
system by implementing a command centre (CC) on clinical outcomes in a
community hospital before and during COVID-19 pandemic from the year
2016 to 2021. A descriptive, retrospective study was conducted at an
acute care community hospital. The administrative data included monthly
average admissions, intensive care unit (ICU) admissions, average length
of stay, total ICU length of stay, and in-hospital mortality.
In-hospital acquired events were recorded and defined as one of the
following: cardiac arrest, cerebral infarction, respiratory arrest, or
sepsis after hospital admissions. A subgroup statistical analysis of
patients with in-hospital acquired events was performed. In addition, a
subgroup statistical analysis was performed for the department of
medicine. The rates of in-hospital acquired events and in-hospital
mortality among all admitted patients did not change significantly
throughout the years 2016 to 2021. In the subgroup of patients with
in-hospital acquired events, the in-hospital mortality rate also did not
change during the years of the study, despite the increase in the ICU
admissions during the COVID-19 pandemic.Although the in-hospital
mortality rate did not increase for all admitted patients, the
in-hospital mortality rate increased in the department of medicine.
Implementation of CC and centralized management systems has the
potential to improve quality of care by supporting early identification
and real-time management of patients at risk of harm and clinical
deterioration, including COVID-19 patients. © 2023. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38190356
</td>
<td style="text-align:left;">
Critical research gaps in treating growth faltering in infants under 6
months: A systematic review and meta-analysis.
</td>
<td style="text-align:left;">
In 2020, 149.2 million children worldwide under 5 years suffered from
stunting, and 45.4 million experienced wasting. Many infants are born
already stunted, while others are at high risk for growth faltering
early after birth. Growth faltering is linked to transgenerational
impacts of poverty and marginalization. Few interventions address growth
faltering in infants under 6 months, despite a likely increasing
prevalence due to the negative global economic impacts of the COVID-19
pandemic. Breastfeeding is a critical intervention to alleviate
malnutrition and improve child health outcomes, but rarely receives
adequate attention in growth faltering interventions. A systematic
review and meta-analysis were undertaken to identify and evaluate
interventions addressing growth faltering among infants under 6 months
that employed supplemental milks. The review was carried out following
guidelines from the USA National Academy of Medicine. A total of 10,405
references were identified, and after deduplication 7390 studies were
screened for eligibility. Of these, 227 were assessed for full text
eligibility and relevance. Two randomized controlled trials were
ultimately included, which differed in inclusion criteria and
methodology and had few shared outcomes. Both studies had small sample
sizes, high attrition and high risk of bias. A Bangladeshi study (n =
153) found significantly higher rates of weight gain for F-100 and
diluted F-100 (DF-100) compared with infant formula (IF), while a DRC
trial (n = 146) did not find statistically significant differences in
rate of weight gain for DF-100 compared with IF offered in the context
of broader lactation and relactation support. The meta-analysis of rate
of weight gain showed no statistical difference and some evidence of
moderate heterogeneity. Few interventions address growth faltering among
infants under 6 months. These studies have limited generalizability and
have not comprehensively supported lactation. Greater investment is
necessary to accelerate research that addresses growth faltering
following a new research framework that calls for comprehensive
lactation support. Copyright: © 2024 Tomori et al. This is an open
access article distributed under the terms of the Creative Commons
Attribution License, which permits unrestricted use, distribution, and
reproduction in any medium, provided the original author and source are
credited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38189860
</td>
<td style="text-align:left;">
COVID-19 vaccination intention and vaccine hesitancy among citizens of
the Métis Nation of Ontario.
</td>
<td style="text-align:left;">
The study objective is to measure the influence of psychological
antecedents of vaccination on COVID-19 vaccine intention among citizens
of the Métis Nation of Ontario (MNO). A population-based online survey
was implemented by the MNO when COVID-19 vaccines were approved in
Canada. Questions included vaccine intention, the short version of the
“5C” psychological antecedents of vaccination scale (confidence,
complacency, constraint, calculation, collective responsibility), and
socio-demographics. Census sampling via the MNO Registry was used
achieving a 39% response rate. Descriptive statistics, bivariate
analyses, and multinomial logistic regression models (adjusted for
sociodemographic variables) were used to analyze the survey data. The
majority of MNO citizens (70.2%) planned to be vaccinated. As compared
with vaccine-hesitant individuals, respondents with vaccine intention
were more confident in the safety of COVID-19 vaccines, believed that
COVID-19 is severe, were willing to protect others from getting
COVID-19, and would research the vaccines (Confident OR = 19.4, 95% CI
15.5-24.2; Complacency OR = 6.21, 95% CI 5.38-7.18; Collective
responsibility OR = 9.83, 95% CI 8.24-11.72; Calculation OR = 1.43, 95%
CI 1.28-1.59). Finally, respondents with vaccine intention were less
likely to let everyday stress prevent them from getting COVID-19
vaccines (OR = 0.47, 95% CI 0.42-0.53) compared to vaccine-hesitant
individuals. This research contributes to the knowledge base for Métis
health and supported the MNO’s information sharing and educational
activities during the COVID-19 vaccines rollout. Future research will
examine the relationship between the 5Cs and actual uptake of COVID-19
vaccines among MNO citizens. © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38189594
</td>
<td style="text-align:left;">
Investigating the perceptions and experiences of Canadian dentists on
dental regulatory bodies’ communications and guidelines during the
COVID-19 pandemic.
</td>
<td style="text-align:left;">
Dental regulatory bodies aim to ensure the health and safety of
dentists, dental staff patients and the public. An important
responsibility during a pandemic is to communicate risk and guidelines
for patient care. Limited data exist on the perceptions and experiences
of dentists navigating new guidelines for mitigating risk in dental care
during the pandemic. The objective of this study was to use a
qualitative approach to explore how dentists in Canada experienced and
perceived their regulatory bodies’ communication about COVID-19 risks
and guidelines during the pandemic. Participants were Canadian dentists
(N = 644) recruited through the email roster of nine provincial dental
associations or regulatory bodies. This qualitative analysis was nested
within a prospective longitudinal cohort study in which data were
collected using online questionnaires at regular intervals from August
2020 to November 2021. To address the objective reported in this paper,
a conventional qualitative content analysis method was applied to
responses to three open-ended questions included in the final
questionnaire. Participants encountered challenges and frustrations amid
the COVID-19 pandemic, grappling with diverse regulations and
communications from dental bodies. While some bodies offered helpful
guidance, many participants felt the need for improved communication on
guidelines. Dentists urged for expedited, clearer and more frequent
updates, expressing difficulty in navigating overwhelming information.
Negative views emerged on the vague and unclear communication of
COVID-19 guidelines, contributing to confusion and frustration among
participants. As COVID-19 persists and in planning for future pandemics,
these experiential findings will help guide regulatory bodies in
providing clear, timely and practical guidelines to protect the health
and safety of dentists, dental staff, patients and the public. © 2024
The Authors. Community Dentistry and Oral Epidemiology published by John
Wiley &amp; Sons Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38189240
</td>
<td style="text-align:left;">
Shifting gears: Creating equity informed leaders for effective learning
health systems.
</td>
<td style="text-align:left;">
Leadership is vital to a well-functioning and effective health system.
This importance was underscored during the COVID-19 pandemic. As
disparities in infection and mortality rates became pronounced, greater
calls for equity-informed healthcare emerged. These calls led some
leaders to use the Learning Health System (LHS) approach to quickly
transform research into healthcare practice to mitigate inequities
causing these rates. The LHS is a relatively new framework informed by
many within and outside health systems, supported by decision-makers and
financial arrangements and encouraged by a culture that fosters quick
learning and improvements. Although studies indicate the LHS can enhance
patients’ health outcomes, scarce literature exists on health system
leaders’ use and incorporation of equity into the LHS. This commentary
begins addressing this gap by examining how equity can be incorporated
into LHS activities and discussing ways leaders can ensure equity is
considered and achieved in rapid learning cycles.
</td>
</tr>
<tr>
<td style="text-align:left;">
38187898
</td>
<td style="text-align:left;">
Appraising the decision-making process concerning COVID-19 policy in
postsecondary education in Canada: A critical scoping review protocol.
</td>
<td style="text-align:left;">
Responses to COVID-19 in Canadian postsecondary education have
overhauled usual norms and practices, with policies of unclear rationale
implemented under the pressure of a reported public health emergency. To
critically appraise the decision-making process informing COVID-19
policy in the postsecondary education sector. Our scoping review will
draw from macro and micro theories of public policy, specifically the
critical tradition exemplified by Carol Bacchi’s approach “What is the
problem represented to be” and will be guided by Arksey and O’Malley’s
framework for scoping reviews and the team-based approach of Levan and
colleagues. Data will include diverse and publicly available documents
to capture multiple stakeholders’ perspectives on the phenomenon of
interest and will be retrieved from university newsletters and legal
websites using combinations of search terms adapted to specific data
types. Two reviewers will independently screen, chart, analyze and
synthesize the data. Disagreements will be resolved through full team
discussion. Despite the unprecedented nature of the mass medical
mandates implemented in the postsecondary sector and their dramatic
impact on millions of lives-students, faculty, staff and their families,
friends and communities-the decision-making process leading to them has
not been documented or appraised. By identifying, summarizing and
appraising the evidence, our review should inform practices that can
contribute to effective and equitable public health policies in
postsecondary institutions moving forward. © 2023 the Author(s),
licensee AIMS Press.
</td>
</tr>
<tr>
<td style="text-align:left;">
38185135
</td>
<td style="text-align:left;">
Risk factors for COVID-19-associated pulmonary aspergillosis: a
systematic review and meta-analysis.
</td>
<td style="text-align:left;">
COVID-19-associated pulmonary aspergillosis (CAPA) has been reported to
be an emerging and potentially fatal complication of severe COVID-19.
However, risk factors for CAPA have not been systematically addressed to
date. In this systematic review and meta-analysis to identify factors
associated with CAPA, we comprehensively searched five medical
databases: Ovid MEDLINE; Ovid Embase; the Cochrane Database of
Systematic Reviews; the Cochrane Central Register of Controlled Trials;
and the WHO COVID-19 Database. All case-control and cohort studies in
adults (aged &gt;18 years) that described at least six cases of CAPA and
evaluated any risk factors for CAPA, published from Dec 1, 2019, to July
27, 2023, were screened and assessed for inclusion. Only studies with a
control population of COVID-19-positive individuals without
aspergillosis were included. Two reviewers independently screened search
results and extracted outcome data as summary estimates from eligible
studies. The primary outcome was to identify the factors associated with
CAPA. Meta-analysis was done with random-effects models, with use of the
Mantel-Haenszel method to assess dichotomous outcomes as potential risk
factors, or the inverse variance method to assess continuous variables
for potential association with CAPA. Publication bias was assessed with
funnel plots for factors associated with CAPA. The study is registered
with PROSPERO, CRD42022334405. Of 3561 records identified, 27 articles
were included in the meta-analysis. 6848 patients with COVID-19 were
included, of whom 1324 (19·3%) were diagnosed with CAPA. Diagnosis rates
of CAPA ranged from 2·5% (14 of 566 patients) to 47·2% (58 of 123). We
identified eight risk factors for CAPA. These factors included
pre-existing comorbidities of chronic liver disease (odds ratio \[OR\]
2·70 \[95% CI 1·21-6·04\], p=0·02; I2=53%), haematological malignancies
(OR 2·47 \[1·27-4·83\], p=0·008; I2=50%), chronic obstructive pulmonary
disease (OR 2·00 \[1·42-2·83\], p&lt;0·0001; I2=26%), and
cerebrovascular disease (OR 1·31 \[1·01-1·71\], p=0·05; I2=46%). Use of
invasive mechanical ventilation (OR 2·83; 95% CI 1·88-4·24; p&lt;0·0001;
I2=69%), use of renal replacement therapy (OR 2·26 \[1·76-2·90\],
p&lt;0·0001; I2=14%), treatment of COVID-19 with interleukin-6
inhibitors (OR 2·88 \[1·52-5·43\], p=0·001; I2=89%), and treatment of
COVID-19 with corticosteroids (OR 1·88 \[1·28-2·77\], p=0·001; I2=66%)
were also associated with CAPA. Patients with CAPA were typically older
than those without CAPA (mean age 66·6 years \[SD 3·6\] vs 63·5 years
\[5·3\]; mean difference 2·90 \[1·48-4·33\], p&lt;0·0001; I2=86%). The
duration of mechanical ventilation in patients with CAPA was longer than
in those without CAPA (n=7 studies; mean duration 19·3 days \[8·9\] vs
13·5 days \[6·8\]; mean difference 5·53 days \[1·30-9·77\], p=0·01;
I2=88%). In post-hoc analysis, patients with CAPA had higher all-cause
mortality than those without CAPA (n=20 studies; OR 2·65 \[2·04-3·45\],
p&lt;0·0001; I2=51%). The identified risk factors for CAPA could
eventually be addressed with targeted antifungal prophylaxis in patients
with severe COVID-19. None. Copyright © 2024 Elsevier Ltd. All rights
reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38184709
</td>
<td style="text-align:left;">
Pornography and sexual function in the post-pandemic period: a narrative
review from psychological, psychiatric, and sexological perspectives.
</td>
<td style="text-align:left;">
The COVID-19 pandemic and lockdowns had significant impacts on sexual
functioning and behavior. Partnered sexual activity decreased overall,
while solo sex activities such as masturbation and pornography
consumption increased exponentially. Given the ongoing debate about the
effects of pornography on sexual function, it was prudent to consider
how the increase in porn consumption during the pandemic might have
impacted sexual function in the post-pandemic period. Results indicated
that despite the increased rates of use during lockdowns, there remains
no evidence supporting the relationship between sexual dysfunction and
porn use during and following the pandemic period. On the contrary,
pornography consumption and solo sex activities offered an alternative
to conventional sexual behavior during a highly stressful period and
were found to have positive effects of relieving psychosocial stress
otherwise induced by the pandemic. Specifically, those who maintained an
active sexual life experienced less anxiety and depression, and greater
relational health than those who were not sexually active. It is
important to consider factors including frequency, context, and type of
consumption when analyzing the impact of pornography on sexual function.
While excessive use can have negative effects, moderate use can be a
natural and healthy part of life. © 2024. The Author(s), under exclusive
licence to Springer Nature Limited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38184559
</td>
<td style="text-align:left;">
Healthcare providers’ perspectives on implementing a brief physical
activity and diet intervention within a primary care smoking cessation
program: a qualitative study.
</td>
<td style="text-align:left;">
Post-smoking-cessation weight gain can be a major barrier to quitting
smoking; however, adding behavior change interventions for physical
activity (PA) and diet may adversely affect smoking cessation outcomes.
The “Picking up the PACE (Promoting and Accelerating Change through
Empowerment)” study assessed change in PA, fruit/vegetable consumption,
and smoking cessation by providing a clinical decision support system
for healthcare providers to utilize at the intake appointment, and found
no significant change in PA, fruits/vegetable consumption, or smoking
cessation. The objective of this qualitative study was to explore the
factors affecting the implementation of the intervention and
contextualize the quantitative results. Twenty-five semi-structured
interviews were conducted with healthcare providers, using questions
based on the National Implementation Research Network’s Hexagon Tool.
The data were analyzed using the framework’s standard analysis approach.
Most healthcare providers reported a need to address PA and
fruit/vegetable consumption in patients trying to quit smoking, and
several acknowledged that the intervention was a good fit since exercise
and diet could improve smoking cessation outcomes. However, many
healthcare providers mentioned the need to explain the fit to the
patients. Social determinants of health (e.g., low income, food
insecurity) were brought up as barriers to the implementation of the
intervention by a majority of healthcare providers. Most healthcare
providers recognized training as a facilitator to the implementation,
but time was mentioned as a barrier by many of healthcare providers.
Majority of healthcare providers mentioned allied health professionals
(e.g., dieticians, physiotherapists) supported the implementation of the
PACE intervention. However, most healthcare providers reported a need
for individualized approach and adaptation of the intervention based on
the patients’ needs when implementing the intervention. The COVID-19
pandemic was found to impact the implementation of the PACE intervention
based on the Hexagon Tool indicators. There appears to be a need to
utilize a flexible approach when addressing PA and fruit/vegetable
consumption within a smoking cessation program, based on the context of
clinic, the patients’ it is serving, and their life circumstances.
Healthcare providers need support and external resources to implement
this particular intervention. Clinicaltrials.gov. NCT04223336. 7 January
2020 Retrospectively registered. URL OF TRIAL REGISTRY RECORD:
<https://classic>. gov/ct2/show/NCT04223336 . © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38180869
</td>
<td style="text-align:left;">
Novel drop-in web-based mindfulness sessions (Pause-4-Providers) to
enhance well-being amongst healthcare workers during the COVID-19
Pandemic: a descriptive and qualitative study.
</td>
<td style="text-align:left;">
The COVID-19 Pandemic exerted extraordinary pressures on healthcare
workers, imperiling their well-being and mental health. In response to
the urgent demand to provide barrier free support for the healthcare
workforce, Pause-4-Providers implemented 30-minute live web-based
drop-in mindfulness sessions for healthcare workers. The objective of
the study was to evaluate the utilization, feasibility, satisfaction and
acceptability of a novel web-based live drop-in, mindfulness program
aimed at enhancing the well-being of healthcare workers during the
COVID-19 Pandemic. Accrual for the study was ongoing throughout the
first three pandemic waves, and attendees of one or more sessions were
invited to participate. The evaluation framework included: 1.
Descriptive characteristics including participant demographics,
Resilience at Work and Single Item Burnout scores; 2. Feedback
questionnaires on reasons attended, benefits and satisfaction; 3.
Qualitative interviews to further understand participant experience,
satisfaction, benefits, enablers, and barriers; and 4. The number of
participants at each session was summarized by pandemic wave. We
collected descriptive statistics from 50 consenting healthcare workers.
Approximately half had attended more than one session (48%, n=24). Study
participants were predominantly female (80%, n=40), and comprised of
physicians (34%, n=17), nurses (18%, n=9), and other healthcare workers
(48%, n=24), that were largely from Ontario (82%, n=41). 52% of
attendees endorsed feeling burned out (n=26). The highest attendance was
in May 2020 and January 2021, corresponding to the first and second
pandemic waves. Participants endorsed high levels of satisfaction
(91.5%, n=43/47). The most cited reasons for attending were to relax
(79%, n=38), to manage stress or anxiety (75%, n=36), the wish for
loving kindness/self-compassion (64%, n=30), to learn mindfulness (64%,
n=30) and help with emotional reactivity (53%, n=25). Qualitative
interviews (n=15) identified positive personal and professional impacts.
Personal impacts revealed that participation helped the healthcare
workers to relax, manage stress, care for themselves, sleep better,
reduce isolation, and feel recognized. Professional impacts included
having a toolbox of mindfulness techniques, an ability to use
mindfulness moments and be calmer at work. Some noted that they shared
techniques with colleagues. Reported barriers included participants’
needing time to prioritize themselves, fatigue, forgetting to apply
skills on-the-job, internet stability and finding a private place to
participate. The Pause-4-Providers participants found the online groups
were accessible, and appreciated the drop-in web-based format, content,
and faculty, and had high levels of satisfaction with the program. Both
the novel format (drop-in, live, web-based, anonymous, brief, shared
activity with other healthcare workers) and content (themed mindfulness
practices including micro-practices, with workplace applications) were
enablers to participation. This study of healthcare worker support
sessions was limited by the low number of consenting participants and
the rolling enrollment project design; however findings suggest that a
drop-in web-based mindfulness program has potential to support the
well-being of healthcare workers.
</td>
</tr>
<tr>
<td style="text-align:left;">
38180538
</td>
<td style="text-align:left;">
Impact of COVID-19 pandemic on prescription stimulant use among children
and youth: a population-based study.
</td>
<td style="text-align:left;">
COVID-19 associated public health measures and school closures
exacerbated symptoms in some children and youth with attention-deficit
hyperactivity disorder (ADHD). Less well understood is how the pandemic
influenced patterns of prescription stimulant use. We conducted a
population-based study of stimulant dispensing to children and youth ≤
24 years old between January 1, 2013, and June 30, 2022. We used
structural break analyses to identify the pandemic month(s) when changes
in the dispensing of stimulants occurred. We used interrupted time
series models to quantify changes in dispensing following the structural
break and compare observed and expected stimulant use. Our main outcome
was the change in the monthly rate of stimulant use per 100,000 children
and youth. Following an initial immediate decline of 60.1 individuals
per 100,000 (95% confidence interval \[CI\] - 99.0 to - 21.2), the
monthly rate of stimulant dispensing increased by 11.8 individuals per
100,000 (95% CI 10.0-13.6), with the greatest increases in trend
observed among females, individuals in the highest income
neighbourhoods, and those aged 20 to 24. Observed rates were between
3.9% (95% CI 1.7-6.2%) and 36.9% (95% CI 34.3-39.5%) higher than
predicted among females from June 2020 onward and between 7.1% (95% CI
4.2-10.0%) and 50.7% (95% CI 47.0-54.4%) higher than expected among
individuals aged 20-24 from May 2020 onward. Additional research is
needed to ascertain the appropriateness of stimulant use and to develop
strategies supporting children and youth with ADHD during future periods
of long-term stressors. © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38178897
</td>
<td style="text-align:left;">
Co-designing solutions to enhance access and engagement in pediatric
telerehabilitation.
</td>
<td style="text-align:left;">
Prior to the COVID-19 pandemic, children’s therapy appointments provided
by Ontario’s publicly-funded Children’s Treatment Centre (CTCs)
primarily occurred in-person. With COVID-19 restrictions, CTCs offered
services via telerehabilitation (e.g., video, phone), which remains a
part of service delivery. CTC data shows that families experience
barriers in attending telerehabilitation appointments and may need
supports in place to ensure service accessibility. Our study aimed to
co-design innovative solutions to enhance access and engagement in
ambulatory pediatric telerehabilitation services. This manuscript
reports the co-design process and findings related to solution
development. This research project used an experience based co-design
(EBCD) approach, where caregivers, clinicians and CTC management worked
together to improve experience with telerehabilitation services.
Interview data were collected from 27 caregivers and 27 clinicians to
gain an in-depth understanding of their barriers and successes with
telerehabilitation. Next, 4 interactive co-design meetings were held
with caregivers, clinicians and CTC management to address priorities
identified during the interviews. Using qualitative content analysis,
data from the interviews and co-design meetings were analyzed and
findings related to the solutions developed are presented. Four topics
were identified from the interview data that were selected as focii for
the co-design meetings. Findings from the co-design meetings emphasized
the importance of communication, consistency and connection (the 3C’s)
in experiences with telerehabilitation. The 3C’s are represented in the
co-designed solutions aimed at changing organizational processes and
generating tools and resources for telerehabilitation services. The 3C’s
influence experiences with telerehabilitation services. By enhancing the
experience with telerehabilitation, families will encounter fewer
barriers to accessing and engaging in this service delivery model. ©
2023 Reitzel, Letts, Lennon, Lasenby-Lessard, Novak-Pavlic, Di Rezze and
Phoenix.
</td>
</tr>
<tr>
<td style="text-align:left;">
38178058
</td>
<td style="text-align:left;">
Dental service utilization and the COVID-19 pandemic, a micro-data
analysis.
</td>
<td style="text-align:left;">
Global crises and disease pandemics, such as COVID-19, negatively affect
dental care utilization by several factors, such as infection anxiety,
disrupted supply chains, economic contraction, and household income
reduction. Exploring the pattern of this effect can help policy makers
to be prepared for future crises. The present study aimed to investigate
the financial impact of COVID-19 disruptions on dental service
utilization. Data on the number of dental services offered in Dental
School Clinics of Tehran University of Medical Sciences was collected
over a period of two years, before and after the initial COVID-19
outbreak in Iran. School of Dentistry operates two clinics; one with
competitive service fees and one with subsidies. Regression analyses
were performed to determine the effect of the pandemic on the number of
dental services divided by dental treatment groups and these clinics.
The analyses were adjusted for seasonal patterns and the capacity of the
clinics. There was a significant drop in dental services offered in both
clinics across all dental groups in the post-COVID period (on average,
77 (39.44%) fewer services per day). The majority of the procedure loss
happened in the Private clinic. Adjusting for seasonal patterns and the
service capacity, regression results documented 54% and 12% service loss
in Private and Subsidized clinics following the pandemic, respectively.
Difference-in-difference analysis documented that the Subsidized clinic
performed 40% more treatments than the Private clinic in the post-COVID
period. Pandemic -reduction in dental care utilization could have
long-term ramifications for the oral health of the population, and
policymakers need to provide supportive packages to the affected
segments of the economy to reverse this trend. © 2024. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38176877
</td>
<td style="text-align:left;">
Impact of the COVID-19 pandemic on prescription drug use and costs in
British Columbia: a retrospective interrupted time series study.
</td>
<td style="text-align:left;">
To assess the impact of the COVID-19 pandemic on prescription drug use
and costs. Interrupted time series analysis of comprehensive
administrative health data linkages in British Columbia, Canada, from 1
January 2018 to 28 March 2021. Retrospective population-based analysis
of all prescription drugs dispensed in community pharmacies and
outpatient hospital pharmacies and irrespective of the drug insurance
payer. Between 4.30 and 4.37 million individuals (52% women) actively
registered with the publicly funded medical services plan. COVID-19
pandemic and associated mitigation measures. Weekly dispensing rates and
costs, both overall and stratified by therapeutic groups and
pharmacological subgroups, before and after the declaration of the
public health emergency related to the COVID-19 pandemic. Relative
changes in post-COVID-19 outcomes were expressed as ratios of observed
to expected rates. After the onset of the pandemic and subsequent
COVID-19 mitigation measures, overall medication dispensing rates
dropped by 2.4% (p&lt;0.01), followed by a sustained weekly increase to
return to predicted levels by the end of January 2021. We observed
abrupt level decreases in antibacterials (30.3%, p&lt;0.01) and
antivirals (22.4%, p&lt;0.01) that remained below counterfactuals over
the first year of the pandemic. In contrast, there was a week-to-week
trend increase in nervous system drugs, yielding an overall increase of
7.3% (p&lt;0.01). No trend changes in the dispensing of respiratory
system agents, ACE inhibitors, antidiabetic drugs and antidepressants
were detected. The COVID-19 pandemic impact on prescription drug
dispensing was heterogeneous across medication subgroups. As data become
available, dispensing trends in nervous system agents, antibiotics and
antivirals warrant further monitoring and investigation. © Author(s) (or
their employer(s)) 2024. Re-use permitted under CC BY-NC. No commercial
re-use. See rights and permissions. Published by BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38173401
</td>
<td style="text-align:left;">
We have reached single-visit testing, diagnosis, and treatment for
hepatitis C infection, now what?
</td>
<td style="text-align:left;">
Progress toward hepatitis C virus (HCV) elimination is impeded by low
testing and treatment due to the current diagnostic pathway requiring
multiple visits leading to loss to follow-up. Point-of-care testing
technologies capable of detecting current HCV infection in one hour are
a ‘game-changer.’ These tests enable diagnosis and treatment in a single
visit, overcoming the barrier of multiple visits that frequently leads
to loss to follow-up. Combining point-of-care HCV antibody and RNA tests
should improve cost-effectiveness, patient/provider acceptability, and
testing efficiency. However, implementing HCV point-of-care testing
programs at scale requires multiple considerations. This commentary
explores the need for point-of-care HCV tests, diagnostic strategies to
improve HCV testing, key considerations for implementing point-of-care
HCV testing programs, and remaining challenges for point-of-care testing
(including operator training, quality management, connectivity and
reporting systems, regulatory approval processes, and the need for more
efficient tests). It is exciting that single-visit testing, diagnosis,
and treatment for HCV infection have been achieved. Innovations afforded
through COVID-19 should facilitate the accelerated development of
low-cost, rapid, and accurate tests to improve HCV testing. The next
challenge will be to address barriers and facilitators for implementing
point-of-care testing to deliver them at scale.
</td>
</tr>
<tr>
<td style="text-align:left;">
38173090
</td>
<td style="text-align:left;">
Clinical decision support to enhance venous thromboembolism
pharmacoprophylaxis prescribing for pediatric inpatients with COVID-19.
</td>
<td style="text-align:left;">
To design and evaluate a clinical decision support (CDS) module to
improve guideline concordant venous thromboembolism (VTE)
pharmacoprophylaxis prescribing for pediatric inpatients with COVID-19.
The proportion of patients who met our institutional clinical practice
guideline’s (CPG) criteria for VTE prophylaxis was compared to those who
triggered a CDS alert, indicating the patient needed VTE prophylaxis,
and to those who were prescribed prophylaxis pre and post the launch of
a new VTE CDS module to support VTE pharmacoprophylaxis prescribing. The
sensitivity, specificity, positive predictive value (PPV), negative
predictive value, F1-score and accuracy of the tool were calculated for
the pre- and post-intervention periods using the CPG recommendation as
the gold standard. Accuracy was defined as the sum of the true positives
and true negatives over the sum of the true positives, false positives,
true negatives, and false negatives. Logistic regression was used to
identify variables associated with correct thromboprophylaxis
prescribing. A significant increase in the proportion of patients
triggering a CDS alert occurred in the post-intervention period (44.3%
vs. 6.9%, p &lt; .001); however, no reciprocal increase in VTE
prophylaxis prescribing was achieved (36.6% vs. 40.9%, p = .53). The
updated CDS module had an improved sensitivity (55.0% vs. 13.3%), NPV
(44.9% vs. 36.3%), F1-score (66.7% vs. 23.5%), and accuracy (62.5%
vs. 42.0%), but an inferior specificity (78.6% vs. 100%) and PPV (84.6%
vs. 100%). The updated CDS model had an improved accuracy and overall
performance in correctly identifying patients requiring VTE prophylaxis.
Despite an increase in correct patient identification by the CDS module,
the proportion of patients receiving appropriate pharmacologic
prophylaxis did not change. CDS tools to support correct VTE prophylaxis
prescribing need ongoing refinement and validation to maximize clinical
utility. © 2024 Wiley Periodicals LLC.
</td>
</tr>
<tr>
<td style="text-align:left;">
38172725
</td>
<td style="text-align:left;">
Transitions of care for older adults discharged home from the emergency
department: an inductive thematic content analysis of patient comments.
</td>
<td style="text-align:left;">
Improving care transitions for older adults can reduce emergency
department (ED) visits, adverse events, and empower community autonomy.
We conducted an inductive qualitative content analysis to identify
themes emerging from comments to better understand ED care transitions.
The LEARNING WISDOM prospective longitudinal observational cohort
includes older adults (≥ 65 years) who experienced a care transition
after an ED visit from both before and during COVID-19. Their comments
on this transition were collected via phone interview and transcribed.
We conducted an inductive qualitative content analysis with randomly
selected comments until saturation. Themes that arose from comments were
coded and organized into frequencies and proportions. We followed the
Standards for Reporting Qualitative Research (SRQR). Comments from 690
patients (339 pre-COVID, 351 during COVID) composed of 351 women (50.9%)
and 339 men (49.1%) were analyzed. Patients were satisfied with acute
emergency care, and the proportion of patients with positive acute care
experiences increased with the COVID-19 pandemic. Negative patient
comments were most often related to communication between health
providers across the care continuum and the professionalism of personnel
in the ED. Comments concerning home care became more neutral with the
COVID-19 pandemic. Patients were satisfied overall with acute care but
reported gaps in professionalism and follow-up communication between
providers. Comments may have changed in tone from positive to neutral
regarding home care over the COVID-19 pandemic due to service slowdowns.
Addressing these concerns may improve the quality of care transitions
and provide future pandemic mitigation strategies. © 2024. The
Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38172020
</td>
<td style="text-align:left;">
“I shall not poison my child with your human experiment”: Investigating
predictors of parents’ hesitancy about vaccinating younger children
(&lt;12) in Canada.
</td>
<td style="text-align:left;">
Since the approval of SARS-CoV-2 vaccines for younger children (those
under the age of 12), uptake has been low. Despite widespread
vaccination among older children and adults, these trends may undermine
public health efforts to manage future waves of SARS-CoV-2 or spill over
into other childhood vaccines. The objectives of this study were to
understand parents’ intentions to vaccinate their children (under age
12) against SARS-CoV-2, and to explore reasons for and against
SARS-CoV-2 vaccination. A representative sample of parents of
school-aged children (ages 3-11 years) from Canada’s four largest
provinces were invited in June 2021 to complete a survey on the impact
of COVID-19 on schooling. The survey included specific questions on
parents’ intentions to vaccinate their child(ren) against SARS-CoV-2.
Multinomial regression models were run to estimate associations between
demographic factors, political affiliation and voting, concerns about
individual / family health and vaccination intention. A total of 74.0 %
of parents (n = 288) intended to vaccinate their children with the
SARS-CoV-2 vaccine, 18.3 % (n = 71) did not intend to vaccinate and 7.7
% (n = 30) were unsure. The strongest predictor of parental hesitancy
was whether a parent had themselves been vaccinated. Other factors
including past voting behaviour, dissatisfaction with the government’s
response to the pandemic, and relatively less concern about contracting
SARS-CoV-2 were also correlated with hesitancy. Parents of older
children were more likely to indicate plans to vaccinate their
child(ren). Analysis of the reasons for hesitancy showed parents are
concerned about the safety and side effects of the vaccine, as well as
with processes of testing and approval. A considerable proportion of
Canadian parents of younger school-aged children (ages 3-11) were unsure
and/or hesitant about vaccinating their children against SARS-CoV-2. As
well, a much larger proportion who are not necessarily hesitant have
also not had their children vaccinated. Given the evolving nature of
SARS-CoV-2, including the continued emergence of new variants, reaching
younger children will be important for population health. Health
providers should continue to work with government institutions to ensure
clear communication regarding the safety, efficacy, and importance of
child vaccines for reaching public health goals. Copyright © 2023.
Published by Elsevier Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38169852
</td>
<td style="text-align:left;">
Trends in outpatient and inpatient visits for separate
ambulatory-care-sensitive conditions during the first year of the
COVID-19 pandemic: a province-based study.
</td>
<td style="text-align:left;">
The COVID-19 pandemic led to global disruptions in non-urgent health
services, affecting health outcomes of individuals with
ambulatory-care-sensitive conditions (ACSCs). We conducted a
province-based study using Ontario health administrative data (Canada)
to determine trends in outpatient visits and hospitalization rates (per
100,000 people) in the general adult population for seven ACSCs during
the first pandemic year (March 2020-March 2021) compared to previous
years (2016-2019), and how disruption in outpatient visits related to
acute care use. ACSCs considered were chronic obstructive pulmonary
disease (COPD), asthma, angina, congestive heart failure (CHF),
hypertension, diabetes, and epilepsy. We used time series
auto-regressive integrated moving-average models to compare observed
versus projected rates. Following an initial reduction (March-May 2020)
in all types of visits, primary care outpatient visits (combined
in-person and virtual) returned to pre-pandemic levels for asthma,
angina, hypertension, and diabetes, remained below pre-pandemic levels
for COPD, and rose above pre-pandemic levels for CHF (104.8 vs. 96.4,
95% CI: 89.4-104.0) and epilepsy (29.6 vs. 24.7, 95% CI: 22.1-27.5) by
the end of the first pandemic year. Specialty visits returned to
pre-pandemic levels for COPD, angina, CHF, hypertension, and diabetes,
but remained above pre-pandemic levels for asthma (95.4 vs. 79.5, 95%
CI: 70.7-89.5) and epilepsy (53.3 vs. 45.6, 95% CI: 41.2-50.5), by the
end of the year. Virtual visit rates increased for all ACSCs. Among
ACSCs, reductions in hospitalizations were most pronounced for COPD and
asthma. CHF-related hospitalizations also decreased, albeit to a lesser
extent. For angina, hypertension, diabetes, and epilepsy,
hospitalization rates reduced initially, but returned to pre-pandemic
levels by the end of the year. This study demonstrated variation in
outpatient visit trends for different ACSCs in the first pandemic year.
No outpatient visit trends resulted in increased hospitalizations for
any ACSC; however, reductions in rates of asthma, COPD, and CHF
hospitalizations persisted. Copyright © 2023 Kendzerska, Zhu, Pugliese,
Manuel, Sadatsafavi, Povitz, Stukel, To, Aaron, Mulpuru, Chin, Kendall,
Thavorn, Robillard and Gershon.
</td>
</tr>
<tr>
<td style="text-align:left;">
38167164
</td>
<td style="text-align:left;">
Eating disorder hospitalizations among children and youth in Canada from
2010 to 2022: a population-based surveillance study using administrative
data.
</td>
<td style="text-align:left;">
Eating disorders (EDs) are severe mental illnesses associated with
significant morbidity and mortality. EDs are more prevalent among
females and adolescents. Limited research has investigated Canadian
trends of ED hospitalizations prior to the COVID-19 pandemic, however
during the pandemic, rates of ED hospitalizations have increased. This
study examined rates of ED hospitalizations among children and youth in
Canada from 2010 to 2022, by sex, age, province/territory, length of
stay, discharge disposition and ED diagnosis. Cases of ED
hospitalizations among children and youth, ages 5 to 17 years, were
identified using available ICD-10 codes in the Discharge Abstract
Database from the 2010/11 to 2022/23 fiscal years. The EDs examined in
this study were anorexia nervosa (F50.0), atypical anorexia nervosa
(F50.1), bulimia nervosa (F50.2), other EDs (F50.3, F50.8) and
unspecified EDs (F50.9). Both cases of total and first-time ED
hospitalizations were examined. Descriptive statistics and trend
analyses were performed. Between 2010/11 and 2022/23, 18,740 children
and youth were hospitalized for an ED, 65.9% of which were first-time
hospitalizations. The most frequent diagnosis was anorexia nervosa
(51.3%). Females had significantly higher rates of ED hospitalization
compared to males (66.7/100,000 vs. 5.9/100,000). Youth had
significantly higher rates compared to children. The average age of ED
hospitalization was 14.7 years. Rates of ED hospitalizations were
relatively stable pre-pandemic, however during the pandemic (2020-2021),
rates increased. Rates of pediatric ED hospitalizations in Canada
increased significantly during the pandemic, suggesting that there may
have been limited access to alternative care for EDs or that ED cases
became more severe and required hospitalization. This emphasizes the
need for continued surveillance to monitor how rates of ED
hospitalizations evolve post-pandemic. © 2024. Crown.
</td>
</tr>
<tr>
<td style="text-align:left;">
38166742
</td>
<td style="text-align:left;">
Burnout among public health workers in Canada: a cross-sectional study.
</td>
<td style="text-align:left;">
This study presents the prevalence of burnout among the Canadian public
health workforce after three years of the COVID-19 pandemic and its
association with work-related factors. Data were collected using an
online survey distributed through Canadian public health associations
and professional networks between November 2022 and January 2023.
Burnout was measured using a modified version of the Oldenburg Burnout
Inventory (OLBI). Logistic regressions were used to model the
relationship between burnout and work-related factors including years of
work experience, redeployment to pandemic response, workplace safety and
supports, and harassment. Burnout and the intention to leave or retire
as a result of the COVID-19 pandemic was explored using multinomial
logistic regressions. In 2,079 participants who completed the OLBI, the
prevalence of burnout was 78.7%. Additionally, 49.1% of participants
reported being harassed because of their work during the pandemic.
Burnout was positively associated with years of work experience,
redeployment to the pandemic response, being harassed during the
pandemic, feeling unsafe in the workplace and not being offered
workplace supports. Furthermore, burnout was associated with greater
odds of intending to leave public health or retire earlier than
anticipated. The high levels of burnout among our large sample of
Canadian public health workers and its association with work-related
factors suggest that public health organizations should consider
interventions that mitigate burnout and promote recovery. © 2024. The
Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38163585
</td>
<td style="text-align:left;">
Long-Term Safety of Dupilumab in Patients With Moderate-to-Severe
Asthma: TRAVERSE Continuation Study.
</td>
<td style="text-align:left;">
Previous clinical trials have demonstrated dupilumab efficacy and safety
in adults and adolescents with moderate-to-severe asthma for up to 3
years. The TRAVERSE continuation study (NCT03620747), a single-arm,
open-label study, assessed safety and tolerability of dupilumab 300 mg
every 2 weeks up to an additional 144 weeks (approximately 3 years) in
patients with moderate to-severe asthma who previously completed
TRAVERSE (NCT02134028). Primary endpoints were incidence and event rates
per 100 patient-years (PY) of treatment-emergent adverse events (TEAEs).
Secondary endpoints included adverse events of special interest (AESIs),
serious adverse events (SAEs), and adverse events (AEs) leading to study
discontinuation. 393 patients participated in the TRAVERSE continuation
study (cumulative dupilumab exposure: 431.7 PY; median treatment
duration 309 days). 29 patients (7.4%) received &gt;958 days of
treatment. 214 (54.5%) patients reported at least 1 TEAE (event rate:
171.4); 37 (9.4%) experienced at least 1 treatment-related TEAE, none of
which were considered severe; 2 patients reported 6 TEAEs of moderate
intensity. 22 (5.6%) patients reported SAEs (event rate: 6.9). AESIs
were reported in 24 patients (6.1%; event rate: 6.0). 5 (1.3%) deaths
occurred (event rate: 1.2) following SAEs of COVID-19-related pneumonia
(3 patients), pancreatitis (1 patient), and pulmonary embolism (1
patient). None of the TEAEs leading to death were considered treatment
related. Dupilumab treatment was well tolerated for up to an additional
3 years. Safety findings were consistent with the known safety profile
of dupilumab. These findings further support the long-term use of
dupilumab in patients with moderate-to-severe asthma. Copyright © 2023.
Published by Elsevier Inc. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38163287
</td>
<td style="text-align:left;">
The association between frailty, long-term care home characteristics and
COVID-19 mortality before and after SARS-CoV-2 vaccination: a
retrospective cohort study.
</td>
<td style="text-align:left;">
The relative contributions of long-term care (LTC) resident frailty and
home-level characteristics on COVID-19 mortality has not been well
studied. We examined the association between resident frailty and
home-level characteristics with 30-day COVID-19 mortality before and
after the availability of SARS-CoV-2 vaccination in LTC. We conducted a
population-based retrospective cohort study of LTC residents with
confirmed SARS-CoV-2 infection in Ontario, Canada. We used multi-level
multivariable logistic regression to examine associations between 30-day
COVID-19 mortality, the Hubbard Frailty Index (FI), and resident and
home-level characteristics. We compared explanatory models before and
after vaccine availability. There were 11,179 and 3,655 COVID-19 cases
in the pre- and post-vaccine period, respectively. The 30-day COVID-19
mortality was 25.9 and 20.0% during the same periods. The median odds
ratios for 30-day COVID-19 mortality between LTC homes were 1.50 (95%
credible interval \[CrI\]: 1.41-1.65) and 1.62 (95% CrI: 1.46-1.96),
respectively. In the pre-vaccine period, 30-day COVID-19 mortality was
higher for males and those of greater age. For every 0.1 increase in the
Hubbard FI, the odds of death were 1.49 (95% CI: 1.42-1.56) times
higher. The association between frailty and mortality remained
consistent in the post-vaccine period, but sex and age were partly
attenuated. Despite the substantial home-level variation, no home-level
characteristic examined was significantly associated with 30-day
COVID-19 mortality during either period. Frailty is consistently
associated with COVID-19 mortality before and after the availability of
SARS-CoV-2 vaccination. Home-level characteristics previously attributed
to COVID-19 outcomes do not explain significant home-to-home variation
in COVID-19 mortality. © The Author(s) 2023. Published by Oxford
University Press on behalf of the British Geriatrics Society. All rights
reserved. For permissions, please email: <journals.permissions@oup.com>.
</td>
</tr>
<tr>
<td style="text-align:left;">
38162948
</td>
<td style="text-align:left;">
COVID-19 outcomes in patients with sickle cell disease and sickle cell
trait compared with individuals without sickle cell disease or trait: a
systematic review and meta-analysis.
</td>
<td style="text-align:left;">
Clinical manifestations and severity of SARS-CoV-2 infection in
individuals with sickle cell disease (SCD) and sickle cell trait (SCT)
are not well understood yet. We performed a systematic review and
meta-analysis to assess COVID-19 outcomes in individuals with SCD or SCT
compared to individuals without sickle cell disease or trait. An
electronic search on PubMed, Embase, and Cochrane Library was performed
on August 3, 2023. Two authors (IFM and ISP) independently screened (IFM
and ISP) and extracted data (IFM and ILC) from included studies. Main
exclusion criterion was the absence of the non-SCD/SCT group. Exposure
effects for binary endpoints were compared using pooled odds ratio (OR)
with 95% confidence intervals (CI). I2 statistics was used to assess the
heterogeneity and DerSimonian and Laird random-effects models were
applied for all analyses to minimize the impact of differences in
methods and outcomes definitions between studies. The overall quality of
evidence was assessed using the GRADE system. Review Manager 5.4 and R
software (v4.2.2) were used for statistical analyses. Registered with
PROSPERO, CRD42022366015. Overall, 22 studies were included, with a
total of 1892 individuals with SCD, 8677 individuals with SCT, and
1,653,369 individuals without SCD/SCT. No difference in all-cause
mortality was seen between SCD/SCT and non-SCD/SCT (OR 1.18; 95% CI
0.78-1.77; p = 0.429; I2 = 82%). When considering only studies adjusted
for confounders (8 studies), patients with SCD/SCT were shown to be at
increased risk of death (OR 1.86; 95% CI 1.30-2.66; p = 0.0007; I2 =
34%). No significant difference was seen between individuals with SCD
and SCT (p = 0.863). The adjusted for confounders analysis for
hospitalisation revealed higher rates for the SCD (OR 5.44; 95% CI
1.55-19.13; p = 0.008; I2 = 97%) and the SCT groups (OR 1.31; 95% CI
1.10-1.55; p = 0.002; I2 = 0) compared to the non-SCD/SCT population.
Moreover, it was significantly higher for the SCD group (test for
subgroup difference; p = 0.028). Our findings suggest that patients with
SCD or SCT may present with a higher mortality and hospitalisation rates
due to COVID-19 infection. None. © 2023 The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38161668
</td>
<td style="text-align:left;">
Access to Specialized Care Across the Lifespan in Tetralogy of Fallot.
</td>
<td style="text-align:left;">
Individuals living with tetralogy of Fallot require lifelong specialized
congenital heart disease care to monitor for and manage potential late
complications. However, access to cardiology care remains a challenge
for many patients, as does access to mental health services, dental
care, obstetrical care, and other specialties required by this
population. Inequities in health care access were highlighted by the
COVID-19 pandemic and continue to exist. Paradoxically, many social
factors influence an individual’s need for care, yet inadvertently
restrict access to it. These include sex and gender, being a member of a
racial or ethnic historically excluded group, lower educational
attainment, lower socioeconomic status, living remotely from tertiary
care centres, transportation difficulties, inadequate health insurance,
occupational instability, and prior experiences with discrimination in
the health care setting. These factors may coexist and have compounding
effects. In addition, many patients believe that they are cured and
unaware of the need for specialized follow-up. For these reasons, lapses
in care are common, particularly around the time of transfer from
paediatric to adult care. The lack of trained health care professionals
for adults with congenital heart disease presents an additional barrier,
even in higher income countries. This review summarizes challenges
regarding access to multiple domains of specialized care for individuals
with tetralogy of Fallot, with a focus on the impact of social
determinants of health. Specific recommendations to improve access to
care within Canadian and American systems are offered. © 2023 Published
by Elsevier Inc. on behalf of the Canadian Cardiovascular Society.
</td>
</tr>
<tr>
<td style="text-align:left;">
38157048
</td>
<td style="text-align:left;">
Cardiac Biomarkers Aid in Differentiation of Kawasaki Disease from
Multisystem Inflammatory Syndrome in Children Associated with COVID-19.
</td>
<td style="text-align:left;">
Kawasaki disease (KD) and Multisystem Inflammatory Syndrome in Children
(MIS-C) associated with COVID-19 show clinical overlap and both lack
definitive diagnostic testing, making differentiation challenging. We
sought to determine how cardiac biomarkers might differentiate KD from
MIS-C. The International Kawasaki Disease Registry enrolled
contemporaneous KD and MIS-C pediatric patients from 42 sites from
January 2020 through June 2022. The study population included 118 KD
patients who met American Heart Association KD criteria and compared
them to 946 MIS-C patients who met 2020 Centers for Disease Control and
Prevention case definition. All included patients had at least one
measurement of amino-terminal prohormone brain natriuretic peptide
(NTproBNP) or cardiac troponin I (TnI), and echocardiography. Regression
analyses were used to determine associations between cardiac biomarker
levels, diagnosis, and cardiac involvement. Higher NTproBNP (≥ 1500
ng/L) and TnI (≥ 20 ng/L) at presentation were associated with MIS-C
versus KD with specificity of 77 and 89%, respectively. Higher biomarker
levels were associated with shock and intensive care unit admission;
higher NTproBNP was associated with longer hospital length of stay.
Lower left ventricular ejection fraction, more pronounced for MIS-C, was
also associated with higher biomarker levels. Coronary artery
involvement was not associated with either biomarker. Higher NTproBNP
and TnI levels are suggestive of MIS-C versus KD and may be clinically
useful in their differentiation. Consideration might be given to their
inclusion in the routine evaluation of both conditions. © 2023. The
Author(s), under exclusive licence to Springer Science+Business Media,
LLC, part of Springer Nature.
</td>
</tr>
<tr>
<td style="text-align:left;">
38156430
</td>
<td style="text-align:left;">
Workforce resilience supporting staff in managing stress: A coherent
breathing intervention for the long-term care workforce.
</td>
<td style="text-align:left;">
Staff in long-term care (LTC) homes have long-standing stressors, such
as short staffing and high workloads. These stressors increased during
the COVID-19 pandemic; better resources are needed to help staff manage
stress and well-being. The purpose of this study was to evaluate the
effect of a simple stress management strategy (coherent breathing). We
conducted a pre-post intervention study to evaluate a self-managed
coherent breathing intervention from February to September 2022. The
intervention included basic (breathing only) and comprehensive
(breathing plus a biofeedback device) groups. Six hundred eighty-six
participants were initially recruited (359 and 327 in the comprehensive
and basic groups respectively) from 31 LTC homes in Alberta, Canada. Two
hundred fifty-four participants completed pre-and post-intervention
questionnaires (142 \[55.9%\] in comprehensive and 112 \[44.1%\] in
basic). Participants were asked to use coherent breathing based on a
schedule increasing from 2 to 10 min daily, 5-7 times a week over 8
weeks. Participants completed self-administered online questionnaires
pre- and post-intervention to assess outcomes-stress, psychological
distress, anxiety, depression, resilience, insomnia, compassion
satisfaction, compassion fatigue, and burnout. We used a mixed-effects
regression model to test the main effect of time (pre- and
post-intervention) and group while testing the interaction between time
and group and controlling for covariates. We found statistically
significant changes from pre- to post-intervention in stress (b = -2.5,
p &lt; 0.001, 95% CI = -3.1, -1.9), anxiety (b = -0.5, p &lt; 0.001, 95%
CI = -0.7, -0.3), depression (b = -0.4, p &lt; 0.001, 95% CI = -0.6,
-0.2), insomnia (b = -1.5, p &lt; 0.001, 95% CI = -2.1, -0.9), and
resilience (b = 0.2, p &lt; 0.001, 95% CI = 0.1, 0.2). We observed no
statistically significant differences between the two intervention
groups on any outcome. Our findings suggest that coherent breathing is a
promising strategy for improving stress-related outcomes and resilience.
This intervention warrants further, more rigorous testing. © 2023 The
Authors. Journal of the American Geriatrics Society published by Wiley
Periodicals LLC on behalf of The American Geriatrics Society.
</td>
</tr>
<tr>
<td style="text-align:left;">
38155032
</td>
<td style="text-align:left;">
Impact of pandemic on use of mechanical chest compression systems.
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
38153737
</td>
<td style="text-align:left;">
Post-COVID-19 Condition in Children 6 and 12 Months After Infection.
</td>
<td style="text-align:left;">
There is a need to understand the long-term outcomes among children
infected with SARS-CoV-2. To quantify the prevalence of post-COVID-19
condition (PCC) among children tested for SARS-CoV-2 infection in
pediatric emergency departments (EDs). Multicenter, prospective cohort
study at 14 Canadian tertiary pediatric EDs that are members of the
Pediatric Emergency Research Canada network with 90-day, 6-month, and
12-month follow-up. Participants were children younger than 18 years who
were tested for SARS-CoV-2 infection between August 2020 and February
2022. Data were analyzed from May to November 2023. The presence of
SARS-CoV-2 infection at or within 14 days of the index ED visit.
Presence of symptoms and QoL reductions that meet the PCC definition.
This includes any symptom with onset within 3 months of infection that
is ongoing at the time of follow-up and affects everyday functioning.
The outcome was quantified at 6 and 12 months following the index ED
visit. Among the 5147 children at 6 months (1152 with SARS-CoV-2
positive tests and 3995 with negative tests) and 5563 children at 12
months (1192 with SARS-CoV-2 positive tests and 4371 with negative
tests) who had sufficient data regarding the primary outcome to enable
PCC classification, the median (IQR) age was 2.0 (0.9-5.0) years, and
2956 of 5563 (53.1%) were male. At 6-month follow-up, symptoms and QoL
changes consistent with the PCC definition were present in 6 of 1152
children with positive SARS-CoV-2 tests (0.52%) and 4 of 3995 children
with negative SARS-CoV-2 tests (0.10%; absolute risk difference, 0.42%;
95% CI, 0.02% to 0.94%). The PCC definition was met at 12 months by 8 of
1192 children with positive SARS-CoV-2 tests (0.67%) and 7 of 4371
children with negative SARS-CoV-2 tests (0.16%; absolute risk
difference, 0.51%; 95% CI, 0.06 to 1.08%). At 12 months, the median
(IQR) PedsQL Generic Core Scale scores were 98.4 (90.0-100) among
children with positive SARS-CoV-2 tests and 98.8 (91.7-100) among
children with negative SARS-CoV-2 tests (difference, -0.3; 95% CI, -1.5
to 0.8; P = .56). Among the 8 children with SARS-CoV-2 positive tests
and PCC at 12-month follow-up, children reported respiratory (7 of 8
patients \[88%\]), systemic (3 of 8 patients \[38%\]), and neurologic (1
of 8 patients \[13%\]) symptoms. In this cohort study of children tested
for SARS-CoV-2 infection in Canadian pediatric EDs, although children
infected with SARS-CoV-2 reported increased chronic symptoms, few of
these children developed PCC, and overall QoL did not differ from
children with negative SARS-CoV-2 tests.
</td>
</tr>
<tr>
<td style="text-align:left;">
38152950
</td>
<td style="text-align:left;">
Anxiety and watching the war in Ukraine.
</td>
<td style="text-align:left;">
On 24 February 2022, Russia attacked Ukraine. Millions of people tuned
into social media to watch the war. Media exposure to disasters and
large-scale violence can precipitate anxiety resulting in intrusive
thoughts. This research investigates factors related to anxiety while
watching the war. Since the war began during the ongoing coronavirus
pandemic, threat from COVID-19 is seen as a predictor of anxiety when
watching the war. A theoretical model is put forward where the outcome
was anxiety when watching the war, and predictors were self-reported
interference of watching the war with one’s studies or work, gender,
worry about the war, self-efficacy and coronavirus threat. Data were
collected online with independent samples of university students from
two European countries close to Ukraine, Germany (n = 348) and Finland
(n = 228), who filled out an anonymous questionnaire. Path analysis was
used to analyse the data. Findings showed that the model was an
acceptable fit to the data in each sample, and standardised regression
coefficients indicated that anxiety, when watching the war, increased
with interference, war worry and coronavirus threat, and decreased with
self-efficacy. Women reported more anxiety when watching the war than
men. Implications of the results are discussed. © 2023 The Authors.
International Journal of Psychology published by John Wiley &amp; Sons
Ltd on behalf of International Union of Psychological Science.
</td>
</tr>
<tr>
<td style="text-align:left;">
38151981
</td>
<td style="text-align:left;">
Examining the Relationship Between Workplace Industry and COVID-19
Infection: A Cross-sectional Study of Canada’s Largest Rapid Antigen
Screening Program.
</td>
<td style="text-align:left;">
To control virus spread while keeping the economy open, this study aimed
to identify individuals at increased risk of COVID-19 transmission in
the workplace using rapid antigen screening data. Among adult
participants in a large Canadian rapid antigen screening program
(January 2021-March 2022), we examined screening, personal, and
workplace characteristics and conducted logistic regressions, adjusted
for COVID-19 wave, screening frequency and location, role, age group,
and geography. Among 145,814 participants across 2707 worksites, 6209
screened positive at least once. Workers in natural resources (odds
ratio \[OR\] = 2.1 \[1.73-2.55\]), utilities (OR = 1.67 \[1.38-2.03\]),
construction (OR = 1.35 \[1.06-1.71\]), and transportation/warehousing
(OR = 1.32 \[1.12-1.56\]) had increased odds of screening positive;
workers in education/health (OR = 0.62 \[0.52-0.73\]),
leisure/hospitality (OR = 0.71 \[0.56-0.90\]), and finance (OR = 0.84
\[0.71-0.99\]) had lesser odds of screening positive, compared with
professional/business services. Certain industries involving in-person
work in close quarters are associated with elevated COVID-19
transmission. Continued reliance on rapid screening in these sectors is
warranted. Copyright © 2023 American College of Occupational and
Environmental Medicine.
</td>
</tr>
<tr>
<td style="text-align:left;">
38150254
</td>
<td style="text-align:left;">
Virtual Visits With Own Family Physician vs Outside Family Physician and
Emergency Department Use.
</td>
<td style="text-align:left;">
Virtual visits became more common after the COVID-19 pandemic, but it is
unclear in what context they are best used. To investigate whether there
was a difference in subsequent emergency department use between patients
who had a virtual visit with their own family physician vs those who had
virtual visits with an outside physician. This propensity score-matched
cohort study was conducted among all Ontario residents attached to a
family physician as of April 1, 2021, who had a virtual family physician
visit in the subsequent year (to March 31, 2022). The type of virtual
family physician visit, with own or outside physician, was determined.
In a secondary analysis, own physician visits were compared with visits
with a physician working in direct-to-consumer telemedicine. The primary
outcome was an emergency department visit within 7 days after the
virtual visit. Among 5 229 240 Ontario residents with a family physician
and virtual visit, 4 173 869 patients (79.8%) had a virtual encounter
with their own physician (mean \[SD\] age, 49.3 \[21.5\] years; 2 420
712 females \[58.0%\]) and 1 055 371 patients (20.2%) had an encounter
with an outside physician (mean \[SD\] age, 41.8 \[20.9\] years; 605 614
females \[57.4%\]). In the matched cohort of 1 885 966 patients, those
who saw an outside physician were 66% more likely to visit an emergency
department within 7 days than those who had a virtual visit with their
own physician (30 748 of 942 983 patients \[3.3%\] vs 18 519 of 942 983
patients \[2.0%\]; risk difference, 1.3% \[95% CI, 1.2%-1.3%\]; relative
risk, 1.66 \[95% CI, 1.63-1.69\]). The increase in the risk of emergency
department visits was greater when comparing 30 216 patients with
definite direct-to-consumer telemedicine visits with 30 216 patients
with own physician visits (risk difference, 4.1% \[95% CI, 3.8%-4.5%\];
relative risk, 2.99 \[95% CI, 2.74-3.27\]). In this study, patients
whose virtual visit was with an outside physician were more likely to
visit an emergency department in the next 7 days than those whose
virtual visit was with their own family physician. These findings
suggest that primary care virtual visits may be best used within an
existing clinical relationship.
</td>
</tr>
<tr>
<td style="text-align:left;">
38149489
</td>
<td style="text-align:left;">
The role of the neutrophil-lymphocyte ratio in predicting poor outcomes
in COVID-19 patients.
</td>
<td style="text-align:left;">
This study examines how the neutrophil-lymphocyte ratio (NLR) predicts
coronavirus disease 2019 (COVID-19) hospitalization, severity, length,
and mortality in adult patients. A study was done using a retrospective,
single-center, observational design. A total of 400 patients who were
admitted to the Ziv Medical Center (Safed, Israel) from April 2020 to
December 2021 with a confirmed diagnosis of COVID-19 through RT-PCR
testing were included in the analysis. Two complete blood count
laboratory tests were conducted for each patient. The first test was
administered upon admission to the hospital, while the second test was
conducted prior to the patient’s discharge from the hospital or a few
days before their death. Four hundred patients were included in the
study, 206 males (51.5%) and 194 females (48.5%). The mean age was 64.5
± 17.1 years. In the group of cases, there were 102 deaths, and 296
survivors were recorded, with a fatality rate of 25.5%. The median NLR
was 6.9 ± 5.8 at the beginning of hospitalization and 15.1 ± 32.9 at the
end of hospitalization (p &lt; 0.001). The median length of hospital
stay was 9.4 ± 8.8 days. NLR in the fatality group was 34.0 ± 49.9
compared to 8.4 ± 20.4 in the survivor group (p &lt; 0.001). Comparison
between the NLR at the time of admission of the patient and before
discharge/death was 6.9 ± 5.8 vs. 15.1 ± 32.9 (p &lt; 0.001). The
analyses conducted revealed a statistically significant correlation
between the NLR and the severity, mortality rates, and the duration of
hospitalization. The consideration of NLR should commence during the
initial phases of the disease when assessing individuals afflicted with
COVID-19.
</td>
</tr>
<tr>
<td style="text-align:left;">
38148877
</td>
<td style="text-align:left;">
Exploring the lived experiences of participants and facilitators of an
online mindfulness program during COVID-19: a phenomenological study.
</td>
<td style="text-align:left;">
The coronavirus pandemic (COVID-19) has placed incredible demands on
healthcare workers (HCWs) and adversely impacted their well-being.
Throughout the pandemic, organizations have sought to implement brief
and flexible mental health interventions to better support employees.
Few studies have explored HCWs’ lived experiences of participating in
brief, online mindfulness programming during the pandemic using
qualitative methodologies. To address this gap, we conducted
semi-structured interviews with HCWs and program facilitators (n = 13)
who participated in an online, four-week, mindfulness-based intervention
program. The goals of this study were to: (1) understand how
participants experienced work during the pandemic; (2) understand how
the rapid switch to online life impacted program delivery and how
participants experienced the mindfulness program; and (3) describe the
role of the mindfulness program in supporting participants’ mental
health and well-being. We utilized interpretive phenomenological
analysis (IPA) to elucidate participants’ and facilitators’ rich and
meaningful lived experiences and identified patterns of experiences
through a cross-case analysis. This resulted in four main themes: (1)
changing environments; (2) snowball of emotions; (3) connection and
disconnection; and (4) striving for resilience. Findings from this study
highlight strategies for organizations to create and support wellness
programs for HCWs in times of public health crises. These include
improving social connection in virtual care settings, providing
professional development and technology training for HCWs to adapt to
rapid environmental changes, and recognizing the difference between
emotions and emotional states in HCWs involved in mindfulness-based
programs. Copyright © 2023 Melvin, Canning, Chowdhury, Hunter and Kim.
</td>
</tr>
<tr>
<td style="text-align:left;">
38148036
</td>
<td style="text-align:left;">
Evaluating fluvoxamine for the outpatient treatment of COVID-19: A
systematic review and meta-analysis.
</td>
<td style="text-align:left;">
This systematic review and meta-analysis of randomised controlled trials
(RCTs) aimed to evaluate the efficacy, safety, and tolerability of
fluvoxamine for the outpatient management of COVID-19. We conducted this
review in accordance with the PRISMA 2020 guidelines. Literature
searches were conducted in MEDLINE, EMBASE, International Pharmaceutical
Abstracts, CINAHL, Web of Science, and CENTRAL up to 14 September 2023.
Outcomes included incidence of hospitalisation, healthcare utilization
(emergency room visits and/or hospitalisation), mortality, supplemental
oxygen and mechanical ventilation requirements, serious adverse events
(SAEs) and non-adherence. Fluvoxamine 100 mg twice a day was associated
with reductions in the risk of hospitalisation (risk ratio \[RR\] 0.75,
95% confidence interval \[CI\] 0.58-0.97; I 2 = 0%) and reductions in
the risk of healthcare utilization (RR 0.68, 95% CI 0.53-0.86; I 2 =
0%). While no increased SAEs were observed, fluvoxamine 100 mg twice a
day was associated with higher treatment non-adherence compared to
placebo (RR 1.61, 95% CI 1.22-2.14; I 2 = 53%). In subgroup analyses,
fluvoxamine reduced healthcare utilization in outpatients with BMI ≥30
kg/m2 , but not in those with lower BMIs. While fluvoxamine offers
potential benefits in reducing healthcare utilization, its efficacy may
be most pronounced in high-risk patient populations. The observed
non-adherence rates highlight the need for better patient education and
counselling. Future investigations should reassess trial endpoints to
include outcomes relating to post-COVID sequelaes. Registration: This
review was prospectively registered on PROSPERO (CRD42023463829). © 2023
The Authors. Reviews in Medical Virology published by John Wiley &amp;
Sons Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38146536
</td>
<td style="text-align:left;">
Impact of the COVID-19 pandemic on antidepressant and antipsychotic use
among children and adolescents: a population-based study.
</td>
<td style="text-align:left;">
The COVID-19 pandemic was associated with increases in the prevalence of
depression, anxiety and behavioural problems among children and youth.
Less well understood is the influence of the pandemic on antidepressant
and antipsychotic use among children. This is important, as it is
possible that antidepressants and antipsychotics were used as a
“stop-gap” measure to treat mental health symptoms when in-person access
to outpatient care and school-based supportive services was disrupted.
Furthermore, antipsychotics and antidepressants have been associated
with harm in children and youth. We examined trends in dispensing of
these medications two years following the pandemic among children 18
years of age and under in Ontario, Canada. We conducted a
population-based time-series study of antidepressant and antipsychotic
medication dispensing to children and adolescents ≤18 years old between
September 1, 2014, and March 31, 2022. We measured monthly
population-adjusted rates of antidepressant and antipsychotics obtained
from the IQVIA Geographic Prescription Monitor (GPM) database. We used
structural break analyses to identify the pandemic month(s) when changes
in the dispensing of antidepressants and antipsychotics occurred. We
used interrupted time series models to quantify changes in dispensing
following the structural break and compare observed and expected use of
these drugs. Overall, we found higher-than-expected dispensing of
antidepressants and antipsychotics in children and youth. Specifically,
we observed an immediate step decrease in antidepressant dispensing
associated with a structural break in April 2020 (-55.8 units per 1,000
individuals; 95% confidence intervals \[CI\] CI: -117.4 to 5.8),
followed by an increased monthly trend in the rate of antidepressant
dispensing of 13.0 units per 1,000 individuals (95% CI: 10.2-15.9).
Antidepressant dispensing was consistently greater than predicted from
September 2020 onward. Antipsychotic dispensing increased immediately
following a June 2020 structural break (26.4 units per 1,000
individuals; 95% CI: 15.8-36.9) and did not change appreciably
thereafter. Antipsychotic dispensing was higher than predicted at all
time points from June 2020 onward. We found higher-than-expected
dispensing of antidepressants and antipsychotics in children and youth.
These increases were sustained through nearly two years of observation
and are especially concerning in light of the potential for harm with
the long-term use of antipsychotics in children. Further research is
required to understand the clinical implications of these findings. ©
2023 Antoniou, Pajer, Gardner, Penner, Lunsky, Tadrous, Mamdani,
Gozdyra, Juurlink and Gomes.
</td>
</tr>
<tr>
<td style="text-align:left;">
38145480
</td>
<td style="text-align:left;">
Design of a Dyadic Digital Health Module for Chronic Disease Shared
Care: Development Study.
</td>
<td style="text-align:left;">
The COVID-19 pandemic forced the spread of digital health tools to
address limited clinical resources for chronic health management. It
also illuminated a population of older patients requiring an informal
caregiver (IC) to access this care due to accessibility, technological
literacy, or English proficiency concerns. For patients with heart
failure (HF), this rapid transition exacerbated the demand on ICs and
pushed Canadians toward a dyadic care model where patients and ICs
comanage care. Our previous work identified an opportunity to improve
this dyadic HF experience through a shared model of dyadic digital
health. We call this alternative model of care “Caretown for Medly,”
which empowers ICs to concurrently expand patients’ self-care abilities
while acknowledging ICs’ eagerness to provide greater support. We
present the systematic design and development of the Caretown for Medly
dyadic management module. While HF is the outlined use case, we outline
our design methodology and report on 6 core disease-invariant features
applied to dyadic shared care for HF management. This work lays the
foundation for future usability assessments of Caretown for Medly. We
conducted a qualitative, human-centered design study based on 25
semistructured interviews with self-identified ICs of loved ones living
with HF. Interviews underwent thematic content analysis by 2 coders
independently for themes derived deductively (eg, based on the interview
guide) and inductively refined. To build the Caretown for Medly model,
we (1) leveraged the Knowledge to Action (KTA) framework to translate
knowledge into action and (2) borrowed Google Sprint’s ability to
quickly “solve big problems and test new ideas,” which has been
effective in the medical and digital health spaces. Specifically, we
blended these 2 concepts into a new framework called the “KTA Sprint.”
We identified 6 core disease-invariant features to support ICs in care
dyads to provide more effective care while capitalizing on dyadic care’s
synergistic benefits. Features were designed for customizability to suit
the patient’s condition, informed by stakeholder analysis, corroborated
with literature, and vetted through user needs assessments. These
features include (1) live reports to enhance data sharing and facilitate
appropriate IC support, (2) care cards to enhance guidance on the
caregiving role, (3) direct messaging to dissolve the disconnect across
the circle of care, (4) medication wallet to improve guidance on
managing complex medication regimens, (5) medical events timeline to
improve and consolidate management and organization, and (6) caregiver
resources to provide disease-specific education and support their
self-care. These disease-invariant features were designed to address
ICs’ needs in supporting their care partner. We anticipate that the
implementation of these features will empower a shared model of care for
chronic disease management through digital health and will improve
outcomes for care dyads. ©Camila Benmessaoud, Kaylen J Pfisterer,
Anjelica De Leon, Ashish Saragadam, Noor El-Dassouki, Karen G M Young,
Raima Lohani, Ting Xiong, Quynh Pham. Originally published in JMIR Human
Factors (<https://humanfactors.jmir.org>), 25.12.2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38145238
</td>
<td style="text-align:left;">
Investigating the impact of the COVID-19 pandemic on the occurrence of
medication incidents in Canadian community pharmacies.
</td>
<td style="text-align:left;">
As the COVID-19 pandemic unfolded, community pharmacies adapted rapidly
to broaden and adjust the services they were providing to patients,
while coping with severe pressure on supply chains and constrained
social interactions. This study investigates whether these events had an
impact on the medication incidents reported by pharmacists. Results
indicate that Canadian pharmacies were able to sustain such stress while
maintaining comparable safety levels. At the same time, it appears that
some risk factors that were either ignored or not meaningful in the past
started to be reported, suggesting that community pharmacists are now
aware of a larger set of contributing factors that can lead to
medication incidents, notably for medication incidents that can lead to
harm. © 2023 The Authors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38142697
</td>
<td style="text-align:left;">
Adaptive servo-ventilation for sleep-disordered breathing in patients
with heart failure with reduced ejection fraction (ADVENT-HF): a
multicentre, multinational, parallel-group, open-label, phase 3
randomised controlled trial.
</td>
<td style="text-align:left;">
In patients with heart failure and reduced ejection fraction,
sleep-disordered breathing, comprising obstructive sleep apnoea (OSA)
and central sleep apnoea (CSA), is associated with increased morbidity,
mortality, and sleep disruption. We hypothesised that treating
sleep-disordered breathing with a peak-flow triggered adaptive
servo-ventilation (ASV) device would improve cardiovascular outcomes in
patients with heart failure and reduced ejection fraction. We conducted
a multicentre, multinational, parallel-group, open-label, phase 3
randomised controlled trial of peak-flow triggered ASV in patients aged
18 years or older with heart failure and reduced ejection fraction (left
ventricular ejection fraction ≤45%) who were stabilised on optimal
medical therapy with co-existing sleep-disordered breathing
(apnoea-hypopnoea index \[AHI\] ≥15 events/h of sleep), with concealed
allocation and blinded outcome assessments. The trial was carried out at
49 hospitals in nine countries. Sleep-disordered breathing was
stratified into predominantly OSA with an Epworth Sleepiness Scale score
of 10 or lower or predominantly CSA. Participants were randomly assigned
to standard optimal treatment alone or standard optimal treatment with
the addition of ASV (1:1), stratified by study site and sleep apnoea
type (ie, CSA or OSA), with permuted blocks of sizes 4 and 6 in random
order. Clinical evaluations were performed and Minnesota Living with
Heart Failure Questionnaire, Epworth Sleepiness Scale, and New York
Heart Association class were assessed at months 1, 3, and 6 following
randomisation and every 6 months thereafter to a maximum of 5 years. The
primary endpoint was the cumulative incidence of the composite of
all-cause mortality, first admission to hospital for a cardiovascular
reason, new onset atrial fibrillation or flutter, and delivery of an
appropriate cardioverter-defibrillator shock. All-cause mortality was a
secondary endpoint. Analysis for the primary outcome was done in the
intention-to-treat population. This trial is registered with
ClinicalTrials.gov (NCT01128816) and the International Standard
Randomised Controlled Trial Number Register (ISRCTN67500535), and the
trial is complete. The first and last enrolments were Sept 22, 2010, and
March 20, 2021. Enrolments terminated prematurely due to
COVID-19-related restrictions. 1127 patients were screened, of whom 731
(65%) patients were randomly assigned to receive standard care (n=375;
mean AHI 42·8 events per h of sleep \[SD 20·9\]) or standard care plus
ASV (n=356; 43·3 events per h of sleep \[20·5\]). Follow-up of all
patients ended at the latest on June 15, 2021, when the trial was
terminated prematurely due to a recall of the ASV device due to
potential disintegration of the motor sound-abatement material. Over the
course of the trial, 41 (6%) of participants withdrew consent and 34
(5%) were lost to follow-up. In the ASV group, the mean AHI decreased to
2·8-3·7 events per h over the course of the trial, with associated
improvements in sleep quality assessed 1 month following randomisation.
Over a mean follow-up period of 3·6 years (SD 1·6), ASV had no effect on
the primary composite outcome (180 events in the control group vs 166 in
the ASV group; hazard ratio \[HR\] 0·95, 95% CI 0·77-1·18; p=0·67) or
the secondary endpoint of all-cause mortality (88 deaths in the control
group vs. 76 in the ASV group; 0·89, 0·66-1·21; p=0·47). For patients
with OSA, the HR for all-cause mortality was 1·00 (0·68-1·46; p=0·98)
and for CSA was 0·74 (0·44-1·23; p=0·25). No safety issue related to ASV
use was identified. In patients with heart failure and reduced ejection
fraction and sleep-disordered breathing, ASV had no effect on the
primary composite outcome or mortality but eliminated sleep-disordered
breathing safely. Canadian Institutes of Health Research and Philips RS
North America. Copyright © 2024 Elsevier Ltd. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38140720
</td>
<td style="text-align:left;">
Environment-based approaches to improve participation of young people
with physical disabilities during COVID-19.
</td>
<td style="text-align:left;">
To examine the effects of the Pathways and Resources for Engagement and
Participation (PREP) intervention during the COVID-19 pandemic on (1)
activity performance and satisfaction, and (2) motor, cognitive, and
affective body functions. An interrupted time-series design with
multiple baselines across 21 young people (13 females, eight males) aged
16 to 25 years (median = 21 years 5 months) with physical disabilities
was employed. The young people engaged in an 8-week self-chosen leisure
activity (e.g. football, piano, photography) at their home or community.
The Canadian Occupational Performance Measure (COPM) assessed activity
performance and satisfaction weekly. Mental health problems, including
affective and cognitive outcomes, were assessed weekly using the
Behavior Assessment System for Children, Third Edition. Motor functions
(e.g. trunk control, reaching, strength) were assessed biweekly. Linear
mixed-effects models were used. The intervention had large effects on
activity performance (0.78) and satisfaction (0.88) with clinically
significant change in COPM scores (2.6 \[95% confidence interval {CI}:
2.0-3.2\] and 3.2 points \[95% CI: 2.4-3.9\] respectively). Young people
without mental health problems at baseline benefited more from the
intervention (p = 0.028). Improvements in at least one domain of body
function occurred in 10 young people especially for motor outcomes.
Results demonstrate the effectiveness of PREP during adverse times and
suggest benefits going beyond participation, involving outcomes at the
body-function level. © 2023 The Authors. Developmental Medicine &amp;
Child Neurology published by John Wiley &amp; Sons Ltd on behalf of Mac
Keith Press.
</td>
</tr>
<tr>
<td style="text-align:left;">
38140216
</td>
<td style="text-align:left;">
Global Analysis of Tracking the Evolution of SARS-CoV-2 Variants.
</td>
<td style="text-align:left;">
Severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2), infamously
known as Coronavirus Disease 2019 (COVID-19), is responsible for the
current pandemic and, to date, has greatly impacted public health and
economy globally \[…\].
</td>
</tr>
<tr>
<td style="text-align:left;">
38135322
</td>
<td style="text-align:left;">
Faith-based organisations and their role in supporting vaccine
confidence and uptake: a scoping review protocol.
</td>
<td style="text-align:left;">
Faith-based organisations (FBOs) and religious actors increase vaccine
confidence and uptake among ethnoracially minoritised communities in
low-income and middle-income countries. During the COVID-19 pandemic and
the subsequent vaccine rollout, global organisations such as the WHO and
UNICEF called for faith-based collaborations with public health agencies
(PHAs). As PHA-FBO partnerships emerge to support vaccine uptake, the
scoping review aims to: (1) outline intervention typologies and
implementation frameworks guiding interventions; (2) describe the roles
of PHAs and FBOs in the design, implementation and evaluation of
strategies and (3) synthesise outcomes and evaluations of PHA-FBO
vaccine uptake initiatives for ethnoracially minoritised communities. We
will perform six library database searches in PROQUEST-Public Health,
OVID MEDLINE, Cochrane Library, CINAHL, SCOPUS- all, PROQUEST - Policy
File index; three theses repositories, four website searches, five niche
journals and 11 document repositories for public health. These databases
will be searched for literature that describe partnerships for vaccine
confidence and uptake for ethnoracially minoritised populations,
involving at least one PHA and one FBO, published in English from
January 2011 to October 2023. Two reviewers will pilot-test 20 articles
to refine and finalise the inclusion/exclusion criteria and data
extraction template. Four reviewers will independently screen and
extract the included full-text articles. An implementation science
process framework outlining the design, implementation and evaluation of
the interventions will be used to capture the array of partnerships and
effectiveness of PHA-FBO vaccine uptake initiatives. This multiphase
Canadian Institutes of Health Research (CIHR) project received ethics
approval from the University of Toronto. Findings will be translated
into a series of written materials for dissemination to CIHR, and
collaborating knowledge users (ie, regional and provincial PHAs), and
panel presentations at conferences to inform the development of a
best-practices framework for increasing vaccine confidence and uptake. ©
Author(s) (or their employer(s)) 2023. Re-use permitted under CC BY-NC.
No commercial re-use. See rights and permissions. Published by BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38135301
</td>
<td style="text-align:left;">
Examining adaptive models of care implemented in hospital ICUs during
the COVID-19 pandemic: a qualitative study.
</td>
<td style="text-align:left;">
The emergence of the COVID-19 pandemic led to an increased demand for
hospital beds, which in turn led to unique changes to both the
organisation and delivery of patient care, including the adoption of
adaptive models of care. Our objective was to understand staff
perspectives on adaptive models of care employed in intensive care units
(ICUs) during the pandemic. We interviewed 77 participants representing
direct care staff (registered nurses) and members of the nursing
management team (nurse managers, clinical educators and nurse
practitioners) from 12 different ICUs. Thematic analysis was used to
code and analyse the data. Our findings highlight effective elements of
adaptive models of care, including appreciation for redeployed staff,
organising aspects of team-based models and ICU culture. Challenges
experienced with the pandemic models of care were heightened workload,
the influence of experience, the disparity between model and practice
and missed care. Finally, debriefing, advanced planning and preparation,
the redeployment process and management support and communication were
important areas to consider in implementing future adaptive care models.
The implementation of adaptive models of care in ICUs during the
COVID-19 pandemic provided a rapid solution for staffing during the
surge in critical care patients. Findings from this study highlight some
of the challenges of implementing redeployment as a staffing strategy,
including how role clarity and accountability can influence the adoption
of care delivery models, lead to workarounds and contribute to adverse
patient and nurse outcomes. © Author(s) (or their employer(s)) 2023.
Re-use permitted under CC BY-NC. No commercial re-use. See rights and
permissions. Published by BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38134724
</td>
<td style="text-align:left;">
Association of SARS-CoV-2 infection with neurological impairments in
pediatric population: A systematic review.
</td>
<td style="text-align:left;">
Neurological manifestations have been widely reported in adults with
COVID-19, yet the extent of involvement among the pediatric population
is currently poorly characterized. The objective of our systematic
review is to evaluate the association of SARS-CoV-2 infection with
neurological symptoms and neuroimaging manifestations in the pediatric
population. A literature search of Cochrane Library; EBSCO CINAHL;
Global Index Medicus; OVID AMED, Embase, Medline, PsychINFO; and Scopus
was conducted in accordance with the Peer Review of Electronic Search
Strategies form (October 1, 2019 to March 15, 2022). Studies were
included if they reported (1) COVID-19-associated neurological symptoms
and neuroimaging manifestations in individuals aged &lt;18 years with a
confirmed, first SARS-CoV-2 infection and were (2) peer-reviewed.
Full-text reviews of 222 retrieved articles were performed, along with
subsequent reference searches. A total of 843 no-duplicate records were
retrieved. Of the 19 identified studies, there were ten retrospective
observational studies, seven case series, one case report, and one
prospective cohort study. A total of 6985 individuals were included,
where 12.8% (n = 892) of hospitalized patients experienced
neurocognitive impairments which includes: 1) neurological symptoms (n =
294 of 892, 33.0%), 2) neurological syndromes and neuroimaging
abnormalities (n = 223 of 892, 25.0%), and 3) other phenomena (n = 233
of 892, 26.1%). Based on pediatric-specific cohorts, children
experienced more drowsiness (7.3% vs. 1.3%) and muscle weakness (7.3%
vs. 6.3%) as opposed to adolescents. Agitation or irritability was
observed more in children (7.3%) than infants (1.3%). Our findings
revealed a high prevalence of immune-mediated patterns of disease among
COVID-19 positive pediatric patients with neurocognitive abnormalities.
Copyright © 2023 Elsevier Ltd. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38134253
</td>
<td style="text-align:left;">
COVID-19 Infection, Symptoms, and Stroke Revascularization Outcomes:
Intriguing Connections.
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
38134127
</td>
<td style="text-align:left;">
Scheduled and urgent inguinal hernia repair in Ontario, Canada between
2010 and 2022: Population-based cross sectional analysis of trends and
outcomes.
</td>
<td style="text-align:left;">
We examine trends in inguinal hernia repairs with respect to the
COVID-19 pandemic and secular trends in Ontario, Canada. This was a
retrospective cohort study. Hernia repairs performed January 1,
2010-December 31, 2022 were captured from health administrative
inpatient and outpatient databases. Patients managed in three clinical
settings were examined: public hospital in-patient, semi-private
hospital in-patient (Shouldice Hospital), and public hospital
out-patient. We examined the effect of the COVID-19 pandemic on surgical
volumes, clinical setting, patient characteristics by setting, time from
diagnosis until surgery, hospital length-of-stay, and patient outcomes
(90-day readmissions, 1-year reoperations). We used multivariable
logistic regression to examine whether patient outcomes were comparable
between the COVID-19 period and the pre-pandemic period, adjusted
sociodemographic and clinical factors. Shouldice Hospital is the only
semi-private hospital in Ontario specializing in hernia repair (patients
pay for the mandated admission, but not for the procedure). During the
pandemic (March 2020-December 2022), there were 8,162 fewer (15%)
scheduled inguinal hernia repairs than expected, but the age-sex
standardized rate of urgent repairs remained unchanged. Shouldice
Hospital performed more surgeries in the COVID-19 era than pre-pandemic
and had a shorter average LOS by 24 hours, despite treating more
patients with older age, higher ASA score \[adjusted odds ratio (aOR)
2.13 (1.93-2.35) III vs I-II\] and greater comorbidity \[aOR 1.36
(1.08-1.70) for 2 vs none\] than pre-pandemic. Patients treated in the
COVID-19 era experienced a longer time until surgery, being the longest
in 2022 (median 133 days). Ninety-day readmissions and 1-year
reoperations were lower in the COVID-19 era and lower for patients
receiving surgery at Shouldice Hospital. During the COVID-19 pandemic,
there were 8,162 fewer scheduled hernia repairs than expected, longer
wait-times until surgery, shorter length-of-stay, and more patients with
comorbidities, but outcomes were not worse compared with the
pre-pandemic period. Copyright: © 2023 Habbous et al. This is an open
access article distributed under the terms of the Creative Commons
Attribution License, which permits unrestricted use, distribution, and
reproduction in any medium, provided the original author and source are
credited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38132023
</td>
<td style="text-align:left;">
Yoga Pose Estimation Using Angle-Based Feature Extraction.
</td>
<td style="text-align:left;">
This research addresses the challenges of maintaining proper yoga
postures, an issue that has been exacerbated by the COVID-19 pandemic
and the subsequent shift to virtual platforms for yoga instruction. This
research aims to develop a mechanism for detecting correct yoga poses
and providing real-time feedback through the application of computer
vision and machine learning (ML) techniques. This study utilized
computer vision-based pose estimation methods to extract features and
calculate yoga pose angles. A variety of models, including extremely
randomized trees, logistic regression, random forest, gradient boosting,
extreme gradient boosting, and deep neural networks, were trained and
tested to classify yoga poses. Our study employed the Yoga-82 dataset,
consisting of many yoga pose images downloaded from the web. The results
of this study show that the extremely randomized trees model
outperformed the other models, achieving the highest prediction accuracy
of 91% on the test dataset and 92% in a fivefold cross-validation
experiment. Other models like random forest, gradient boosting, extreme
gradient boosting, and deep neural networks achieved accuracies of 90%,
89%, 90%, and 85%, respectively, while logistic regression
underperformed, having the lowest accuracy. This research concludes that
the extremely randomized trees model presents superior predictive power
for yoga pose recognition. This suggests a valuable avenue for future
exploration in this domain. Moreover, the approach has significant
potential for implementation on low-powered smartphones with minimal
latency, thereby enabling real-time feedback for users practicing yoga
at home.
</td>
</tr>
<tr>
<td style="text-align:left;">
38131688
</td>
<td style="text-align:left;">
Patient Presentations in a Community Pain Clinic after COVID-19
Infection or Vaccination: A Case-Series Approach.
</td>
<td style="text-align:left;">
Early case report studies and anecdotes from patients, medical
colleagues, and social media suggest that patients may present to
chronic pain clinics with a number of complaints post COVID-19 infection
or vaccination. The aim of this study is to systematically report on a
consecutive series of chronic pain patients seen in a community-based
pain clinic, who acquired symptoms after COVID-19 infection or
vaccination. This retrospective cross-sectional descriptive study
identified all patients seen at the clinic over a 4-month period
(January-April 2022) with persistent symptoms after COVID-19 infection,
vaccination, or both. Information was collected on sex, gender, age,
details of vaccination, new pains, or exacerbation of old pain plus the
development of novel symptoms. The study identified 21 community
dwellers (17 females and 4 males; F/M 4.25/1; age range 22-79 years;
mean age 46.3 years), with symptoms attributed to COVID-19 infection or
vaccination. Several patients suffered exacerbation of previous pains or
developed novel pains, as well as high levels of anxiety and mood
disorders. A review of the existing literature provides support for the
spectrum of symptoms displayed by the study group. Information collected
in this study will add to the body of COVID-19-related literature and
assist particularly community practitioners in recognizing and managing
these conditions.
</td>
</tr>
<tr>
<td style="text-align:left;">
38131026
</td>
<td style="text-align:left;">
What are effective strategies to respond to the psychological impacts of
working on the frontlines of a public health emergency?
</td>
<td style="text-align:left;">
The COVID-19 pandemic has disrupted the healthcare and public health
sectors. The impact of working on the frontlines as a healthcare or
public health professional has been well documented. Healthcare
organizations must support the psychological and mental health of those
responding to future public health emergencies. This systematic review
aims to identify effective interventions to support healthcare workers’
mental health and wellbeing during and following a public health
emergency. Eight scientific databases were searched from inception to 1
November 2022. Studies that described strategies to address the
psychological impacts experienced by those responding to a public health
emergency (i.e., a pandemic, epidemic, natural disaster, or mass
casualty event) were eligible for inclusion. No limitations were placed
based on study design, language, publication status, or publication
date. Two reviewers independently screened studies, extracted data, and
assessed methodological quality using the Joanna Briggs Institute
critical appraisal tools. Discrepancies were resolved through discussion
and a third reviewer when needed. Results were synthesized narratively
due to the heterogeneity of populations and interventions. Outcomes were
displayed graphically using harvest plots. A total of 20,018 records
were screened, with 36 unique studies included in the review, 15
randomized controlled trials, and 21 quasi-experimental studies. Results
indicate that psychotherapy, psychoeducation, and mind-body
interventions may reduce symptoms of anxiety, burnout, depression, and
Post Traumatic Stress Disorder, with the lowest risk of bias found among
psychotherapy interventions. Psychoeducation appears most promising to
increase resilience, with mind-body interventions having the most
substantial evidence for increases in quality of life. Few
organizational interventions were identified, with highly heterogeneous
components. Promoting healthcare workers’ mental health is essential at
an individual and health system level. This review identifies several
promising practices that could be used to support healthcare workers at
risk of adverse mental health outcomes as they respond to future public
health emergencies.Systematic review registration:
<https://www.crd.york.ac.uk/prospero/display_record.php?RecordID=203810>,
identifier \#CRD42020203810 (PROSPERO). Copyright © 2023 Neil-Sztramko,
Belita, Hopkins, Sherifali, Anderson, Apatu, Kapiriri, Tarride,
Bellefleur, Kaasalainen, Marr and Dobbins.
</td>
</tr>
<tr>
<td style="text-align:left;">
38129798
</td>
<td style="text-align:left;">
Association between biochemical and hematologic factors with COVID-19
using data mining methods.
</td>
<td style="text-align:left;">
Coronavirus disease (COVID-19) is an infectious disease that can spread
very rapidly with important public health impacts. The prediction of the
important factors related to the patient’s infectious diseases is
helpful to health care workers. The aim of this research was to select
the critical feature of the relationship between demographic,
biochemical, and hematological characteristics, in patients with and
without COVID-19 infection. A total of 13,170 participants in the age
range of 35-65 years were recruited. Decision Tree (DT), Logistic
Regression (LR), and Bootstrap Forest (BF) techniques were fitted into
data. Three models were considered in this study, in model I, the
biochemical features, in model II, the hematological features, and in
model II, both biochemical and homological features were studied. In
Model I, the BF, DT, and LR algorithms identified creatine phosphokinase
(CPK), blood urea nitrogen (BUN), fasting blood glucose (FBG), total
bilirubin, body mass index (BMI), sex, and age, as important predictors
for COVID-19. In Model II, our BF, DT, and LR algorithms identified BMI,
sex, mean platelet volume (MPV), and age as important predictors. In
Model III, our BF, DT, and LR algorithms identified CPK, BMI, MPV, BUN,
FBG, sex, creatinine (Cr), age, and total bilirubin as important
predictors. The proposed BF, DT, and LR models appear to be able to
predict and classify infected and non-infected people based on CPK, BUN,
BMI, MPV, FBG, Sex, Cr, and Age which had a high association with
COVID-19. © 2023. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38128935
</td>
<td style="text-align:left;">
Long COVID in long-term care: a rapid realist review.
</td>
<td style="text-align:left;">
The goals of this rapid realist review were to ask: (a) what are the key
mechanisms that drive successful interventions for long COVID in
long-term care (LTC) and (b) what are the critical contexts that
determine whether the mechanisms produce the intended outcomes? Rapid
realist review. Medline, CINAHL, Embase, PsycINFO and Web of Science for
peer-reviewed literature and Google for grey literature were searched up
to 23 February 2023. We included sources focused on interventions,
persons in LTC, long COVID or post-acute phase at least 4 weeks
following initial COVID-19 infection and ones that had a connection with
source materials. Three independent reviewers searched, screened and
coded studies. Two independent moderators resolved conflicts. A data
extraction tool organised relevant data into context-mechanism-outcome
configurations using realist methodology. Twenty-one sources provided 51
intervention data excerpts used to develop our programme theory.
Synthesised findings were presented to a reference group and expert
panel for confirmatory purposes. Fifteen peer-reviewed articles and six
grey literature sources were eligible for inclusion. Eleven
context-mechanism-outcome configurations identify those contextual
factors and underlying mechanisms associated with desired outcomes, such
as clinical care processes and policies that ensure timely access to
requisite resources for quality care delivery, and resident-centred
assessments and care planning to address resident preferences and needs.
The underlying mechanisms associated with enhanced outcomes for LTC long
COVID survivors were: awareness, accountability, vigilance and
empathetic listening. Although the LTC sector struggles with
organisational capacity issues, they should be aware that
comprehensively assessing and monitoring COVID-19 survivors and
providing timely interventions to those with long COVID is imperative.
This is due to the greater care needs of residents with long COVID, and
coordinated efficient care is required to optimise their quality of
life. © Author(s) (or their employer(s)) 2023. Re-use permitted under CC
BY-NC. No commercial re-use. See rights and permissions. Published by
BMJ.
</td>
</tr>
<tr>
<td style="text-align:left;">
38127861
</td>
<td style="text-align:left;">
The impact of the COVID-19 pandemic on the rate of primary care visits
for substance use among patients in Ontario, Canada.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has led to an increase in the prevalence of
substance use presentations. This study aims to assess the impact of the
COVID-19 pandemic on the rate of primary care visits for substance use
including tobacco, alcohol, and other drug use among primary care
patients in Ontario, Canada. Diagnostic and service fee code data were
collected from a longitudinal cohort of family medicine patients during
pre-pandemic (March 14, 2019-March 13, 2020) and pandemic periods (March
14, 2020-March 13, 2021). Generalized linear models were used to compare
the rate of substance-use related visits pre-pandemic and during the
pandemic. The effects of demographic characteristics including age, sex,
and income quintile were also assessed. Relative to the pre-pandemic
period, patients were less likely to have a primary care visit during
the pandemic for tobacco-use related reasons (OR = 0.288, 95% CI
\[0.270-0.308\]), and for alcohol-use related reasons (OR = 0.851, 95%
CI \[0.780-0.929\]). In contrast, patients were more likely to have a
primary care visit for other drug-use related reasons (OR = 1.150, 95%
CI \[1.080-1.225\]). In the face of a known increase in substance use
during the COVID-19 pandemic, a decrease in substance use-related
primary care visits likely represents an unmet need for this patient
population. This study highlights the importance of continued research
in the field of substance use, especially in periods of heightened
vulnerability such as during the COVID-19 pandemic. Copyright: © 2023
Siu et al. This is an open access article distributed under the terms of
the Creative Commons Attribution License, which permits unrestricted
use, distribution, and reproduction in any medium, provided the original
author and source are credited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38127207
</td>
<td style="text-align:left;">
Metabolic disturbances potentially attributable to clogging during
continuous renal replacement therapy.
</td>
<td style="text-align:left;">
Clogging is characterized by a progressive impairment of transmembrane
patency in renal replacement devices and occurs due to obstruction of
pores by unknown molecules. If citrate-based anti-coagulation is used,
clogging can manifest as a metabolic alkalosis accompanied by
hypernatremia and hypercalcemia, primarily a consequence of Na3Citrate
infusion. An increased incidence of clogging has been observed during
the COVID-19 pandemic. However, precise factors contributing to the
formation remain uncertain. This investigation aimed to analyze its
incidence and assessed time-varying trajectories of associated factors
in critically ill patients on continuous renal replacement therapy
(CRRT). In this retrospective, single-center data analysis, we evaluated
COVID-19 patients undergoing CRRT and admitted to critical care between
March 2020 and December 2021. We assessed the proportional incidence of
clogging surrogates in the overall population and subgroups based on the
specific CRRT devices employed at our institution, including
multiFiltrate (Fresenius Medical Care) and Prismaflex System (Baxter).
Moderate and severe clogging were defined as Na &gt; 145 or ≥ 150 mmol/l
and HCO3- &gt; 28.0 or ≥ 30 mmol/l, respectively, with a total
albumin-corrected calcium &gt; 2.54 mmol/l. A mixed effect model was
introduced to investigate factors associated with development of
clogging. Fifty-three patients with 240 CRRT runs were analyzed.
Moderate and severe clogging occurred in 15% (8/53) and 19% (10/53) of
patients, respectively. Twenty-seven percent (37/136) of CRRTs conducted
with a multiFiltrate device met the criteria for clogging, whereas no
clogging could be observed in patients dialyzed with the Prismaflex
System. Occurrence of clogging was associated with elevated triglyceride
plasma levels at filter start (p = 0.013), amount of enteral nutrition
(p = 0.002) and an increasing white blood cell count over time (p =
0.002). Clogging seems to be a frequently observed phenomenon in
critically ill COVID-19 patients. The presence of hypertriglyceridemia,
combined with systemic inflammation, may facilitate the development of
an impermeable secondary membrane within filters, thereby contributing
to compromised membrane patency. © 2023. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38126062
</td>
<td style="text-align:left;">
Pharmaceutical and non-pharmaceutical interventions for controlling the
COVID-19 pandemic.
</td>
<td style="text-align:left;">
Disease spread can be affected by pharmaceutical interventions (such as
vaccination) and non-pharmaceutical interventions (such as physical
distancing, mask-wearing and contact tracing). Understanding the
relationship between disease dynamics and human behaviour is a
significant factor to controlling infections. In this work, we propose a
compartmental epidemiological model for studying how the infection
dynamics of COVID-19 evolves for people with different levels of social
distancing, natural immunity and vaccine-induced immunity. Our model
recreates the transmission dynamics of COVID-19 in Ontario up to
December 2021. Our results indicate that people change their behaviour
based on the disease dynamics and mitigation measures. Specifically,
they adopt more protective behaviour when mandated social distancing
measures are in effect, typically concurrent with a high number of
infections. They reduce protective behaviour when vaccination coverage
is high or when mandated contact reduction measures are relaxed,
typically concurrent with a reduction of infections. We demonstrate that
waning of infection and vaccine-induced immunity are important for
reproducing disease transmission in autumn 2021. © 2023 The Authors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38123810
</td>
<td style="text-align:left;">
Dissecting the heterogeneity of “in the wild” stress from multimodal
sensor data.
</td>
<td style="text-align:left;">
Stress is associated with numerous chronic health conditions, both
mental and physical. However, the heterogeneity of these associations at
the individual level is poorly understood. While data generated from
individuals in their day-to-day lives “in the wild” may best represent
the heterogeneity of stress, gathering these data and separating signals
from noise is challenging. In this work, we report findings from a major
data collection effort using Digital Health Technologies (DHTs) and
frontline healthcare workers. We provide insights into stress “in the
wild”, by using robust methods for its identification from multimodal
data and quantifying its heterogeneity. Here we analyze data from the
Stress and Recovery in Frontline COVID-19 Workers study following 365
frontline healthcare workers for 4-6 months using wearable devices and
smartphone app-based measures. Causal discovery is used to learn how the
causal structure governing an individual’s self-reported symptoms and
physiological features from DHTs differs between non-stress and
potential stress states. Our methods uncover robust representations of
potential stress states across a population of frontline healthcare
workers. These representations reveal high levels of inter- and
intra-individual heterogeneity in stress. We leverage multiple stress
definitions that span different modalities (from subjective to
physiological) to obtain a comprehensive view of stress, as these
differing definitions rarely align in time. We show that these different
stress definitions can be robustly represented as changes in the
underlying causal structure on and off stress for individuals. This
study is an important step toward better understanding potential
underlying processes generating stress in individuals. © 2023. The
Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38118118
</td>
<td style="text-align:left;">
Surviving pandemic control measures: The experiences of female sex
workers during COVID-19 in Nairobi, Kenya.
</td>
<td style="text-align:left;">
At the beginning of the COVID-19 pandemic, the Kenya Ministry of Health
instituted movement cessation measures and limits on face-to-face
meetings. We explore the ways in which female sex workers (FSWs) in
Nairobi were affected by the COVID-19 control measures and the ways they
coped with the hardships. Forty-seven women were randomly sampled from
the Maisha Fiti study, a longitudinal study of 1003 FSWs accessing
sexual reproductive health services in Nairobi for an in-depth
qualitative interview 4-5 months into the pandemic. We sought to
understand the effects of COVID-19 on their lives. Data were
transcribed, translated, and coded inductively. The COVID-19 measures
disenfranchised FSWs reducing access to healthcare, decreasing income
and increasing sexual, physical, and financial abuse by clients and law
enforcement. Due to the customer-facing nature of their work, sex
workers were hit hard by the COVID-19 restrictions. FSWs experienced
poor mental health and strained interpersonal relationships. To cope
they skipped meals, reduced alcohol use and smoking, started small
businesses to supplement sex work or relocated to their rural homes.
Interventions that ensure continuity of access to health services,
prevent exploitation, and ensure the social and economic protection of
FSWs during times of economic strain are required.
</td>
</tr>
<tr>
<td style="text-align:left;">
38117443
</td>
<td style="text-align:left;">
Covid-19 Vaccine Hesitancy and Under-Vaccination among Marginalized
Populations in the United States and Canada: A Scoping Review.
</td>
<td style="text-align:left;">
Amid persistent disparities in Covid-19 vaccination and burgeoning
research on vaccine hesitancy (VH), we conducted a scoping review to
identify multilevel determinants of Covid-19 VH and under-vaccination
among marginalized populations in the U.S. and Canada. Using the scoping
review methodology developed by the Joanna Briggs Institute, we designed
a search string and explored 7 databases to identify peer-reviewed
articles published from January 1, 2020-October 25, 2022. We combine
frequency analysis and narrative synthesis to describe factors
influencing Covid-19 VH and under-vaccination among marginalized
populations. The search captured 11,374 non-duplicated records, scoped
to 103 peer-reviewed articles. Among 14 marginalized populations
identified, African American/Black, Latinx, LGBTQ+, American
Indian/Indigenous, people with disabilities, and justice-involved people
were the predominant focus. Thirty-two factors emerged as influencing
Covid-19 VH, with structural racism/stigma and institutional mistrust
(structural)(n = 71) most prevalent, followed by vaccine safety
(vaccine-specific)(n = 62), side effects (vaccine-specific)(n = 50),
trust in individual healthcare provider (social/community)(n = 38), and
perceived risk of infection (individual)(n = 33). Structural factors
predominated across populations, including structural racism/stigma and
institutional mistrust, barriers to Covid-19 vaccine access due to
limited supply/availability, distance/lack of transportation, no/low
paid sick days, low internet/digital technology access, and lack of
culturally- and linguistically-appropriate information. We identified
multilevel and complex drivers of Covid-19 under-vaccination among
marginalized populations. Distinguishing vaccine-specific, individual,
and social/community factors that may fuel decisional ambivalence, more
appropriately defined as VH, from structural racism/structural stigma
and systemic/institutional barriers to vaccination access may better
support evidence-informed interventions to promote equity in access to
vaccines and informed decision-making among marginalized populations. ©
2023. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38116645
</td>
<td style="text-align:left;">
The Effects of Cognitive Ability, Mental Health, and Self-Quarantining
on Functional Ability of Older Adults During the COVID-19 Pandemic:
Results From the Canadian Longitudinal Study on Aging.
</td>
<td style="text-align:left;">
Determine whether levels of anxiety and depression, cognitive ability,
and self-quarantining during and prior to the pandemic predict decreases
in perceived functional ability. Longitudinal data collected from the
Canadian Longitudinal Study on Aging (CLSA) COVID-19 Questionnaire Study
(2020) and core CLSA study (Follow-Up 1; 2014-2018). 17 541 CLSA
participants. Self-quarantining behaviours from questionnaires
administered at Baseline (April 2020), Monthly, and Exit (December 2020)
time points of the CLSA COVID-19 Questionnaire Study, levels of anxiety
and depression at Baseline, perceived change in functional ability at
Exit, and performance on neuropsychological tests (Rey Auditory Verbal
Learning Task, Mental Alternation Task, Animal Fluency Test) and
functional ability (Older Americans Resources and Services \[OARS\]
Multidimensional Assessment Questionnaire) from the core CLSA study.
Greater cognitive ability pre-pandemic (B = -.003, P &lt; .01), higher
levels of anxiety (B = -.024, P &lt; .01) and depressive symptoms (B =
-.110, P &lt; .01) at Baseline, and higher frequency of engaging in
self-quarantining throughout the COVID-19 survey period (B = -.098, P
&lt; .01) were associated with perceived loss in functional ability at
Exit. Self-quarantining behaviour was associated with perceived loss in
functional ability only at average and high levels of depressive
symptoms (B = -.013, P &lt; .01). Older adults with higher cognitive and
lower functional ability prior to the pandemic were at greater risk of
decreased perceived functional ability during the first year of the
pandemic, as were those who experienced greater levels of anxiety and
depressive symptoms during the pandemic. Strategies/interventions to
preserve functional ability in older adults with cognitive independence
prior to future pandemics are warranted.
</td>
</tr>
<tr>
<td style="text-align:left;">
38114867
</td>
<td style="text-align:left;">
Impact of Elevated Body Mass Index (BMI) on Hedonic Tone in Persons with
Post-COVID-19 Condition: A Secondary Analysis.
</td>
<td style="text-align:left;">
The post-COVID-19 condition (PCC) is characterized by persistent,
distressing symptoms following an acute COVID-19 infection. These
symptoms encompass various domains, including hedonic tone, which is
critical for overall well-being. Furthermore, obesity is both a risk
factor for COVID-19 and PCC and associated with impaired hedonic tone.
This study aims to investigate whether elevated body mass index (BMI) is
associated with hedonic tone in persons with PCC. We perform a post hoc
analysis of a randomized, double-blind, placebo-controlled clinical
trial investigating the impact of vortioxetine on cognitive impairment
in persons with PCC. Statistical analysis of baseline data using a
generalized linear model was undertaken to determine the relationship of
BMI to hedonic tone measured by Snaith-Hamilton Pleasure Scale (SHAPS)
scores. The model was adjusted for covariates including age, sex, race,
suspected versus confirmed COVID-19 cases, alcohol amount consumed per
week, and annual household income. The baseline data of 147 participants
were available for analysis. BMI had a statistically significant
positive association with baseline SHAPS total scores (β = 0.003, 95% CI
\[6.251E-5, 0.006\], p = 0.045), indicating elevated BMI is associated
with deficits in self-reported reward system functioning. Higher BMI is
associated with greater deficits in hedonic tone in persons with PCC,
which may impact reward functioning processes such as reward prediction
and processing. The mediatory effect of BMI on reward function
underscores the need to investigate the neurobiologic interactions to
elucidate preventative and therapeutic interventions for persons with
PCC. Therapeutic development targeting debilitating features of PCC
(e.g., motivation, cognitive dysfunction) could consider stratification
on the basis of baseline BMI. NCT05047952. © 2023. The Author(s), under
exclusive licence to Springer Healthcare Ltd., part of Springer Nature.
</td>
</tr>
<tr>
<td style="text-align:left;">
38113220
</td>
<td style="text-align:left;">
Initiations of safer supply hydromorphone increased during the COVID-19
pandemic in Ontario: An interrupted time series analysis.
</td>
<td style="text-align:left;">
Calls to prescribe safer supply hydromorphone (SSHM) as an alternative
to the toxic drug supply increased during the COVID-19 pandemic but it
is unknown whether prescribing behaviour was altered. We aimed to
evaluate how the number of new SSHM dispensations changed during the
pandemic in Ontario. We conducted a retrospective interrupted
time-series analysis using provincial administrative databases. We
counted new SSHM dispensations in successive 28-day periods from March
22, 2016 to August 30, 2021. We used segmented Poisson regression
methods to test for both a change in level and trend of new
dispensations before and after March 17, 2020, the date Ontario’s
pandemic-related emergency was declared. We adjusted the models to
account for seasonality and assessed for over-dispersion and residual
autocorrelation. We used counterfactual analysis methods to estimate the
number of new dispensations attributable to the pandemic. We identified
1489 new SSHM dispensations during the study period (434 \[mean of 8 per
28-day period\] before and 1055 \[mean of 56 per 28-day period\] during
the pandemic). Median age of individuals initiating SSHM was 40
(interquartile interval 33-48) with 61.7% (N = 919) male sex. Before the
pandemic, there was a small trend of increased prescribing (incidence
rate ratio \[IRR\] per period 1.002; 95% confidence interval \[95CI\]
1.001-1.002; p&lt;0.001), with a change in level (immediate increase) at
the pandemic date (relative increase in IRR 1.674; 95CI 1.206-2.322; p =
0.002). The trend during the pandemic was not statistically significant
(relative increase in IRR 1.000; 95CI 1.000-1.001; p = 0.251). We
estimated 511 (95CI 327-695) new dispensations would not have occurred
without the pandemic. The pandemic led to an abrupt increase in SSHM
prescribing in Ontario, although the rate of increase was similar before
and during the pandemic. The absolute number of individuals who accessed
SSHM remained low throughout the pandemic. Copyright: © 2023 Young et
al. This is an open access article distributed under the terms of the
Creative Commons Attribution License, which permits unrestricted use,
distribution, and reproduction in any medium, provided the original
author and source are credited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38112009
</td>
<td style="text-align:left;">
Patient and families’ perspectives on telepalliative care: A systematic
integrative review.
</td>
<td style="text-align:left;">
Telepalliative care is increasingly used in palliative care, but has yet
to be examined from a patient and family perspective. A synthesis of
evidence may provide knowledge on how to plan and provide telepalliative
care that caters specifically to patients and families’ needs. To
synthesise evidence on patients and families’ perspectives on
telepalliative care. A systematic integrative review (PROSPERO
\#CRD42022301206) reported in accordance with PRISMA 2020 guidelines.
Inclusion criteria; primary peer-reviewed studies published 2011-2022,
patient and family perspective, &gt;18 years, telepalliative care and
English/Danish language. Quality was appraised using the mixed-methods
appraisal tool, version 2020. Guided by Toronto and Remington, data were
extracted, thematically analysed and synthesised. MEDLINE, EMBASE,
PsycINFO and CINAHL were searched in March 2022 and updated in February
2023. Forty-four studies were included. Analysis revealed five themes;
the effect of the Covid-19 pandemic on telepalliative care, adding value
for patients and families, synchronous and asynchronous telepalliative
care, the integration of telepalliative care with other services and the
tailoring and timing of telepalliative care. Enhanced access to care and
convenience, as attributes of telepalliative care, are highly valued.
Patients and families have varying needs during the illness trajectory
that may be addressed by early integration of telepalliative care based
on models of care that are flexible and combine synchronous and
asynchronous solutions. Further research should examine telepalliative
care in a post-pandemic context, use of models of care and identify
meaningful outcome measures from patient and family perspectives for
evaluation of telepalliative care.
</td>
</tr>
<tr>
<td style="text-align:left;">
38111331
</td>
<td style="text-align:left;">
Telemedicine-Based Cognitive Examinations During COVID-19 and Beyond:
Perspective of the Massachusetts General Hospital Behavioral Neurology
&amp; Neuropsychiatry Group.
</td>
<td style="text-align:left;">
Telehealth and telemedicine have encountered explosive growth since the
beginning of the COVID-19 pandemic, resulting in increased access to
care for patients located far from medical centers and clinics.
Subspecialty clinicians in behavioral neurology &amp; neuropsychiatry
(BNNP) have implemented the use of telemedicine platforms to perform
cognitive examinations that were previously office based. In this
perspective article, BNNP clinicians at Massachusetts General Hospital
(MGH) describe their experience performing cognitive examinations via
telemedicine. The article reviews the goals, prerequisites, advantages,
and potential limitations of performing a video- or telephone-based
telemedicine cognitive examination. The article shares the approaches
used by MGH BNNP clinicians to examine cognitive and behavioral areas,
such as orientation, attention and executive functions, language, verbal
learning and memory, visual learning and memory, visuospatial function,
praxis, and abstract abilities, as well as to survey for
neuropsychiatric symptoms and assess activities of daily living.
Limitations of telemedicine-based cognitive examinations include limited
access to and familiarity with telecommunication technologies on the
patient side, limitations of the technology itself on the clinician
side, and the limited psychometric validation of virtual assessments.
Therefore, an in-person examination with a BNNP clinician or a formal
in-person neuropsychological examination with a neuropsychologist may be
recommended. Overall, this article emphasizes the use of standardized
cognitive and behavioral assessment instruments that are either in the
public domain or, if copyrighted, are nonproprietary and do not require
a fee to be used by the practicing BNNP clinician.
</td>
</tr>
<tr>
<td style="text-align:left;">
38110945
</td>
<td style="text-align:left;">
What motivates individuals to share information with governments when
adopting health technologies during the COVID-19 pandemic?
</td>
<td style="text-align:left;">
While digital governance has been adopted by governments around the
world to assist in the management of the COVID-19 pandemic, the
effectiveness of its implementation relies on the collection and use of
personal information. This study examines the willingness of individuals
to engage in information-sharing with governments when adopting health
technologies during the COVID-19 pandemic. Data were obtained from a
cross-sectional survey of 4,800 individuals drawn from 16 cities in
China in 2021. Tobit regression models were used to assess the impacts
of an array of determinants on an individual’s willingness to share
information with governments when adopting health technologies.
Individuals who perceived a higher level of helpfulness, risk,
expectations from others, weariness toward privacy issues, and were
sensitive to positive outcomes were more willing to share information
with governments when adopting health technologies during the COVID-19
pandemic. Across all the subgroups, self-efficacy only reduced the
willingness to share information with governments for individuals who
spent more than seven hours per day online. The negative impacts of
being sensitive to negative outcomes on the willingness to share
information were only found among females and the less educated group.
This study revealed the seemingly paradoxical behavior of individuals
who perceived high risks of sharing information and a sense of fatigue
toward privacy issues yet continued to be willing to share their
information with their governments when adopting health technologies
during the COVID-19 pandemic. This work highlighted significant
differential motivations for sharing information with governments when
using health technologies during a pandemic. Tailored policies that
resonate with population sub-groups were suggested to be proposed to
facilitate crisis management in future situations. © 2023. The
Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38109428
</td>
<td style="text-align:left;">
Disruption to Pattern but No Overall Increase in the Expected Incidence
of Pediatric Diabetes During the First Three Years of the COVID-19
Pandemic in Ontario, Canada (March 2020-March 2023).
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
38107835
</td>
<td style="text-align:left;">
Integrated Care: A Person-Centered and Population Health Strategy for
the COVID-19 Pandemic Recovery and Beyond.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has mandated a re-imagination of how healthcare is
administered and delivered, with a view towards focusing on
person-centred care and advancing population health while increasing
capacity, access and equity in the healthcare system. These goals can be
achieved through healthcare integration. In 2019, the University Health
Network (UHN), a consortium of four quaternary care hospitals in
Ontario, Canada, established the first stage of a pilot program to
increase healthcare integration at the institutional level and
vertically with other primary, secondary and tertiary institutions in
the Ontario healthcare system. Implementation of the program was
accelerated during the COVID-19 pandemic and demonstrated how healthcare
integration improves person-centred care and population health;
therefore serving as the foundation for a health system response for the
COVID-19 pandemic recovery and beyond. Copyright: © 2023 The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38105668
</td>
<td style="text-align:left;">
Practice- and System-Based Interventions to Reduce COVID-19 Transmission
in Primary Care Settings: A Qualitative Study.
</td>
<td style="text-align:left;">
Using qualitative interviews with 68 family physicians (FPs) in Canada,
we describe practice- and system-based approaches that were used to
mitigate COVID-19 exposure in primary care settings across Canada to
ensure the continuation of primary care delivery. Participants described
how they applied infection prevention and control procedures (risk
assessment, hand hygiene, control of environment, administrative
control, personal protective equipment) and relied on centralized
services that directed patients with COVID-19 to settings outside of
primary care, such as testing centres. The multi-layered approach
mitigated the risk of COVID-19 exposure while also conserving resources,
preserving capacity and supporting supply chains. Copyright © 2023
Longwoods Publishing.
</td>
</tr>
<tr>
<td style="text-align:left;">
38105389
</td>
<td style="text-align:left;">
Decision making in continuing professional development organisations
during a crisis.
</td>
<td style="text-align:left;">
Early in COVID-19, continuing professional development (CPD) providers
quickly made decisions about program content, design, funding and
technology. Although experiences during an earlier pandemic cautioned
providers to make disaster plans, CPD was not entirely prepared for this
event. We sought to better understand how CPD organisations make
decisions about CPD strategy and operations during a crisis. This is a
descriptive qualitative research study of decision making in two
organisations: CPD at the University of Toronto (UofT) and the US-based
Society for Academic Continuing Medical Education (SACME). In March
2021, using purposive and snowball sampling, we invited faculty and
staff who held leadership positions to participate in semi-structured
interviews. The interview focused on the individual’s role and
organisation, their decision-making process and reflections on how their
units had changed because of COVID-19. Transcripts were reviewed, coded
and analysed using thematic analysis. We used Mazmanian et al.’s
Ecological Framework as a further conceptual tool. We conducted eight
interviews from UofT and five from SACME. We identified that decision
making during the pandemic occurred over four phases of reactions and
impact from COVID-19, including shutdown, pivot, transition and the ‘new
reality’. The decision-making ability of CPD organisations changed
throughout the pandemic, ranging from having little or no independent
decision-making ability early on to having considerable control over
choosing appropriate pathways forward. Decision making was strongly
influenced by the creativity, adaptability and flexibility of the CPD
community and the need for social connection. This adds to literature on
the changes CPD organisations faced due to COVID-19, emphasising CPD
organisations’ adaptability in making decisions. Applying the Ecological
Framework further demonstrates the importance of time to decision-making
processes and the relational aspect of CPD. To face future crises, CPD
will need to embrace creative, flexible and socially connected
solutions. Future scholarship could explore an organisation’s ability to
rapidly adapt to better prepare for future crises. © 2023 The Authors.
Medical Education published by Association for the Study of Medical
Education and John Wiley &amp; Sons Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38101150
</td>
<td style="text-align:left;">
The impact of COVID-19 three years on: Introduction to the 2023 special
issue.
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
38100397
</td>
<td style="text-align:left;">
Long-term care transitions during a global pandemic: Planning and
decision-making of residents, care partners, and health professionals in
Ontario, Canada.
</td>
<td style="text-align:left;">
The COVID-19 pandemic appears to have shifted the care trajectories of
many residents and care partners in Ontario who considered leaving LTC
to live in the community for a portion or the duration of the pandemic.
This type of care transition-from LTC to home care-was highly uncommon
prior to the pandemic, therefore we know relatively little about the
planning and decision-making involved. The aim of this study was to
describe who was involved in LTC to home care transitions in Ontario
during the COVID-19 pandemic, to what extent, and the factors that
guided their decision-making. A qualitative description study involving
semi-structured interviews with 32 residents, care partners and health
professionals was conducted. Transition decisions were largely made by
care partners, with varied input from residents or health professionals.
Stakeholders considered seven factors, previously identified in a
scoping review, when making their transition decisions: (a)
institutional priorities and requirements; (b) resources; (c) knowledge;
(d) risk; (e) group structure and dynamic; (f) health and support needs;
and (g) personality preferences and beliefs. Participants’ emotional
responses to the pandemic also influenced the perceived need to pursue a
care transition. The findings of this research provide insights towards
the planning required to support LTC to home care transitions, and the
many challenges that arise during decision-making. Copyright: © 2023
Carbone et al. This is an open access article distributed under the
terms of the Creative Commons Attribution License, which permits
unrestricted use, distribution, and reproduction in any medium, provided
the original author and source are credited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38100009
</td>
<td style="text-align:left;">
The Evolving Roles and Expectations of Inpatient Palliative Care Through
COVID-19: a Systematic Review and Meta-synthesis.
</td>
<td style="text-align:left;">
Palliative care performed a central role in responding to the systemic
suffering incurred by the COVID-19 pandemic. Yet, few studies have
elucidated the inpatient palliative care specialists’ experiences and
perceptions. Systematically review and synthesize the evolving roles and
expectations of inpatient palliative care specialists in response to
COVID-19. A systematic review and meta-synthesis informed by Thomas and
Harden’s framework and Pozzar et al.’s approach was conducted in
accordance with Preferred Reporting Items for Systematic Reviews and
Meta-Analysis (PRISMA) guidelines. MEDLINE, EMBASE, CINAHL, and PubMed
were systematically searched for articles published between December
2019 and March 2023. We included all peer-reviewed qualitative and
mixed-method literature studying the roles and expectations of inpatient
palliative care specialists. A mixed-method appraisal tool was used for
quality assessment. Of 3869 unique articles, 52 were included. Studies
represented North American (n = 23), European (n = 16), South American
(n = 4), Oceanic (n = 2), Asian (n = 2), West African (n = 1), Middle
Eastern (n = 1), and inter-continental settings (n = 3). Most were
reported in English (n = 50), conducted in 2020 (n = 28), and focused on
the perspectives of inpatient palliative care clinicians (n = 28). Three
descriptive themes captured the roles and expectations of inpatient
palliative care specialists: shifting foundations, reorienting to
relationships, and evolving identity. Two analytical themes were
synthesized: palliative care propagates compassion through a healing
presence, and palliative care enhances the systemic response to
suffering through nimble leadership. Inpatient palliative care
specialists responded to the COVID-19 pandemic by establishing their
healing presence and leading with their adaptability. To develop
institutionally tailored and collaborative responses to future
pandemics, future studies are needed to understand how inpatient
palliative care clinicians are recognized and valued within their
institutions. © 2023. The Author(s), under exclusive licence to Society
of General Internal Medicine.
</td>
</tr>
<tr>
<td style="text-align:left;">
38098159
</td>
<td style="text-align:left;">
Effectiveness of a Fourth COVID-19 mRNA Vaccine Dose Against the Omicron
Variant in Solid Organ Transplant Recipients.
</td>
<td style="text-align:left;">
The effectiveness of booster doses of COVID-19 vaccines in solid organ
transplant recipients is unclear. We conducted a population-based
matched cohort study using linked administrative healthcare databases
from Ontario, Canada to estimate the marginal vaccine effectiveness of a
fourth versus third dose of the BNT162b2 and mRNA-1273 vaccines against
clinically important outcomes (ie, hospitalization or death) and
infection during the era of the Omicron variant. We matched 3120 solid
organ transplant recipients with a third COVID-19 vaccine dose
(reference) to 3120 recipients with a fourth dose. Recipients were
matched on the third dose date (±7 d). We used a multivariable Cox
proportional hazards model to estimate the marginal vaccine
effectiveness with outcomes occurring between December 21, 2021 and
April 30, 2022. The cumulative incidence of COVID-19-related
hospitalization or death was 2.8% (95% confidence interval \[CI\],
2.0-3.7) in the third dose group compared with 1.1% (95% CI, 0.59-1.8)
in the fourth dose group after 84 d of follow-up (P &lt; 0.001). The
adjusted marginal vaccine effectiveness was 70% (95% CI, 47-83) against
clinically important outcomes and 39% (95% CI, 21-52) against SARS-CoV-2
infection. Compared with a third dose, a fourth dose of the COVID-19
vaccine was associated with improved protection against hospitalization,
death, and SARS-CoV-2 infection during the Omicron era. Results
highlight the importance of a booster COVID-19 vaccine dose in solid
organ transplant recipients. Copyright © 2023 Wolters Kluwer Health,
Inc. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38096173
</td>
<td style="text-align:left;">
The mental health impacts of the COVID-19 pandemic among individuals
with depressive, anxiety, and stressor-related disorders: A scoping
review.
</td>
<td style="text-align:left;">
A scoping review of studies published in the first year of the COVID-19
pandemic focused on individuals with pre-existing symptoms of
depression, anxiety, and specified stressor-related disorders, with the
objective of mapping the research conducted. (1) direct study of
individuals with pre-existing depressive, anxiety, and/or specified
stressor-related (i.e., posttraumatic stress, acute stress)
disorders/issues; (2) focus on mental health-related pandemic effects,
and; (3) direct study of mental health symptoms related to depression,
anxiety, or psychological distress. Database-specific subject headings
and natural language keywords were searched in Medline, Embase, APA
PsycInfo, and Cumulative Index to Nursing &amp; Allied Health Literature
(CINAHL) up to March 3, 2021. Review of potentially relevant studies was
conducted by two independent reviewers and proceeded in two stages: (1)
title and abstract review, and; (2) full paper review. Study details
(i.e., location, design and methodology, sample or population, outcome
measures, and key findings) were extracted from included studies by one
reviewer and confirmed by the Principal Investigator. 66 relevant
articles from 26 countries were identified. Most studies adopted a
cross-sectional design and were conducted via online survey. About half
relied on general population samples, with the remainder assessing
special populations, primarily mental health patients. The most commonly
reported pre-existing category of disorders or symptoms was depression,
followed closely by anxiety. Most studies included depressive and
anxiety symptoms as outcome measures and demonstrated increased
vulnerability to mental health symptoms among individuals with a
pre-existing mental health issue. These findings suggest that improved
mental health supports are needed during the pandemic and point to
future research needs, including reviews of other diagnostic categories
and reviews of research published in subsequent years of the pandemic.
Copyright: © 2023 Wickens et al. This is an open access article
distributed under the terms of the Creative Commons Attribution License,
which permits unrestricted use, distribution, and reproduction in any
medium, provided the original author and source are credited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38093007
</td>
<td style="text-align:left;">
A synthesis of evidence for policy from behavioural science during
COVID-19.
</td>
<td style="text-align:left;">
Scientific evidence regularly guides policy decisions1, with behavioural
science increasingly part of this process2. In April 2020, an
influential paper3 proposed 19 policy recommendations (‘claims’)
detailing how evidence from behavioural science could contribute to
efforts to reduce impacts and end the COVID-19 pandemic. Here we assess
747 pandemic-related research articles that empirically investigated
those claims. We report the scale of evidence and whether evidence
supports them to indicate applicability for policymaking. Two
independent teams, involving 72 reviewers, found evidence for 18 of 19
claims, with both teams finding evidence supporting 16 (89%) of those 18
claims. The strongest evidence supported claims that anticipated
culture, polarization and misinformation would be associated with policy
effectiveness. Claims suggesting trusted leaders and positive social
norms increased adherence to behavioural interventions also had strong
empirical support, as did appealing to social consensus or bipartisan
agreement. Targeted language in messaging yielded mixed effects and
there were no effects for highlighting individual benefits or protecting
others. No available evidence existed to assess any distinct differences
in effects between using the terms ‘physical distancing’ and ‘social
distancing’. Analysis of 463 papers containing data showed generally
large samples; 418 involved human participants with a mean of 16,848
(median of 1,699). That statistical power underscored improved
suitability of behavioural science research for informing policy
decisions. Furthermore, by implementing a standardized approach to
evidence selection and synthesis, we amplify broader implications for
advancing scientific evidence in policy formulation and prioritization.
© 2023. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38091618
</td>
<td style="text-align:left;">
Delivering health promotion during school closures in public health
emergencies: building consensus among Canadian experts.
</td>
<td style="text-align:left;">
School-based health promotion is drastically disrupted by school
closures during public health emergencies or natural disasters. Climate
change will likely accelerate the frequency of these events and hence
school closures. We identified innovative health promotion practices
delivered during COVID-19 school closures and sought consensus among
education experts on their future utility. Fifteen health promotion
practices delivered in 87 schools across Alberta, Canada during COVID-19
school closures in Spring 2020, were grouped into: ‘awareness of healthy
lifestyle behaviours and mental wellness’, ‘virtual events’, ‘tangible
supports’ and ‘school-student-family connectedness’. Two expert panels
(23 school-level practitioners and 20 decision-makers at the school
board and provincial levels) rated practices on feasibility,
acceptability, reach, effectiveness, cost-effectiveness and other
criteria in three rounds of online Delphi surveys. Consensus was reached
if 70% or more participants (strongly) agreed with a statement,
(strongly) disagreed or neither. Participants agreed all practices
require planning, preparation and training before implementation and
additional staff time and most require external support or partnerships.
Participants rated ‘awareness of healthy lifestyle behaviours and mental
wellness’ and ‘virtual events’ as easy and quick to implement, effective
and cost-effective, sustainable, easy to integrate into curriculum, well
received by students and teachers, benefit school culture and require no
additional funding/resources. ‘Tangible supports’ (equipment, food) and
‘school-student-family connectedness’ were rated as most likely to reach
vulnerable students and families. Health promotion practices presented
herein can inform emergency preparedness plans and are critical to
ensuring health remains a priority during public health emergencies and
natural disasters. © The Author(s) 2023. Published by Oxford University
Press.
</td>
</tr>
<tr>
<td style="text-align:left;">
38090725
</td>
<td style="text-align:left;">
Integration of hospital with congregate care homes in response to the
COVID-19 pandemic.
</td>
<td style="text-align:left;">
The coronavirus disease 2019 (COVID-19) pandemic has highlighted the
need to improve the safety of the environments where we care for older
adults in Canada. After providing assistance during the first wave, many
Ontario hospitals formally partnered with local congregate care homes in
a “hub and spoke” model during second pandemic wave onward. The
objective of this article is to describe the implementation and
longitudinal outcomes of residents in one hub and spoke model composed
of a hospital partnered with 18 congregate care homes including four
long-term care and 14 retirement or other congregate care homes. Homes
were provided continuous seven-day per week access to hospital support,
including infection prevention and control (IPAC), testing, vaccine
delivery and clinical support as needed. Any COVID-19 exposure or
transmission triggered a same-day meeting to implement initial control
measures. A minimum of weekly on-site visits occurred for long-term care
homes and biweekly for other congregate care homes, with up to daily
on-site presence during outbreaks. Case detection among residents
increased following implementation in context of increased testing, then
decreased post-immunization until the Omicron wave when it peaked. After
adjusting for the correlation within homes, COVID-related mortality
decreased following implementation (OR=0.51, 95% CI, 0.30-0.88; p=0.01).
In secondary analysis, homes without pre-existing IPAC programs had
higher baseline COVID-related mortality rate (OR=19.19, 95% CI,
4.66-79.02; p&lt;0.001) and saw a larger overall decrease during
implementation (3.76% to 0.37%-0.98%) as compared to homes with
pre-existing IPAC programs (0.21% to 0.57%-0.90%). The outcomes for
older adults residing in congregate care homes improved steadily
throughout the COVID-19 pandemic. While this finding is multifactorial,
integration with a local hospital partner supported key interventions
known to protect residents.
</td>
</tr>
<tr>
<td style="text-align:left;">
38087299
</td>
<td style="text-align:left;">
“None of us are lying”: an interpretive description of the search for
legitimacy and the journey to access quality health services by
individuals living with Long COVID.
</td>
<td style="text-align:left;">
Understanding of Long COVID has advanced through patient-led
initiatives. However, research about barriers to accessing Long COVID
services is limited. This study aimed to better understand the need for,
access to, and quality of, Long COVID services. We explored health needs
and experiences of services, including ability of services to address
needs. Our study was informed by the Levesque et al.’s (2013)
“conceptual framework of access to health care.” We used Interpretive
Description, a qualitative approach partly aimed at informing clinical
decisions. We recruited participants across five settings. Participants
engaged in one-time, semi-structured, virtual interviews. Interviews
were transcribed verbatim. We used reflexive thematic analysis. Best
practice to ensure methodological rigour was employed. Three key themes
were generated from 56 interviews. The first theme illustrated the
rollercoaster-like nature of participants’ Long COVID symptoms and the
resulting impact on function and health. The second theme highlighted
participants’ attempts to access Long COVID services. Guidance received
from healthcare professionals and self-advocacy impacted initial access.
When navigating Long COVID services within the broader system,
participants encountered barriers to access around stigma; appointment
logistics; testing and ‘normal’ results; and financial precarity and
affordability of services. The third theme illuminated common factors
participants liked and disliked about Long COVID services. We framed
each sub-theme as the key lesson (stemming from all likes and dislikes)
that, if acted upon, the health system can use to improve the quality of
Long COVID services. This provides tangible ways to improve the system
based directly on what we heard from participants. With Long COVID
services continuously evolving, our findings can inform decision makers
within the health system to better understand the lived experiences of
Long COVID and tailor services and policies appropriately. © 2023. The
Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38086432
</td>
<td style="text-align:left;">
Impact of Antenatal Care Modifications on Gestational Diabetes Outcomes
During the Corona Virus 19 Pandemic.
</td>
<td style="text-align:left;">
Many of the adverse outcomes of gestational diabetes mellitus (GDM) are
linked to excessive fetal growth, which is strongly mediated by the
adequacy of maternal glycemic control. The COVID-19 pandemic led to a
rapid adoption of virtual care models. We aimed to compare glycemic
control, fetal growth, and perinatal outcomes before and during the
COVID-19 pandemic. A retrospective cohort study was conducted between
2017 and 2020. Singleton pregnancies complicated by GDM were included in
the study. The cohort was stratified into “before” and “during” COVID-19
subgroups, using March 11, 2020 as the demarcation time point. Women who
began their GDM follow up starting March 11, 2020 and thereafter were
allocated to the COVID-19 era, whereas women who delivered before the
demarcation point served as the pre-COVID-19 era. The primary outcome
was the rate of large-for-gestational-age (LGA) neonates. Secondary
outcomes included select maternal and neonatal adverse outcomes. Seven
hundred seventy-five women were included in the analysis, of which 187
(24.13%) were followed during the COVID-19 era and 588 (75.87%) before
the COVID-19 era. One hundred seventy-one of the 187 women (91.44%)
followed during COVID-19 had at least 1 virtual follow-up visit. No
virtual follow-up visits occurred before the COVID-19 era. There was no
difference in the rate of LGA neonates between groups on both univariate
(5.90% vs 7.30%, p=0.5) and multivariate analyses, controlling for age,
ethnicity, parity, body mass index, gestational weight gain, chronic
hypertension, smoking, and hypertensive disorders in pregnancy (adjusted
odds ratio \[aOR\] 1.11, 95% confidence interval \[CI\] 0.49 to 2.51,
p=0.80). In the multivariate analysis, there was no difference in
composite neonatal outcome between groups (GDM diet: aOR 1.40, 95% CI
0.81 to 2.43, p=0.23; GDM medical treatment: aOR 1.20, 95% CI 0.63 to
2.43, p=0.5). After adjusting for differences in baseline variables, the
combined virtual mode of care was not associated with a higher rate of
LGA neonates or other adverse perinatal outcomes in women with GDM.
Larger studies are needed to better understand the specific impact of
virtual care on less common outcomes in pregnancies with GDM. Copyright
© 2023 Canadian Diabetes Association. Published by Elsevier Inc. All
rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38086185
</td>
<td style="text-align:left;">
Unintentional pediatric poisonings before and during the COVID-19
pandemic: A population-based study.
</td>
<td style="text-align:left;">
The impact of coronavirus disease 2019 (COVID-19) on unintentional
pediatric poisonings is unclear. We examined changes in emergency
department (ED) visits and hospitalizations for poisonings before and
during the COVID-19 pandemic. We compared changes in cannabis vs
non-cannabis poisoning events given the recent legalization of cannabis
in October 2018 and cannabis edibles in January 2020. Interrupted
time-series (ITS) analyses of changes in population-level ED visits and
hospitalizations for poisonings in children aged 0-9 years in Ontario,
Canada (annual population of 1.4 million children), over two time
periods: pre-pandemic (January 2010-March 2020) and pandemic (April
2020-December 2021). Overall, there were 28,292 ED visits and 2641
hospitalizations for unintentional poisonings. During the pandemic,
poisonings per 100,000 person-years decreased by 14.6% for ED visits
(40.15 pre- vs. 34.29 during) and increased by 35.9% for
hospitalizations (3.48 pre- vs. 4.73 during). ED visits dropped
immediately (Incidence Rate Ratio \[IRR\], 0.76; 95% CI, 0.70-0.82) at
the onset of the pandemic, followed by a gradual return to baseline
(quarterly change, IRR 1.04, 95%CI 1.03-1.06), while hospitalizations
had an immediate increase (IRR 1.34; 95% CI, 1.08-1.66) and no gradual
change. The only increase in poisonings was for cannabis which had a
10.7-fold for ED visits (0.45 to 4.83 per 100,000 person-years) and a
12.1-fold increase for hospitalizations (0.16 to 1.91 per 100,000
person-years). Excluding cannabis, there was no overall increase in
poisoning hospitalizations. The COVID-19 pandemic was not associated
with increases in any type of unintentional pediatric poisonings, with
the exception of cannabis poisonings. Increased cannabis poisonings may
be explained by the legalization of non-medical cannabis edibles in
Canada in January 2020. Copyright © 2023 Elsevier Inc. All rights
reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38086061
</td>
<td style="text-align:left;">
Defining the Clinicoradiologic Syndrome of SARS-CoV-2 Acute Necrotizing
Encephalopathy: A Systematic Review and 3 New Pediatric Cases.
</td>
<td style="text-align:left;">
We characterize clinical and neuroimaging features of SARS-CoV-2-related
acute necrotizing encephalopathy (ANE). Systematic review of English
language publications in PubMed and reference lists between January 1,
2020, and June 30, 2023, in accordance with PRISMA guidelines. Patients
with SARS-CoV-2 infection who fulfilled diagnostic criteria for sporadic
and genetic ANE were included. From 899 articles, 20 cases (17 single
case reports and 3 additional cases) were curated for review (50%
female; 8 were children). Associated COVID-19 illnesses were febrile
upper respiratory tract infections in children while adults had
pneumonia (45.6%) and myocarditis (8.2%). Children had early neurologic
deterioration (median day 2 in children vs day 4 in adults), seizures (5
(62.5%) children vs 3 of 9 (33.3%) adults), and motor abnormalities (6
of 7 (85.7%) children vs 3 of 7 (42.9%) adults). Eight of 12 (66.7%)
adults and 4 (50.0%) children had high-risk ANE scores. Five (62.5%)
children and 12 (66.7%) adults had brain lesions bilaterally and
symmetrically in the putamina, external capsules, insula cortex, or
medial temporal lobes, in addition to typical thalamic lesions of ANE.
Hypotension was only seen in adults (30%). Hematologic derangements were
common: lymphopenia (66.7%), coagulopathy (60.0%), or elevated D-dimers
(100%), C-reactive protein (91.7%), and ferritin (62.5%). A pathogenic
heterozygous c/.1754 C&gt;T variant in RANBP2 was present in 2 children:
one known to have this before SARS-CoV-2 infection, and a patient tested
because the SARS-CoV-2 infection was the second encephalopathic illness.
Three other children with no prior encephalopathy or family history of
encephalopathy were negative for this variant. Fifteen (75%) received
immunotherapy (with IV methylprednisolone, immunoglobulins, tocilizumab,
or plasma exchange): 6 (40.0%) with monotherapy and 9 (60.0%) had
combination therapy. Deaths were in 8 of 17 with data (47.1%): a
2-month-old male infant and 7 adults (87.5%) of median age 56 years
(33-70 years), 4 of whom did not receive immunotherapy. Children and
adults with SARS-CoV-2 ANE have similar clinical features and
neuroimaging characteristics. Mortality is high, predominantly in
patients not receiving immunotherapy and at the extremes of age.
</td>
</tr>
<tr>
<td style="text-align:left;">
38083979
</td>
<td style="text-align:left;">
Risk of COVID-19 death for people with a pre-existing cancer diagnosis
prior to COVID-19-vaccination: A systematic review and meta-analysis.
</td>
<td style="text-align:left;">
While previous reviews found a positive association between pre-existing
cancer diagnosis and COVID-19-related death, most early studies did not
distinguish long-term cancer survivors from those recently
diagnosed/treated, nor adjust for important confounders including age.
We aimed to consolidate higher-quality evidence on risk of
COVID-19-related death for people with recent/active cancer (compared to
people without) in the pre-COVID-19-vaccination period. We searched the
WHO COVID-19 Global Research Database (20 December 2021), and Medline
and Embase (10 May 2023). We included studies adjusting for age and sex,
and providing details of cancer status. Risk-of-bias assessment was
based on the Newcastle-Ottawa Scale. Pooled adjusted odds or risk ratios
(aORs, aRRs) or hazard ratios (aHRs) and 95% confidence intervals (95%
CIs) were calculated using generic inverse-variance random-effects
models. Random-effects meta-regressions were used to assess associations
between effect estimates and time since cancer diagnosis/treatment. Of
23 773 unique title/abstract records, 39 studies were eligible for
inclusion (2 low, 17 moderate, 20 high risk of bias). Risk of
COVID-19-related death was higher for people with active or recently
diagnosed/treated cancer (general population: aOR = 1.48, 95% CI:
1.36-1.61, I2 = 0; people with COVID-19: aOR = 1.58, 95% CI: 1.41-1.77,
I2 = 0.58; inpatients with COVID-19: aOR = 1.66, 95% CI: 1.34-2.06, I2 =
0.98). Risks were more elevated for lung (general population: aOR = 3.4,
95% CI: 2.4-4.7) and hematological cancers (general population: aOR =
2.13, 95% CI: 1.68-2.68, I2 = 0.43), and for metastatic cancers.
Meta-regression suggested risk of COVID-19-related death decreased with
time since diagnosis/treatment, for example, for any/solid cancers,
fitted aOR = 1.55 (95% CI: 1.37-1.75) at 1 year and aOR = 0.98 (95% CI:
0.80-1.20) at 5 years post-cancer diagnosis/treatment. In conclusion,
before COVID-19-vaccination, risk of COVID-19-related death was higher
for people with recent cancer, with risk depending on cancer type and
time since diagnosis/treatment. © 2023 The Authors. International
Journal of Cancer published by John Wiley &amp; Sons Ltd on behalf of
UICC.
</td>
</tr>
<tr>
<td style="text-align:left;">
38082295
</td>
<td style="text-align:left;">
Publisher Correction: Early corticosteroids are associated with lower
mortality in critically ill patients with COVID‑19: a cohort study.
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
38077465
</td>
<td style="text-align:left;">
Forensic investigations of disasters: Past achievements and new
directions.
</td>
<td style="text-align:left;">
In the 2020s, understanding disaster risk requires a strong and clear
recognition of values and goals that influence the use of political and
economic power and social authority to guide growth and development.
This configuration of values, goals, power and authority may also lead
to concrete drivers of risk at any one time. Building on previous
disaster risk frameworks and experiences from practice, since 2010, the
‘Forensic Investigations of Disasters (FORIN)’ approach has been
developed to support transdisciplinary research on the transformational
pathways societies may follow to recognise and address root causes and
drivers of disaster risk. This article explores and assesses the
achievements and failures of the FORIN approach. It also focuses on
shedding light upon key requirements for new approaches and
understandings of disaster risk research. The new requirements stem not
only from the uncompleted ambitions of FORIN and the forensic approach
but also from dramatic and ongoing transformational changes
characterised by climate change, the coronavirus disease 2019 (COVID-19)
pandemic and the threat of global international confrontation, among
other potential crises, both those that can be identified and those not
yet identified or unknown. Disasters associated with extreme natural
events cannot be treated in isolation. A comprehensive “all risks” or
“all disasters” approach is essential for a global transformation, which
could lead to a better world order. To achieve this, an
Intergovernmental Panel for Disaster Risk is suggested to assess risk
science periodically and work towards sustainability, human rights, and
accountability, within a development and human security frame and on a
systemic basis and integrated perspective. © 2023. The Authors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38075698
</td>
<td style="text-align:left;">
Unlocking the potential of RNA-based therapeutics in the lung: current
status and future directions.
</td>
<td style="text-align:left;">
Awareness of RNA-based therapies has increased after the widespread
adoption of mRNA vaccines against SARS-CoV-2 during the COVID-19
pandemic. These mRNA vaccines had a significant impact on reducing lung
disease and mortality. They highlighted the potential for rapid
development of RNA-based therapies and advances in nanoparticle delivery
systems. Along with the rapid advancement in RNA biology, including the
description of noncoding RNAs as major products of the genome, this
success presents an opportunity to highlight the potential of RNA as a
therapeutic modality. Here, we review the expanding compendium of
RNA-based therapies, their mechanisms of action and examples of
application in the lung. The airways provide a convenient conduit for
drug delivery to the lungs with decreased systemic exposure. This review
will also describe other delivery methods, including local delivery to
the pleura and delivery vehicles that can target the lung after systemic
administration, each providing access options that are advantageous for
a specific application. We present clinical trials of RNA-based therapy
in lung disease and potential areas for future directions. This review
aims to provide an overview that will bring together researchers and
clinicians to advance this burgeoning field. Copyright © 2023 Man,
Moosa, Singh, Wu, Granton, Juvet, Hoang and de Perrot.
</td>
</tr>
<tr>
<td style="text-align:left;">
38074111
</td>
<td style="text-align:left;">
Eleven-month SARS-CoV-2 binding antibody decay, and associated factors,
among mRNA vaccinees: implications for booster vaccination.
</td>
<td style="text-align:left;">
We examined the 11 month longitudinal antibody decay among two-dose mRNA
vaccinees, and identified factors associated with faster decay. The
study included samples from the COVID-19 Occupational Risk,
Seroprevalence and Immunity among Paramedics (CORSIP) longitudinal
observational study of paramedics in Canada. Participants were included
if they had received two mRNA vaccines without prior SARS-CoV-2
infection and provided two blood samples post-vaccination. The outcomes
of interest were quantitative SARS-CoV-2 antibody concentrations. We
employed spaghetti and scatter plots (with kernel-weighted local
polynomial smoothing curve) to describe the trend of the antibody decay
over 11 months post-vaccine and fit a mixed effect exponential decay
model to examine the loss of immunogenicity and factors associated with
antibody waning over time. This analysis included 652 blood samples from
326 adult paramedics. Total anti-spike antibody levels peaked on the
twenty-first day (antibody level 9042 U ml-1) after the second mRNA
vaccine dose. Total anti-spike antibody levels declined thereafter, with
a half-life of 94 \[95 % CI: 70, 143\] days, with levels plateauing at
295 days (antibody level 1021 U ml-1). Older age, vaccine dosing
interval &lt;35 days, and the BNT162b2 vaccine (compared to mRNA-1273
vaccine) were associated with faster antibody decay. Antibody levels
declined after the initial mRNA series with a half-life of 94 days,
plateauing at 295 days. These findings may inform the timing of booster
vaccine doses and identifying individuals with faster antibody decay. ©
2023 The Authors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38074102
</td>
<td style="text-align:left;">
The relationship between the number of COVID-19 vaccines and infection
with Omicron ACE2 inhibition at 18-months post initial vaccination in an
adult cohort of Canadian paramedics.
</td>
<td style="text-align:left;">
The coronavirus disease 2019 (COVID-19) pandemic, caused by the
SARS-CoV-2 virus, has rapidly evolved since late 2019, due to highly
transmissible Omicron variants. While most Canadian paramedics have
received COVID-19 vaccination, the optimal ongoing vaccination strategy
is unclear. We investigated neutralizing antibody (NtAb) response
against wild-type (WT) Wuhan Hu-1 and Omicron BA.4/5 lineages based on
the number of doses and past SARS-CoV-2 infection, at 18 months
post-initial vaccination (with a Wuhan Hu-1 platform mRNA vaccine
\[BNT162b2 or mRNA-1273\]). Demographic information, previous COVID-19
vaccination, infection history, and blood samples were collected from
paramedics 18 months post-initial mRNA COVID-19 vaccine dose. Outcome
measures were ACE2 percent inhibition against Omicron BA.4/5 and WT
antigens. We compared outcomes based on number of vaccine doses (two
vs. three) and previous SARS-CoV-2 infection status, using the
Mann-Whitney U test. Of 657 participants, the median age was 40 years
(IQR 33-50) and 251 (42 %) were females. Overall, median percent
inhibition to BA.4/5 and WT was 71.61 % (IQR 39.44-92.82) and 98.60 %
(IQR 83.07-99.73), respectively. Those with a past SARS-CoV-2 infection
had a higher median percent inhibition to BA.4/5 and WT, when compared
to uninfected individuals overall and when stratified by two or three
vaccine doses. When comparing two vs. three WT vaccine doses among
SARS-CoV-2 negative participants, we did not detect a difference in
BA.4/5 percent inhibition, but there was a difference in WT percent
inhibition. Among those with previous SARS-CoV-2 infection(s), when
comparing two vs. three WT vaccine doses, there was no observed
difference between groups. These findings demonstrate that additional
Whttps://www.covid19immunitytaskforce.ca/citf-databank/#accessing
<https://www.covid19immunitytaskforce.ca/citf-databank/#accessinguhan>
Hu-1 platform mRNA vaccines did not improve NtAb response to BA.4/5, but
prior SARS-CoV-2 infection enhances NtAb response. © 2023 The Authors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38073634
</td>
<td style="text-align:left;">
Electrophysiological neuromuscular alterations and severe fatigue
predict long-term muscle weakness in survivors of COVID-19 acute
respiratory distress syndrome.
</td>
<td style="text-align:left;">
Long-term weakness is common in survivors of COVID-19-associated acute
respiratory distress syndrome (CARDS). We longitudinally assessed the
predictors of muscle weakness in patients evaluated 6 and 12 months
after intensive care unit discharge with in-person visits. Muscle
strength was measured by isometric maximal voluntary contraction (MVC)
of the tibialis anterior muscle. Candidate predictors of muscle weakness
were follow-up time, sex, age, mechanical ventilation duration, use of
steroids in the intensive care unit, the compound muscle action
potential of the tibialis anterior muscle (CMAP-TA-S100), a 6-min walk
test, severe fatigue, depression and anxiety, post-traumatic stress
disorder, cognitive assessment, and body mass index. We also compared
the clinical tools currently available for the evaluation of muscle
strength (handgrip strength and Medical Research Council sum score) and
electrical neuromuscular function (simplified peroneal nerve test
\[PENT\]) with more objective and robust measures of force (MVC) and
electrophysiological evaluation of the neuromuscular function of the
tibialis anterior muscle (CMAP-TA-S100) for their essential role in
ankle control. MVC improved at 12 months compared with 6 months.
CMAP-TA-S100 (P = 0.016) and the presence of severe fatigue (P = 0.036)
were independent predictors of MVC. MVC was strongly associated with
handgrip strength, whereas CMAP-TA-S100 was strongly associated with
PENT. Electrical neuromuscular abnormalities and severe fatigue are
independently associated with reduced MVC and can be used to predict the
risk of long-term muscle weakness in CARDS survivors. Copyright © 2023
Benedini, Cogliati, Lulic-Kuryllo, Peli, Mombelli, Calza, Guarneri,
Cudicio, Rizzardi, Bertoni, Gazzina, Renzi, Gitti, Rasulo, Goffi, Pozzi,
Orizio, Negro, Latronico and Piva.
</td>
</tr>
<tr>
<td style="text-align:left;">
38072756
</td>
<td style="text-align:left;">
Test negative design for vaccine effectiveness estimation in the context
of the COVID-19 pandemic: A systematic methodology review.
</td>
<td style="text-align:left;">
During the height of the global COVID-19 pandemic, the test-negative
design (TND) was extensively used in many countries to evaluate COVID-19
vaccine effectiveness (VE). Typically, the TND involves the recruitment
of care-seeking individuals who meet a common clinical case definition.
All participants are then tested for an infection of interest. To review
and describe the variation in TND methodology, and disclosure of
potential biases, as applied to the evaluation of COVID-19 VE during the
early vaccination phase of the pandemic. We conducted a systematic
review by searching four biomedical databases using defined keywords to
identify peer-reviewed articles published between January 1, 2020, and
January 25, 2022. We included only original articles that employed a TND
to estimate VE of COVID-19 vaccines in which cases and controls were
evaluated based on SARS-CoV-2 laboratory test results. We identified 96
studies, 35 of which met the defined criteria. Most studies were from
North America (16 studies) and targeted the general population (28
studies). Outcome case definitions were based primarily on COVID-19-like
symptoms; however, several papers did not consider or specify symptoms.
Cases and controls had the same inclusion criteria in only half of the
studies. Most studies relied upon administrative or hospital databases
assembled for a different (non-evaluation) clinical purpose. Potential
unmeasured confounding (20 studies), misclassification of current
SARS-CoV-2 infection (16 studies) and selection bias (10 studies) were
disclosed as limitations by some studies. We observed potentially
meaningful deviations from the validated design in the application of
the TND during the COVID-19 pandemic. Copyright © 2023 Elsevier Ltd. All
rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38071626
</td>
<td style="text-align:left;">
Pharmacists’ role and experiences with delivering mental health care
within team-based primary care settings during the COVID-19 pandemic.
</td>
<td style="text-align:left;">
Pharmacists have been increasingly integrated into primary care teams,
leading to improved health outcomes for patients. The two objectives of
this study were (i) to describe how the COVID-19 pandemic impacted
pharmacists’ role in mental health care within Canadian primary care
teams and (ii) to describe Canadian pharmacists’ experiences
collaborating with other healthcare providers in the delivery of mental
health services during the COVID-19 pandemic. Cross-sectional
observational study utilizing an online survey consisting of
closed-ended and open-ended questions. Primary care pharmacists in
Ontario were eligible to participate. Descriptive statistics were
collated, and qualitative data underwent thematic analysis. A total of
51 pharmacists participated in the study. The COVID-19 pandemic has led
to the expanding role of pharmacists in attending to the mental health
care of patients. Working within a collaborative, interprofessional
healthcare environment, pharmacists support patients’ mental health in a
variety of ways, including medication education and management,
non-pharmacologic approaches and supportive conversations, and
identification of resources, including referrals, wellness checks, and
consulting with physicians. Increasing demand for mental health services
has led to higher referrals to pharmacists, which will likely persist
and require further education of pharmacists in mental health along with
better access to deliver virtual care. In response to the increasing
mental health care needs of patients since the COVID-19 pandemic,
primary care pharmacists reported increased attention spent on mental
health care. Building capacity and ensuring support for pharmacists to
continue to address the increasing mental health care demands is
essential. © The Author(s) 2023. Published by Oxford University Press on
behalf of the Royal Pharmaceutical Society.
</td>
</tr>
<tr>
<td style="text-align:left;">
38066033
</td>
<td style="text-align:left;">
Canadian Covid-19 pandemic public health mitigation measures at the
province level.
</td>
<td style="text-align:left;">
The Covid-19 pandemic has prompted governments across the world to
enforce a range of public health interventions. We introduce the
Covid-19 Policy Response Canadian tracker (CPRCT) database that tracks
and records implemented public health measures in every province and
territory in Canada. The implementations are recorded on a four-level
ordinal scale (0-3) for three domains, (Schools, Work, and Other),
capturing differences in degree of response. The data-set allows the
exploration of the effects of public health mitigation on the spread of
Covid-19, as well as provides a near-real-time record in an accessible
format that is useful for a diverse range of modeling and research
questions. © 2023. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38064711
</td>
<td style="text-align:left;">
Figure Correction: Using Social Media to Help Understand
Patient-Reported Health Outcomes of Post-COVID-19 Condition: Natural
Language Processing Approach.
</td>
<td style="text-align:left;">
\[This corrects the article DOI: 10.2196/45767.\]. ©Elham Dolatabadi,
Diana Moyano, Michael Bales, Sofija Spasojevic, Rohan Bhambhoria, Junaid
Bhatti, Shyamolima Debnath, Nicholas Hoell, Xin Li, Celine Leng, Sasha
Nanda, Jad Saab, Esmat Sahak, Fanny Sie, Sara Uppal, Nirma Khatri
Vadlamudi, Antoaneta Vladimirova, Artur Yakimovich, Xiaoxue Yang, Sedef
Akinli Kocak, Angela M Cheung. Originally published in the Journal of
Medical Internet Research (<https://www.jmir.org>), 08.12.2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38064393
</td>
<td style="text-align:left;">
Worldwide scientific efforts on nursing in the field of SARS-CoV-2: a
cross-sectional survey analysis.
</td>
<td style="text-align:left;">
Severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2) infection
has been a global public health issue. This study aimed to characterize
global nursing research on SARS-CoV-2. Nursing-related publications
through December 31, 2022, were identified using Scopus. The number of
studies, study types, countries, institutions, journals, authors,
h-index, total confirmed cases, total deaths, and the highest-cited
studies were investigated. In total, 12,427 studies were identified. The
number of studies increased rapidly, particularly between 2020 and 2021,
with a 2.36-fold increase. The United States published the most studies
(3,289, 26.47%), followed by the United Kingdom (1,059, 8.52%) and China
(877, 7.06%). Scientific productivity significantly correlated with the
total confirmed cases (r = 0.701, p = 0.024) and total deaths (r =
0.804, p = 0.005). The United States had the highest h-index (80),
followed by China (59), and the United Kingdom (57). The University of
Toronto published the most studies (181), followed by Harvard Medical
School (165), and the University of São Paulo (107). Gravenstein S (23)
was the most prolific author, followed by Mor V (22), and Rosa WE (19).
The International Journal of Environmental Research and Public Health
published the most papers (436), followed by PLOS ONE (219), and BMJ
Open (185). Several countries, institutions, journals, and authors
contributed greatly to SARS-CoV-2-related nursing studies. Countries
with larger numbers of confirmed cases and deaths tended to publish more
nursing studies. The United States, United Kingdom, and China had the
highest quantity and quality of studies. Copyright (c) 2023 Yanping
Xiao, Lele Xiao, Ruizhi Zhu, Xueyin Liu.
</td>
</tr>
<tr>
<td style="text-align:left;">
38064213
</td>
<td style="text-align:left;">
Obesity and Outcomes of Kawasaki Disease and COVID-19-Related
Multisystem Inflammatory Syndrome in Children.
</td>
<td style="text-align:left;">
Obesity may affect the clinical course of Kawasaki disease (KD) in
children and multisystem inflammatory syndrome in children (MIS-C)
associated with COVID-19. To compare the prevalence of obesity and
associations with clinical outcomes in patients with KD or MIS-C. In
this cohort study, analysis of International Kawasaki Disease Registry
(IKDR) data on contemporaneous patients was conducted between January 1,
2020, and July 31, 2022 (42 sites, 8 countries). Patients with MIS-C
(defined by Centers for Disease Control and Prevention criteria) and
patients with KD (defined by American Heart Association criteria) were
included. Patients with KD who had evidence of a recent COVID-19
infection or missing or unknown COVID-19 status were excluded. Patient
demographic characteristics, clinical features, disease course, and
outcome variables were collected from the IKDR data set. Using body mass
index (BMI)/weight z score percentile equivalents, patient weight was
categorized as normal weight (BMI &lt;85th percentile), overweight (BMI
≥85th to &lt;95th percentile), and obese (BMI ≥95th percentile). The
association between adiposity category and clinical features and
outcomes was determined separately for KD and MIS-C patient groups. Of
1767 children, 338 with KD (median age, 2.5 \[IQR, 1.2-5.0\] years;
60.4% male) and 1429 with MIS-C (median age, 8.7 \[IQR, 5.3-12.4\]
years; 61.4% male) were contemporaneously included in the study. For
patients with MIS-C vs KD, the prevalence of overweight (17.1% vs 11.5%)
and obesity (23.7% vs 11.5%) was significantly higher (P &lt; .001),
with significantly higher adiposity z scores, even after adjustment for
age, sex, and race and ethnicity. For patients with KD, apart from
intensive care unit admission rate, adiposity category was not
associated with laboratory test features or outcomes. For patients with
MIS-C, higher adiposity category was associated with worse laboratory
test values and outcomes, including a greater likelihood of shock,
intensive care unit admission and inotrope requirement, and increased
inflammatory markers, creatinine levels, and alanine aminotransferase
levels. Adiposity category was not associated with coronary artery
abnormalities for either MIS-C or KD. In this international cohort
study, obesity was more prevalent for patients with MIS-C vs KD, and
associated with more severe presentation, laboratory test features, and
outcomes. These findings suggest that obesity as a comorbid factor
should be considered at the clinical presentation in children with
MIS-C.
</td>
</tr>
<tr>
<td style="text-align:left;">
38064166
</td>
<td style="text-align:left;">
The Fragility of Scientific Rigour and Integrity in “Sped up Science”:
Research Misconduct, Bias, and Hype and in the COVID-19 Pandemic.
</td>
<td style="text-align:left;">
During the early years of the COVID-19 pandemic, preclinical and
clinical research were sped up and scaled up in both the public and
private sectors and in partnerships between them. This resulted in some
extraordinary advances, but it also raised a range of issues regarding
the ethics, rigour, and integrity of scientific research, academic
publication, and public communication. Many of the failures of
scientific rigour and integrity that occurred during the pandemic were
exacerbated by the rush to generate, disseminate, and implement research
findings, which not only created opportunities for unscrupulous actors
but also compromised the methodological, peer review, and advisory
processes that would usually identify sub-standard research and prevent
compromised clinical or policy-level decisions. While it would be
tempting to attribute these failures of science and its translation
solely to the “unprecedented” circumstances of the COVID-19 pandemic,
the reality is that they preceded the pandemic and will continue to
arise once it is over. Existing strategies for promoting scientific
rigour and integrity need to be made more rigorous, better integrated
into research training and institutional cultures, and made more
sophisticated. They might also need to be modified or supplemented with
other strategies that are fit for purpose not only in public health
emergencies but in any research that is sped-up and scaled up to address
urgent unmet medical needs. © 2023. Journal of Bioethical Inquiry Pty
Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38063756
</td>
<td style="text-align:left;">
NanoBubble-Mediated Oxygenation: Elucidating the Underlying Molecular
Mechanisms in Hypoxia and Mitochondrial-Related Pathologies.
</td>
<td style="text-align:left;">
Worldwide, hypoxia-related conditions, including cancer, COVID-19, and
neuro-degenerative diseases, often lead to multi-organ failure and
significant mortality. Oxygen, crucial for cellular function, becomes
scarce as levels drop below 10 mmHg (&lt;2% O2), triggering
mitochondrial dysregulation and activating hypoxia-induced factors
(HiFs). Herein, oxygen nanobubbles (OnB), an emerging versatile oxygen
delivery platform, offer a novel approach to address hypoxia-related
pathologies. This review explores OnB oxygen delivery strategies and
systems, including diffusion, ultrasound, photodynamic, and
pH-responsive nanobubbles. It delves into the nanoscale mechanisms of
OnB, elucidating their role in mitochondrial metabolism (TFAM,
PGC1alpha), hypoxic responses (HiF-1alpha), and their interplay in
chronic pathologies including cancer and neurodegenerative disorders,
amongst others. By understanding these dynamics and underlying
mechanisms, this article aims to contribute to our accruing knowledge of
OnB and the developing potential in ameliorating hypoxia- and metabolic
stress-related conditions and fostering innovative therapies.
</td>
</tr>
<tr>
<td style="text-align:left;">
38063647
</td>
<td style="text-align:left;">
Surviving the Storm: The Impact of COVID-19 on Cervical Cancer Screening
in Low- and Middle-Income Countries.
</td>
<td style="text-align:left;">
According to the Center for Disease Control and Prevention’s National
Breast and Cervical Cancer Early Detection Program, the cervical cancer
screening rate dropped by 84% soon after the declaration of the COVID-19
pandemic. The challenges facing cervical cancer screening were largely
attributed to the required in-person nature of the screening process and
the measures implemented to control the spread of the virus. While the
impact of the COVID-19 pandemic on cancer screening is well-documented
in high-income countries, less is known about the low- and middle-income
countries that bear 90% of the global burden of cervical cancer deaths.
In this paper, we aim to offer a comprehensive view of the impact of
COVID-19 on cervical cancer screening in LMICs. Using our study,
“Prevention of Cervical Cancer in India through Self-Sampling” (PCCIS),
as a case example, we present the challenges COVID-19 has exerted on
patients, healthcare practitioners, and health systems, as well as
potential opportunities to mitigate these challenges.
</td>
</tr>
<tr>
<td style="text-align:left;">
38063612
</td>
<td style="text-align:left;">
Anxiety and Coping Strategies among Italian-Speaking Physicians: A
Comparative Analysis of the Contractually Obligated and Voluntary Care
of COVID-19 Patients.
</td>
<td style="text-align:left;">
This study aims to explore the differences in the psychological impact
of COVID-19 on physicians, specifically those who volunteered or were
contractually obligated to provide care for COVID-19 patients. While
previous research has predominantly focused on the physical health
consequences and risk of exposure for healthcare workers, limited
attention has been given to their work conditions. This sample comprised
300 physicians, with 68.0% of them men (mean age = 54.67 years; SD =
12.44; range: 23-73). Participants completed measurements including the
State-Trait Anxiety Inventory (STAI), Coping Inventory in Stressful
Situations (CISS), and Coronavirus Anxiety Scale (C.A.S.). Pearson’s
correlations were conducted to examine the relationships between the
variables of interest. This study employed multivariate models to test
the differences between work conditions: (a) involvement in COVID-19
patient care, (b) volunteering for COVID-19 patient management, (c)
contractual obligation to care for COVID-19 patients, and (d) COVID-19
contraction in the workplace. The results of the multivariate analysis
revealed that direct exposure to COVID-19 patients and contractual
obligation to care for them significantly predicted state anxiety and
dysfunctional coping strategies \[Wilks’ Lambda = 0.917 F = 3.254 p &lt;
0.001\]. In contrast, volunteering or being affected by COVID-19 did not
emerge as significant predictors for anxiety or dysfunctional coping
strategies. The findings emphasize the importance of addressing the
psychological well-being of physicians involved in COVID-19 care and
highlight the need for targeted interventions to support their mental
and occupational health.
</td>
</tr>
<tr>
<td style="text-align:left;">
38063440
</td>
<td style="text-align:left;">
Shared health governance, mutual collective accountability, and
transparency in COVAX: A qualitative study triangulating data from
document sampling and key informant interviews.
</td>
<td style="text-align:left;">
To facilitate global COVID-19 vaccine equity, the World Health
Organization, the Coalition for Epidemic Preparedness Innovations, the
Global Alliance for Vaccines and Immunizations, and the United Nations
Children’s Fund supported the COVID-19 Vaccine Global Access (COVAX)
partnership. COVAX’s goals may have best been pursued through shared
health governance - a theory of global health governance based on six
premises, in which global health actors collaborate to achieve a shared
goal. Shared health governance employs a framework for accountability
termed “mutual collective accountability”, in which actors hold each
other accountable for achieving their goal, thus relying on transparency
with one another. We conducted a multi-method qualitative study
triangulating document analysis and key informant interviews to address
the question: To what extent did COVAX employ shared health governance,
mutual collective accountability, and transparency? We thus aimed to
explore the governance structures and accountability and transparency
mechanisms in COVAX and determine whether these constituted shared
health governance and mutual collective accountability. We identified
117 documents and interviewed 20 key informants. Our findings suggest
that COVAX’s co-convening organisations were governed by their
individual formal governance mechanisms, while each was formally
accountable to its own leadership team, resulting in challenges when
activities and decisions involved collaboration between organisations.
Furthermore, COVAX’s governance lacked transparency, as there was little
public information about their decision-making processes and operations,
including information about the algorithm with which they make vaccine
allocation decision, possibly contributing to its inability to achieve
its goals. The COVAX partnership only achieved four of the six premises
of shared health governance. Since actors involved in COVAX did not hold
one another accountable for their role in the partnership, it did not
employ mutual collective accountability, while also lacking in
transparency. Although these results do not entirely explain COVAX’s
shortcomings, they contribute to evidence about the roles of good
governance, transparency, and accountability in large global health
initiatives and underscore failures of the current global governance
system. Copyright © 2023 by the Journal of Global Health. All rights
reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38063155
</td>
<td style="text-align:left;">
Disproportionate Sociodemographic Effects of Suspended Breast Cancer
Screening During the COVID-19 Pandemic.
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
38062437
</td>
<td style="text-align:left;">
Age and sex-related comparison of referral-based telemedicine service
utilization during the COVID-19 pandemic in Ontario: a retrospective
analysis.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has led to increased utilization of telemedicine
services. A retrospective analysis of all referral-based ambulatory
telemedicine services in Ontario from November 2019 to June 2021 was
collected from the Ontario Health Insurance Plan (OHIP) billing
database. Only fee-for-service billings were included in the present
analysis. Coincident COVID-19 cases were obtained from Public Health
Ontario. Comparisons were made based on age bracket, sex, telemedicine
and in-person care. Billings for telemedicine services in Ontario
increased from \$1.7 million CAD in November 2019 to \$64 million CAD in
April 2020 and the proportions reached a mean peak of 72% in April 2020
and declined to 46% in June 2021. A positive correlation was found
between the use of telemedicine and COVID-19 cases (p = 0.05). The age
group with the highest proportion of telemedicine use was the
10-20-year-olds, followed by the 20-50-year-olds (61 ± 9.0%, 55 ± 7.3%,
p = 0.01). Both age groups remained above 50% telemedicine services at
the end of the study period. There seemed to be higher utilization by
females (females 54.2 ± 8.0%, males 47.9 ± 7.7%, ANCOVA p = 0.05) for
all specialties, however, after adjusting for male to female ratio m:f
of 0.952:1.0 according to the 2016 census, this was no longer
significant. The use of telemedicine services remained at a high level
across groups, particularly the 10-50-year-olds. There were clear age
preferences for using telemedicine. Studying these differences may
provide insights into how the delivery of non-hospital-based medicine
has changed during the COVID-19 pandemic. © 2023. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38061186
</td>
<td style="text-align:left;">
Enhancing the value of digital health tools for mental health
help-seeking in Canadian transitional aged youth during the pandemic:
Qualitative study.
</td>
<td style="text-align:left;">
While the COVID-19 pandemic has greatly exacerbated the mental health
challenges of transition-aged youth (TAY) between 17 and 29 years old,
it has also led to the rapid adoption of digital tools for mental health
help-seeking and treatment. However, to date, there has been limited
work focusing on how this shift has impacted perceptions, needs and
challenges of this population in using digital tools. The current study
aims to understand their perspectives on mental health help-seeking
during the pandemic and emerging issues related to digital tools (e.g.,
digital health equity, inclusivity). A total of 16 TAY were invited from
three post-secondary institutions in the Greater Toronto Area. A total
of two streams of focus groups were held and participants were invited
to share their perceptions, needs and experiences. Five main themes were
identified: 1) Helpfulness of a centralized resource encompassing a
variety of diverse mental health supports help-seeking; 2) The impact of
the shift to online mental health support on the use of informal
supports; 3) Digital tool affordability and availability; 4) Importance
of inclusivity for digital tools; and 5) Need for additional support for
mental health seeking and digital tool navigation. Future work should
examine how these needs can be addressed through new and existing
digital mental health help-seeking tools for TAY. Copyright © 2023.
Published by Elsevier B.V.
</td>
</tr>
<tr>
<td style="text-align:left;">
38060560
</td>
<td style="text-align:left;">
Combinatorial design of ionizable lipid nanoparticles for
muscle-selective mRNA delivery with minimized off-target effects.
</td>
<td style="text-align:left;">
Ionizable lipid nanoparticles (LNPs) pivotal to the success of COVID-19
mRNA (messenger RNA) vaccines hold substantial promise for expanding the
landscape of mRNA-based therapies. Nevertheless, the risk of mRNA
delivery to off-target tissues highlights the necessity for LNPs with
enhanced tissue selectivity. The intricate nature of biological systems
and inadequate knowledge of lipid structure-activity relationships
emphasize the significance of high-throughput methods to produce
chemically diverse lipid libraries for mRNA delivery screening. Here, we
introduce a streamlined approach for the rapid design and synthesis of
combinatorial libraries of biodegradable ionizable lipids. This led to
the identification of iso-A11B5C1, an ionizable lipid uniquely apt for
muscle-specific mRNA delivery. It manifested high transfection
efficiencies in muscle tissues, while significantly diminishing
off-targeting in organs like the liver and spleen. Moreover, iso-A11B5C1
also exhibited reduced mRNA transfection potency in lymph nodes and
antigen-presenting cells, prompting investigation into the influence of
direct immune cell transfection via LNPs on mRNA vaccine effectiveness.
In comparison with SM-102, while iso-A11B5C1’s limited immune
transfection attenuated its ability to elicit humoral immunity, it
remained highly effective in triggering cellular immune responses after
intramuscular administration, which is further corroborated by its
strong therapeutic performance as cancer vaccine in a melanoma model.
Collectively, our study not only enriches the high-throughput toolkit
for generating tissue-specific ionizable lipids but also encourages a
reassessment of prevailing paradigms in mRNA vaccine design. This study
encourages rethinking of mRNA vaccine design principles, suggesting that
achieving high immune cell transfection might not be the sole criterion
for developing effective mRNA vaccines.
</td>
</tr>
<tr>
<td style="text-align:left;">
38060296
</td>
<td style="text-align:left;">
Postpandemic Evaluation of the Eco-Efficiency of Personal Protective
Equipment Against COVID-19 in Emergency Departments: Proposal for a
Mixed Methods Study.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has had a profound impact on emergency department
(ED) care in Canada and around the world. To prevent transmission of
COVID-19, personal protective equipment (PPE) was required for all ED
care providers in contact with suspected cases. With mass vaccination
and improvements in several infection prevention components, our
hypothesis is that the risks of transmission of COVID-19 will be
significantly reduced and that current PPE use will have economic and
ecological consequences that exceed its anticipated benefits. Evidence
is needed to evaluate PPE use so that recommendations can ensure the
clinical, economic, and environmental efficiency (ie, eco-efficiency) of
its use. To support the development of recommendations for the
eco-efficient use of PPE, our research objectives are to (1) estimate
the clinical effectiveness (reduced transmission, hospitalizations,
mortality, and work absenteeism) of PPE against COVID-19 for health care
workers; (2) estimate the financial cost of using PPE in the ED for the
management of suspected or confirmed COVID-19 patients; and (3) estimate
the ecological footprint of PPE use against COVID-19 in the ED. We will
conduct a mixed method study to evaluate the eco-efficiency of PPE use
in the 5 EDs of the CHU de Québec-Université Laval (Québec, Canada). To
achieve our goals, the project will include four phases: systematic
review of the literature to assess the clinical effectiveness of PPE
(objective 1; phase 1); cost estimation of PPE use in the ED using a
time-driven activity-based costing method (objective 2; phase 2);
ecological footprint estimation of PPE use using a life cycle assessment
approach (objective 3; phase 3); and cost-consequence analysis and focus
groups (integration of objectives 1 to 3; phase 4). The first 3 phases
have started. The results of these phases will be available in 2023.
Phase 4 will begin in 2023 and results will be available in 2024. While
the benefits of PPE use are likely to diminish as health care workers’
immunity increases, it is important to assess its economic and
ecological impacts to develop recommendations to guide its eco-efficient
use. PROSPERO CRD42022302598;
<https://www.crd.york.ac.uk/prospero/display_record.php?RecordID=302598>.
DERR1-10.2196/50682. ©Simon Berthelot, Yves Longtin, Manuele Margni,
Jason Robert Guertin, Annie LeBlanc, Tania Marx, Khadidiatou Mangou,
Ariane Bluteau, Diego Mantovani, Sergey Mikhaylin, Frédéric Bergeron,
Valérie Dancause, Anne Desjardins, Nadia Lahrichi, Danielle Martin,
Charles Jérôme Sossa, Philippe Lachapelle, Isabelle Genest, Stéphane
Schaal, Anne Gignac, Stéphane Tremblay, Éric Hufty, Lynda Bélanger,
Erica Beatty. Originally published in JMIR Research Protocols
(<https://www.researchprotocols.org>), 07.12.2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38058761
</td>
<td style="text-align:left;">
A case of probable COVID-19 and mononucleosis reactivation complicating
the presentation of travel-acquired measles.
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
38058498
</td>
<td style="text-align:left;">
Randomized trial of the safety and efficacy of anti-SARS-CoV-2 mAb in
the treatment of patients with nosocomial COVID-19 (CATCO-NOS).
</td>
<td style="text-align:left;">
Patients with nosocomial acquisition of COVID-19 have poor outcomes but
have not been included in therapeutic trials to date. A pragmatic
open-label randomized controlled trial of anti-SARS-CoV-2 monoclonal
antibodies (mAb) was performed in hospitalized patients with nosocomial
COVID-19 infection in acute care hospitals spanning a provincial health
care network. Participants within 5 days of first positive test or
symptom onset were randomized to standard of care (SOC) plus a single
dose intravenous mAb treatment (bamlanivimab or casirivimab/imdevimab)
or SOC alone on a 2:1 basis. The primary study endpoint was the need for
invasive mechanical ventilation (IMV) or inpatient mortality by day 60
after randomization. Forty-six participants were enrolled from 13
hospitals between February 14 and October 8, 2021: 31 in the mAb and 15
in the SOC arm. IMV or inpatient mortality up to day 60 occurred in 4
(12.9%) participants in the mAb versus 3 in the SOC arm (20.0%),
difference of -7.1% (95% CI -22.5 to 13.4, p = 0.67). The study was
terminated early due to lack of equipoise as effectiveness of anti-viral
therapies and mAb was published in similar high-risk patient
populations. The trial was underpowered to detect meaningful differences
given its early termination. The study does highlight the feasibility of
undertaking trials in this patient population using a pragmatic approach
allowing for trial participation and treatment access across a large
health care network and may serve as a template for future designs. ©
Association of Medical Microbiology and Infectious Disease Canada (AMMI
Canada), 2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38058495
</td>
<td style="text-align:left;">
Highly pathogenic avian influenza: Unprecedented outbreaks in Canadian
wildlife and domestic poultry.
</td>
<td style="text-align:left;">
Canada experienced a wave of HPAI H5N1 outbreaks in the spring of 2022
with millions of wild and farmed birds being infected. Seabird
mortalities in Canada have been particularly severe on the Atlantic
Coast over the summer of 2022. Over 7 million birds have been culled in
Canada, and outbreaks continue to profoundly affect commercial bird
farms across the world. This new H5N1 virus can and has infected
multiple mammalian species, including skunks, foxes, bears, mink, seals,
porpoises, sea lions, and dolphins. Viruses with mammalian adaptations
such as the mutations PB2-E627K, E627V, and D701N were found in the
brain of various carnivores in Europe and Canada. To date this specific
clade of H5N1 virus has been identified in less than 10 humans. At the
ground level, awareness should be raised among frontline practitioners
most likely to encounter patients with HPAI. © Association of Medical
Microbiology and Infectious Disease Canada (AMMI Canada), 2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38058238
</td>
<td style="text-align:left;">
Building solidarity during COVID-19 and HIV/AIDS.
</td>
<td style="text-align:left;">
While the WHO, public health experts, and political leaders have
referenced solidarity as an important part of our responses to COVID-19,
I consider how we build solidarity during pandemics in order to improve
the effectiveness of our responses. I use Prainsack and Buyx’s
definition of solidarity, which highlights three different tiers: (1)
interpersonal solidarity, (2) group solidarity, and (3) institutional
solidarity. Each tier of solidarity importantly depends on the actions
and norms established at the lower tiers. Although empathy and
solidarity are distinct moral concepts, I argue that the affective
component of solidarity is important for motivating solidaristic action,
and empathetic accounts of solidarity help us understand how we actually
build solidarity from tier to tier. During pandemics, public health
responses draw on different tiers of solidarity depending on the nature,
scope, and timeline of the pandemic. Therefore, I analyze both COVID-19
and HIV/AIDS using this framework to learn lessons about how solidarity
can more effectively contribute to our ongoing public health responses
during pandemics. Whereas we used institutional solidarity during
COVID-19 in a top-down approach to building solidarity that often
overlooked interpersonal and group solidarity, we used those lower tiers
during HIV/AIDS in a bottom-up approach because governments and public
health institutions were initially unresponsive to the crisis. Thus, we
need to ensure that we have a strong foundation of respect, trust, and
so forth, on which to build solidarity from tier to tier and promote
whichever tiers of solidarity are lacking during a given pandemic to
improve our responses. © 2023 The Authors. Bioethics published by John
Wiley &amp; Sons Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38057820
</td>
<td style="text-align:left;">
Post-COVID-19 vaccination myocarditis: a prospective cohort study pre
and post vaccination using cardiovascular magnetic resonance.
</td>
<td style="text-align:left;">
Concerns about COVID-19 vaccination induced myocarditis or subclinical
myocarditis persists in some populations. Cardiac magnetic resonance
imaging (CMR) has been used to detect signs of COVID-19 vaccination
induced myocarditis. This study aims to: (i) characterise myocardial
tissue, function, size before and after COVID-19 vaccination, (ii)
determine if there is imaging evidence of subclinical myocardial
inflammation or injury after vaccination using CMR. Subjects aged ≥
12yrs old without prior COVID-19 or COVID-19 vaccination underwent two
CMR examinations: first, ≤ 14 days before the first COVID-19 vaccination
and a second time ≤ 14 days after the second COVID-19 vaccination.
Biventricular indices, ejection fraction (EF), global longitudinal
strain (GLS), late gadolinium enhancement (LGE), left ventricular (LV)
myocardial native T1, T2, extracellular volume (ECV) quantification,
lactate dehydrogenase (LDH), white cell count (WCC), C-reactive protein
(CRP), NT-proBNP, troponin-T, electrocardiogram (ECG), and 6-min walk
test were assessed in a blinded fashion. 67 subjects were included.
First and second CMR examinations were performed a median of 4 days
before the first vaccination (interquartile range 1-8 days) and 5 days
(interquartile range 3-6 days) after the second vaccination
respectively. No significant change in global native T1, T2, ECV, LV EF,
right ventricular EF, LV GLS, LGE, ECG, LDH, troponin-T and 6-min walk
test was demonstrated after COVID-19 vaccination. There was a
significant WCC decrease (6.51 ± 1.49 vs 5.98 ± 1.65, p = 0.003) and CRP
increase (0.40 ± 0.22 vs 0.50 ± 0.29, p = 0.004). This study found no
imaging, biochemical or ECG evidence of myocardial injury or
inflammation post COVID-19 vaccination, thus providing some reassurance
that COVID-19 vaccinations do not typically cause subclinical
myocarditis. © 2023. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38057504
</td>
<td style="text-align:left;">
A Qualitative Analysis of Medical Student Reflections Following
Participation in a Canadian Radiation Oncology Studentship.
</td>
<td style="text-align:left;">
Exposure to radiation oncology in medical school curricula is limited;
thus, mentorship and research opportunities like the Dr. Pamela Catton
Summer Studentship Program attempt to bridge this gap and stimulate
interest in the specialty. In 2021, the studentship was redesigned as
virtual research, mentorship, and case-based discussions due to the
COVID-19 pandemic. This study explores the impact of COVID-19 on the
studentship, on students’ perceptions of the program, and on medical
training and career choice. Fifteen studentship completion essays during
2021-2022 were obtained and anonymized. Thematic analysis was performed
to interpret the essays with NVivo. Two independent reviewers coded the
essays. Themes were established by identifying connections between coded
excerpts. Consensus was achieved through multiple rounds of discussion
and iteratively reviewing each theme. Representative quotes were used to
illustrate the themes. The themes confirmed the studentship was feasible
during the pandemic. Perceived benefits of the program included
mentorship and networking opportunities; gaining practical and
fundamental knowledge in radiation oncology; developing clinical and
research skills; and creating positive attitudes towards radiation
oncology and the humanistic aspect of the field. The studentship
supported medical specialty selection by helping define student values,
shaping perceptions of the specialty, and promoting self-reflection upon
students’ personal needs. This study informs future iterations of the
studentship to promote radiation oncology in Canadian medical school
curricula. It serves as a model for studentships in other specialties
that have limited exposure and similar challenges with medical student
recruitment. © 2023. The Author(s) under exclusive licence to American
Association for Cancer Education.
</td>
</tr>
<tr>
<td style="text-align:left;">
38055885
</td>
<td style="text-align:left;">
Hyperbaric oxygen therapy for treatment of COVID-19-related parosmia: a
case report.
</td>
<td style="text-align:left;">
Parosmia is a qualitative olfactory dysfunction characterized by
distortion of odor perception. Traditional treatments for parosmia
include olfactory training and steroids. Some patients infected with
COVID-19 have developed chronic parosmia as a result of their infection.
Here, we present the case of a patient who developed parosmia after a
COVID-19 infection that was not improved by traditional treatments but
found significant improvement after hyperbaric oxygen therapy\[A1\].
Copyright© Undersea and Hyperbaric Medical Society.
</td>
</tr>
<tr>
<td style="text-align:left;">
38054579
</td>
<td style="text-align:left;">
Population-level effectiveness of pre-exposure prophylaxis for HIV
prevention among men who have sex with men in Montréal (Canada): a
modelling study of surveillance and survey data.
</td>
<td style="text-align:left;">
HIV pre-exposure prophylaxis (PrEP) has been recommended and partly
subsidized in Québec, Canada, since 2013. We evaluated the
population-level impact of PrEP on HIV transmission among men who have
sex with men (MSM) in Montréal, Québec’s largest city, over 2013-2021.
We used an agent-based mathematical model of sexual HIV transmission to
estimate the fraction of HIV acquisitions averted by PrEP compared to a
counterfactual scenario without PrEP. The model was calibrated to local
MSM survey, surveillance, and cohort data and accounted for COVID-19
pandemic impacts on sexual activity, HIV prevention, and care. PrEP was
modelled from 2013 onwards, assuming 86% individual-level effectiveness.
The PrEP eligibility criteria were: any anal sex unprotected by condoms
(past 6 months) and either multiple partnerships (past 6 months) or
multiple uses of post-exposure prophylaxis (lifetime). To assess
potential optimization strategies, we modelled hypothetical scenarios
prioritizing PrEP to MSM with high sexual activity (≥11 anal sex
partners annually) or aged ⩽45 years, increasing coverage to levels
achieved in Vancouver, Canada (where PrEP is free-of-charge), and
improving retention. Over 2013-2021, the estimated annual HIV incidence
decreased from 0.4 (90% credible interval \[CrI\]: 0.3-0.6) to 0.2 (90%
CrI: 0.1-0.2) per 100 person-years. PrEP coverage among HIV-negative MSM
remained low until 2015 (&lt;1%). Afterwards, coverage increased to a
maximum of 10% of all HIV-negative MSM, or about 16% of the 62%
PrEP-eligible HIV-negative MSM in 2020. Over 2015-2021, PrEP averted an
estimated 20% (90% CrI: 11%-30%) of cumulative HIV acquisitions. The
hypothetical scenarios modelled showed that, at the same coverage level,
prioritizing PrEP to high sexual activity MSM could have averted 30%
(90% CrI: 19%-42%) of HIV acquisitions from 2015-2021. Even larger
impacts could have resulted from higher coverage. Under the provincial
eligibility criteria, reaching 10% coverage among HIV-negative MSM in
2015 and 30% in 2019, like attained in Vancouver, could have averted up
to 63% (90% CrI: 54%-70%) of HIV acquisitions from 2015 to 2021. PrEP
reduced population-level HIV transmission among Montréal MSM. However,
our study suggests missed prevention opportunities and adds support for
public policies that reduce PrEP barriers, financial or otherwise, to
MSM at risk of HIV acquisition. © 2023 The Authors. Journal of the
International AIDS Society published by John Wiley &amp; Sons Ltd on
behalf of International AIDS Society.
</td>
</tr>
<tr>
<td style="text-align:left;">
38053406
</td>
<td style="text-align:left;">
COVID-19 in Long-Term Care: A Two-Part Commentary.
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
38051737
</td>
<td style="text-align:left;">
Effect of mode of healthcare delivery on job satisfaction and intention
to quit among nurses in Canada during the COVID-19 pandemic.
</td>
<td style="text-align:left;">
The COVID-19 pandemic resulted in a major shift in the delivery of
healthcare services with the adoption of care modalities to address the
diverse needs of patients. Besides, nurses, the largest profession in
the healthcare sector, were imposed with challenges caused by the
pandemic that influenced their intention to leave their profession. The
aim of the study was to examine the influence of mode of healthcare
delivery on nurses’ intention to quit job due to lack of satisfaction
during the pandemic in Canada. This cross-sectional study utilized data
from the Health Care Workers’ Experiences During the Pandemic (SHCWEP)
survey, conducted by Statistics Canada, that targeted healthcare workers
aged 18 and over who resided in the ten provinces of Canada during the
COVID-19 pandemic. The main outcome of the study was nurses’ intention
to quit within two years due to lack of job satisfaction. The mode of
healthcare delivery was categorized into; in-person, online, or blended.
Multivariable logistic regression was performed to examine the
association between mode of healthcare delivery and intention to quit
job after adjusting for sociodemographic, job-, and health-related
factors. Analysis for the present study was restricted to 3,430 nurses,
weighted to represent 353,980 Canadian nurses. Intention to quit job,
within the next two years, due to lack of satisfaction was reported by
16.4% of the nurses. Results showed that when compared to participants
who provided in-person healthcare services, those who delivered online
or blended healthcare services were at decreased odds of intention to
quit their job due to lack of job satisfaction (OR = 0.47, 95% CI:
0.43-0.50 and OR = 0.64, 95% CI: 0.61-0.67, respectively). Findings from
this study can inform interventions and policy reforms to address
nurses’ needs and provide organizational support to enhance their
retention and improve patient care during times of crisis. Copyright: ©
2023 Zangiabadi, Ali-Hassan. This is an open access article distributed
under the terms of the Creative Commons Attribution License, which
permits unrestricted use, distribution, and reproduction in any medium,
provided the original author and source are credited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38051660
</td>
<td style="text-align:left;">
Investigating the Telerehabilitation with Aims to Improve Lower
Extremity Recovery Post-Stroke (TRAIL) Program: A Feasibility Study.
</td>
<td style="text-align:left;">
The purpose of this study was to examine the feasibility of a
progressive virtual exercise and self-management intervention, the
TeleRehabilitation with Aims to Improve Lower extremity recovery
post-stroke program (TRAIL), in individuals with stroke. A single group
pre-post study design was used. Thirty-two participants were recruited
who were aged 19 years or older, had a stroke within 18 months of the
beginning of the study, had hemiparesis of the lower extremity, and were
able to tolerate 50 minutes of activity. Participants completed TRAIL, a
synchronous exercise and self-management program delivered via
videoconferencing. Participants received 8 telerehabilitation sessions
over 4 weeks that were 60 to 90 minutes, with a trained physical
therapist in a ≤ 2 to 1 participant-to-therapist ratio. Feasibility
indicators in the areas of process (recruitment and retention rates,
perceived satisfaction), resources (treatment fidelity and adherence,
participant and assessor burden, therapist burden), management
(equipment, processing time), and scientific indicators (safety,
treatment response, treatment effect) were collected throughout the
study using a priori criteria for success. The treatment effect was
examined on the Timed “Up &amp; Go” Test (TUG), the virtual Fugl-Meyer
Lower Extremity Assessment (FM-Tele), the 30-Second Sit-to-Stand Test
(30s S2S), the Functional Reach, the Tandem Stand, the
Activities-specific Balance Confidence Scale, the Stroke Impact Scale,
and the Goal Attainment Scale (GAS). Forty-seven individuals were
screened, of which 32 (78% male; median age of 64.5 years) were included
for the study from 5 sites across Canada. Nine feasibility indicators
met our study-specific threshold criteria for success: retention rate (0
dropouts), perceived satisfaction, treatment fidelity, adherence,
therapist burden, equipment, and safety. In terms of treatment response
and effect, improvements were observed in TUG (Cohen d = 0.57); FM-Tele
(d = 0.76); 30s S2S (d = 0.89); and GAS (d = 0.95). The delivery of
TRAIL, a lower extremity stroke rehabilitation program using
video-conferencing technology, is feasible and appears to have positive
influences on mobility, lower extremity impairment, strength, and goal
attainment. Community-based telerehabilitation programs, such as TRAIL,
could extend the continuum of care during the transition back to
community post-discharge, or during global disruptions, such as
COVID-19. Delivery of synchronous lower extremity rehabilitation via
videoconferencing to community-dwelling stroke survivors is feasible. ©
The Author(s) 2023. Published by Oxford University Press on behalf of
the American Physical Therapy Association. All rights reserved. For
permissions, please e-mail: <journals.permissions@oup.com>.
</td>
</tr>
<tr>
<td style="text-align:left;">
38051562
</td>
<td style="text-align:left;">
The Implementation of a Virtual Emergency Department: Multimethods Study
Guided by the RE-AIM (Reach, Effectiveness, Adoption, Implementation,
and Maintenance) Framework.
</td>
<td style="text-align:left;">
While the COVID-19 pandemic dramatically increased virtual care uptake
across many health settings, it remains significantly underused in
urgent care. This study evaluated the implementation of a pilot virtual
emergency department (VED) at an Ontario hospital that connected
patients to emergency physicians through a web-based portal. We sought
to (1) assess the acceptability of the VED model, (2) evaluate whether
the VED was implemented as intended, and (3) explore the impact on
quality of care, access to care, and continuity of care. This evaluation
used a multimethods approach informed by the RE-AIM (Reach,
Effectiveness, Adoption, Implementation, and Maintenance) framework.
Data included semistructured interviews with patients and physicians as
well as postvisit surveys from patients. Interviews were transcribed and
analyzed using thematic analysis. Data from the surveys were described
using summary statistics. From December 2020 to December 2021, the VED
had a mean of 153 (SD 25) visits per month. Among them, 67% (n=677) were
female, and 75% (n=758) had a family physician. Patients reported that
the VED provided high-quality, timely access to care and praised the
convenience, shorter appointments, and benefit of the calm, safe space
afforded through virtual appointments. In instances where patients were
directed to come into the emergency department (ED), physicians were
able to provide a “warm handoff” to improve efficiency. This helped
manage patient expectations, and the direct advice of the ED physician
reassured them that the visit was warranted. There was broad initial
uptake of VED shifts among ED physicians with 60% (n=22) completing
shifts in the first 2 months and 42% (n=15) completing 1 or more shifts
per month over the course of the pilot. There were no difficulties
finding sufficient ED physicians for shifts. Most physicians enjoyed
working in the VED, saw value for patients, and were motivated by
patient satisfaction. However, some physicians were hesitant as they
felt their expertise and skills as ED physicians were underused. The VED
was implemented using an iterative staged approach with increased
service capabilities over time, including access to ultrasounds, virtual
follow-ups after a recent ED visit, and access to blood work, urine
tests, and x-rays (at the hospital or a local community laboratory).
Physicians recognized the value in supporting patients by advising on
the need for an in-person visit, booking a diagnostic test, or referring
them to a specialist. The VED had the support of physicians and
facilitated care for low-acuity presentations with immediate benefits
for patients. It has the potential to benefit the health care system by
seeing patients through the web and guiding patients to in-person care
only when necessary. Long-term sustainability requires a focus on
understanding digital equity and enhanced access to rapid testing or
investigations. ©Jennifer Shuldiner, Diya Srinivasan, Laura Desveaux,
Justin N Hall. Originally published in JMIR Formative Research
(<https://formative.jmir.org>), 05.12.2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38048423
</td>
<td style="text-align:left;">
Proteomic Evolution from Acute to Post-COVID-19 Conditions.
</td>
<td style="text-align:left;">
Many COVID-19 survivors have post-COVID-19 conditions, and females are
at a higher risk. We sought to determine (1) how protein levels change
from acute to post-COVID-19 conditions, (2) whether females have a
plasma protein signature different from that of males, and (3) which
biological pathways are associated with COVID-19 when compared to
restrictive lung disease. We measured protein levels in 74 patients on
the day of admission and at 3 and 6 months after diagnosis. We
determined protein concentrations by multiple reaction monitoring (MRM)
using a panel of 269 heavy-labeled peptides. The predicted forced vital
capacity (FVC) and diffusing capacity of the lungs for carbon monoxide
(DLCO) were measured by routine pulmonary function testing. Proteins
associated with six key lipid-related pathways increased from admission
to 3 and 6 months; conversely, proteins related to innate immune
responses and vasoconstriction-related proteins decreased. Multiple
biological functions were regulated differentially between females and
males. Concentrations of eight proteins were associated with FVC, %, and
they together had c-statistics of 0.751 (CI:0.732-0.779); similarly,
concentrations of five proteins had c-statistics of 0.707
(CI:0.676-0.737) for DLCO, %. Lipid biology may drive evolution from
acute to post-COVID-19 conditions, while activation of innate immunity
and vascular regulation pathways decreased over that period.
(ProteomeXchange identifiers: PXD041762, PXD029437).
</td>
</tr>
<tr>
<td style="text-align:left;">
38048017
</td>
<td style="text-align:left;">
HIV Vulnerabilities Associated with Water Insecurity, Food Insecurity,
and Other COVID-19 Impacts Among Urban Refugee Youth in Kampala, Uganda:
Multi-method Findings.
</td>
<td style="text-align:left;">
Food insecurity (FI) and water insecurity (WI) are linked with HIV
vulnerabilities, yet how these resource insecurities shape HIV
prevention needs is understudied. We assessed associations between FI
and WI and HIV vulnerabilities among urban refugee youth aged 16-24 in
Kampala, Uganda through individual in-depth interviews (IDI) (n = 24),
focus groups (n = 4), and a cross-sectional survey (n = 340) with
refugee youth, and IDI with key informants (n = 15). Quantitative data
was analysed via multivariable logistic and linear regression to assess
associations between FI and WI with: reduced pandemic sexual and
reproductive health (SRH) access; past 3-month transactional sex (TS);
unplanned pandemic pregnancy; condom self-efficacy; and sexual
relationship power (SRP). We applied thematic analytic approaches to
qualitative data. Among survey participants, FI and WI were commonplace
(65% and 47%, respectively) and significantly associated with: reduced
SRH access (WI: adjusted odds ratio \[aOR\]: 1.92, 95% confidence
interval \[CI\]: 1.19-3.08; FI: aOR: 2.31. 95%CI: 1.36-3.93), unplanned
pregnancy (WI: aOR: 2.77, 95%CI: 1.24-6.17; FI: aOR: 2.62, 95%CI:
1.03-6.66), and TS (WI: aOR: 3.09, 95%CI: 1.22-7.89; FI: aOR: 3.51,
95%CI: 1.15-10.73). WI participants reported lower condom self-efficacy
(adjusted β= -3.98, 95%CI: -5.41, -2.55) and lower SRP (adjusted β=
-2.58, 95%CI= -4.79, -0.37). Thematic analyses revealed: (1) contexts of
TS, including survival needs and pandemic impacts; (2) intersectional
HIV vulnerabilities; (3) reduced HIV prevention/care access; and (4)
water insecurity as a co-occurring socio-economic stressor. Multi-method
findings reveal FI and WI are linked with HIV vulnerabilities,
underscoring the need for HIV prevention to address co-occurring
resource insecurities with refugee youth. © 2023. The Author(s), under
exclusive licence to Springer Science+Business Media, LLC, part of
Springer Nature.
</td>
</tr>
<tr>
<td style="text-align:left;">
38047755
</td>
<td style="text-align:left;">
Co-development and evaluation of the Musculoskeletal Telehealth Toolkit
for physiotherapists.
</td>
<td style="text-align:left;">
In-person physiotherapy services are not readily available to all
individuals with musculoskeletal conditions, especially those in rural
regions or with time-intensive responsibilities. The COVID-19 pandemic
highlighted that telehealth may facilitate access to, and continuity of
care, yet many physiotherapists lack telehealth confidence and training.
This project co-developed and evaluated a web-based professional
development toolkit supporting physiotherapists to provide telehealth
services for musculoskeletal conditions. A mixed-methods exploratory
sequential design applied modified experience-based co-design methods
(physiotherapists \[n = 13\], clinic administrators \[n = 2\], and
people with musculoskeletal conditions \[n = 7\]) to develop an
evidence-informed toolkit. Semi-structured workshops were conducted,
recorded, transcribed, and thematically analysed, refining the toolkit
prototype. Subsequently, the toolkit was promoted via webinars and
social media. The usability of the toolkit was examined with pre-post
surveys examining changes in confidence, knowledge, and perceived
telehealth competence (19 statements modelled from the theoretical
domains framework) between toolkit users (&gt;30 min) and non-users (0
min) using chi-squared tests for independence. Website analytics were
summarised. Twenty-two participants engaged in co-design workshops.
Feedback led to the inclusion of more patient-facing resources,
increased assessment-related visual content, streamlined toolkit
organisation, and simplified, downloadable infographics. Three hundred
and twenty-nine physiotherapists from 21 countries completed the
baseline survey, with 172 (52%) completing the 3-month survey. Toolkit
users had greater improvement in knowledge, confidence, and competence
than non-users in 42% of statements. Seventy-two percentage of toolkit
users said it changed their practice, and 95% would recommend the
toolkit to colleagues. During the evaluation period, the toolkit
received 5486 total views. The co-designed web-based Musculoskeletal
Telehealth Toolkit is a professional development resource that may
increase physiotherapist’s confidence, knowledge, and competence in
telehealth. © 2023 The Authors. Musculoskeletal Care published by John
Wiley &amp; Sons Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38047336
</td>
<td style="text-align:left;">
Overcoming financial and social barriers during COVID-19: A medical
student-led medical education innovation.
</td>
<td style="text-align:left;">
Underrepresented minorities in medicine (URMM) may face financial and
social limitations when applying to medical schools. The computer-based
assessment for sampling personal characteristics (CASPER) test is used
by many medical schools to assess the nonacademic competencies of
applicants. Performance on CASPER can be enhanced by coaching and
mentorship, which URMMs often lack, for affordability reasons, when
applying to medical schools. The CASPER Preparation Program (CPP) is a
free, online, 4-week program to help URMM prepare for the CASPER test.
CPP features free medical ethics resources, homework and practice tests,
and feedback from tutors. Two of CPPs major objectives include relieving
URMM of financial burdens and increasing their accessibility to
mentorship during the COVID-19 pandemic. A program evaluation was
conducted using anonymous, voluntary postprogram questionnaires to
assess CPPs efficacy in achieving the aforementioned objectives. Sixty
URMMs completed the survey. The majority of the respondents strongly
agreed or agreed that CPP relieves students of financial burden (97%),
is beneficial for applicants with low-socioeconomic statuses (98%),
provides students with resources they could not afford (n = 55; 92%),
and enables access to mentors during the pandemic (90%). Pathway
coaching programs, such as the CASPER Preparation Program, have the
potential to offer URMMs mentorship and financial relief, and increase
their confidence and familiarity with standardized admission tests to
help them matriculate into medical schools.
</td>
</tr>
<tr>
<td style="text-align:left;">
38046546
</td>
<td style="text-align:left;">
The Effect of Telerehabilitation on Physical Fitness and
Depression/Anxiety in Post-COVID-19 Patients: A Randomized Controlled
Trial.
</td>
<td style="text-align:left;">
The aim of this research was to evaluate the impact of a
telerehabilitation program on physical fitness, muscle strength, and
levels of depression and anxiety in post-COVID-19 patients. Thirty-two
individuals recovered from COVID-19 (48.20±12.82 years) were allocated
into either a telerehabilitation (TG n=16) or control (CG n=16) group.
Physical fitness, handgrip strength, depression and anxiety levels were
assessed before and after an 8-week intervention. There was a
significant improvement in muscle strength in both groups. Physical
fitness significantly increased compared to the CG at the end of the
intervention. Levels of anxiety and depression significantly decreased
after the intervention when compared to the CG. Eight weeks of
functional telerehabilitation training is a viable and efficient way to
rehabilitate patients affected by COVID-19, as it improved physical
conditioning and mental health. Copyright © 2023 Paloma Lopes de Araújo
Furtado, Maria do Socorro Brasileiro-Santos, Brenda Lopes Cavalcanti de
Mello, Alex Andrade Araújo, Maria Alessandra Sipriano da Silva, Jennifer
Arielly Suassuna, Gabriella Brasileiro-Santos, Renata de Lima Martins,
Amilton da Cruz Santos.
</td>
</tr>
<tr>
<td style="text-align:left;">
38045081
</td>
<td style="text-align:left;">
Zoomification of medical education: can the rapid online educational
responses to COVID-19 prepare us for another educational disruption? A
scoping review.
</td>
<td style="text-align:left;">
In response to the COVID-19 pandemic, educators have increasingly
shifted delivery of medical education to online/distance learning. Given
the rapid and heterogeneous nature of adaptations; it is unclear what
interventions have been developed, which strategies and technologies
have been leveraged, or, more importantly, the rationales given for
designs. Capturing the content and skills that were shifted to online,
the type of platforms used for the adaptations, as well as the
pedagogies, theories, or conceptual frameworks used to inform the
adapted educational deliveries can bolster continued improvement and
sustainability of distance/online education while preparing medical
education for future large-scale disruptions. We conducted a scoping
review to map the rapid medical educational interventions that have been
adapted or transitioned to online between December 2019 and August 2020.
We searched MEDLINE, EMBASE, Education Source, CINAHL, and Web of
Science for articles pertaining to COVID-19, online (distance) learning,
and education for medical students, residents, and staff. We included
primary research articles and reports describing adaptations of previous
educational content to online learning. From an initial 980 articles, we
identified 208 studies for full-text screening and 100 articles for data
extraction. The majority of the reported scholarship came from Western
Countries and was published in clinical science journals. Cognitive
content was the main type of content adapted (over psychomotor, or
affective). More than half of the articles used a video-conferencing
software as the platform to pivot their educational intervention into
virtual. Unfortunately, most of the reported work did not disclose their
rationale for choosing a platform. Of those that did, the majority chose
technological solutions based on availability within their institutions.
Similarly, most of the articles did not report the use of any pedagogy,
theory, or framework to inform the educational adaptations. © 2023
Rojas, Tailor, Fournier, Cheung, Rangel; licensee Synergies Partners.
</td>
</tr>
<tr>
<td style="text-align:left;">
38044629
</td>
<td style="text-align:left;">
Development and Evaluation of a Nurse Practitioner Huddles Toolkit for
Long Term Care Homes.
</td>
<td style="text-align:left;">
Long-term care homes (LTCHs) were disproportionately affected by the
coronavirus disease (COVID-19) pandemic, creating stressful
circumstances for LTCH employees, residents, and their care partners.
Team huddles may improve staff outcomes and enable a supportive climate.
Nurse practitioners (NPs) have a multifaceted role in LTCHs, including
facilitating implementation of new practices. Informed by a
community-based participatory approach to research, this mixed-methods
study aimed to develop and evaluate a toolkit for implementing NP-led
huddles in an LTCH. The toolkit consists of two sections. Section one
describes the huddles’ purpose and implementation strategies. Section
two contains six scripts to guide huddle discussions. Acceptability of
the intervention was evaluated using a quantitative measure (Treatment
Acceptability Questionnaire) and through qualitative interviews with
huddle participants. Descriptive statistics and manifest content
analysis were used to analyse quantitative and qualitative data. The
project team rated the toolkit as acceptable. Qualitative findings
provided evidence on design quality, limitations, and recommendations
for future huddles.
</td>
</tr>
<tr>
<td style="text-align:left;">
38041406
</td>
<td style="text-align:left;">
Perspectives of Older Adults on COVID-19 and Influenza Vaccination in
Ontario, Canada.
</td>
<td style="text-align:left;">
Addressing vaccine hesitancy has become an increasingly important public
health priority in recent years. There is a paucity of studies that have
focused on vaccine hesitancy among older adults, who are known to be at
greater risk of complications from infections such as COVID-19. We aim
to explore the attitudes and beliefs of older adults regarding COVID-19
and influenza vaccines in Toronto, Ontario. Older adults enrolled in the
Student Senior Isolation Prevention Partnership (SSIPP) program at the
University of Toronto were contacted to participate in a phone survey
and semi-structured interview. Survey data was analyzed descriptively,
and attitude toward vaccination was compared between sociodemographic
groups by using Fisher’s exact test. Interview audio files were
transcribed verbatim and analyzed inductively for themes and sub-themes.
All thirty-three (100%) older adults reported that they had received the
first and second doses of the COVID-19 vaccine. Twenty-six (78.8%)
participants reported intent to get vaccinated against influenza or had
already received the influenza vaccine that year. Notably, only 2 out 7
(28.6%) individuals who did not plan to get vaccinated against influenza
believed that vaccines offered by health providers are beneficial and
only 3 out of 7 (42.9%) agreed that getting vaccines is a good way to
protect oneself from disease. No other significant differences in
attitudes among participants were found when compared by gender,
ethnicity, or education level. The qualitative data analysis of
interview transcripts identified 5 themes that impact vaccine decision
making: safety, trust, mistrust, healthcare experience, and information
dissemination and education. Our data showed that older adults in the
SSIPP program generally had positive views toward vaccination,
especially toward the COVID-19 vaccines. However, several concerns
regarding the effectiveness of the vaccines were brought up in
interviews, such as the speed at which the vaccines were produced and
the inconsistency in government messaging.
</td>
</tr>
<tr>
<td style="text-align:left;">
38041026
</td>
<td style="text-align:left;">
Humoral and T cell responses to SARS-CoV-2 reveal insights into immunity
during the early pandemic period in Pakistan.
</td>
<td style="text-align:left;">
Protection against SARS-CoV-2 is mediated by humoral and T cell
responses. Pakistan faced relatively low morbidity and mortality from
COVID-19 through the pandemic. To examine the role of prior immunity in
the population, we studied IgG antibody response levels, virus
neutralizing activity and T cell reactivity to Spike protein in a
healthy control group (HG) as compared with COVID-19 cases and
individuals from the pre-pandemic period (PP). HG and COVID-19
participants were recruited between October 2020 and May 2021.
Pre-pandemic sera was collected before 2018. IgG antibodies against
Spike and its Receptor Binding Domain (RBD) were determined by ELISA.
Virus neutralization activity was determined using a PCR-based
micro-neutralization assay. T cell - IFN-γ activation was assessed by
ELISpot. Overall, the magnitude of anti-Spike IgG antibody levels as
well as seropositivity was greatest in COVID-19 cases (90%) as compared
with HG (39.8%) and PP (12.2%). During the study period, Pakistan
experienced three COVID-19 waves. We observed that IgG seropositivity to
Spike in HG increased from 10.3 to 83.5% during the study, whilst
seropositivity to RBD increased from 7.5 to 33.3%. IgG antibodies to
Spike and RBD were correlated positively in all three study groups.
Virus neutralizing activity was identified in sera of COVID-19, HG and
PP. Spike reactive T cells were present in COVID-19, HG and PP groups.
Individuals with reactive T cells included those with and without IgG
antibodies to Spike. Antibody and T cell responses to Spike protein in
individuals from the pre-pandemic period suggest prior immunity against
SARS-CoV-2, most likely from cross-reactive responses. The rising
seroprevalence observed in healthy individuals through the pandemic
without known COVID-19 may be due to the activation of adaptive immunity
from cross-reactive memory B and T cells. This may explain the more
favourable COVID-19 outcomes observed in this population. © 2023. The
Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38040779
</td>
<td style="text-align:left;">
A novel multiplex biomarker panel for profiling human acute and chronic
kidney disease.
</td>
<td style="text-align:left;">
Acute and chronic kidney disease continues to confer significant
morbidity and mortality in the clinical setting. Despite high prevalence
of these conditions, few validated biomarkers exist to predict kidney
dysfunction. In this study, we utilized a novel kidney multiplex panel
to measure 21 proteins in plasma and urine to characterize the spectrum
of biomarker profiles in kidney disease. Blood and urine samples were
obtained from age-/sex-matched healthy control subjects (HC),
critically-ill COVID-19 patients with acute kidney injury (AKI), and
patients with chronic or end-stage kidney disease (CKD/ESKD). Biomarkers
were measured with a kidney multiplex panel, and results analyzed with
conventional statistics and machine learning. Correlations were examined
between biomarkers and patient clinical and laboratory variables. Median
AKI subject age was 65.5 (IQR 58.5-73.0) and median CKD/ESKD age was
65.0 (IQR 50.0-71.5). Of the CKD/ESKD patients, 76.1% were on
hemodialysis, 14.3% of patients had kidney transplant, and 9.5% had CKD
without kidney replacement therapy. In plasma, 19 proteins were
significantly different in titer between the HC versus AKI versus
CKD/ESKD groups, while NAG and RBP4 were unchanged. TIMP-1 (PPV 1.0, NPV
1.0), best distinguished AKI from HC, and TFF3 (PPV 0.99, NPV 0.89) best
distinguished CKD/ESKD from HC. In urine, 18 proteins were significantly
different between groups except Calbindin, Osteopontin and TIMP-1.
Osteoactivin (PPV 0.95, NPV 0.95) best distinguished AKI from HC, and
β2-microglobulin (PPV 0.96, NPV 0.78) best distinguished CKD/ESKD from
HC. A variety of correlations were noted between patient variables and
either plasma or urine biomarkers. Using a novel kidney multiplex
biomarker panel, together with conventional statistics and machine
learning, we identified unique biomarker profiles in the plasma and
urine of patients with AKI and CKD/ESKD. We demonstrated correlations
between biomarker profiles and patient clinical variables. Our
exploratory study provides biomarker data for future hypothesis driven
research on kidney disease. © 2023. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38040496
</td>
<td style="text-align:left;">
Implementing digital devices to maintain family connections during the
COVID-19 pandemic: Experience of a large academic urban hospital.
</td>
<td style="text-align:left;">
The COVID-19 pandemic lockdown imposed drastic measures at hospitals
internationally to rapidly pivot their approaches to patient care. Early
on when the pandemic was declared, hospitals responded to local public
health restrictions by limiting all non-essential visits for their
patients. Digital devices allowed Canadians to remain connected with
their friends and families during the imposed isolation restrictions.
The aim of this clinical perspective is to share the experience with the
immediate implementation of digital connections for patients by
exploring the impact as well as to describe key learnings from the
initiative. 150 iPads were distributed to clinical teams for use by
patients, for either clinical care or for social connection. An
iterative evaluation process, guided by quality improvement methodology,
was followed which ensured that any changes to the iPad implementation
process were responsive in real time. The evaluation measures included a
clinician survey and collection of narratives from patients and
clinicians. All clinician respondents (n=7) indicated that the iPads
were a valuable tool to support patients and families. Narratives from
patients indicated that virtual connections brought joy to them
especially given that they were isolated from their support systems
during the initial days of COVID-19. Key learnings for the
Implementation Team were: the importance in maintaining cognitive
stimulation as an enabler to recovery for patients; staff members
provide support beyond direct clinical care; technology needs to be
harnessed to facilitate clinical care; technology should be leveraged to
support clinical care; and interprofesssional collaboration of the
entire Implementation Team is a key enabler of success. Since
implementation, iPads have been integrated as a supporting digital tool
for both social connections as well as clinical care as a result of the
benefits seen during this initiative. The iPads have also been
recognized as a tool to complement patient connection and care and
currently being expanded as part of services to patients, families and
their clinicians. Copyright © 2023. Published by Elsevier Inc. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38038182
</td>
<td style="text-align:left;">
Here to stay? Policy changes in alcohol home delivery and “to-go” sales
during and after COVID-19 in the United States.
</td>
<td style="text-align:left;">
During the early phase of the COVID-19 pandemic, legislative changes
that expanded alcohol home delivery and options for “to-go” alcohol
sales were introduced across the United States to provide economic
relief to establishments and retailers. Using data from the Alcohol
Policy Information System, we examined whether these changes have
persisted beyond the peak phase of the COVID-19 emergency and explored
the implications for public health. Illustration of state-level policy
data reveals that the liberalisation of alcohol delivery and “to-go”
alcohol sales has continued throughout a 2-year period (2020 and 2021),
with indications that many of these changes have or will become
permanent after the pandemic. This raises concerns about inadequate
regulation, particularly in preventing underage access to alcohol, and
ensuing changes in drinking practices. In this commentary, we highlight
the need for rigorous empirical evaluation of the public health impact
of this changing policy landscape and underscore the potential risks
associated with increased alcohol availability, including a
corresponding increase in alcohol-attributable mortality and other
alcohol-related harm, such as domestic violence. Policy makers should
carefully consider public health consequences, whose costs may surpass
short-term economic interests in the long term. © 2023 The Authors. Drug
and Alcohol Review published by John Wiley &amp; Sons Australia, Ltd on
behalf of Australasian Professional Society on Alcohol and other Drugs.
</td>
</tr>
<tr>
<td style="text-align:left;">
38037883
</td>
<td style="text-align:left;">
New causes of occupational allergic contact dermatitis.
</td>
<td style="text-align:left;">
Occupational allergic contact dermatitis (OACD) is an important
work-related skin disease. Information about the causative agents comes
from many sources, including patch test databases, registries, case
series and case reports. This review summarizes new information about
common causative allergens and diagnosis. Common causes of OACD include
rubber components, epoxies and preservatives. New exposure sources for
these allergens continue to be described. Often these exposure sources
are related to the changing world around us, such as allergens related
to smartphones and technology, and personal protective equipment-related
exposures during the COVID-19 pandemic. New allergens are also being
described, some of which are related to known allergens (e.g. a new
epoxy or acrylate component).Accurate diagnosis is critical to effective
management of OACD, which may include removing the worker from exposure
to the causative allergen. Safety data sheets may not contain complete
information and patch testing with specialized series of allergens and
workplace materials may be necessary. This review provides current
evidence about causes of OACD and important aspects of diagnosis. This
is important for clinical practice to ensure cases of OACD are not
missed. Copyright © 2023 Wolters Kluwer Health, Inc. All rights
reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38037575
</td>
<td style="text-align:left;">
Physical Rehabilitation Before and After Lung Transplantation for
COVID-19 ARDS: A Case Report.
</td>
<td style="text-align:left;">
To describe the functional trajectory and physical rehabilitation of an
individual who underwent lung transplantation for COVID-19 acute
respiratory distress syndrome (ARDS). A previously healthy 60-year-old
man admitted to critical care pre-transplantation and followed six
months post-transplant. Physical rehabilitation in the critical care,
acute ward and in-patient rehabilitation settings. Despite a successful
surgery, a long and complex acute care admission contributed to a slow
and variable functional recovery. Significant functional limitations and
physical frailty were present in the early post-transplant period.
Little is known of the effects of COVID-19 superimposed upon lung
transplantation on muscle function, exercise capacity, and physical
activity. Future research should include case series to further
understand the functional deficits and trajectory of recovery in this
emerging clinical population. Standard core outcome measures should be
identified for this population to enable synthesis of findings and
inform short- and long-term rehabilitation strategies. © Canadian
Physiotherapy Association, 2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38033248
</td>
<td style="text-align:left;">
Critical care delivery across health care systems in low-income and
low-middle-income country settings: A systematic review.
</td>
<td style="text-align:left;">
Prior research has demonstrated that low- and low-middle-income
countries (LLMICs) bear a higher burden of critical illness and have a
higher rate of mortality from critical illness than high-income
countries (HICs). There is a pressing need for improved critical care
delivery in LLMICs to reduce this inequity. This systematic review aimed
to characterise the range of critical care interventions and services
delivered within LLMIC health care systems as reported in the
literature. A search strategy using terms related to critical care in
LLMICs was implemented in multiple databases. We included English
language articles with human subjects describing at least one critical
care intervention or service in an LLMIC setting published between 1
January 2008 and 1 January 2020. A total of 1620 studies met the
inclusion criteria. Among the included studies, 45% of studies reported
on pediatric patients, 43% on adults, 23% on infants, 8.9% on geriatric
patients and 4.2% on maternal patients. Most of the care described (94%)
was delivered in-hospital, with the remainder (6.2%) taking place in
out-of-hospital care settings. Overall, 49% of critical care described
was delivered outside of a designated intensive care unit. Specialist
physicians delivered critical care in 60% of the included studies.
Additional critical care was delivered by general physicians (40%), as
well as specialist physician trainees (22%), pharmacists (16%), advanced
nursing or midlevel practitioners (8.9%), ambulance providers (3.3%) and
respiratory therapists (3.1%). This review represents a comprehensive
synthesis of critical care delivery in LLMIC settings. Approximately 50%
of critical care interventions and services were delivered outside of a
designated intensive care unit. Specialist physicians were the most
common health care professionals involved in care delivery in the
included studies, however generalist physicians were commonly reported
to provide critical care interventions and services. This study
additionally characterised the quality of the published evidence guiding
critical care practice in LLMICs, demonstrating a paucity of
interventional and cost-effectiveness studies. Future research is needed
to understand better how to optimise critical care interventions,
services, care delivery and costs in these settings. PROSPERO
CRD42019146802. Copyright © 2023 by the Journal of Global Health. All
rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38028902
</td>
<td style="text-align:left;">
Variability in changes in physician outpatient antibiotic prescribing
from 2019 to 2021 during the COVID-19 pandemic in Ontario, Canada.
</td>
<td style="text-align:left;">
To evaluate inter-physician variability and predictors of changes in
antibiotic prescribing before (2019) and during (2020/2021) the
coronavirus disease 2019 (COVID-19) pandemic. We conducted a
retrospective cohort analysis of physicians in Ontario, Canada
prescribing oral antibiotics in the outpatient setting between January
1, 2019 and December 31, 2021 using the IQVIA Xponent data set. The
primary outcome was the change in the number of antibiotic prescriptions
between the prepandemic and pandemic period. Secondary outcomes were
changes in the selection of broad-spectrum agents and long-duration
(&gt;7 d) antibiotic use. We used multivariable linear regression models
to evaluate predictors of change. There were 17,288 physicians included
in the study with substantial inter-physician variability in changes in
antibiotic prescribing (median change of -43.5 antibiotics per
physician, interquartile range -136.5 to -5.0). In the multivariable
model, later career stage (adjusted mean difference \[aMD\] -45.3, 95%
confidence interval \[CI\] -52.9 to -37.8, p &lt; .001), family medicine
(aMD -46.0, 95% CI -62.5 to -29.4, p &lt; .001), male patient sex (aMD
-52.4, 95% CI -71.1 to -33.7, p &lt; .001), low patient comorbidity (aMD
-42.5, 95% CI -50.3 to -34.8, p &lt; .001), and high prescribing to new
patients (aMD -216.5, 95% CI -223.5 to -209.5, p &lt; .001) were
associated with decreases in antibiotic initiation. Family medicine and
high prescribing to new patients were associated with a decrease in
selection of broad-spectrum agents and prolonged antibiotic use.
Antibiotic prescribing changed throughout the COVID-19 pandemic with
overall decreases in antibiotic initiation, broad-spectrum agents, and
prolonged antibiotic courses with inter-physician variability. These
findings present opportunities for community antibiotic stewardship
interventions. © The Author(s) 2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38028707
</td>
<td style="text-align:left;">
Enrollment of dengue patients in a prospective cohort study in Umphang
District, Thailand, during the COVID-19 pandemic: Implications for
research and policy.
</td>
<td style="text-align:left;">
Dengue is endemic in Thailand and imposes a high burden on the health
system and society. We conducted a prospective cohort study in Umphang
District, Tak Province, Thailand, to investigate the share of dengue
cases with long symptoms and their duration. Here we present the results
of the enrollment process during the COVID-19 pandemic with implications
and challenges for research and policy. In a prospective cohort study
conducted in Umphang District, Thailand, we examined the prevalence of
persistent symptoms in dengue cases. Clinically diagnosed cases were
offered free laboratory testing, We enrolled ambulatory dengue patients
regardless of age who were confirmed through a highly sensitive
laboratory strategy (positive NS1 and/or IgM), agreed to follow-up
visits, and gave informed consent. We used multivariate logistic
regressions to assess the probability of clinical dengue being
laboratory confirmed. To determine the factors associated with study
enrollment, we analyzed the relationship of patient characteristics and
month of screening to the likelihood of participation. To identify
underrepresented groups, we compared the enrolled cohort to external
data sources. The 150 clinical cases ranged from 1 to 85 years old. Most
clinical cases (78%) were confirmed by a positive laboratory test, but
only 19% of those confirmed enrolled in the cohort study. Women, who
were half as likely to enroll as men, were underrepresented in the
cohort. The Thai physicians’ clinical diagnoses at this rural district
hospital had good agreement with laboratory diagnoses. By identifying
underrepresented groups and disparities, future studies can ensure the
creation of statistically representative cohorts to maximize their
scientific value. This involves recruiting and retaining
underrepresented groups in health research, such as women in this study.
Promising strategies for meaningful inclusion include multi-site
enrollment, offering in-home or virtual services, and providing in-kind
benefits like childcare for underrepresented groups. © 2023 The Authors.
Health Science Reports published by Wiley Periodicals LLC.
</td>
</tr>
<tr>
<td style="text-align:left;">
38028120
</td>
<td style="text-align:left;">
Why Did Home Care Personal Support Service Volumes Drop During the
COVID-19 Pandemic? The Contributions of Client Choice and Personal
Support Worker Availability.
</td>
<td style="text-align:left;">
Home care personal support service delivery decreased during the
COVID-19 pandemic, and qualitative studies have suggested many potential
contributors to these reductions. This paper provides insight into the
source (client or provider) of reductions in home care service volumes
early in the pandemic through analysis of a retrospective administrative
dataset from a large provider organization. The percentage of authorized
services not delivered was 17.2% in Wave 1, 12.6% in Wave 2 and 10.5% in
Wave 3, nearing the pre-pandemic baseline of 8.9%. The dominant
contribution to reduced home care service volumes was client-initiated
holds and cancellations, collectively accounting for 99.3% of the
service volume; missed care visits by the provider accounted for 0.7%.
Worker availability also declined due to long-term absences (which
increased 5-fold early in Wave 1 and remained 4× above baseline in Waves
2 and 3); short-term absences rose sharply for 6 early-pandemic weeks,
then dropped below the pre-pandemic baseline. These data reveal that
service volume reductions were primarily driven by client-initiated
holds and cancellations; despite unprecedented decreases in Personal
Support Worker availability, missed care did not increase, indicating
that the decrease in demand was more substantial and occurred earlier
than the decrease in worker availability. © The Author(s) 2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38027260
</td>
<td style="text-align:left;">
Overnight staffing in Canadian neonatal and pediatric intensive care
units.
</td>
<td style="text-align:left;">
Infants and children who require specialized medical attention are
admitted to neonatal and pediatric intensive care units (ICUs) for
continuous and closely supervised care. Overnight in-house physician
coverage is frequently considered the ideal staffing model. It remains
unclear how often this is achieved in both pediatric and neonatal ICUs
in Canada. The aim of this study is to describe overnight in-house
physician staffing in Canadian pediatric and level-3 neonatal ICUs
(NICUs) in the pre-COVID-19 era. A national cross-sectional survey was
conducted in 34 NICUs and 19 pediatric ICUs (PICUs). ICU directors or
their delegates completed a 29-question survey describing overnight
staffing by resident physicians, fellow physicians, nurse practitioners,
and attending physicians. A comparative analysis was conducted between
ICUs with and without in-house physicians. We obtained responses from
all 34 NICUs and 19 PICUs included in this study. A total of 44 ICUs
(83%) with in-house overnight physician coverage provided advanced
technologies, such as extracorporeal life support, and included all ICUs
that catered to patients with cardiac, transplant, or trauma conditions.
Residents provided the majority of overnight coverage, followed by the
Critical Care Medicine fellows. An attending physician was in-house
overnight in eight (15%) out of the 53 ICUs, seven of which were NICUs.
Residents participating in rotations in the ICU would often have
rotation durations of less than 6 weeks and were often responsible for
providing care during shifts lasting 20-24 h. Most PICUs and level-3
NICUs in Canada have a dedicated in-house physician overnight. These
physicians are mainly residents or fellows, but a notable variation
exists in this arrangement. The potential effects on patient outcomes,
resident learning, and physician satisfaction remain unclear and warrant
further investigation. © 2023 Maratta, Hutchison, Nicoll, Bagshaw,
Granton, Kirpalani, Stelfox, Ferguson, Cook, Parshuram and Moore.
</td>
</tr>
<tr>
<td style="text-align:left;">
38026412
</td>
<td style="text-align:left;">
Impact of the COVID-19 pandemic on community-based brain injury
associations across Canada: a cross-sectional survey study.
</td>
<td style="text-align:left;">
The COVID-19 pandemic created new difficulties for people living with
brain injury, their families, and caregivers while amplifying the
challenges of community-based associations that support them. We aimed
to understand the effects of the pandemic on clients who live with brain
injury, as well as on the provision of community brain injury
services/programs in Canada. Online cross-sectional survey conducted in
January 2022. Representatives of brain injury associations across Canada
completed the 31 open- and closed-ended questions about meeting clients’
needs, addressing public health guidelines, and sustaining the
association. Data were analyzed using descriptive statistics
(close-ended questions) and qualitative content analysis (open-ended
questions). Of the 45 key representatives from associations in
Pacific/Western (40%), Central (56%), and Atlantic Canada (4%), the
majority were paid executive directors (67%). Participants reported that
the most frequent psychosocial challenges experienced by their clients
during the pandemic were social isolation (98%), loneliness (96%), and
anxiety (93%). To alleviate these challenges, associations implemented
wellness checks and psychosocial support. Most respondents (91%)
affirmed that clients faced multiple technological barriers, such as a
lack of technological knowledge and financial resources for devices
and/or internet. In the open-ended questions, twenty-nine (64%)
associations reported providing clients with devices, technology
training, and assistance. Regarding public health measures, thirty (67%)
respondents reported that clients had challenges understanding and/or
following public health guidelines. Forty-two associations (93%)
provided tailored information to help clients understand and comply with
public health measures. Although associations (67%) received
pandemic-related funding from the Canadian government they still
struggled with the association’s sustainability. Thirty-four (76%) lost
funding or financial resources that prevented them from delivering
programs or required the use of reserve funds to continue to do so. Only
56% reported receiving sufficient funding to address additional
COVID-19-related expenses. Although the pandemic added further
challenges to the sustainability of brain injury associations across
Canada, they quickly adapted services/programs to respond to the
increasing and varied needs of clients, while complying with protective
measures. To ensure community associations’ survival it is essential to
aptly recognize the vital role played by these associations within the
brain injury care continuum. Copyright © 2023 Salazar, Bottari, Lecours,
McDonald, Gignac, Swaine, Schmidt, Lemsky, Brosda and Engel.
</td>
</tr>
<tr>
<td style="text-align:left;">
38026065
</td>
<td style="text-align:left;">
What are COVID-19 Patient Preferences for and Experiences with Virtual
Care? Findings From a Scoping Review.
</td>
<td style="text-align:left;">
Virtual care became a routine method for healthcare delivery during the
coronavirus disease 2019 (COVID-19) pandemic. Patient preferences are
central to delivering patient-centered and high-quality care. The
pandemic challenged healthcare organizations and providers to quickly
deliver safe healthcare to COVID-19 patients. This resulted in varied
implementation of virtual healthcare services. With an increased focus
on remote COVID-19 monitoring, little research has examined patient
experiences with virtual care. This scoping review examined patient
experiences and preferences with virtual care among community-based
self-isolating COVID-19 patients. We identified a paucity of literature
related to patient experiences and preferences regarding virtual care.
Few articles focused on patient experiences and preferences as a primary
outcome. Our research suggests that (1) patients view virtual care
positively and to be feasible to use; (2) patient access to technology
impacts patient satisfaction and experiences; and (3) to enhance the
patient experience, healthcare organizations and providers need to
support patient use of technology and resolve technology-related issues.
When planning virtual care modalities, purposeful consideration of
patient experiences and preferences is needed to deliver quality
patient-centered care. © The Author(s) 2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38025438
</td>
<td style="text-align:left;">
New and continuing physician-based outpatient mental health care among
children and adolescents during the COVID-19 pandemic in Ontario,
Canada: a population-based study.
</td>
<td style="text-align:left;">
To assess physician-based mental health care utilization during the
COVID-19 pandemic among children and adolescents new to care and those
already engaged with mental health services, and to evaluate differences
by sociodemographic factors. We performed a population-based repeated
cross-sectional study using linked health and administrative databases
in Ontario, Canada among all children and adolescents 3-17 years. We
examined outpatient visit rates per 1,000 population for mental health
concerns for those new to care (no physician-based mental healthcare for
≥1 year) and those with continuing care needs (any physician-based
mental healthcare &lt;1 year) following onset of the pandemic. Among
~2.5 million children and adolescents (48.7% female, mean age 10.1 ± 4.3
years), expected monthly mental health outpatient visits were 1.5/1,000
for those new to mental health care and 5.4/1,000 for those already
engaged in care. Following onset of the pandemic, visit rates for both
groups were above expected \[adjusted rate ratio (aRR) 1.22, 95% CI
1.17, 1.27; aRR 1.10, 95% CI 1.07, 1.12\] for new and continuing care,
respectively. The greatest increase above expected was among females
(new: aRR 1.33, 95% CI 1.25, 1.42; continuing: aRR 1.22 95% CI 1.17,
1.26) and adolescents ages 13-17 years (new: aRR 1.31, 95% CI 1.27,
1.34; continuing: aRR 1.15 95% CI 1.13, 1.17). Mood and anxiety concerns
were prominent among those new to care. In the 18 months following onset
of the pandemic, outpatient mental health care utilization increased for
those with new and continuing care needs, especially among females and
adolescents. Copyright © 2023 Toulany, Vigod, Kurdyak, Stukel, Strauss,
Fu, Guttmann, Guan, Cohen, Chiu, Hepburn, Moran, Gardner, Cappelli,
Sundar and Saunders.
</td>
</tr>
<tr>
<td style="text-align:left;">
38025206
</td>
<td style="text-align:left;">
“A prison is no place for a pandemic”: Canadian prisoners’ collective
action in the time of COVID-19.
</td>
<td style="text-align:left;">
Since the onset of COVID-19, social protest has expanded significantly.
Little, however, has been written on prison-led and prison justice
organizing in the wake of the pandemic-particularly in the Canadian
context. This article is a case study of prisoner organizing in Canada
throughout the first 18 months of COVID-19, which draws on qualitative
interviews, media, and documentary analysis. We argue that the pandemic
generated conditions under which the grievances raised by prisoners, and
the strategies through which they were articulated, made possible a
discursive bridge to the anxieties and grievances experienced by those
in the community, thinning the walls of state-imposed societal
exclusion. We demonstrate that prisons are sites of fierce contestation
and are deeply embedded in, rather than separate from, our society. An
important lesson learned from this case study is the need for prison
organizing campaigns to strategically embrace multi-issue framing and
engage in sustained coalition building. © The Author(s) 2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38025099
</td>
<td style="text-align:left;">
Opening the digital front door for individuals using long-term in-home
ventilation (LIVE) during a pandemic- implementation, feasibility and
acceptability.
</td>
<td style="text-align:left;">
The COVID-19 pandemic led to an unprecedented need for virtual
healthcare that was safe, acceptable and feasible to deliver. In May
2020, we launched the Long-term In-Home Ventilator Engagement (LIVE)
program for ventilator assisted individuals using ventilators hosted on
an e-platform in Ontario, Canada. To assess the acceptability,
appropriateness, feasibility and usability of the LIVE program reported
by patients, family caregivers, and healthcare providers (HCP). We
conducted a cross-sectional study. We provided HCPs participating in the
LIVE program anonymized questionnaires (Acceptability of Intervention
Measure (AIM), Intervention Appropriateness Measure (IAM), Feasibility
of Intervention Measure (FIM), and mHealth App Usability (MAUQ).
Patients and family caregivers completed the AIM and MAUQ.
Questionnaires were administered via an e-platform. We recruited 105/251
(42%) patients and family caregivers and 42/48 (87.5%) HCPs. Patients
and caregivers rated a mean (SD) overall AIM score of 4.3 (0.7) (maximum
score 5; higher scores indicate greater acceptability) and a mean (SD)
overall MAUQ score of 5.8 (1.5) (maximum score 7; higher scores indicate
greater useability). HCPs rated a mean (SD) overall AIM score of 4.3
(0.7), IAM score of 4.3 (0.8), FIM score of 4.2 (0.7) and overall MAUQ
score of 5.6 ± 1.5. There were no differences in AIM ((4.3 (0.7) vs 4.3
(0.8), p = 1) or MAUQ (5.8 (1.5) vs 5.6 (1.5), p = 0.5) scores between
patients/ family caregivers and HCPs. This study suggests that the LIVE
program was acceptable, appropriate, feasible, and usable from the
perspective of patients, family caregivers and HCPs. © The Author(s)
2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
38021457
</td>
<td style="text-align:left;">
Parents’ attitudes regarding their children’s play during COVID-19:
Impact of socioeconomic status and urbanicity.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has had a profound impact on the daily routines of
parents and children. This study explored the influence of socioeconomic
status (SES) and urbanicity on parents’ attitudes toward their
children’s active play opportunities 6 months and 1.5 years into
COVID-19. A sample of 239 Ontario parents of children aged 12 and
younger completed two online surveys (August-December 2020; 2021) to
assess parents’ intentions, beliefs, and comforts concerning their
child’s eventual return to play, in addition to various sociodemographic
and physical activity variables. Descriptive analyses were run as well
as an exploratory factor analysis (EFA) was conducted to group the 14
attitude items into subscales for analysis, to ensure reliability and
validity of attitude measures. In general, parents in communities with
more urban features (e.g., densely populated areas), single-parents,
full-time employed parents, and parents with lower-incomes were more
hesitant to return their children to active play during the pandemic.
Findings from this work highlight SES and urbanicity disparities that
continue to exist during COVID-19. © 2023 The Authors. Published by
Elsevier Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38018786
</td>
<td style="text-align:left;">
Was Virtual Care as Safe as In-Person Care? Analyzing Patient Outcomes
at Seven and Thirty Days in Ontario during the COVID-19 Pandemic.
</td>
<td style="text-align:left;">
In 2020, almost overnight, the paradigm for healthcare interactions
changed in Ontario. To limit person-to-person transmission of COVID-19,
the norm of in-person interactions shifted to virtual care. While this
shift was part of broader public health measures and an acknowledgment
of patient and societal concerns, it also represented a change in care
modalities that had the potential to affect the quality of care
provided, as well as short- and long-term patient outcomes. While public
policy decisions were being made to moderate the use of virtual care at
the end of the declared pandemic, a thorough analysis of short-term
patient outcomes was needed to quantify the impact of virtual care on
the population of Ontario. Copyright © 2023 Longwoods Publishing.
</td>
</tr>
<tr>
<td style="text-align:left;">
38018781
</td>
<td style="text-align:left;">
The Commonwealth Fund Survey of Primary Care Physicians Reveals
Challenges Experienced by Family Doctors and Emphasizes the Need for
Interoperability of Health Information Technologies.
</td>
<td style="text-align:left;">
Electronic health information that is easily accessible and shareable
among healthcare providers and their patients can provide substantial
improvements in Canada’s primary care system and population health
outcomes. The Commonwealth Fund’s (CMWF’s) 2022 International Health
Policy Survey of Primary Care Physicians (CIHI 2023) highlights the
views and experiences of primary care doctors in 10 developed countries,
including Canada. The survey covered various topics related to physician
workload, the use of information technology and coordination of care.
While the COVID-19 pandemic contributed to an increased physician
workload that may have impacted the ability to efficiently coordinate
care with other healthcare providers, Canadian family doctors did close
the gap with other countries as 93% of family doctors are now using
electronic medical records (EMRs) in their practices. The CMWF’s 2022
survey revealed challenges faced by Canadian family doctors in their
practices. However, international comparisons provide opportunities to
learn from other countries and build on the implementation of EMRs as
part of Canada’s shared health priorities. Copyright © 2023 Longwoods
Publishing.
</td>
</tr>
<tr>
<td style="text-align:left;">
38018780
</td>
<td style="text-align:left;">
The Importance of Race and Ethnicity Data in Cardiovascular Health
Research.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has underscored the importance of addressing race
and ethnic disparities in healthcare worldwide. In Canada, however, the
lack of consistent capture of race and ethnicity data has hindered a
comprehensive understanding of these potential disparities. This article
explores the importance of and current progress in collecting race and
ethnic data in Canada and provides examples of its importance in
cardiovascular health outcomes. We believe that a successful
implementation of standardized data collection tools on race and
ethnicity data will shape evidence-based policies to minimize health
disparities in Canada in the future. Copyright © 2023 Longwoods
Publishing.
</td>
</tr>
<tr>
<td style="text-align:left;">
38017498
</td>
<td style="text-align:left;">
Associations between changes in habitual sleep duration and lower
self-rated health among COVID-19 survivors: findings from a survey
across 16 countries/regions.
</td>
<td style="text-align:left;">
Self-rated health (SRH) is widely recognized as a clinically significant
predictor of subsequent mortality risk. Although COVID-19 may impair
SRH, this relationship has not been extensively examined. The present
study aimed to examine the correlation between habitual sleep duration,
changes in sleep duration after infection, and SRH in subjects who have
experienced SARS-CoV-2 infection. Participants from 16 countries
participated in the International COVID Sleep Study-II (ICOSS-II) online
survey in 2021. A total of 10,794 of these participants were included in
the analysis, including 1,509 COVID-19 individuals (who reported that
they had tested positive for COVID-19). SRH was evaluated using a 0-100
linear visual analog scale. Habitual sleep durations of &lt; 6 h and
&gt; 9 h were defined as short and long habitual sleep duration,
respectively. Changes in habitual sleep duration after infection of ≤ -2
h and ≥ 1 h were defined as decreased or increased, respectively.
Participants with COVID-19 had lower SRH scores than non-infected
participants, and those with more severe COVID-19 had a tendency towards
even lower SRH scores. In a multivariate regression analysis of
participants who had experienced COVID-19, both decreased and increased
habitual sleep duration after infection were significantly associated
with lower SRH after controlling for sleep quality (β = -0.056 and
-0.058, respectively, both p &lt; 0.05); however, associations between
current short or long habitual sleep duration and SRH were negligible.
Multinomial logistic regression analysis showed that decreased habitual
sleep duration was significantly related to increased fatigue (odds
ratio \[OR\] = 1.824, p &lt; 0.01), shortness of breath (OR = 1.725, p
&lt; 0.05), diarrhea/nausea/vomiting (OR = 2.636, p &lt; 0.01), and
hallucinations (OR = 5.091, p &lt; 0.05), while increased habitual sleep
duration was significantly related to increased fatigue (OR = 1.900, p
&lt; 0.01). Changes in habitual sleep duration following SARS-CoV-2
infection were associated with lower SRH. Decreased or increased
habitual sleep duration might have a bidirectional relation with
post-COVID-19 symptoms. Further research is needed to better understand
the mechanisms underlying these relationships for in order to improve
SRH in individuals with COVID-19. © 2023. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38016758
</td>
<td style="text-align:left;">
Utilization of physician mental health services by birthing parents with
young children during the COVID-19 pandemic: a population-based,
repeated cross-sectional study.
</td>
<td style="text-align:left;">
The COVID-19 pandemic and nonpharmaceutical interventions that reduced
the spread of infection had impacts on social interaction, schooling and
employment. Concerns have been raised about the impact of these
disruptions on the mental health of high-risk groups, including birthing
parents of young children. This population-based, repeated
cross-sectional study used health administrative databases in Ontario,
Canada, to link children to birth parents and to measure subsequent
mental health visits of birthing parents of younger (age 0-5 yr) and
school-aged (6-12 yr) children. We used a repeated cross-sectional study
design to estimate expected rates for visits to physicians for mental
health diagnoses, based on prepandemic trends (March 2016-February
2020), and to compare those to observed visit rates during the March
2020-November 2021 period of the pandemic. We identified 2 cohorts: 986
870 birthing parents of younger children and 1 012 997 birthing parents
of school-aged children. In both cohorts, observed visit rates were
higher than expected in the June 2020-August 2020 quarter (incidence
rate ratio \[IRR\] 1.13, 95% confidence interval \[CI\] 1.10-1.16; and
IRR 1.10, 95% CI 1.07-1.13, respectively), peaked in December
2020-February 2021 (IRR 1.24, 95% CI 1.20-1.27; and IRR 1.20, 95% CI
1.16-1.23) and remained higher than expected in September 2021-November
2021 (IRR 1.12, 95% CI 1.08-1.16; and IRR 1.09, 95% CI 1.06-1.13). The
increases were driven mostly by visits for mood and anxiety disorders,
and trends in increases were similar across physician type,
birthing-parent age and deprivation quintile. The COVID-19 pandemic was
associated with increased mental health visits for parents of young
children. This raises concerns about mental health impacts and
highlights the need to address these concerns. © 2023 CMA Impact Inc. or
its licensors.
</td>
</tr>
<tr>
<td style="text-align:left;">
38016119
</td>
<td style="text-align:left;">
Future leaders in a Learning Health System: Exploring the Health System
Impact Fellowship.
</td>
<td style="text-align:left;">
The Canadian health system is reeling following the COVID-19 pandemic.
Strains have become growing cracks, with long emergency department wait
times, shortage of human health resources, and growing dissatisfaction
from both clinicians and patients. To address long needed health system
reform in Canada, a modernization of training is required for the next
generation health leaders. The Canadian Institutes of Health Research
Health System Impact Fellowship is an example of a well-funded and
connected training program which prioritizes embedded research and
embedding technically trained scholars with health system partners. The
program has been successful in the scope and impact of its training
outcomes as well as providing health system partners with a pool of
connected and capable scholars. Looking forward, integrating aspects of
evidence synthesis from both domestic and international sources and
adapting a general contractor approach to implementation within the HSIF
could help catalyze Learning Health System reform in Canada.
</td>
</tr>
<tr>
<td style="text-align:left;">
38012843
</td>
<td style="text-align:left;">
SARS-CoV-2 Variants Omicron BA.4/5 and XBB.1.5 Significantly Escape T
Cell Recognition in Solid-organ Transplant Recipients Vaccinated Against
the Ancestral Strain.
</td>
<td style="text-align:left;">
Immune-suppressed solid-organ transplant recipients (SOTRs) display
impaired humoral responses to COVID-19 vaccination, but T cell responses
are incompletely understood. SARS-CoV-2 variants Omicron BA.4/5 (BA.4/5)
and XBB.1.5 escape neutralization by antibodies induced by vaccination
or infection with earlier strains, but T cell recognition of these
lineages in SOTRs is unclear. We characterized Spike-specific T cell
responses to ancestral SARS-CoV-2 and BA.4/5 peptides in 42 kidney,
liver, and lung transplant recipients throughout a 3- or 4-dose
ancestral Spike mRNA vaccination schedule. As the XBB.1.5 variant
emerged during the study, we tested vaccine-induced T cell responses in
10 additional participants using recombinant XBB.1.5 Spike protein.
Using an optimized activation-induced marker assay, we quantified
circulating Spike-specific CD4+ and CD8+ T cells based on
antigen-stimulated expression of CD134, CD69, CD25, CD137, and/or
CD107a. Vaccination strongly induced SARS-CoV-2-specific T cells,
including BA.4/5- and XBB.1.5-reactive T cells, which remained
detectable over time and further increased following a fourth dose.
However, responses to BA.4/5 (1.34- to 1.67-fold lower) XBB.1.5 (2.0- to
18-fold lower) were significantly reduced in magnitude compared with
ancestral strain responses. CD4+ responses correlated with
anti-receptor-binding domain antibodies and predicted subsequent
antibody responses in seronegative individuals. Lung transplant
recipients receiving prednisone and older adults displayed weaker
responses. Ancestral strain vaccination stimulates BA.4/5 and
XBB.1.5-cross-reactive T cells in SOTRs, but at lower magnitudes.
Antigen-specific T cells can predict future antibody responses. Our data
support monitoring both humoral and cellular immunity in SOTRs to track
COVID-19 vaccine immunogenicity against emerging variants. Copyright ©
2023 Wolters Kluwer Health, Inc. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38012311
</td>
<td style="text-align:left;">
Impact of the coronavirus disease 2019 pandemic on equity of access to
hip and knee replacements: a population-level study.
</td>
<td style="text-align:left;">
The COVID-19 pandemic had innumerable impacts on healthcare delivery. In
Canada, this included limitations on inpatient capacity, which resulted
in an increased focus on outpatient surgery for non-emergent cases such
as joint replacements. The objective of this study was to assess whether
the pandemic and the shift towards outpatient surgery had an impact on
access to joint replacement for marginalized patients. Data from
Ontario’s administrative healthcare databases were obtained for all
patients undergoing an elective hip or knee replacement between January
1, 2018 and August 31, 2021. All surgeries performed before March 15,
2020 were classified as “pre-COVID,” while all procedures performed
after that date were classified as “post-COVID.” The Ontario
Marginalization Index domains were used to analyze proportion of
marginalized patients undergoing surgery pre- and post-COVID. A total of
102,743 patients were included-42,812 hip replacements and 59,931 knee
replacements. There was a significant shift towards outpatient surgery
during the post-COVID period (1.1% of all cases pre-COVID to 13.2%
post-COVID, p &lt; 0.001). In the post-COVID cohort, there were
significantly fewer patients from some marginalized groups, as well as
fewer patients with certain co-morbidities, such as congestive heart
failure and chronic obstructive pulmonary disease. The most important
finding of this population-level database study is that, compared to
before the COVID-19 pandemic, there has been a change in the profile of
patients undergoing hip and knee replacements in Ontario, specifically
across a range of indicators. Fewer marginalized patients are undergoing
joint replacement surgery since the COVID-19 pandemic. Further
monitoring of access to joint replacement surgery is required in order
to ensure that surgery is provided to those who are most in need. ©
2023. The Author(s) under exclusive licence to SICOT aisbl.
</td>
</tr>
<tr>
<td style="text-align:left;">
38012044
</td>
<td style="text-align:left;">
Practice Facilitation to Support Family Physicians in Encouraging
COVID-19 Vaccine Uptake: A Multimethod Process Evaluation.
</td>
<td style="text-align:left;">
We offered a practice facilitation intervention to family physicians in
Ontario, Canada, known to have large numbers of patients not yet
vaccinated against coronavirus disease 2019 (COVID-19). We conducted a
multimethod process evaluation embedded within a randomized controlled
trial (clinical trial \#NCT05099497). We collected descriptive
statistics regarding engagement and qualitative interview data from
family physicians and practice facilitators, as well as data from
facilitator field notes. We analyzed and triangulated the data using
thematic analysis and mapped barriers to and enablers for implementation
to structural, organizational, physician, and patient factors. Of the
300 approached, 90 family physicians (30%) accepted facilitation. Of
these, 57% received technical support to identify unvaccinated patients,
29% used trained medical student volunteers to contact patients on their
behalf, and 30% used automated calling to reach patients. Key factors
affecting engagement with the intervention were staff shortages owing to
COVID-19 (structural), clinic characteristics such as technical issues
and gatekeeping by staff, which prevented facilitators from talking with
physicians (organizational), burnout (physician), and specialized
populations that required targeted resources (patient). The
facilitator’s ability to address technical issues and connect family
physicians with medical students helped with engagement. Strategies to
help underresourced family physicians serving high-needs populations for
issues of public health importance, such as vaccine promotion, must
acknowledge the scarcity of physicians’ time and provide new resources.
To successfully engage family physicians, practice facilitators should
seek to build trust and relationships over time, including with
front-office staff. © 2023 Annals of Family Medicine, Inc. 
</td>
</tr>
<tr>
<td style="text-align:left;">
38011077
</td>
<td style="text-align:left;">
The successful and safe conversion of joint arthroplasty to same-day
surgery: A necessity after the COVID-19 pandemic.
</td>
<td style="text-align:left;">
A key strategy to address system pressures on hip and knee arthroplasty
through the COVID-19 pandemic has been to shift procedures to the
outpatient setting. This was a retrospective cohort and case-control
study. Using the Discharge Abstract Database and the National Ambulatory
Care Reporting System databases, we estimated the use of outpatient hip
and knee arthroplasty in Ontario, Canada. After propensity-score
matching, we estimated rates of 90-day readmission, 90-day emergency
department (ED) visit, 1-year mortality, and 1-year infection or
revision. 204,066 elective hip and 341,678 elective knee arthroplasties
were performed from 2010-2022. Annual volumes of hip and knee
arthroplasties increased steadily until 2020. Following the start of the
COVID-19 pandemic (March 1, 2020) through December 31, 2022 there were
7,561 (95% CI 5,435 to 9,688) fewer hip and 20,777 (95% CI 17,382 to
24,172) fewer knee replacements performed than expected. Outpatient
arthroplasties increased as a share of all surgeries from 1%
pre-pandemic to 39% (hip) and 36% (knee) by 2022. Among inpatient
arthroplasties, the tendency to discharge to home did not change since
the start of the pandemic. During the COVID-19 era, patients receiving
arthroplasty in the outpatient setting had a similar or lower risk of
readmission than matched patients receiving inpatient arthroplasty
\[hip: RR 0.65 (0.56-0.76); knee: RR 0.86 (0.76-0.97)\]; ED visits
\[hip: RR 0.78 (0.73-0.83); knee: RR 0.92 (0.88-0.96)\]; and mortality,
infection, or revision \[hip: RR 0.65 (0.45-0.93); knee: 0.90
(0.64-1.26)\]. Following the start of the COVID-19 pandemic in Ontario,
the volume of outpatient hip and knee arthroplasties performed increased
despite a reduction in overall arthroplasty volumes. This shift in
surgical volumes from the inpatient to outpatient setting coincided with
pressures on hospitals to retain inpatient bed capacity. Patients
receiving arthroplasty in the outpatient setting had relatively similar
outcomes to those receiving inpatient surgery after matching on known
sociodemographic and clinical characteristics. Copyright: © 2023 Habbous
et al. This is an open access article distributed under the terms of the
Creative Commons Attribution License, which permits unrestricted use,
distribution, and reproduction in any medium, provided the original
author and source are credited.
</td>
</tr>
<tr>
<td style="text-align:left;">
38008266
</td>
<td style="text-align:left;">
A multimethods randomized trial found that plain language versions
improved adults understanding of health recommendations.
</td>
<td style="text-align:left;">
To make informed decisions, the general population should have access to
accessible and understandable health recommendations. To compare
understanding, accessibility, usability, satisfaction, intention to
implement, and preference of adults provided with a digital “Plain
Language Recommendation” (PLR) format vs. the original “Standard
Language Version” (SLV). An allocation-concealed, blinded, controlled
superiority trial and a qualitative study to understand participant
preferences. An international on-line survey. 488 adults with some
English proficiency. 67.8% of participants identified as female, 62.3%
were from the Americas, 70.1% identified as white, 32.2% had a
bachelor’s degree as their highest completed education, and 42% said
they were very comfortable reading health information. In collaboration
with patient partners, advisors, and the Cochrane Consumer Network, we
developed a plain language format of guideline recommendations (PLRs) to
compare their effectiveness vs. the original standard language versions
(SLVs) as published in the source guideline. We selected two
recommendations about COVID-19 vaccine, similar in their content, to
compare our versions, one from the World Health Organization (WHO) and
one from Centers for Disease Control and Prevention (CDC). The primary
outcome was understanding, measured as the proportion of correct
responses to seven comprehension questions. Secondary outcomes were
accessibility, usability, satisfaction, preference, and intended
behavior, measured on a 1-7 scale. Participants randomized to the PLR
group had a higher proportion of correct responses to the understanding
questions for the WHO recommendation (mean difference \[MD\] of 19.8%,
95% confidence interval \[CI\] 14.7-24.9%; P &lt; 0.001) but this
difference was smaller and not statistically significant for the CDC
recommendation (MD of 3.9%, 95% CI -0.7% to 8.3%; P = 0.096). However,
regardless of the recommendation, participants found the PLRs more
accessible, (MD of 1.2 on the seven-point scale, 95% CI 0.9-1.4%; P &lt;
0.001) and more satisfying (MD of 1.2, 95% CI 0.9-1.4%; P &lt; 0.001).
They were also more likely to follow the recommendation if they had not
already followed it (MD of 1.2, 95% CI 0.7-1.8%; P &lt; 0.001) and share
it with other people they know (MD of 1.9, 95% CI 0.5-1.2%; P &lt;
0.001). There was no significant difference in the preference between
the two formats (MD of -0.3, 95% CI -0.5% to 0.03%; P = 0.078). The
qualitative interviews supported and contextualized these findings.
Health information provided in a PLR format improved understanding,
accessibility, usability, and satisfaction and thereby has the potential
to shape public decision-making behavior. Copyright © 2023 The Authors.
Published by Elsevier Inc. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
38003268
</td>
<td style="text-align:left;">
SCARF Genes in COVID-19 and Kidney Disease: A Path to
Comorbidity-Specific Therapies.
</td>
<td style="text-align:left;">
Severe acute respiratory syndrome coronavirus-2 (SARS-CoV-2) causes
coronavirus disease 2019 (COVID-19), which has killed ~7 million persons
worldwide. Chronic kidney disease (CKD) is the most common risk factor
for severe COVID-19 and one that most increases the risk of
COVID-19-related death. Moreover, CKD increases the risk of acute kidney
injury (AKI), and COVID-19 patients with AKI are at an increased risk of
death. However, the molecular basis underlying this risk has not been
well characterized. CKD patients are at increased risk of death from
multiple infections, to which immune deficiency in non-specific host
defenses may contribute. However, COVID-19-associated AKI has specific
molecular features and CKD modulates the local (kidney) and systemic
(lung, aorta) expression of host genes encoding coronavirus-associated
receptors and factors (SCARFs), which SARS-CoV-2 hijacks to enter cells
and replicate. We review the interaction between kidney disease and
COVID-19, including the over 200 host genes that may influence the
severity of COVID-19, and provide evidence suggesting that kidney
disease may modulate the expression of SCARF genes and other key host
genes involved in an effective adaptive defense against coronaviruses.
Given the poor response of certain CKD populations (e.g., kidney
transplant recipients) to SARS-CoV-2 vaccines and their suboptimal
outcomes when infected, we propose a research agenda focusing on CKD to
develop the concept of comorbidity-specific targeted therapeutic
approaches to SARS-CoV-2 infection or to future coronavirus infections.
</td>
</tr>
<tr>
<td style="text-align:left;">
38002510
</td>
<td style="text-align:left;">
Alexithymia, Burnout, and Hopelessness in a Large Sample of Healthcare
Workers during the Third Wave of COVID-19 in Italy.
</td>
<td style="text-align:left;">
In the present study, we aimed to assess the frequency of and the
relationships between alexithymia, burnout, and hopelessness in a large
sample of healthcare workers (HCWs) during the third wave of COVID-19 in
Italy. Alexithymia was evaluated by the Italian version of the 20-item
Toronto Alexithymia Scale (TAS-20) and its subscales Difficulty in
Identifying Feelings (DIF), Difficulty in Describing Feelings (DDF), and
Externally Oriented Thinking (EOT), burnout was measured with the scales
emotional exhaustion (EE), depersonalisation (DP), and personal
accomplishment (PA) of the Maslach Burnout Test (MBI), hopelessness was
measured using the Beck Hopelessness Scale (BHS), and irritability
(IRR), depression (DEP), and anxiety (ANX) were evaluated with the
Italian version of the Irritability’ Depression’ Anxiety Scale (IDA).
This cross-sectional study recruited a sample of 1445 HCWs from a large
urban healthcare facility in Italy from 1 May to 31 June 2021. The
comparison between individuals that were positive (n = 214, 14.8%) or
not for alexithymia (n = 1231, 85.2%), controlling for age, gender, and
working seniority, revealed that positive subjects showed higher scores
on BHS, EE, DP IRR, DEP, ANX, DIF, DDF, and EOT and lower on PA than the
not positive ones (p &lt; 0.001). In the linear regression model, higher
working seniority as well as higher EE, IRR, DEP, ANX, and DDF scores
and lower PA were associated with higher hopelessness. In conclusion,
increased hopelessness was associated with higher burnout and
alexithymia. Comprehensive strategies should be implemented to support
HCWs’ mental health and mitigate the negative consequences of
alexithymia, burnout, and hopelessness.
</td>
</tr>
<tr>
<td style="text-align:left;">
38001619
</td>
<td style="text-align:left;">
Impact of the COVID-19 Pandemic on Staging Oncologic PET/CT Imaging and
Patient Outcome in a Public Healthcare Context: Overview and Follow Up
of the First Two Years of the Pandemic.
</td>
<td style="text-align:left;">
To assess the impact of the COVID-19 pandemic on the diagnosis, staging
and outcome of a selected population throughout the first two years of
the pandemic, we evaluated oncology patients undergoing PET/CT at our
institution. A retrospective population of lung cancer, melanoma,
lymphoma and head and neck cancer patients staged using PET/CT during
the first 6 months of the years 2019, 2020 and 2021 were included for
analysis. The year in which the PET was performed was our exposure
variable, and our two main outcomes were stage at the time of the PET/CT
and overall survival (OS). A total of 1572 PET/CTs were performed for
staging purposes during the first 6 months of 2019, 2020 and 2021. The
median age was 66 (IQR 16), and 915 (58%) were males. The most prevalent
staged cancer was lung cancer (643, 41%). The univariate analysis of
staging at PET/CT and OS by year of PET/CT were not significantly
different. The multivariate Cox regression of non-COVID-19 significantly
different variables at univariate analysis and the year of PET/CT
determined that lung cancer (HR 1.76 CI95 1.23-2.53, p &lt; 0.05), stage
III (HR 3.63 CI95 2.21-5.98, p &lt; 0.05), stage IV (HR 11.06 CI95
7.04-17.36, p &lt; 0.05) and age at diagnosis (HR 1.04 CI95 1.02-1.05, p
&lt; 0.05) had increased risks of death. We did not find significantly
higher stages or reduced OS when assessing the year PET/CT was
performed. Furthermore, OS was not significantly modified by the year
patients were staged, even when controlled for non-COVID-19 significant
variables (age, type of cancer, stage and gender).
</td>
</tr>
<tr>
<td style="text-align:left;">
38001483
</td>
<td style="text-align:left;">
Corruption risks in health procurement during the COVID-19 pandemic and
anti-corruption, transparency and accountability (ACTA) mechanisms to
reduce these risks: a rapid review.
</td>
<td style="text-align:left;">
Health systems are often susceptible to corruption risks. Corruption
within health systems has been found to negatively affect the efficacy,
safety, and, significantly, equitable distribution of health products.
Enforcing effective anti-corruption mechanisms is important to reduce
the risks of corruption but requires first an understanding of the ways
in which corruption manifests. When there are public health crises, such
as the COVID-19 pandemic, corruption risks can increase due to the need
for accelerated rates of resource deployment that may result in the
bypassing of standard operating procedures. A rapid review was conducted
to examine factors that increased corruption risks during the COVID-19
pandemic as well as potential anti-corruption, transparency and
accountability (ACTA) mechanisms to reduce these risks. A search was
conducted including terms related to corruption, COVID-19, and health
systems from January 2020 until January 2022. In addition, relevant grey
literature websites were hand searched for items. A single reviewer
screened the search results removing those that did not meet the
inclusion criteria. This reviewer then extracted data relevant to the
research objectives from the included articles. 20 academic articles and
17 grey literature pieces were included in this review. Majority of the
included articles described cases of substandard and falsified products.
Several papers attributed shortages of these products as a major factor
for the emergence of falsified versions. Majority of described
corruption instances occurred in low- and middle-income countries. The
main affected products identified were chloroquine tablets, personal
protective equipment, COVID-19 vaccine, and diagnostic tests. Half of
the articles were able to offer potential anti-corruption strategies.
Shortages of health products during the COVID-19 pandemic seemed to be
associated with increased corruption risks. We found that low- and
middle-income countries are particularly vulnerable to corruption during
global emergencies. Lastly, there is a need for additional research on
effective anti-corruption mechanisms. © 2023. The Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
38001328
</td>
<td style="text-align:left;">
Implementing digital emergency medicine: a call to action.
</td>
<td style="text-align:left;">
As digital technologies continue to impact medicine, emergency medicine
providers have an opportunity to work together to harness these
technologies and shape their implementation within our healthcare
system. COVID-19 and the rapid scaling of virtual care provide an
example of how profoundly emergency medicine can be affected by digital
technology, both positively and negatively. This example also
strengthens the case for why EM providers can help lead the integration
of digital technologies within our broader healthcare system. As virtual
care becomes a permanent fixture of our system, and other technologies
such as AI and wearables break into Canadian healthcare, more advocacy,
research, and health system leadership will be required to best leverage
these tools. This paper outlines the purpose and outputs of the newly
founded CAEP Digital Emergency Medicine (DigEM) Committee, with the hope
of inspiring further interest amongst CAEP members and creating
opportunities to collaborate with other organizations within CAEP and
across EM groups nationwide. © 2023. The Author(s), under exclusive
licence to Canadian Association of Emergency Physicians (CAEP)/
Association Canadienne de Médecine d’Urgence (ACMU).
</td>
</tr>
<tr>
<td style="text-align:left;">
38001037
</td>
<td style="text-align:left;">
Protection conferred by COVID-19 vaccination, prior SARS-CoV-2
infection, or hybrid immunity against Omicron-associated severe outcomes
among community-dwelling adults.
</td>
<td style="text-align:left;">
We assessed protection from COVID-19 vaccines and/or prior SARS-CoV-2
infection against Omicron-associated severe outcomes during successive
sublineage-predominant periods. We used a test-negative design to
estimate protection by vaccines and/or prior infection against
hospitalization/death among community-dwelling, PCR-tested adults aged
≥50 years in Ontario, Canada between January 2, 2022 and June 30, 2023.
Multivariable logistic regression was used to estimate the relative
change in the odds of hospitalization/death with each vaccine dose (2-5)
and/or prior PCR-confirmed SARS-CoV-2 infection (compared with
unvaccinated, uninfected subjects) up to 15 months since the last
vaccination or infection. We included 18,526 cases with
Omicron-associated severe outcomes and 90,778 test-negative controls.
Vaccine protection was high during BA.1/BA.2 predominance, but was
generally &lt;50% during periods of BA.4/BA.5 and BQ/XBB predominance
without boosters. A third/fourth dose transiently increased protection
during BA.4/BA.5 predominance (third-dose, 6-month: 68%, 95%CI 63%-72%;
fourth-dose, 6-month: 80%, 95%CI 77%-83%), but was lower and waned
quickly during BQ/XBB predominance (third-dose, 6-month: 59%, 95%CI
48%-67%; 12-month: 49%, 95%CI 41%-56%; fourth-dose, 6-month: 62%, 95%CI
56%-68%, 12-months: 51%, 95%CI 41%-56%). Hybrid immunity conferred
nearly 90% protection throughout BA.1/BA.2 and BA.4/BA.5 predominance,
but was reduced during BQ/XBB predominance (third-dose, 6-month: 60%,
95%CI 36%-75%; fourth-dose, 6-month: 63%, 95%CI 42%-76%). Protection was
restored with a fifth dose (bivalent; 6-month: 91%, 95%CI 79%-96%).
Prior infection alone did not confer lasting protection. Protection from
COVID-19 vaccines and/or prior SARS-CoV-2 infections against severe
outcomes is reduced when immune-evasive variants/subvariants emerge and
may also wane over time. Our findings support a variant-adapted booster
vaccination strategy with periodic review. © The Author(s) 2023.
Published by Oxford University Press on behalf of Infectious Diseases
Society of America.
</td>
</tr>
<tr>
<td style="text-align:left;">
38000083
</td>
<td style="text-align:left;">
A review of the reliability of remote neuropsychological assessment.
</td>
<td style="text-align:left;">
The provision of clinical neuropsychological services has predominately
been undertaken by way of standardized administration in a face-to-face
setting. Interpretation of psychometric findings in this context is
dependent on the use of normative comparison. When the standardization
in which such psychometric measures are employed deviates from how they
were employed in the context of the development of its associated norms,
one is left to question the reliability and hence, validity of any such
findings and in turn, diagnostic decision making. In light of the
current COVID-19 pandemic and resultant social distancing direction,
face-to-face neuropsychological assessment has been challenging to
undertake. As such, remote (i.e., virtual) neuropsychological assessment
has become an obvious solution. Here, and before the results from remote
neuropsychological assessment can be said to stand on firm scientific
grounds, it is paramount to ensure that results garnered remotely are
reliable and valid. To this end, we undertook a review of the literature
and present an overview of the landscape. To date, the literature shows
evidence for the reliability of remote administration and the clinical
implications are paramount. When and where needed, neuropsychologists,
psychometric technicians and examinees may no longer need to be in the
same physical space to undergo an assessment. These findings are most
relevant given the physical distancing practices because of COVID-19.
And whilst remote assessment should never supplant face-to-face
neuropsychological assessments, it does serve as a valid alternative
when necessary.
</td>
</tr>
<tr>
<td style="text-align:left;">
37995548
</td>
<td style="text-align:left;">
A qualitative analysis of gestational surrogates’ healthcare experiences
during the COVID-19 pandemic.
</td>
<td style="text-align:left;">
No empirical data are available on the healthcare experiences of
surrogates during the COVID-19 pandemic. This study aimed to examine the
impact of pandemic-control measures on surrogates’ fertility, pregnancy
and birthing experiences. Sampling frame included eligible surrogates
who were actively involved in a surrogacy process at an academic IVF
centre during the pandemic (03/2020 to 02/2022). Data were collected
between 29/04/2022 and 31/07/2022 using an anonymous 85-item online
survey that included twelve open-ended questions. Free-text comments
were analysed by thematic analysis. The response rate was 50.7%
(338/667). Of the 320 completed surveys used for analysis, 609 comments
were collected from 206 respondents. Twelve main themes and thirty-six
sub-themes grouped under ‘vaccination’, ‘fertility treatment’,
‘pregnancy care’, and ‘surrogacy birth’ were identified. Three in five
surrogates found the control measures highly or moderately affected
their surrogacy experiences. Themes involving loneliness and isolation
frequently emerged when essential surrogacy support was restricted by
the visitor protocols implemented at healthcare facilities. Our findings
show that restricting or limiting intended parents’ in-person
involvement increased surrogates’ feelings of isolation and made the
overall surrogacy experience less rewarding and fulfilling. Furthermore,
the childbirth experiences of surrogates were mostly negative,
suggesting that hospitals were ill-equipped to manage all births,
including surrogacy births, during the pandemic. Our findings highlight
the needs to rethink how surrogacy care and maternity services could be
strengthened to better serve the needs of surrogates during times of
public health crises, such as COVID-19, while still allowing for risk
mitigation and maximising patient safety. Copyright © 2023 Elsevier
Ltd. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
37993984
</td>
<td style="text-align:left;">
Efficacy of an Electronic Cognitive Behavioral Therapy Program Delivered
via the Online Psychotherapy Tool for Depression and Anxiety Related to
the COVID-19 Pandemic: Pre-Post Pilot Study.
</td>
<td style="text-align:left;">
Lockdowns and social distancing resulting from the COVID-19 pandemic
have worsened the population’s mental health and made it more difficult
for individuals to receive care. Electronic cognitive behavioral therapy
(e-CBT) is a cost-effective and evidence-based treatment for anxiety and
depression and can be accessed remotely. The objective of the study was
to investigate the efficacy of online psychotherapy tailored to
depression and anxiety symptoms during the pandemic. The pilot study
used a pre-post design to evaluate the efficacy of a 9-week e-CBT
program designed for individuals with depression and anxiety affected by
the pandemic. Participants were adults (N=59) diagnosed with major
depressive disorder and generalized anxiety disorder, whose mental
health symptoms initiated or worsened during the COVID-19 pandemic. The
online psychotherapy program focused on teaching coping, mindfulness,
and problem-solving skills. Symptoms of anxiety and depression,
resilience, and quality of life were assessed. Participants demonstrated
significant improvements in symptoms of anxiety (P=.02) and depression
(P=.03) after the intervention. Similar trends were observed in the
intention-to-treat analysis. No significant differences were observed in
resilience and quality-of-life measures. The sample comprised mostly
females, making it challenging to discern the benefits of the
intervention in males. Although a pre-post design is less rigorous than
a controlled trial, this design was selected to observe changes in
scores during a critical period. e-CBT for COVID-19 is an effective and
accessible treatment option. Improvements in clinical symptoms of
anxiety and depression can be observed in individuals whose mental
health is affected by the COVID-19 pandemic. ClinicalTrials.gov
NCT04476667; <https://clinicaltrials.gov/study/NCT04476667>.
RR2-10.2196/24913. ©Elnaz Moghimi, Callum Stephenson, Anika Agarwal,
Niloofar Nikjoo, Niloufar Malakouti, Gina Layzell, Anne O’Riordan,
Jasleen Jagayat, Amirhossein Shirazi, Gilmar Gutierrez, Ferwa Khan,
Charmy Patel, Megan Yang, Mohsen Omrani, Nazanin Alavi. Originally
published in JMIR Mental Health (<https://mental.jmir.org>), 25.12.2023.
</td>
</tr>
<tr>
<td style="text-align:left;">
37993365
</td>
<td style="text-align:left;">
Parental mental health trajectories over the COVID-19 pandemic and links
with childhood adversity and pandemic stress.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has created significant disruptions, with parents
of school-age children being identified as a vulnerable population.
Limited research has longitudinally tracked the mental health
trajectories of parents over the active pandemic period. In addition,
parents’ history of adverse (ACEs) and benevolent (BCEs) childhood
experiences may compound or attenuate the effect of COVID-19 stressors
on parental psychopathology. To identify distinct longitudinal
trajectories of parental mental health over the COVID-19 pandemic and
how these trajectories are associated with parental ACEs, BCEs, and
COVID-19 stress. 547 parents of 5-18-year-old children from the U.K.,
U.S., Canada, and Australia. Growth mixture modelling was used to
identify trajectories of parental mental health (distress, anxiety,
post-traumatic stress, and substance use) from May 2020 to October 2021.
COVID-19 stress, ACEs, and BCEs were assessed as predictors of mental
health trajectories via multinomial logistic regression. Two-class
trajectories of “Low Stable” and “Moderate Stable” symptoms were
identified for psychological distress and anxiety. Three-class
trajectories of “Low Stable”, “High Stable”, and “High Decreasing”
symptoms were observed for post-traumatic stress. Reliable trajectories
for substance use could not be identified. Multinomial logistic
regression showed that COVID-19 stress and ACEs independently predicted
membership in trajectories of greater mental health impairment, while
BCEs independently predicted membership in trajectories of lower
psychological distress. Parents experienced mostly stable mental health
symptomatology, with trajectories varying by overall symptom severity.
COVID-19 stress, ACEs, and BCEs each appear to play a role in parents’
mental health during this unique historical period. Copyright © 2023
Elsevier Ltd. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
37992799
</td>
<td style="text-align:left;">
Impact of the COVID-19 pandemic on Canadian emergency medical system
management of out-of-hospital cardiac arrest: A retrospective cohort
study.
</td>
<td style="text-align:left;">
We sought to describe the impact of the COVID-19 pandemic on the care
provided by Canadian emergency medical system (EMS) clinicians to
patients suffering out of hospital cardiac arrest (OHCA), and whether
any observed changes persisted beyond the initial phase of the pandemic.
We analysed cases of adult, non-traumatic, OHCA from the Canadian
Resuscitation Outcome Consortium (CanROC) registry who were treated
between January 27th, 2018, and December 31st, 2021. We used adjusted
regression models and interrupted time series analysis to examine the
impact of the COVID-19 pandemic (January 27th, 2020 - December 31st,
2021)on the care provided to patients with OHCA by EMS clinicians. There
were 12,947 cases of OHCA recorded in the CanROC registry in the
pre-COVID-19 period and 17,488 during the COVID-19 period. We observed a
reduction in the cumulative number of defibrillations provided by EMS
(aRR 0.91, 95% CI 0.89 - 0.93, p &lt; 0.01), a reduction in the odds of
attempts at intubation (aOR 0.33, 95% CI 0.31 - 0.34, p &lt; 0.01),
higher rates of supraglottic airway use (aOR 1.23, 95% CI 1.16-1.30, p
&lt; 0.01), a reduction in vascular access (aOR for intravenous access
0.84, 95% CI 0.79 - 0.89, p &lt; 0.01; aOR for intraosseous access 0.89,
95% CI 0.82 - 0.96, p &lt; 0.01), a reduction in the odds of epinephrine
administration (aOR 0.89, 95% CI 0.85 - 0.94, p &lt; 0.01), and higher
odds of resuscitation termination on scene (aOR 1.38, 95% CI 1.31 -
1.46, p &lt; 0.01). Delays to initiation of chest compressions (2 min.
vs. 3 min., p &lt; 0.01), intubation (16 min. vs. 19 min., p = 0.01),
and epinephrine administration (11 min. vs. 13 min., p &lt; 0.01) were
observed, whilst supraglottic airways were inserted earlier (11 min.
vs. 10 min., p &lt; 0.01). The COVID-19 pandemic was associated with
substantial changes in EMS management of OHCA. EMS leaders should
consider these findings to optimise current OHCA management and prepare
for future pandemics. Copyright © 2023 The Author(s). Published by
Elsevier B.V. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
37992183
</td>
<td style="text-align:left;">
The Novavax Heterologous COVID Booster Demonstrates Lower Reactogenicity
Than mRNA: A Targeted Review.
</td>
<td style="text-align:left;">
COVID-19 continues to be a global health concern and booster doses are
necessary for maintaining vaccine-mediated protection, limiting the
spread of SARS-CoV-2. Despite multiple COVID vaccine options, global
booster uptake remains low. Reactogenicity, the occurrence of adverse
local/systemic side effects, plays a crucial role in vaccine uptake and
acceptance, particularly for booster doses. We conducted a targeted
review of the reactogenicity of authorized/approved mRNA and
protein-based vaccines demonstrated by clinical trials and real-world
evidence. It was found that mRNA-based boosters show a higher incidence
and an increased severity of reactogenicity compared with the Novavax
protein-based COVID vaccine, NVX-CoV2373. In a recent NIAID study, the
incidence of pain/tenderness, swelling, erythema, fatigue/malaise,
headache, muscle pain, or fever was higher in individuals boosted with
BNT162b2 (0.4 to 41.6% absolute increase) or mRNA-1273 (5.5 to 55.0%
absolute increase) compared with NVX-CoV2373. Evidence suggests that
NVX-CoV2373, when utilized as a heterologous booster, demonstrates less
reactogenicity compared with mRNA vaccines, which, if communicated to
hesitant individuals, may strengthen booster uptake rates worldwide. ©
The Author(s) 2023. Published by Oxford University Press on behalf of
Infectious Diseases Society of America.
</td>
</tr>
<tr>
<td style="text-align:left;">
37991889
</td>
<td style="text-align:left;">
Canadian respiratory therapists who considered leaving their clinical
position experienced elevated moral distress and adverse psychological
and functional outcomes during the COVID-19 pandemic.
</td>
<td style="text-align:left;">
Respiratory therapists (RTs) faced morally distressing situations
throughout the COVID-19 pandemic, including working with limited
resources and facilitating video calls for families of dying patients.
Moral distress is associated with a host of adverse psychological and
functional outcomes (e.g. depression, anxiety, symptoms of posttraumatic
stress disorder \[PTSD\] and functional impairment) and consideration of
position departure. The purpose of this study was to understand the
impact of moral distress and its associated psychological and functional
outcomes on consideration to leave a clinical position among Canadian
RTs during the COVID-19 pandemic. Canadian RTs (N = 213) completed an
online survey between February and June 2021. Basic demographic
information (e.g. age, sex, gender) and psychometrically validated
measures of moral distress, depression, anxiety, stress, PTSD,
dissociation, functional impairment, resilience and adverse childhood
experiences were collected. One in four RTs reported considering leaving
their position. RTs considering leaving reported elevated levels of
moral distress and adverse psychological and functional outcomes
compared to RTs not considering leaving. Over half (54.5%) of those
considering leaving scored above the cut-off for potential diagnosis of
PTSD. Previous consideration to leave a position and having left a
position in the past each significantly increased the odds of currently
considering leaving, along with system-related moral distress and
symptoms of PTSD, but the contribution of these latter factors was
small. Canadian RTs considering leaving their position reported elevated
levels of distress and adverse psychological and functional outcomes,
yet these individual-level factors appear unlikely to be the primary
factors underlying RTs’ consideration to leave, because their effects
were small. Further research is required to identify broader,
organizational factors that may contribute to consideration of position
departure among Canadian RTs.
</td>
</tr>
<tr>
<td style="text-align:left;">
37991692
</td>
<td style="text-align:left;">
Factors affecting hesitancy toward COVID-19 vaccine booster doses in
Canada: a cross-national survey.
</td>
<td style="text-align:left;">
COVID-19 transmission, emergence of variants of concern, and weakened
immunity have led to recommended vaccine booster doses for COVID-19.
Vaccine hesitancy challenges broad immunization coverage. We deployed a
cross-national survey to investigate knowledge, beliefs, and behaviours
toward continued COVID-19 vaccination. We administered a national,
cross-sectional online survey among adults in Canada between March 16
and March 26, 2022. We utilized descriptive statistics to summarize our
sample, and tested for demographic differences, perceptions of vaccine
effectiveness, recommended doses, and trust in decisions, using the
Rao-Scott correction for weighted chi-squared tests. Multivariable
logistic regression was adjusted for relevant covariates to identify
sociodemographic factors and beliefs associated with vaccine hesitancy.
We collected 2202 completed questionnaires. Lower education status (high
school: odds ratio (OR) 1.90, 95% confidence interval (CI) 1.29, 2.81)
and having children (OR 1.89, CI 1.39, 2.57) were associated with
increased odds of experiencing hesitancy toward a booster dose, while
higher income (\$100,000-\$149,999: OR 0.60, CI 0.39, 0.91; \$150,000 or
more: OR 0.49, CI 0.29, 0.82) was associated with decreased odds.
Disbelief in vaccine effectiveness (against infection: OR 3.69, CI 1.98,
6.90; serious illness: OR 3.15, CI 1.69, 5.86), disagreeing with
government decision-making (somewhat disagree: OR 2.70, CI 1.38, 5.29;
strongly disagree: OR 4.62, CI 2.20, 9.7), and beliefs in
over-vaccinating (OR 2.07, CI 1.53, 2.80) were found associated with
booster dose hesitancy. COVID-19 vaccine hesitancy may develop or
increase regarding subsequent vaccines. Our findings indicate factors to
consider when targeting vaccine-hesitant populations. © 2023. The
Author(s).
</td>
</tr>
<tr>
<td style="text-align:left;">
37989512
</td>
<td style="text-align:left;">
SARS-CoV-2 vaccination prevalence by mental health diagnosis: a
population-based cross-sectional study in Ontario, Canada.
</td>
<td style="text-align:left;">
Since the onset of the COVID-19 pandemic, there has been concern about
the impact of SARS-CoV-2 infection among individuals with mental
illnesses. We analyzed the SARS-CoV-2 vaccination status of Ontarians
with and without a history of mental illness. We conducted a
population-based cross-sectional study of all community-dwelling Ontario
residents aged 19 years and older as of Sept. 17, 2021. We used health
administrative data to categorize Ontario residents with a mental
disorder (anxiety, mood, substance use, psychotic or other disorder)
within the previous 5 years. Vaccine receipt as of Sept. 17, 2021, was
compared between individuals with and without a history of mental
illness. Our sample included 11 900 868 adult Ontario residents. The
proportion of individuals not fully vaccinated (2 doses) was higher
among those with substance use disorders (37.7%) or psychotic disorders
(32.6%) than among those with no mental disorders (22.9%), whereas there
were similar proportions among those with anxiety disorders (23.5%),
mood disorders (21.5%) and other disorders (22.1%). After adjustment for
age, sex, neighbourhood income and homelessness, individuals with
psychotic disorders (adjusted prevalence ratio 1.19, 95% confidence
interval \[CI\] 1.18-1.20) and substance use disorders (adjusted
prevalence ratio 1.35, 95% CI 1.34-1.35) were more likely to be
partially vaccinated or unvaccinated relative to individuals with no
mental disorders. Our study found that psychotic disorders and substance
use disorders were associated with an increased prevalence of being less
than fully vaccinated. Efforts to ensure such individuals have access to
vaccinations, while challenging, are critical to ensuring the ongoing
risks of death and other adverse consequences of SARS-CoV-2 infection
are mitigated in this high-risk population. © 2023 CMA Impact Inc. or
its licensors.
</td>
</tr>
<tr>
<td style="text-align:left;">
37987194
</td>
<td style="text-align:left;">
The role of thriving in mental health among people with intellectual and
developmental disabilities during the COVID-19 pandemic in Canada.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has had a negative impact on the mental health of
people with intellectual and developmental disabilities. Numerous
pandemic-related stressors experienced by people with intellectual and
developmental disabilities may have impacted their ability to thrive,
which has been linked to mental health outcomes. The current study
examined the associations among COVID-19 stressors, thriving, and mental
health problems among youth and adults with intellectual and
developmental disabilities. Caregivers of 159 people with intellectual
and developmental disabilities between 12 and 35 years of age from
Canada completed an online questionnaire. A mediation analysis revealed
that COVID-19 stressors were positively associated with mental health
problems, and that thriving partially mediated this association. Our
findings suggest that experiences of thriving may be an important target
for mental health support for people with intellectual and developmental
disabilities. © 2023 The Authors. Journal of Applied Research in
Intellectual Disabilities published by John Wiley &amp; Sons Ltd. 
</td>
</tr>
<tr>
<td style="text-align:left;">
37986769
</td>
<td style="text-align:left;">
Post-Vaccination Syndrome: A Descriptive Analysis of Reported Symptoms
and Patient Experiences After Covid-19 Immunization.
</td>
<td style="text-align:left;">
A chronic post-vaccination syndrome (PVS) after covid-19 vaccination has
been reported but has yet to be well characterized. We included 241
individuals aged 18 and older who self-reported PVS after covid-19
vaccination and who joined the online Yale Listen to Immune, Symptom and
Treatment Experiences Now (LISTEN) Study from May 2022 to July 2023. We
summarized their demographics, health status, symptoms, treatments
tried, and overall experience. The median age of participants was 46
years (interquartile range \[IQR\]: 38 to 56), with 192 (80%)
identifying as female, 209 (87%) as non-Hispanic White, and 211 (88%)
from the United States. Among these participants with PVS, 127 (55%) had
received the BNT162b2 \[Pfizer-BioNTech\] vaccine, and 86 (37%) received
the mRNA-1273 \[Moderna\] vaccine. The median time from the day of index
vaccination to symptom onset was three days (IQR: 1 day to 8 days). The
time from vaccination to symptom survey completion was 595 days (IQR:
417 to 661 days). The median Euro-QoL visual analogue scale score was 50
(IQR: 39 to 70). The five most common symptoms were exercise intolerance
(71%), excessive fatigue (69%), numbness (63%), brain fog (63%), and
neuropathy (63%). In the week before survey completion, participants
reported feeling unease (93%), fearfulness (82%), and overwhelmed by
worries (81%), as well as feelings of helplessness (80%), anxiety (76%),
depression (76%), hopelessness (72%), and worthlessness (49%) at least
once. Participants reported a median of 20 (IQR: 13 to 30) interventions
to treat their condition. In this study, individuals who reported PVS
after covid-19 vaccination had low health status, high symptom burden,
and high psychosocial stress despite trying many treatments. There is a
need for continued investigation to understand and treat this condition.
</td>
</tr>
<tr>
<td style="text-align:left;">
37984936
</td>
<td style="text-align:left;">
Novel obesity treatments.
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
37983393
</td>
<td style="text-align:left;">
Detour or New Direction: The Impact of the COVID-19 Pandemic on the
Professional Identity Formation of Postgraduate Residents.
</td>
<td style="text-align:left;">
The COVID-19 pandemic has resulted in numerous disruptions to health
professions education training programs. Much attention has been given
to the impact of these disruptions on formal learning opportunities in
training; however, little attention has been given to the impact on
professional socialization and professional identity formation. This
study explored the impact of the pandemic and resultant curricular
changes on the professional identity of family medicine residents. 23
family medicine residents at the University of Toronto were interviewed
between September 2020 and September 2022. Using symbolic interactionism
as a theoretical framework, thematic analysis explored the meanings
residents attributed to both experiences that were disrupted due to the
pandemic, and new experiences that resulted from these disruptions.
Participant responses reflected that disruptions in training did not
always align with their expectations for family medicine and plans for
future practice; however, these new experiences also reinforced their
understanding of what it means to be a family physician. While
participants felt the pandemic represented a loss of agency and
negatively impacted relationships in their training program, it also
provided a sense of belonging and membership in their profession.
Finally, these new experiences continually blurred the line between
professional and personal identities through the impact of the pandemic
on participants’ sense of well-being and safety. The impact of the
pandemic on training experiences extends beyond the loss of formal
learning opportunities. Participant responses reflect the collective
influence of the formal, informal, and hidden curriculum on the
professional socialization and professional identity formation of
residents-and how these different curricular influences were disrupted
due to the pandemic. These training experiences have important
implications for the future practice of residents who completed their
training during the pandemic and highlight the role of training programs
in supporting the professional identity formation of residents.
Copyright © 2023 by the Association of American Medical Colleges.
</td>
</tr>
<tr>
<td style="text-align:left;">
37983033
</td>
<td style="text-align:left;">
Cancer Screening Disparities Before and After the COVID-19 Pandemic.
</td>
<td style="text-align:left;">
Breast, cervical, and colorectal cancer-screening disparities existed
prior to the COVID-19 pandemic, and it is unclear whether those have
changed since the pandemic. To assess whether changes in screening from
before the pandemic to after the pandemic varied for immigrants and for
people with limited income. This population-based, cross-sectional
study, using data from March 31, 2019, and March 31, 2022, included
adults in Ontario, Canada, the country’s most populous province, with
more than 14 million people, almost 30% of whom are immigrants. At both
dates, the screening-eligible population for each cancer type was
assessed. Neighborhood income quintile, immigrant status, and primary
care model type. For each cancer screening type, the main outcome was
whether the screening-eligible population was up to date on screening (a
binary outcome) on March 31, 2019, and March 31, 2022. Up to date on
screening was defined as having had a mammogram in the previous 2 years,
a Papanicolaou test in the previous 3 years, and a fecal test in the
previous 2 years or a flexible sigmoidoscopy or colonoscopy in the
previous 10 years. The overall cohort on March 31, 2019, included 1 666
943 women (100%) eligible for breast screening (mean \[SD\] age, 59.9
\[5.1\] years), 3 918 225 women (100%) eligible for cervical screening
(mean \[SD\] age, 45.5 \[13.2\] years), and 3 886 345 people eligible
for colorectal screening (51.4% female; mean \[SD\] age, 61.8 \[6.4\]
years). The proportion of people up to date on screening in Ontario
decreased for breast, cervical, and colorectal cancers, with the largest
decrease for breast screening (from 61.1% before the pandemic to 51.7%
\[difference, -9.4 percentage points\]) and the smallest decrease for
colorectal screening (from 65.9% to 62.0% \[difference, -3.9 percentage
points\]). Preexisting disparities in screening for people living in
low-income neighborhoods and for immigrants widened for breast screening
and colorectal screening. For breast screening, compared with income
quintile 5 (highest), the β estimate for income quintile 1 (lowest) was
-1.16 (95% CI, -1.56 to -0.77); for immigrant vs nonimmigrant, the β
estimate was -1.51 (95% CI, -1.84 to -1.18). For colorectal screening,
compared with income quintile 5, the β estimate for quntile 1 was -1.29
(95% CI, 16 -1.53 to -1.06); for immigrant vs nonimmigrant, the β
estimate was -1.41 (95% CI, -1.61 to -1.21). The lowest screening rates
both before and after the COVID-19 pandemic were for people who had no
identifiable family physician (eg, moving from 11.3% in 2019 to 9.6% in
2022 up to date for breast cancer). In addition, patients of
interprofessional, team-based primary care models had significantly
smaller reductions in β estimates for breast (2.14 \[95% CI, 1.79 to
2.49\]), cervical (1.72 \[95% CI, 1.46 to 1.98\]), and colorectal (2.15
\[95% CI, 1.95 to 2.36\]) postpandemic screening and higher uptake of
screening in general compared with patients of other primary care
models. In this cross-sectional study in Ontario that included 2 time
points, widening disparities before compared with after the COVID-19
pandemic were found for breast cancer and colorectal cancer screening
based on income and immigrant status, but smaller declines in
disparities were found among patients of interprofessional, team-based
primary care models than among their counterparts. Policy makers should
investigate the value of prioritizing and investing in improving access
to team-based primary care for people who are immigrants and/or with
limited income.
</td>
</tr>
<tr>
<td style="text-align:left;">
37981484
</td>
<td style="text-align:left;">
Investigating the impact of COVID-19 on the provision of pediatric burn
care.
</td>
<td style="text-align:left;">
The COVID-19 pandemic had widespread effects on the healthcare system
due to public health regulations and restrictions. The following study
shares trends observed during these extraordinary circumstances to
investigate the impact of the COVID-19 pandemic on the provision of
pediatric burn care at an American-Burn-Association verified tertiary
pediatric hospital in Ontario, Canada. Pediatric burn patient data for
new burn patients between March 17th, 2019, and March 17th, 2021, was
retrospectively extracted and two cohorts of patients were formed:
pre-pandemic and pandemic, through which statistical analysis was
performed. No significant changes in the number of admitted patients,
age, and sex of patients were observed. However, a significant increase
in fire/flame burns was observed during the pandemic period.
Additionally, a decrease in follow-up care was observed while an
increase in acute burn care (wound care and surgical interventions) was
found for the pandemic cohort. Despite changes to hospital care
facilities to maximize resources for COVID-19-related care, our findings
demonstrate that burn care remained an essential service and significant
reductions in patient volumes were not observed. Overall, this study
will aid in future planning and management for the provision of
pediatric burn resources during similar public health emergencies.
Copyright © 2023 Elsevier Ltd and International Society of Burns
Injuries. All rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
37979717
</td>
<td style="text-align:left;">
Therapeutic Heparin in Non-ICU Patients Hospitalized for COVID-19 in the
Accelerating COVID-19 Therapeutic Interventions and Vaccines 4 Acute
Trial: Effect on 3-Month Symptoms and Quality of Life.
</td>
<td style="text-align:left;">
Therapeutic-dose heparin decreased days requiring organ support in
noncritically ill patients hospitalized for COVID-19, but its impact on
persistent symptoms or quality of life (QoL) is unclear. In the ACTIV-4a
trial, was randomization of patients hospitalized for COVID-19 illness
to therapeutic-dose vs prophylactic heparin associated with fewer
symptoms and better QoL at 90 days? This was an open-label randomized
controlled trial at 34 hospitals in the United States and Spain. A total
of 727 noncritically ill patients hospitalized for COVID-19 from
September 2020 to June 2021 were randomized to therapeutic-dose vs
prophylactic heparin. Only patients with 90-day data on symptoms and QoL
were analyzed. We ascertained symptoms and QoL by the EuroQol
5-Dimension 5-Level (EQ-5D-5L) at 90-day follow-up in a preplanned
analysis for the ACTIV-4a trial. Individual domains assessed by the
EQ-5D-5L included mobility, self-care, usual activities,
pain/discomfort, and anxiety/depression. Univariate and multivariate
analyses were performed. Among 571 patients, 288 (50.4%) reported at
least one symptom. Among 410 patients, 148 (36.1%) reported moderate to
severe impairment in one or more domains of the EQ-5D-5L. The presence
of 90-day symptoms was associated with moderate-severe impairment in the
EQ-5D-5L domains of mobility (adjusted OR \[aOR\], 2.37; 95% CI,
1.22-4.59), usual activities (aOR, 3.66; 95% CI, 1.75-7.65), pain (aOR,
2.43; 95% CI, 1.43-4.12), and anxiety (aOR, 4.32; 95% CI, 2.06-9.02),
compared with patients reporting no symptoms There were no differences
in symptoms or in the overall EQ-5D-5L index score between treatment
groups. Therapeutic-dose heparin was associated with less
moderate-severe impairment in all physical functioning domains
(mobility, self-care, usual activities) but was independently
significant only in the self-care domain (aOR, 0.32; 95% CI, 0.11-0.96).
In a randomized controlled trial of hospitalized noncritically ill
patients with COVID-19, therapeutic-dose heparin was associated with
less severe impairment in the self-care domain of EQ-5D-5L. However,
this type of impairment was uncommon, affecting 23 individuals.
ClinicalTrials.gov; No.: NCT04505774; URL: www. gov. Copyright © 2023
American College of Chest Physicians. Published by Elsevier Inc. All
rights reserved.
</td>
</tr>
<tr>
<td style="text-align:left;">
37975914
</td>
<td style="text-align:left;">
How long elective surgery should be delayed from COVID-19 infection in
pediatric patients?
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
37975711
</td>
<td style="text-align:left;">
The impact of the COVID-19 pandemic on blood culture practices and
bloodstream infections.
</td>
<td style="text-align:left;">
Bacterial infections are a significant cause of morbidity and mortality
worldwide. In the wake of the COVID-19 pandemic, previous studies have
demonstrated pandemic-related shifts in the epidemiology of bacterial
bloodstream infections (BSIs) in the general population and in specific
hospital systems. Our study uses a large, comprehensive data set
stratified by setting \[community, long-term care (LTC), and hospital\]
to uniquely demonstrate how the effect of the COVID-19 pandemic on BSIs
and testing practices varies by healthcare setting. We showed that,
while the number of false-positive blood culture results generally
increased during the pandemic, this effect did not apply to hospitalized
patients. We also found that many infections were likely
under-recognized in patients in the community and in LTC, demonstrating
the importance of maintaining healthcare for these groups during crises.
Last, we found a decrease in infections caused by certain pathogens in
the community, suggesting some secondary benefits of pandemic-related
public health measures.
</td>
</tr>
</tbody>
</table>

</div>

``` r
# The table will likely be huge. How can we make the output look better?
# (one idea: kableExtra::scroll_box())
```

Done! Knit the document, commit, and push.

## Final Pro Tip (optional)

You can still share the HTML document on github. You can include a link
in your `README.md` file as the following:

``` md
View [here](https://htmlpreview.github.io/?https://github.com/Jennifer-xxx/JSC370-labs/blob/master/lab06/README.html)
```

For example, if we wanted to add a direct link the HTML page of lecture
6, we could do something like the following:

``` md
View Week 6 Lecture [here]()
```

# GOThresher: a program to remove annotation biases from protein function annotation datasets

GOThresher removes annotation bias from [GAF](http://www.geneontology.org/page/go-annotation-file-formats) files based on annotation information content, GO evidence, annotation source, number of proteins annotated from a given source, and date.  GOThresher accepts one or more GAF files as input. The motivation for GAF lies in the observation that many organism annotations are biased due to high throughput experimental studies ([1](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003063)). Removing such annotation biases can help present a more balanaced picture of protein annotations for a given organism or set of proteins.  

-------------

## Prerequisites

### Required modules:

GoThresher requires Python 3.5 or newer with the following libraries installed:
* [networkx](https://networkx.github.io/)
* [matplotlib](https://matplotlib.org/)
* [numpy](http://www.numpy.org/)
* [Biopython](http://biopython.org/)
* [xlsxwriter](http://xlsxwriter.readthedocs.io/)

Modules can be automatically installed using `pip`, or obtained from their respective websites.

### Required files:

GoThresher requires an obo formatted version of the Gene Ontology. Depending on your needs, this would usually be one of [go-basic.obo](http://purl.obolibrary.org/obo/go/go-basic.obo) or [go.obo](http://purl.obolibrary.org/obo/go.obo). For more details and to download either the most recent daily version or the latest version go to the [Gene Ontology website](http://geneontology.org/page/download-ontology). 

<!--
A program debias_prep.py has been provided in the package. This program builds the graphs for each of the ontologies and puts them in three different files. Hence the .obo files are not needed. This program has been provided so that if the hierarchy changes then this program can be used to regenerate the  files. In addition to the three hierarchy graphs for the three ontologies it also generates the mapping for alternate GO_ID to actual GO_ID. It also generates the mapping from one GO_ID to all its ancestors. 
-->

-------------

## Installation

GOThresher is available on PyPi, so the best way to install GOThresher is through `pip`.

You can install GOThresher by running:
```
$ pip install debias
```
OR
```
$ pip install git+git://github.com/FriedbergLab/debias
```

Alternatively, it is possible to **manually download** from GitHub or **clone the repository** using the following command:
```
$ git clone https://github.com/FriedbergLab/debias
$ cd debias
```
and install GOThresher by running: 
```
$ pip install .
```

-------------

## Generate initial mapping files

<!--
A program debias_prep.py has been provided in the package. This program builds the graphs for each of the ontologies and puts them in three different files. Hence the .obo files are not needed. This program has been provided so that if the hierarchy changes then this program can be used to regenerate the  files. In addition to the three hierarchy graphs for the three ontologies it also generates the mapping for alternate GO_ID to actual GO_ID. It also generates the mapping from one GO_ID to all its ancestors. 
-->

GOThresher requires graphs of the three ontologies (MF, CC, BP), mapping of GO terms to all of its ancestors, and mapping of alternate GO IDs to actual GO IDs. These files can be generated by running `debias_prep`.

**Run this command only once to generate the mapping files**

```
$ mkdir data
$ debias_prep -i ./data/<GOFILE>
```

`<GOFILE>` will usually be one of `go.obo` or `go-basic.obo`.

`debias_prep` will generate seven files in total:
1. Three files corresponds to the three ontologies
2. Three files corresponds to the mapping between each GO_term and its ancestors in its own respective ontology
3. One file containing mapping from alternate GO_ID to actual GO_ID. 
 
IMPORTANT: This command needs to be run again when a new version of ontology is available and updated graphs/mapping files need to be used for analysis. In that case, please use `debias_prep` after downloading a new go.obo file.

Following files will be generated within `./data` folder:

```
1. ./data/alt_to_id.graph : Needed to obtain mapping from alternate GO_ID to actual GO_ID
2. ./data/mf.graph : The MFO Ontology graph
3. ./data/bp.graph : The BPO Ontology graph
4. ./data/cc.graph : The CCO Ontology graph
5. ./data/mf_ancestors.map : The MFO Ancestors map
6. ./data/bp_ancestors.map : The BPO Ancestors map
7. ./data/cc_ancestors.map : The CCO Ancestors map
```

-------------

## Quick setup

1. Download the latest `go.obo` or `go-basic.obo` file from http://www.geneontology.org/ontology/ 

2. Run the program `debias_prep` program and provide the downloaded `obo` file. See the usage details [here](https://github.com/parnaljoshi/debias#generate-initial-mapping-files). This program needs to be run only when a new `obo` file needs to be used.

3. Run the program `debias` 

-------------

## GOThresher usage

```
usage: debias [-h] [--prefix PREFIX] [--cutoff_prot CUTOFF_PROT]
                 [--cutoff_attn CUTOFF_ATTN] [--output OUTPUT]
                 [--evidence EVIDENCE [EVIDENCE ...] | --evidence_inverse
                 EVIDENCE_INVERSE [EVIDENCE_INVERSE ...]] --input INPUT
                 [INPUT ...] [--aspect ASPECT [ASPECT ...]]
                 [--assigned_by ASSIGNED_BY [ASSIGNED_BY ...] |
                 --assigned_by_inverse ASSIGNED_BY_INVERSE
                 [ASSIGNED_BY_INVERSE ...]] [--recalculate RECALCULATE]
                 [--info_threshold_Wyatt_Clark_percentile INFO_THRESHOLD_WYATT_CLARK_PERCENTILE | --info_threshold_Wyatt_Clark INFO_THRESHOLD_WYATT_CLARK]
                 [--info_threshold_Phillip_Lord_percentile INFO_THRESHOLD_PHILLIP_LORD_PERCENTILE | --info_threshold_Phillip_Lord INFO_THRESHOLD_PHILLIP_LORD]
                 [--verbose VERBOSE] [--date_before DATE_BEFORE]
                 [--date_after DATE_AFTER] [--single_file SINGLE_FILE]
                 [--select_references SELECT_REFERENCES [SELECT_REFERENCES ...]
                 | --select_references_inverse SELECT_REFERENCES_INVERSE
                 [SELECT_REFERENCES_INVERSE ...]] [--report REPORT]
                 [-histogram HISTOGRAM]

optional arguments:
  -h, --help            show this help message and exit
  --prefix PREFIX, -pref PREFIX
                        Add a prefix to the name of your output files.
  --cutoff_prot CUTOFF_PROT, -cprot CUTOFF_PROT
                        The threshold level for deciding to eliminate
                        annotations which come from references that annotate
                        more than the given 'threshold' number of PROTEINS
  --cutoff_attn CUTOFF_ATTN, -cattn CUTOFF_ATTN
                        The threshold level for deciding to eliminate
                        annotations which come from references that annotate
                        more than the given 'threshold' number of ANNOTATIONS
  --output OUTPUT, -odir OUTPUT
                        Writes the final outputs to the directory in this
                        path.
  --evidence EVIDENCE [EVIDENCE ...], -e EVIDENCE [EVIDENCE ...]
                        Accepts Standard Evidence Codes outlined in
                        http://geneontology.org/page/guide-go-evidence-codes.
                        All 3 letter code for each standard evidence is
                        acceptable. In addition to that EXPEC is accepted
                        which will pull out all annotations which are made
                        experimentally. COMPEC will extract all annotations
                        which have been done computationally. Similarly,
                        AUTHEC and CUREC are also accepted. Cannot be provided
                        if -einv is provided
  --evidence_inverse EVIDENCE_INVERSE [EVIDENCE_INVERSE ...], -einv EVIDENCE_INVERSE [EVIDENCE_INVERSE ...]
                        Leaves out the provided Evidence Codes. Cannot be
                        provided if -e is provided
  --aspect ASPECT [ASPECT ...], -a ASPECT [ASPECT ...]
                        Enter P, C or F for Biological Process, Cellular
                        Component or Molecular Function respectively
  --assigned_by ASSIGNED_BY [ASSIGNED_BY ...], -assgn ASSIGNED_BY [ASSIGNED_BY ...]
                        Choose only those annotations which have been
                        annotated by the provided list of databases. Cannot be
                        provided if -assgninv is provided
  --assigned_by_inverse ASSIGNED_BY_INVERSE [ASSIGNED_BY_INVERSE ...], -assgninv ASSIGNED_BY_INVERSE [ASSIGNED_BY_INVERSE ...]
                        Choose only those annotations which have NOT been
                        annotated by the provided list of databases. Cannot be
                        provided if -assgn is provided
  --recalculate RECALCULATE, -recal RECALCULATE
                        Set this to 1 if you wish to enforce the recalculation
                        of the Information Accretion for every GO term.
                        Calculation of the information accretion is time
                        consuming. Therefore keep it to zero if you are
                        performing rerun on old data. The program will then
                        read the information accretion values from a file
                        which it wrote to in the previous run of the program
  --info_threshold_Wyatt_Clark_percentile INFO_THRESHOLD_WYATT_CLARK_PERCENTILE, -WCTHRESHp INFO_THRESHOLD_WYATT_CLARK_PERCENTILE
                        Provide the percentile p. All annotations having
                        information content below p will be discarded
  --info_threshold_Wyatt_Clark INFO_THRESHOLD_WYATT_CLARK, -WCTHRESH INFO_THRESHOLD_WYATT_CLARK
                        Provide a threshold value t. All annotations having
                        information content below t will be discarded
  --info_threshold_Phillip_Lord_percentile INFO_THRESHOLD_PHILLIP_LORD_PERCENTILE, -PLTHRESHp INFO_THRESHOLD_PHILLIP_LORD_PERCENTILE
                        Provide the percentile p. All annotations having
                        information content below p will be discarded. So if 5 is provided, proteins annotated by 
                        terms whose score is in the top 5%  will be  left in, the rest will be discarded.
  --info_threshold_Phillip_Lord INFO_THRESHOLD_PHILLIP_LORD, -PLTHRESH INFO_THRESHOLD_PHILLIP_LORD
                        Provide a  value t. All annotations having
                        information content below t will be discarded
  --verbose VERBOSE, -v VERBOSE
                        Set this argument to 1 if you wish to view the outcome
                        of each operation on the console
  --date_before DATE_BEFORE, -dbfr DATE_BEFORE
                        The date entered here will be parsed by the parser
                        from dateutil package. For more information on
                        acceptable date formats please visit
                        https://github.com/dateutil/dateutil/. All annotations
                        made prior to this date will be picked up
  --date_after DATE_AFTER, -daftr DATE_AFTER
                        The date entered here will be parsed by the parser
                        from dateutil package. For more information on
                        acceptable date formats please visit
                        https://github.com/dateutil/dateutil/. All annotations
                        made after this date will be picked up
  --single_file SINGLE_FILE, -single SINGLE_FILE
                        Set to 1 in order to output the results of each
                        individual species in a single file.
  --select_references SELECT_REFERENCES [SELECT_REFERENCES ...], -selref SELECT_REFERENCES [SELECT_REFERENCES ...]
                        Provide the paths to files which contain references
                        you wish to select. It is possible to include
                        references in case you wish to select annotations made
                        by a few references. This will prompt the program to
                        interpret string which have the keywords
                        'GO_REF','PMID' and 'Reactome' as a GO reference.
                        Strings which do not contain that keyword will be
                        interpreted as a file path which the program will
                        except to contain a list of GO references. The program
                        will accept a mixture of GO_REF and file names. It is
                        also possible to choose all references of a particular
                        category and a handful of references from another. For
                        example if you wish to choose all PMID references,
                        just put PMID. The program will then select all PMID
                        references. Currently the program can accept PMID,
                        GO_REF and Reactome
  --select_references_inverse SELECT_REFERENCES_INVERSE [SELECT_REFERENCES_INVERSE ...], -selrefinv SELECT_REFERENCES_INVERSE [SELECT_REFERENCES_INVERSE ...]
                        Works like -selref but does not select the references
                        which have been provided as input
  --report REPORT, -r REPORT
                        Provide the path where the report file will be stored.
                        If you are providing a path please make sure your path
                        ends with a '/'. Otherwise the program will assume the
                        last string after the final '/' as the name of the
                        report file. A single report file will be generated.
                        Information for each species will be put into
                        individual worksheets.
  --histogram HISTOGRAM, -hist HISTOGRAM
                        Set this option to 1 if you wish to view the histogram
                        of GO_TERM frequency before and after debiasing is
                        performed with respect to cutoffs based on number of
                        proteins or annotations. If you wish to save the file
                        then please enter a filepath. If you are providing a
                        path please make sure your path ends with a '/'.
                        Otherwise the program will assume the last string
                        after the final '/' as the name of the image file.
                        Separate histograms will be generated for each
                        species.

Required arguments:
  --input INPUT [INPUT ...], -i INPUT [INPUT ...]
                        The input file path. Please remember the name of the
                        file must start with goa in front of it, with the name
                        of the species following separated by an underscore
```

NOTE: Files inside the folder "temp" are generated when `-recal` is set to 1.

-------------

## Example usage

### Step 1: Generating graphs and mapping files

```debias_prep -i data/go.obo```

This command will generate seven files in total. Three files corresponds
to the three ontologies. Three files corresponds to the mapping between
each GO_term and its ancestors in its own respective ontology. The last
file contains mapping from alternate GO_ID to actual GO_ID. Please use
this command every time you update GOFILE. 

### Step 2: Running GOThresher

1. ```debias -cprot 100 -i example_data/goa_yeast.gaf example_data/goa_dicty.gaf -a C -WCTHRESHp 2 -recal 1```

This command reads from two input files one for yeast and the other for
dicty. The -a C only selects the annotations which are CCO. The
-WCTHRESHp argument specifies that the Wyatt Clark Threshold is a 2
percentile, which means all annotations having a Wyatt Clark Information
content below 2% will be removed. Instead of providing a percentage
value one can also provide a threshold value using the argument
-WCTHRESH. In addition to that, those annotations will be removed which
have been annotated by references that have in turn annotated more than
100 **proteins**. The output will be put in the current directory. It is
necessary to have -recal 1 in this command since the GO_term to IC has
to be created. Subsequent runs with different threshold and all other
parameters fised is possible **WITHOUT** providing the argument -recal.
This command will lead to 3 output files. One each for the two organisms
and the third one is where both the organisms are combined. 

2. ```debias -i example_data/goa_yeast.gaf example_data/goa_dicty.gaf -a C P -PLTHRESHp 30 -e EXPEC IBA -odir example_data/output -single 1```

This command will read from two input files, select CCO and BPO
annotations. Further, it will **choose** only those annotations which
have been made experimentally or have been annotated computationally as
"IBA" (Inferred from Biological aspect of Ancestor). In addition to that
it will discard all annotations which have a Phillip Lord information
content less than 30%. Instead of providing a percentage value one can
also provide a threshold value using the argument -PLTHRESH. The final
output will be put inside the data/output directory. You can include non
existent paths. The program will attempt to create the folders if
required permissions are present. This will lead to only one file, since
the -single argument has been provided, which will contain all the
selected annotations from both the organisms. 

3. ```debias -cattn 1000 -i example_data/goa_yeast.gaf example_data/goa_dicty.gaf -a C P -einv COMPEC -pref testing -selrefinv Reactome```

This command will read from two input files, select CCO and BPO
annotations. Further, it will **discard**  those annotations which have
been made computationally. The program further filters out all
annotations made by "Reactome". All files will be prefixed with the
string "testing". Since the program creates a meaningful name for each
file, the user has been given the opportunity to give a prefix.

### Running test data

To test all the commands mentioned above, you can run the shell script named test.sh in the tests directory.

```
git clone https://github.com/FriedbergLab/debias
cd ./debias/tests
bash test.sh
```


# WIKIR Thai
A python tool for building large scale Wikipedia-based Information Retrieval datasets

Currently supported languages:  
**English**  
**French**  
**Spanish**  
**Italian**  
**Thai** (Added in this fork)

### Changelog for this fork
1. feat: add support for Thai language including pipeline, reproducible scripts, and downloadable datasets (THwikIR10k and THwikIRS10k)
2. fix: makes [BM25 optional depedency](https://github.com/getalp/wikIR/pull/2)
3. fix: catch json decode error when reading wiki.json file
4. fix: update docs to correctly describe [default len_doc](https://github.com/getalp/wikIR/issues/4)

# Table of Contents
1. [Requirements](#Requirements)
2. [Installation](#Installation)
3. [Usage](#Usage)
4. [Details](#Details)
5. [Example](#Example)
6. [Reproducibility](#Reproducibility)
7. [Downloads](#Downloads)
8. [More languages](#More-languages)
9. [Citation](#Citation)
10. [References](#References)

## Requirements
  * Python 3.6+
  * [NumPy](https://numpy.org) and [SciPy](https://www.scipy.org) 
  * [pytrec_eval](https://github.com/cvangysel/pytrec_eval) to evaluate the runs 
  * [nltk](https://www.nltk.org/) library to perform stemming and stopword removal in several languages
  * [Pandas](https://pandas.pydata.org) library to be able to save the dataset as a dataframe compatible with [MatchZoo](https://github.com/NTMC-Community/MatchZoo) 
  * **Optional**:
    * [Rank-BM25](https://github.com/dorianbrown/rank_bm25) as a first efficient ranking stage if you want to use [MatchZoo](https://github.com/NTMC-Community/MatchZoo)
    * [MatchZoo](https://github.com/NTMC-Community/MatchZoo) in order to train and evaluate neural networks on the collection
    * [PythaiNLP](https://github.com/PyThaiNLP/pythainlp) and [PyICU](https://github.com/ovalhub/pyicu) in order to tokenize Thai language and remove stopwords for BM25 ranking (We use ICU tokenizer simply for speed, but you can easily modify the code in `build-wikiIR.py` for other tokenizers)

*****
## Installation

Install wikIR
```bash
git clone --recurse-submodules https://github.com/getalp/wikIR.git
cd wikIR
pip install -r requirements.txt
```

Install [Rank-BM25](https://github.com/dorianbrown/rank_bm25) (optional)
```bash
pip install git+ssh://git@github.com/dorianbrown/rank_bm25.git
```

Install [MatchZoo](https://github.com/NTMC-Community/MatchZoo) (optional)
```bash
git clone https://github.com/NTMC-Community/MatchZoo.git
cd MatchZoo
python setup.py install
```

Install [PythaiNLP](https://github.com/PyThaiNLP/pythainlp) and [PyICU](https://github.com/ovalhub/pyicu) (optional)
```bash
pip install pythainlp==3.1.*
pip install PyICU
```
*****
## Usage

  * Download and extract a XML wikipedia dump file from [here](https://dumps.wikimedia.org/backup-index.html) 
  * Use [Wikiextractor](https://github.com/attardi/wikiextractor) to get the text of the wikipedia pages in a signle json file, for example : 
```bash
python wikiextractor/WikiExtractor.py input --output - --bytes 100G --links --quiet --json > output.json
```
Where input is the XML wikipedia dump file and output is the output in json format

  * Call our script
```
python build_wikIR.py [-i,--input] [-o,--output_dir] 
                      [-m,--max_docs] [-d,--len_doc] [-q,--len_query] [-l,--min_len_doc]
                      [-e,--min_nb_rel_doc] [-v,--validation_part] [-t,--test_part]
                      [-k,--k] [-i,--title_queries] [-f,--only_first_links] 
                      [-s,--skip_first_sentence] [-c,--lower_cased] [-j,--json] 
                      [-x,--xml] [-b,--bm25] [-r,--random_seed] 
```

```
arguments : 

    [-i,--input]                  The json file produced by wikiextractor
    
    [-o,--output_dir]             Directory where the collection will be stored

optional argument:

    [--language]                  Language of the input json file
                                  Possible values: 'en','fr','es','it', 'th'
                                  Default value: 'en'

    [-m,--max_docs]               Maximum number of documents in the collection
                                  Default value None

    [-d,--len_doc]                Number of max tokens in documents
                                  Default value 200: all tokens are preserved
                                  
    [-q,--len_query]              Number of max tokens in queries
                                  Default value None: all tokens are preserved
    
    [-l,--min_len_doc]            Mininum number of tokens required for an article 
                                  to be added to the dataset as a document
                                  Default value 200
    
    [-e,--min_nb_rel_doc]         Mininum number of relevant documents required for 
                                  a query to be added to the dataset
                                  Default value 5
    
    [-v,--validation_part]        Number of queries in the validation set
                                  Default value 1000
    
    [-t,--test_part]              Number of queries in the test set
                                  Default value 1000
    
    [-i,--title_queries]          If used, queries are build using the title of 
                                  the article 
                                  If not used, queries are build using the first
                                  sentence of the article
    
    [-f,--only_first_links]       If used, only the links in the first sentence of 
                                  articles will be used to build qrels
                                  If not used, all links up to len_doc token will
                                  be used to build qrels
    
    [-s,--skip_first_sentence]    If used, the first sentence of articles is not 
                                  used in documents
    
    [-c,--lower_cased]            If used, all characters are lowercase
    
    [-j,--json]                   If used, documents and queries are saved in json
                                  If not used, documents and queries are saved in
                                  csv as dataframes compatible with matchzoo
                                  
    [-x,--xml]                    If used, documents and queries are saved in xml
                                  format compatible with Terrier IRS 
                                  If not used, documents and queries are saved in
                                  csv as dataframes compatible with matchzoo                                  
                                  
    [-b,--bm25]                   If used, perform and save results of BM25 ranking 
                                  model on the collection
                                
    [-k,--k]                      If BM25 is used, indicates the number of documents 
                                  per query saved 
                                  Default value 100
    
    [-r,--random_seed]            Random seed
                                  Default value 27355
        
```

*****
## Details
  * The data construction process is similar to [[1]](#References) and [[2]](#References)
  * Article used to build the documents (article titles are removed from documents)	
  * Title or first sentence of each article is used to build the queries
  * We assign a **relevance of 2** if the query and document were extracted from the **same article**
  * We assign a **relevance of 1** if there is a **link from the article of the document to the article of the query**
    * For example the document [Autism](https://en.wikipedia.org/wiki/Autism) is relevant to the query [Developmental disorder](https://en.wikipedia.org/wiki/Developmental_disorder).


*****
## Example

Execute the follwing lines in the wikIR directory

Download the english wikipedia dump from 01/11/2019
```bash
wget https://dumps.wikimedia.org/enwiki/20191101/enwiki-20191101-pages-articles-multistream.xml.bz2
```

Extract the file 
```bash
bzip2 -dk enwiki-20191101-pages-articles-multistream.xml.bz2
```

Use Wikiextractor (ignore the WARNING: Template errors in article)
```bash
python wikiextractor/WikiExtractor.py enwiki-20191101-pages-articles-multistream.xml --output - --bytes 100G --links --quiet --json > enwiki.json
```

Use wikIR builder
```bash
python build_wikIR.py --input enwiki.json --output_dir wikIR1k --max_docs 370000 -tfscb 
```

:warning: **Do not forget to delete the dowloaded and intermediary files** :warning:

```bash
rm enwiki-20191101-pages-articles-multistream.xml.bz2
rm enwiki-20191101-pages-articles-multistream.xml
rm wiki.json
```
Train and evaluate neural networks with Matchzoo

```bash
python wikIR/matchzoo_experiment.py -c config.json
```

Display results in a format compatible with a latex table

```bash
python wikIR/display_res.py -c config.json
```


*****
## Downloads

The *wikIR1k* and *wikIR59k* datasets presented in our [paper](https://arxiv.org/abs/1912.01901) are available for download

You can download *wikIR1k* [here](https://zenodo.org/record/3565761#.Xep7Z-ZKg5k).

You can download *wikIR59k* [here](https://zenodo.org/record/3557342#.XeD3c-ZKg5k).

*****
## More languages

Datasets in more languages are also available:

|   | English | French | Spanish | Italian | Thai
|:-------------:|:-------------:|:-------------:|:-----:|:-----:|:---:|
| title queries | [ENwikIR59k](https://zenodo.org/record/3557342#.Xe_gQ-ZKhhE) | [FRwikIR14k](https://zenodo.org/record/3569718#.Xe_gYOZKhhE) | [ESwikIR13k](https://zenodo.org/record/3569724#.Xe_geOZKhhE) | [ITwikIR16k](https://zenodo.org/record/3569732#.Xe_gj-ZKhhE) | [THwikIR10k](https://github.com/new5558/wikIR-thai/blob/master/data/THwikIR10k.zip) |
| first sentence queries| [ENwikIRS59k](https://zenodo.org/record/3569708#.Xe_gT-ZKhhE) | [FRwikIRS14k](https://zenodo.org/record/3569720#.Xe_ga-ZKhhE) | [ESwikIRS13k](https://zenodo.org/record/3569726#.Xe_ggeZKhhE) | [ITwikIRS16k](https://zenodo.org/record/3569734#.Xe_gnOZKhhE) | [THwikIRS10k](https://github.com/new5558/wikIR-thai/blob/master/data/THwikIRS10k.zip) |


We propose datasets with short and well defined queries built from titles (ENwikIR59k, FRwikIR14k) and datasets with long and noisy queries built from first sentences (ENwikIRS59k, FRwikIRS14k) to study robustness of IR models to noise.


*****
## Reproducibility

### Reproduce *wikIR1k* dataset

To reproduce the *wikIR1k* dataset, execute the follwing lines in the wikIR directory

```bash
wget https://dumps.wikimedia.org/enwiki/20191101/enwiki-20191101-pages-articles-multistream.xml.bz2
bzip2 -d enwiki-20191101-pages-articles-multistream.xml.bz2
python wikiextractor/WikiExtractor.py enwiki-20191101-pages-articles-multistream.xml --output - --bytes 100G --links --quiet --json > enwiki.json
rm enwiki-20191101-pages-articles-multistream.xml.bz2
rm enwiki-20191101-pages-articles-multistream.xml
python build_wikIR.py --input enwiki.json --output_dir COLLECTION_PATH/wikIR1k --max_docs 370000 --validation_part 100 --test_part 100 -tfscb
rm enwiki.json
```

COLLECTION_PATH is the directory where *wikIR1k* will be stored


### Reproduce *wikIR59k* dataset

To reproduce the *wikIR59k* dataset, execute the follwing lines in the wikIR directory

```bash
wget https://dumps.wikimedia.org/enwiki/20191101/enwiki-20191101-pages-articles-multistream.xml.bz2
bzip2 -d enwiki-20191101-pages-articles-multistream.xml.bz2
python wikiextractor/WikiExtractor.py enwiki-20191101-pages-articles-multistream.xml --output - --bytes 100G --links --quiet --json > enwiki.json
rm enwiki-20191101-pages-articles-multistream.xml.bz2
rm enwiki-20191101-pages-articles-multistream.xml
python build_wikIR.py --input enwiki.json --output_dir COLLECTION_PATH/wikIR59k --validation_part 1000 --test_part 1000 -tfscb
rm enwiki.json
```

COLLECTION_PATH is the directory where *wikIR59k* will be stored

### Reproduce *THwikIR10k* dataset

To reproduce the *THwikIR10k* dataset, execute the follwing lines in the wikIR directory

```bash
wget https://dumps.wikimedia.org/thwiki/20230201/thwiki-20230201-pages-articles-multistream.xml.bz2
bzip2 -d thwiki-20230201-pages-articles-multistream.xml.bz2
python wikiextractor/WikiExtractor.py thwiki-20230201-pages-articles-multistream --output - --bytes 100G --links --quiet --json > thwiki.json
rm thwiki-20230201-pages-articles-multistream.xml.bz2
rm thwiki-20230201-pages-articles-multistream.xml
python build_wikIR.py --input thwiki.json --output_dir COLLECTION_PATH/THwikIR10k --validation_part 1000 --test_part 1000 --min_len_doc 300 --min_nb_rel_doc 5 -f -s -c --language th -u
rm thwiki.json
```

COLLECTION_PATH is the directory where *THwikIR10k* will be stored

### Reproduce *THwikIRS10k* dataset

To reproduce the *THwikIRS10k* dataset, execute the follwing lines in the wikIR directory

```bash
wget https://dumps.wikimedia.org/thwiki/20230201/thwiki-20230201-pages-articles-multistream.xml.bz2
bzip2 -d thwiki-20230201-pages-articles-multistream.xml.bz2
python wikiextractor/WikiExtractor.py thwiki-20230201-pages-articles-multistream --output - --bytes 100G --links --quiet --json > thwiki.json
rm thwiki-20230201-pages-articles-multistream.xml.bz2
rm thwiki-20230201-pages-articles-multistream.xml
python build_wikIR.py --input thwiki.json --output_dir COLLECTION_PATH/THwikIRS10k --validation_part 1000 --test_part 1000 --min_len_doc 300 --min_nb_rel_doc 5 -f -s -c --language th
rm thwiki.json
```

COLLECTION_PATH is the directory where *THwikIRS10k* will be stored



:warning: bm25 can take several days to solve all the queires on *wikIR59k*, therefore the bm25 results files are provided in the dowloadable datasets.


### Reproduce both datasets

To create both *wikIR1k* and *wikIR59k* datasets just call the following script

```bash
./reproduce_datasets.sh COLLECTION_PATH
```
COLLECTION_PATH is the directory where the datasets will be stored

### Train and evaluate neural networks for ad-hoc IR with matchzoo

To reproduce our results with matchzoo models on the dev dataset, call
```bash
python matchzoo_experiment.py -c config.json
```
:warning: bm25 results files are needed by matchzoo_experiment.py 

### Display results
To compute statistical significance against BM25 with Student t-test with Bonferroni correction 
and display the results of the dev dataset, call

```bash
python display_res.py -c config.json
```
:warning: Change "collection_path" in the config.json file if you want to train and display results on the full dataset

*****
## Citation

If you use wikIR tool or the dataset(s) we provide to produce results for your scientific publication, please refer to our [paper](https://arxiv.org/abs/1912.01901v4):

```bash
@inproceedings{frej-etal-2020-wikir,
title = "{WIKIR}: A Python Toolkit for Building a Large-scale {W}ikipedia-based {E}nglish Information Retrieval Dataset",
author = "Frej, Jibril and
Schwab, Didier and
Chevallet, Jean-Pierre",
booktitle = "Proceedings of the 12th Language Resources and Evaluation Conference",
month = may,
year = "2020",
address = "Marseille, France",
publisher = "European Language Resources Association",
url = "https://www.aclweb.org/anthology/2020.lrec-1.237",
pages = "1926--1933",
abstract = "Over the past years, deep learning methods allowed for new state-of-the-art results in ad-hoc information retrieval. However such methods usually require large amounts of annotated data to be effective. Since most standard ad-hoc information retrieval datasets publicly available for academic research (e.g. Robust04, ClueWeb09) have at most 250 annotated queries, the recent deep learning models for information retrieval perform poorly on these datasets. These models (e.g. DUET, Conv-KNRM) are trained and evaluated on data collected from commercial search engines not publicly available for academic research which is a problem for reproducibility and the advancement of research. In this paper, we propose WIKIR: an open-source toolkit to automatically build large-scale English information retrieval datasets based on Wikipedia. WIKIR is publicly available on GitHub. We also provide wikIR59k: a large-scale publicly available dataset that contains 59,252 queries and 2,617,003 (query, relevant documents) pairs.",
language = "English",
ISBN = "979-10-95546-34-4",
}
```

*****
## References

[1] Shota Sasaki, Shuo Sun, Shigehiko Schamoni, Kevin Duh, and Kentaro Inui. 2018. Cross-lingual learning-to-rank with shared representations, [pdf](https://cs.jhu.edu/~kevinduh/papers/sasaki18letor.pdf)

[2] Shigehiko Schamoni, Felix Hieber, Artem Sokolov, and Stefan Riezler. 2014. Learning translational and knowledge-based similarities from relevance rankings for cross-language retrieval, [pdf](https://pdfs.semanticscholar.org/5b5a/83e5ad9f4096cd6286ec11c31d796c82bcd0.pdf)

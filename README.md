# WikiQA-Salience: A dataset for evaluating entity salience prediction on extremely short documents

## Introduction

WikiQA-Salience is a dataset for evaluating entity salience prediction on extremely short question-answer pair passages.

We leveraged the [WikiQA](https://www.microsoft.com/en-us/research/publication/wikiqa-a-challenge-dataset-for-open-domain-question-answering/) dataset as a starting point to create a new entity salience dataset from publicly available data.  The WikiQA corpus is an answer sentence selection (AS2) dataset where the questions are derived from query logs of the Bing search engine, and the answer candidates are extracted from Wikipedia. The examples are Q/A pairs in natural language with full-sentence (non-factoid) answers, which resemble the type of responses provided by conversational assistants.

Our dataset augments a subset of the Q/A pairs in WikiQA with named entities extracted from the question and answer text and linked to Wikidata and ground truth labels of the salience of each entity to the Q\A pair.

## Construction

We first selected the Q/A pairs in the WikiQA corpus where the answer is labeled as correctly answering the question (positive pairs).  Then we applied the [ReFinED](https://github.com/amazon-science/ReFinED) named entity resolution model to the combined question-answer text to extract named entities and augmented each entity with the name, description and aliases of the entity from WikiData.  Since WikiData descriptions are typically extremely brief, we further augment the entities in the dataset with more detailed information from Wikipedia pages (wherever these are available) including the Wikipedia summary (i.e., the first section of the page) and the first 100 noun-phrases from the article. Ground truth labels were generated by crowd workers on the Amazon Mechanical Turk platform who rated the relevance of each entity to the Q/A pair it was extracted from on a three level scale (“High”, “Moderate”, “Low”).

The finished dataset consists of 687 annotated Q/A pairs with the linked entity data from ReFinED, entity details from WikiData, and the (5-pass) crowd worker salience ratings.  The 687 Q/A pairs contain 2113 entities (unique at the Q/A pair level), and the mean length of the question-answer text is just 190.6 characters and 32.9 words.  The distribution of the entity salience labels is significantly skewed towards salient entities with 1089 rated “High”, 535 rated “Moderate” and 489 rated “Low”.  Additional details on the construction of the dataset can be found in our paper (TBD).

## Statistics

|            | mean | std | min | max |
|------------|------|-----|---|----|
| characters | 190.6 | 68.4 | 49 | 589 |
| words      | 32.9 | 11.2 | 8 | 89 |

**Table 1**: Q/A pair context size


| number of entities | count |
|--------------------|-------|
| 2                  | 281 |
| 3                  | 188 |
| 4                  | 125 |
| 5                  | 71 |
| 6                  | 22 |

**Table 2**: Distribution of number of entities

| median rating | count |
|---------------|-------|
| High          | 1089 |
| Moderate      | 535 |
| Low           | 489 |
| Total         | 2113 |

**Table 3**: Distribution of ground truth labels

## Contents

The dataset (`wikiqa_salience.jsonl`), in the JSON lines format, contains the following fields:
-  QuestionID: QuestionID from the original WikiQA dataset
-  SentenceID: SentenceID from the original WikiQA dataset
-  entities: a list of entity objects

Each entity object contains the following fields:
-  text: the mention text
-  category: the coarse mention type (from ReFinED)
-  predicted-entity-types: the predicted entity types
-  wikidata-entity-id: the WikiData entity ID
-  el-score: the ReFinED entity linking model confidence score
-  start-char: the start character of the mention text within the passage
-  end-char: the end character of the mention text within the passage
-  backend: the name of the entity linking model (i.e. "refined")
-  wikidata-entity-name: the canonical name of the entity in WikiData
-  wikidata-entity-description: a short textual description of the entity from WikiData
-  wikidata-entity-aliases: a list of aliases for the entity from WikiData
-  gt-rating-mean: the mean normalized numeric rating in the range [0, 1]
-  gt-rating-std: the standard deviation of the normalized numeric ratings.
-  gt-rating-median: the median normalized numeric rating in the range [0, 1]
-  gt-ratings-raw: a list of strings containing the ratings from each pass of annotation from the set {"High", "Moderate", "Low"}.
-  sum-first-section: Wikipedia page summary from the first section of the page
-  sum-noun-phrase-spacy: the first 100 non-phrases from the article
-  sum-keywords-spacy: first 100 key phrases using Spacy
-  sum-keywords-rake: first 100 key phrases using Rake

## Joining with the original WIKIQA dataset

The WikiQA-Salience dataset does not contain the question and answer pairs from the original WikiQA dataset only the extracted entities, enrichments from Wikidata/Wikipedia, and the ground truth labels.  The datasets can be easily merge as described below, where `WIKIQA_PATH` point to the `WikiQA.tsv` file from the [WikiQA](https://www.microsoft.com/en-us/research/publication/wikiqa-a-challenge-dataset-for-open-domain-question-answering/) corpus and the `ENTITY_SALIENCE_DATA_PATH` points to the `wikiqa_salience.jsonl` file from this dataset.

```
import pandas as pd
wikiqa_df = pd.read_csv(WIKIQA_PATH, sep="\t")
entities_df = pd.read_json(ENTITY_SALIENCE_DATA_PATH, lines=True)
joined_df = entities_df.join(wikiqa_df.set_index(["QuestionID", "SentenceID"]), on=["QuestionID", "SentenceID"], how="inner")
```

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## Cite

Please cite our paper if you use this dataset for your own research:

```BibTeX
@article{bullough2024,
title={Predicting Entity Salience in Extremely Short Documents},
author={Benjamin L. Bullough and Harrison Lundberg and Chen Hu and Weihang Xiao},
year={2024},
eprint={TBD},
archivePrefix={arXiv}
}
```
## License

This project is licensed under the CC-BY-SA 4.0 License.

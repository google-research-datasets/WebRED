# WebRED - Web Relation Extraction Dataset

A large and diverse dataset for extracting relationships from a variety of text
found on the World Wide Web. The dataset can be downloaded from
https://github.com/google-research-datasets/WebRED.

The dataset consists of sentences from a web documents whose entities are
tagged as subject `SUBJ{...}` and object `OBJ{...}` which are connected by a
relationship that the sentence might indicate.

<!--
More information about the dataset can be found in the paper
[ADD_CITATION W/ ARXIV LINK].
Please if you use the dataset cite the above paper as:
[ADD CITATION]
-->

We compare our dataset against other publicly available relationship extraction
corpus in the table below. Notably, ours is the only dataset with text that is
found on the web, where the input sources and writing styles vary wildly.

<!--
**TODOs**
- Make a note that the number here is what we used.
- Delete the weakly-supervised numbers in the table. 
- Add a note about pre-training here.
-->

| Dataset                    | No of relations     | No of examples            |
|----------------------------|---------------------|---------------------------|
| TACRED <!-- [link] -->     | 42                  | 106,264                   |
| DocRED (human-annotated)   | 96                  | 63,427                    |
| DocRED (weakly-supervised) | 96                  | 1,508,320                 |
| WebRED (human-annotated)   | 420 <!--[CHANGE]--> | 173,140 <!-- [UPDATE] --> |

<!-- Robert: I wouldn't show the size of the weakly supervised data here since
     it is not published. -->
<!--
| WebRED (weakly-supervised) | 420                 | 199,786,781 [UPDATE]      |
-->

## Preparation
First, download the data onto your disk:

```bash
cd ~
git clone https://github.com/google-research-datasets/WebRED.git
cd WebRED/
```

## Using the data
The dataset is distributed in `Tensorflow.Example` format encoded as
[`TFRecord`](https://www.tensorflow.org/tutorials/load_data/tfrecord).

One can easily read the content of the dataset using
[Tensorflow's data API](https://www.tensorflow.org/api_docs/python/tf/data):

```python
import tensorflow as tf

path_to_webred = '...'          # Path to where the WebRED data was downloaded.

def read_examples(*dataset_paths):
  examples = []
  dataset = tf.data.TFRecordDataset(dataset_paths)
  for raw_sentence in dataset:
    sentence = tf.train.Example()
    sentence.ParseFromString(raw_sentence.numpy())
    examples.append(sentence)
  return examples

webred_sentences = read_examples(path_to_webred)
sentence = webred_sentences[0]  # As an instance of `tf.Example`.
```

Description of the features:

  * `num_pos_raters`: Number of unique human raters who thought that the
    sentence expresses the given relation.
  * `num_raters`: Number of unique human raters whou annotated the sentence fact pair
  * `relation_id`: The
    [WikiData relation ID](https://www.wikidata.org/wiki/Wikidata:Identifiers)
    of the fact.
  * `relation_name`: Human readable name of the relation of the fact.
  * `sentence`: The sentence with object and subject annotation in it.
  * `source name`: The name of the subject (source) entity.
  * `target_name`: The name of the object (target) entity.
  * `url`: Original url containing the annotated sentence.

The individual features of an example can be accessed e.g. by the following way:

```python
def get_feature(sentence, feature_name, idx=0):
  feature = sentence.features.feature[feature_name]
  return getattr(feature, feature.WhichOneof('kind')).value[idx]

annotated_sentence_text = get_feature(sentence, 'sentence').decode('utf-8')
relation_name = get_feature(sentence, 'relation_name').decode('utf-8')
empirical_probability_of_the_sentence_expresses_the_relation = (
    get_feature(sentence, 'num_pos_raters') /
    get_feature(sentence, 'num_raters'))
```

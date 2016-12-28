#Aspect Extraction Project Note

-----------
## Multistage Run
### 1. Preprocess
* change `name` in the config.py file
   generate data/hotel/original/  from json files
`nohup python3 generateHotelData.py > log &`
   > save each review as a file, named from 1 to #reviews (1591911).
* generate lemmatized/ data
`python pre.py`
   > lemmatize reviews (1591911 files in data/hotel/lemmatized/ directory)
       lemmatized files format: sentence_id \t lemmatized_sentence 
* generate name_all, name_sentences, name_readable
`python sentences.py`
  >  `goodset` 存放 sent_ids
* `hotel_sentences` format: doc_id \t lemmatized_sentence
* `hotel_all` format: each line is a lemmatized sentence
* `hotel_readable` format: 存放 与 hotel_sentences 相应的没有lemmatized的句子，doc_id \t sentence

### 2. Sentence2Vec
`cd ../sentence2vec`
`python run.py`
> 17414065 sentences in total

* generate bigrams
```python
words=['a','b','c']
zip(words,words[1:])
[('a', 'b'), ('b', 'c')]
```
    - Install gensim
    Since I want to make local changes to the gensim code, I use the following preferred way to install gensim:
    `git clone https://github.com/RaRe-Technologies/gensim.git`
    `python setup.py develop`
    I also use `virtualenv` to make a virtual environment for developing my own aspect project.
    `virtualenv aspect_env`
    `source aspect_env/bin/activate`
    `deactivate`
     
[`collections.Counter`][1], A Counter is a dict subclass for counting hashable objects. It is an unordered collection where elements are stored as dictionary keys and their counts are stored as dicitionary values. Counts are allowed to be any integer value including zero or negative counts. The `Counter` class is similar to bags or multisets in other languages.
`Counter.elements`: Return an iterator over elements repeating each as many times as its count. Elements are returned in arbitrary order. if an element's count is less than one, elements() will ignore it.
```python
>>> c = Counter(a=4, b=2, c=0, d=-2)
>>> sorted(c.elements())
['a', 'a', 'a', 'a', 'b', 'b']
```
`Counter.most_common([n])`: Return a list of the *n* most common elements and their counts from the most common to the least. If *n* is omitted or None, most_common() returns all elements in the counter. Elements with equal counts are ordered arbitrarily.

### 3. Sentence Clustering
* cluster sentences
`cd ../KMeans`
`nohup python kmeans.py > hotel.kmeans.log &`
> kmeans.py, generate config.N 个 sentence clusters
> vector_file 是文件 hotel.sentvec 的句柄
> entry_vector {'sent_1':vector_representation_of_sent1}
> vectors: np.arrray, shape=(#sentences, D), vector[i] 是 sent_i 的向量表示
* aggregate sentences into documents (i.e. reviews)
`nohup python aggregate.py > hotel.agg.log &`
> 
### 4. Topic Generation
`cd ../LDA`
`nohup java -Xmx100g -cp lda.jar lda.main.LdaGibbsSampling > lda.log &`

### 5. Topic Cluster & Noise Isolation
`nohup python vectorize.py > hotel.vectorize.log &`
`nohup python cluster.py > hotel.cluster.log`

### 6. Clustering/Word Ranking
`cd ../ranking`
`nohup python overlap_with_mutual.py > hotel.overlap.log &`


[1]: https://docs.python.org/3/library/collections.html?highlight=collections#collections.Counter

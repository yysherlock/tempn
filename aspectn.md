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
